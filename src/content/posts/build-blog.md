---
title: 使用Astro与Github Pages搭建博客记录
date: 2024-05-17
tags:
  - Astro
  - Github
  - Pages
  - 博客
category: 实践
comments: false
draft: false
summary: 这篇文章记录了使用Astro与GitHub Pages基于模板（gyoza）搭建博客的流程。
---

### 参考链接：

[Astro教程](https://docs.astro.build/zh-cn/getting-started/)

[Gyoza 使用指南](https://github.com/lxchapu/astro-gyoza)

### 前置条件：

- node 版本 >= 18.18.0
- pnpm 版本 > 8.1.0
- git

## 1. 使用Astro下载模板

在一个合适的目录下，打开cmd，运行下面的命令。

```shell
# 基于某个 GitHub 仓库的 main 分支创建一个新项目
npm create astro@latest -- --template <用户名>/<仓库名>
# 以gyoza为例
npm create astro@latest -- --template lxchapu/astro-gyoza
```

## 2.安装依赖

```shell
# 进入astro生成的目录
cd astro-gyoza
# 安装依赖
pnpm insatll
```

## 3. 修改配置文件

gyoza项目中的绝大部分配置都定义在 `src/config.json` 文件中。
将 `site.url` 修改成 `https://<用户名>.github.io` ，避免导航错误。

## 4.创建GitHub Action配置文件

在`astro-gyoza`中新建目录`.github\workflows\` ,在该目录下新建文件`deploy.yml`，并粘贴以下 YAML 配置信息。

```yml
name: Deploy to GitHub Pages
on:
  # Trigger the workflow every time you push to the `main` branch
  # Using a different branch name? Replace `main` with your branch’s name
  push:
    branches: [main]
  # Allows you to run this workflow manually from the Actions tab on GitHub.
  workflow_dispatch:
# Allow this job to clone the repo and create a page deployment
permissions:
  contents: read
  pages: write
  id-token: write
# Allow one concurrent deployment
concurrency:
  group: "pages"
  cancel-in-progress: true
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout your repository using git
        uses: actions/checkout@v2
      - name: Install, build, and upload your site output
        uses: withastro/action@v0
        # with:
            # path: . # The root location of your Astro project inside the repository. (optional)
            # node-version: 16 # The specific version of Node that should be used to build your site. Defaults to 16. (optional)
            # package-manager: yarn # The Node package manager that should be used to install dependencies and build your site. Automatically detected based on your lockfile. (optional)
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v1
```

## 5.创建Github仓库

1. 创建一个名为 **<用户名>.github.io**的仓库，设置为公开仓库。
2. 在 GitHub 上，跳转到仓库的 **Settings** 选项卡并找到设置的 **Pages** 部分。
3. 选择 **GitHub Actions** 作为网站的 **Source**，然后按 **Save**。

## 6.部署

将`astro-gyoza`文件夹中的文件上传至新建的仓库中。

### 克隆仓库至本地

```shell
# 基于HTTPS克隆
git clone <仓库地址>
# 基于Github CLI克隆，这种方式更快
gh repo clone <用户名>/<仓库名>
```

### 同步文件

将`astro-gyoza`目录下的文件移动至上一步创建的文件夹中。
为了方便，使用VS Code同步仓库。
遇到的问题：

1. VS Code 一直处于commit状态：这是VS Code的bug，commit时必须提交信息。
2. 因commit格式要求导致的报错：修改`commitlint.config.js`为

```javascript
export default {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'subject-case': [2, 'never', ['upper-case', 'pascal-case', 'start-case']],
  },
}
```
