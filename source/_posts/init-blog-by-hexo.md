---
title: init blog by hexo
date: 2018-04-04 01:53:15
tags: 
- hexo
---

### Hexo博客搭建

#### 关于搭建的流程
- 首先在自己的 github 上创建仓库，http://yourgithubname.github.io；
- 创建两个分支：master 与 dev；
- 设置 dev 为默认分支（因为我们只需要手动管理这个分支上的 Hexo 网站文件）；
- 使用git clone git@github.com:yourgithubname/yourgithubnamegithub.io 拷贝仓库；

<!-- more -->

-  在本地 http://yourgithubname.github.io 文件夹下通过 Git bash 依次执行 `npm install hexo`、`hexo init`、`npm install` 和 `npm install hexo-deployer-git`（此时当前分支应显示为 dev）;
- 修改 _config.yml 中的 deploy 参数，分支应为 master；
- 依次执行 `git add .`、`git commit -m "..."`、`git push origin hexo`提交网站相关的文件；
- 执行hexo g -d生成网站并部署到 github 上。这样一来，在 github 上的 http://yourgithubname.github.io 仓库就有两个分支，一个 dev 分支用来存放网站的原始文件，一个master分支用来存放生成的静态网页。

#### 搭建踩坑
- 在你从 github `git clone` 下来的代码仓中执行 hexo init 时，会报错，说当前不是一个空文件，无法进行 hexo 初始化，你可以在一个新的空文件加重执行 hexo init，然后删除里面的 package-lock.json 和 node_modules 文件，然后把剩余的文件 copy 进站点根目录，然后执行 `npm i` 和 `npm install hexo-deployer-git`操作。

- 在配置站点的 _config.yml 文件的 deploy 参数是应该这样配置：
![deploy-params](/images/deploy.png)
