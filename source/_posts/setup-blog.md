---
title: 个人博客：Hexo + Github Pages + travis
---

记录一下使用 **Hexo**, **Github pages** 和 **Travis** 搭建个人博客的过程。

目前实现的效果是，本地使用 **Markdown** 编写博客文章，travis 自动基于最新仓库源码，构建博客。
以上均为**免费**服务，无需花钱租用虚拟主机。

## 1. Github Pages

Github Pages 是 Github 提供的静态页面服务，依据其[官网](https://pages.github.com/)，使用的方式比较简单。
大体的 Gist 就是

1. 创建名称为 username.github.io 的代码仓库
2. 将 Web 静态资源通过 Git push 到 username.github.io

其中 username 为具体用户登录 Github 网站的账号名称，当浏览器访问 https://username.github.io 时，仓库 master 目录下的静态资源就会被加载。

| Branch  | Description  |
|---|---|
|  develop | 开发分支，管理 Markdown 源码与项目配置文件 |
|  master | 存储基于 develop 分支源码生成的静态资源文件 |

## 2. 流程

![](/images/blog.png)

1. Hexo 用于将 **Markdown** 编写的博客文章源文件生成为静态页面
2. Github pages serve 生成的静态文件，在公网上提供访问接口
3. travis 由 develop 分支的新 commit 触发，执行 hexo generate 命令生成静态文件，并将生成的内容 push 到 master 分支

## 3. Hexo

> A fast, simple & powerful blog framework.

### 安装命令行工具

```bash
npm install hexo-cli -g
```

### Hexo 的常用命令

创建新文章

``` bash
hexo new "My New Post"
```

本地服务器

``` bash
hexo server
```

生成静态页面

``` bash
hexo generate
```

部署

``` bash
hexo deploy
```

## 4. Travis

> A hosted continuous integration service used to build and test software projects hosted at GitHub

Travis 为 Github 的公开仓库提供免费的构建服务。

### 整合 Github

1. https://github.com/marketplace/travis-ci 添加 Travis App.
2. https://github.com/settings/installations 为 Travis 添加访问 Github 公开仓库的权限
3. 打开 https://github.com/settings/tokens 生成 Token 供 Travis 使用
   ![](/images/github_token.png)

### .travis.yml

完成整合后，在项目根目录下添加 `.travis.yml` 配置文件。

```yml
sudo: false
language: node_js
node_js:
  - 10
cache: npm
branches:
  only:
    - develop
script:
  - hexo generate # generate static files
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  target_branch: master
  on:
    branch: develop
  local-dir: public
```

- github-token: 需要在 travis 中添加 **GH_TOKEN** 环境变量，环境变量的值为之前生成的 token
- develop 分支用于管理博客 markdown 内容，hexo 配置文件
- `target_branch: master` 指定 travis 将 `hexo generate` 生成的静态资源文件 push 到 master 分支

## 5. Summary

详细配置请参考 [repo](https://github.com/RockingJavaBean/rockingjavabean.github.io)，博客访问地址为 [rockingjavabean.github.io](https://rockingjavabean.github.io)。

以上
RockingJavaBean
