---
title: github-actions
date: 2025-05-22 11:52:16
tags:[git, web前端]

---

尝试了一下github的自动部署网站，进入actions后new一个workflow，添加以下代码并commit：

```
name: Deploy Vue Frontend

on:
  push:
    branches: [ "main" ]
  workflow_dispatch: # 允许手动触发

# 为 GITHUB_TOKEN 设置默认权限，以便部署到 GitHub Pages
permissions:
  contents: read      # checkout 代码需要
  pages: write        # 部署到 GitHub Pages 需要
  id-token: write     # OIDC 认证需要 (actions/deploy-pages 使用)

# 只允许一个并发部署，跳过在队列中等待的运行。
# 但是，不要取消进行中的运行，因为我们希望允许那些部署完成。
concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20" # 或者你的项目使用的版本
          cache: 'npm'
          cache-dependency-path: client/package-lock.json # 更精确的缓存路径

      - name: Install Dependencies
        working-directory: ./client
        run: npm ci # 使用 ci 更稳定

      - name: Build
        working-directory: ./client
        run: npm run build # 确保你的 vite.config.js 中 base: '/仓库名/' 已设置

      - name: Setup Pages
        uses: actions/configure-pages@v4 # 官方action

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3 # 官方action
        with:
          # 从 client/dist 目录上传
          path: './client/dist'

  deploy:
    needs: build # 依赖 build job
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }} # 输出部署后的 URL
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4 # 官方action
```

在本地进行git pull后，在client目录下的vite.config.js里面添加这行代码base:'/{仓库名}/',package.json里面添加这行代码"homepage": "https://kkzyu.github.io/{仓库名}"。

再到github的setting下进行以下操作：

- pages下的source选择github actions；
- environment下检查是否存在github-pages，点击进入查看是否有master这个分支；
- actions的general下，勾选read and write permission以及Allow GitHub Actions to create and approve pull requests。



在部署last1km的时候出现了很多路径上的问题，导致信息无法正常显示，这主要是因为在vite中把base改成了/last1km/后在解析的时候路径发生了变化，这也是写代码的时候不太规范导致的问题。

以chatpage.vue为例，下面是对应的解决办法：

```
# 定义baseurl为vite下记录的url
const BASE_URL = import.meta.env.BASE_URL;
# 由于在json文件里面保留了/对应的public路径，所以这里需要对路径格式进行修改，由于baseurl的last1km后面还跟着一个/，所以需要去掉其中的一个/
const resolveAssetPath = (relativePath) => {
  if (!relativePath) return undefined;
  let path = relativePath;
  if (BASE_URL.endsWith('/') && path.startsWith('/')) {
    path = path.substring(1);
  } else if (!BASE_URL.endsWith('/') && !path.startsWith('/')) {
  }
  return `${BASE_URL}${path}`;
};
# 这个可以直接放到stores下用pinia管理，并导入
# ......
# 对应的在获取数据时也要进行以下更改
    const fetchPath = 'data/messages.json'; // Path relative to public directory
    const response = await fetch(`${BASE_URL}${fetchPath}`);
    const riderData = currentChatData.rider ? {
        ...currentChatData.rider,
        // chatInfo.rider.avatar 的值如 "/images/avatar1.jpg"
        // resolveAssetPath会处理它，变成 "/last1km/images/avatar1.jpg"
        avatar: currentChatData.rider.avatar ? resolveAssetPath(currentChatData.rider.avatar) : undefined
      } : null;
```

