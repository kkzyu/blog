---
title: hexo blog部署记录
date: 2025-04-12 16:00:00
tags: web前端
---
2025年4月28日，第一次尝试把hexo集成到个人主页中，记录一下部署过程，以及后期的更新和测试步骤。

## 部署过程

### 创建hexo项目
    hexo是github上一个开源的博客模板，首先可以在新目录下创建一个hexo项目，然后进入该目录，执行如下命令：
``` bash
$ npm install -g hexo-cli
```
接着进行一个初始化，进行初始化之后会创建一个新的hexo项目，自动克隆hexo-start和landscape主题：
``` bash
$ hexo init myblog
$ cd myblog
$ npm install
```
初始化之后，项目文件夹如下所示：
.
├── _config.yml（网站配置文件，以下的修改调整主要在这个文件下进行）
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes


### 本地测试
默认地址是http://localhost:4000/
``` bash
$ hexo server
```

### 选择主题
    hexo默认的主题比较简约，但它提供了非常多的模板，在https://hexo.io/themes/上可以挑选主题，我这里用的是vivia（https://github.com/saicaca/hexo-theme-vivia）。接着按README步骤安装执行即可。

### 部署到github pages上
#### 1.部署个人网站
    首先需要确认github上已有<username>.github.io的仓库，用于存放个人主页的代码。这里对创建过程进行一个简单的介绍：
    把个人主页网站的所有代码copy到一个本地的新文件夹下，然后再在github上新建一个仓库（命名为<username>.github.io,比如我的username是kkzyu），接着在本地文件夹中执行如下命令：
``` bash
$ git remote add origin git@github.com:kkzyu/kkzyu.github.io.git
$ git branch -M main
$ git push -u origin main
```
push成功后，进入setting->pages->branch，选择main和root，点击save。等待十分钟之后即可在https://kkzyu.github.io/查看自己的个人主页。

#### 2.部署博客
    首先需要编辑myblog/_config.yml文件，修改/增加如下代码：
    
``` yaml
# 部署配置
deploy:
  type: git
  repo: https://github.com/kkzyu/blog.git 
  branch: gh-pages  # 必须使用 gh-pages 分支
  message: "更新博客：{{ now('YYYY-MM-DD HH:mm:ss') }}"

# URL 设置
url: https://kkzyu.github.io/blog
root: /blog/
```
    由于hexo博客和个人主页独立部署更优（少冲突），因此在github上新建一个仓库（可以命名为myblog），注意无需初始化readme或其他文件。创建完成后，同样需要将本地的仓库推送上去：
``` bash
$ git remote add origin git@github.com:kkzyu/myblog.git
$ git branch -M main
$ git push -u origin main
```  
    接着在本地myblog文件夹下安装部署插件并部署：
``` bash
$ npm install hexo-deployer-git --save
$ hexo clean && hexo generate && hexo deploy
```
    接着进入myblog仓库的settings->pages，选择branch为gh-pages，选择root目录。之后等待几分钟，进入到https://kkzyu.github.io/blog/，即可看到自己的博客。

### 将博客集成到vue个人主页中
    在vue项目中，修改路由配置(router/index.js)：
``` js
const routes = [{
    path: '/blog',
    name: 'Blog',
    beforeEnter() {
      window.open('https://kkzyu.github.io/blog/', '_blank')
      // 或者不新开标签页：
      // window.location.href = 'https://kkzyu.github.io/blog/'
    }
  }]
```
在APP.vue中添加博客链接：
``` vue
<router-link to="/blog">Blog</router-link>
```
等待几分钟之后，即可在vue项目中看到博客链接，点击后会在新标签页打开博客。

## 后期更新和测试
### 更新博客
    在本地对myblog进行了修改后，网页并不会自动更新，需要进行一下操作才可在网页中观察到更新后内容：
``` bash
$ hexo clean && hexo generate && hexo deploy
#可选：将源码变更推送到main分支备份
git add .
git commit -m "更新博客内容"
git push origin main
```
    如果想要现在本地进行测试可以执行：
``` bash
$ hexo server
```
待测试成功后，再进行更新。