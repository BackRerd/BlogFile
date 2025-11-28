---
title: 如何使用github仓库
tags:
  - git
  - 其他
cover: 'https://s2.loli.net/2024/04/22/NzD6FRuQ3wKJ4ZI.png'
abbrlink: 5093d991
date: 2024-11-15 10:43:30
categories: ["小技巧"]
---
# 仓库的创建与使用
## 第一步创建本地仓库
找到一个文件夹在文件夹内使用git软件输入指令
```ssh
git init
```
这样就创建好了本地仓库
## 第二步添加文件，上传第一步
使用指令`git add .`将当前所有文件都添加到本地仓库中
然后使用`git status`查看添加的情况

## 第三步提交到本地仓库，上传第二部
使用命令,将当前文件上传到本地仓库中，`-m`后面接参数为本次提交的注释
```ssh
git commit -m "first commit"
```
## 第四步与GitHub做连接
### 生成密钥
使用命令`ssh-keygen`
```ssh
ssh-keygen -t rsa -C "401265881@qq.com"
```
这里我提供了一个邮箱`401265881@qq.com`
输入后会有提示需要按Y的地方按一下Y即可,不需要的地方直接回车
### 添加密钥
到GitHub页面点击头像->settings->SSH and GPG keys->New SSH keygen
对key进行添加

## 第五步关联仓库
新建一个仓库，将GitHub仓库与本地仓库关联
```ssh
git remote add origin git@github.com:BackRerd/FileBlog.git
```
这里使用的我自己的git。
关联好后使用`git push -u origin master`与GitHub仓库第一次提交。
完成后就可以使用`git push origin master`进行文件的上传了
# 仓库的使用
直接从第四步开始。
## 上传命令，上传第三部
```ssh
git push origin master
```
## 与本地合并命令！！！上传之前首要这个！！
```ssh
git pull --rebase origin master
```

### hexo博客上传到github
## 第一步准备工作
### 验证ssh
```ssh
ssh -T git@github.com
```
出现类似字样就是正确的
`Hi BackRerd! You've successfully authenticated, but GitHub does not provide shell access.`
### 安装组件
安装deploy组件
```ssh
npm install -g hexo-cli
npm config set registry https://registry.npmmirror.com
npm install
npm install hexo-deployer-git --save
```
### 添加账户密码
```ssh
git config --global user.name "BackRerd"
git config --global user.email "4012658814@qq.com"
```
## 第二步配置文件
配置config.yml文件,与仓库连接
```yaml
deploy:
  type: 'git'
  repository: 'git@github.com:BackRerd/backrer.github.io.git'
  branch: main
```
## 第三步上传，上传最后一步
先生成网页
```ssh
hexo cl
hexo g
```
上传
```ssh
hexo d
```

# 新建网页

```
hexo new 文章名称
```

