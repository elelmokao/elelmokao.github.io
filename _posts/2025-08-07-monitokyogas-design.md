---
title: MonitokyoGas

date: 2025-08-07 00:00:00 +0800
categories: [MonitokyoGas]
tags: [project]
# TAG names should always be lowercase
---
## Prototype of MonitokyoGas
Tokyo Gas（東京ガス）在主頁上可以查看自己每期、每天、每小時的耗電量，但是每天都要開APP檢查實在是有點麻煩，因此想寫一個前後端架構來實現：
* 後端：
    * 在每天13:30（JST），抓取儲存前一天的耗電量
    * 如果前一天超過4kWh，將使用discord通知用戶。
* 前端：
    * 做一個前端架構來檢視過去7/30/90天/任意時間段的耗電量
    * 計算當期在第一階累進費率下剩餘的電量
    * 能夠顯示歷史電價、預估電費

## Tech Stack
* 前後端：使用Vue、Typescript，其實沒有特別原因，只是因為我的Github Repository 裡沒有這兩個，不然原本想要用Golang來寫後端抓取資料的。
* 數據庫：先簡單使用csv儲存資料，依後續做調整。
* 其餘：
    * 想嘗試使用[Projen](https://github.com/projen/projen)
    * Github Actions作為自動CICD，每天執行後端抓取資料、前端在有變更時做github page deployment

## Development History
1. 2025-08-07: 原型完成，可以模擬網站登入獲得Cookie並抓取歷史用電紀錄，透過Github Action 自動抓取
    * [2025-08-07-monitokyogas-backend-1.md](https://elelmokao.github.io/posts/tokyogas-backend-1/)
    * [2025-08-07-monitokyogas-frontend-1.md](https://elelmokao.github.io/posts/tokyogas-frontend-1/)
2. 2025-08-17: 設計自動前端部署CICD，並且實作前端頁面
    * [2025-08-17-monitokyogas-frontend-2.md](https://elelmokao.github.io/posts/tokyogas-frontend-2/)

---
## Ref
* [https://github.com/elelmokao/monitokyogas/tree/main](https://github.com/elelmokao/monitokyogas/tree/main)
