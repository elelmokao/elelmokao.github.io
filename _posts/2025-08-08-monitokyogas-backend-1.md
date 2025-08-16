---
title: MonitokyoGas Backend 1 - Retrieve Data

date: 2025-08-08 00:00:00 +0800
categories: [MonitokyoGas]
tags: [project]
---
## Project Overview
* [MonitokyoGas](https://elelmokao.github.io/posts/monitokyogas/)

在這個篇中，將紀錄
* 抓取東京ガス的歷史用電紀錄
* 使用Github action設計自動抓取資料的流程
* 將資料儲存到csv檔案中

## 抓取東京ガス的歷史用電紀錄

### 取得資料
為了抓取東京ガス的歷史用電紀錄，必須知道從登入到我們點擊獲取用電紀錄的過程是怎麼發生的。在我們登入並點擊「使用量」後，從開發者模式中可以看到網路中有向`https://members.tokyo-gas.co.jp/graphql`傳送POST，Query為
```graphql
query DailyElectricityUsage(
  $contractIndexNumber: Int!
  $electricityContractNumber: String!
  $fromDate: String!
  $toDate: String
) {
  dailyElectricityUsage(
    contractIndexNumber: $contractIndexNumber
    electricityContractNumber: $electricityContractNumber
    fromDate: $fromDate
    toDate: $toDate
  ) {
    averageUsageForSameContract
    date
    usage
    __typename
  }
}
```
以及variables：`contractIndexNumber 1`、`electricityContractNumber "XXXXXXXXXX"`、`fromDate "YYYY-MM-DD"`、`toDate	null`
便可以獲得歷史用電紀錄：
```
{"data":
  {"dailyElectricityUsage":
    [{"averageUsageForSameContract":9.07,
      "date":"2025-08-13T15:00:00.000Z",
      "usage":4.1,
      "__typename":"DailyElectricityUsage"
      }, ...]
  }
}
```
其中，`averageUsageForSameContract`代表同一個用電方案下其他人的平均，`usage`代表自己的用電量。

因此我們可以透過下列程式碼抓取資料：
```typescript
async function fetchElectricityUsage(cookie: string): Promise<UsageData[]> {
  const fromDate = dayjs().subtract(14, "day").format("YYYY-MM-DD");
  const response = await axios.post(
    "https://members.tokyo-gas.co.jp/graphql",
    {
      operationName: "DailyElectricityUsage",
      variables: {
        contractIndexNumber: 1,
        electricityContractNumber: process.env.CONTRACT_NUMBER,
        fromDate: fromDate,
        toDate: null,
      },
      query: `
        query DailyElectricityUsage(
          $contractIndexNumber: Int!
          $electricityContractNumber: String!
          $fromDate: String!
          $toDate: String
        ) {
          dailyElectricityUsage(
            contractIndexNumber: $contractIndexNumber
            electricityContractNumber: $electricityContractNumber
            fromDate: $fromDate
            toDate: $toDate
          ) {
            averageUsageForSameContract
            date
            usage
            __typename
          }
        }
      `,
    },
    {
      headers: {
        "Content-Type": "application/json",
        Origin: "https://members.tokyo-gas.co.jp",
        Referer: "https://members.tokyo-gas.co.jp/usage?tab=electricity",
        Cookie: cookie,
        "User-Agent": "Mozilla/5.0",
      },
    }
  );
  console.log("Response status:", response.status);
  return response.data.data.dailyElectricityUsage.map((entry: any) => ({
    date: entry.date.slice(0, 10),
    usage: entry.usage,
    contract_number: process.env.CONTRACT_NUMBER,
  }));
}
```


仔細看資料，在N日時執行時，第一筆資料日期為N-1日並且`usage`為null，而第二筆資料雖然日期是N-2日，但才是N-1日的用電量。理由是因為他使用UTC + 0時間，所以Local Time N-1日的date（`N-1T00:00:00.000`）會變成N-2 的15時（`N-2T15:00:00.000`）。這個在資料抓取後必須做處理。


### 獲取Cookie
因為網站有Cookie來請求資料，因此在設計自動化時，必須獲取Cookie才能夠進行抓取。
在模擬登入中，使用Typescript中的`puppeteer`來進行網站抓取Cookie的Header：
```typescript
export async function loginAndGetCookie(): Promise<string> {
  const email = process.env.TOKYOGAS_EMAIL!;
  const password = process.env.TOKYOGAS_PASSWORD!;
  if (!email || !password) {
    throw new Error("Please set TOKYOGAS_EMAIL and TOKYOGAS_PASSWORD in your .env file");
  }

const browser = await puppeteer.launch({
  headless: true,
  args: ['--no-sandbox', '--disable-setuid-sandbox'],
});
  const page = await browser.newPage();

  // Visit the login page
  await page.goto("https://members.tokyo-gas.co.jp/", { waitUntil: "networkidle0" });

  // Click the login button
  const goToLoginButtonSelector = 'a.text-center.flex.justify-center.w-full.rounded-lg.py-5.px-6.text-labelLarge.font-normal.bg-primary.text-onPrimary.cursor-pointer[href="/api/mtg/v1/auth/login"]';

  await page.waitForSelector(goToLoginButtonSelector, { visible: true, timeout: 10000 });

  await Promise.all([
    page.waitForNavigation({ waitUntil: "networkidle0" }),
    page.click(goToLoginButtonSelector),
  ]);

  // Fill in email and password
  await page.type('input[name="loginId"]', email);
  await page.type('input[name="password"]', password);

  // Click the login button
  await Promise.all([
    page.click('button[type="submit"]'),
    page.waitForNavigation({ waitUntil: "networkidle0" }),
  ]);

  // After successful login, retrieve cookies
  const cookies = await page.cookies();

  const cookieHeader = cookies.map(c => `${c.name}=${c.value}`).join("; ");

  await browser.close();

  return cookieHeader;
}
```

## 使用Github action設計自動抓取資料的流程
因為Tokyogas 大概都是在13:00 JST時間更新前一天的用電量，在github action 設定上除了手動trigger外，也加上了cron:
```yml
on:
  schedule:
    - cron: '30 4 * * *'
  workflow_dispatch:
```
主要流程為
1. checkout 
2. check the existence of `data` branch 
3. setup nodejs
4. install dependencies
5. Grep new data by fetchElectricity.ts
6. Push the generated csv to `data` branch

在第5步Grep new data by fetchElectricity.ts中，我們會先檢查是否有cookie存放在`backend/cookie_store`，若沒有的話會先抓取Cookie。

在Github Repo裡的Screts 中設定自己的`CONTRACT_NUMBER`、`TOKYOGAS_EMAIL`、`TOKYOGAS_PASSWORD`。

## 將資料儲存到csv檔案中
在還沒想到更好的方法前，採取方法為將資料推送到`data` branch，避免每天抓取資料時，會使得`main` branch 受到CICD的影響，讓每次開發都要處理合併問題。

但這也造成幾個新問題是「該怎麼將main branch 新變動同步到data branch」以及「該怎麼push csv 到data branch」。
* 將main branch 新變動同步到data branch，可以查看[update_branch.yml](https://github.com/elelmokao/monitokyogas/blob/main/.github/workflows/update_branch.yml)
  ```yml
      - name: Sync and push to data branch
        run: |
          git fetch --all
          # echo all branches
          git branch -a
          # Check if data branch exists and switch to it, otherwise exit
          if git show-ref --verify --quiet refs/remotes/origin/data; then
            git switch data
            git pull origin data
          else
            echo "Data branch does not exist. Exiting."
            exit 0
          fi
          git merge --no-ff origin/main -m "Merge main into data [skip ci]"
          git push origin data
  ```
* push csv 到data branch，可以查看[crawler.yml](https://github.com/elelmokao/monitokyogas/blob/main/.github/workflows/crawler.yml)
  ```yml
      - name: Ensure data branch exists and checkout
        run: |
          git fetch origin
          if git show-ref --verify --quiet refs/remotes/origin/data; then
            git checkout data
          else
            git checkout -b data
          fi

      - name: Push the csv file to the repository
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          git config --global user.name "github-actions[bot]"
          git config --global user.email "github-actions[bot]@users.noreply.github.com"
          git add backend/csv_store/*.csv
          git commit -m "Update CSV files [skip ci]" || echo "No changes to commit"
          git push https://x-access-token:${GITHUB_TOKEN}@github.com/${{ github.repository }}.git data
  ```

---
## Ref
* [https://github.com/elelmokao/monitokyogas/tree/main](https://github.com/elelmokao/monitokyogas/tree/main)
