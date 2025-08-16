---
title: MonitokyoGas Frontend 1 - Design & CICD

date: 2025-08-16 00:00:00 +0800
categories: [MonitokyoGas]
tags: [project]
---
## Project Overview
* [MonitokyoGas](https://elelmokao.github.io/posts/monitokyogas/)

在這個篇中，將紀錄
* 使用bolt.new 設計前端原型
* 使用Github Action 自動部署Github Page

## Bolt.new
因為實在不想寫前端，使用Copilot 來從零設計前端似乎容易跟我想像的不太一樣。因此使用bolt.new 來做原型。
這次使用Vue 來寫。
但也沒啥好說的：[https://github.com/elelmokao/monitokyogas/tree/main/frontend](https://github.com/elelmokao/monitokyogas/tree/main/frontend)

## Github Page CICD
但最重要的是使用Github Action 部署Github page 來展示歷史用電量。我參考了Chirpy的CICD過程，改成了適合我目前同時配置前後端的repo。

主要為幾個重點：
1. Setup Node.js 時，必須指定`package-lock.json`在`frontend`裡。
2. 在build, setup page, Upload artifact 時都必須在frontend 裡執行。
3. Deploy時，因為是要生成`name.github.io/monitokyogas`的子網頁，因此vite.config.ts中也必須調整為

```typescript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig(({ mode }) => {
  const repoName = 'monitokyogas';

  return {
    base: mode === 'production' ? `/${repoName}/` : '/',
    plugins: [vue()],
  }
})
```

完整個Github Action YML為下：
```yml
name: "Build and Deploy Vue to GitHub Pages"

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        working-directory: frontend
        run: npm install

      - name: Build project
        working-directory: frontend
        run: npm run build

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: './frontend/dist'

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
如此一來，就可以在{name}.github.io/monitokyogas下看到自己的前端。


---
## Ref
* [https://github.com/elelmokao/monitokyogas/tree/main](https://github.com/elelmokao/monitokyogas/tree/main)
