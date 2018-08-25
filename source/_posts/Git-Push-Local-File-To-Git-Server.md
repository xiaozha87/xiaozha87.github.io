---
title: git将本地文件推送到Github远程仓库
date: 2018-08-25 10:00:58
tags:
- Github
category: Github
---

### 1. 在github上创建一个项目：例如 vue-demo
### 2. 在本地vue-demo 项目中使用 git init 把其变成git可以管理的仓库
`git init`

### 3. 若要忽略本地的文件或文件夹不被提交到github ，则需要在项目根目录下创建 .gitignore 文件
`touch .gitignore`
<!-- more -->
### 4. 打开文件，编辑内容，例如：
```bash
node_modules/
update.txt
```

则可以忽略目录下node_modules 文件夹及update.txt 文件.

[配置文件可以在github在线浏览](https://github.com/github/gitignore)

### 5. 添加文件夹下所有文件到暂存区 git add .
`git add .`

### 6. 把文件提交到仓库
`git commit -m 'decriptions'`

### 7. 关联远程仓库 （第一次使用需要添加github公钥）

`git remote add origin git@github.com:***/vue2.1-sell.git`
或者
`git remote add origin https://github.com/***/vue2.1-sell.git`

### 8. 获取远程库与本地同步（远程仓库不为空需要这一步）

`git pull --rebase origin master`


### 9. 把本地内容推送到远程库 使用 git-push,（第一次需要加-u，后面就不用加了）

`git push -u origin master`


以上内容就可以将本地文件推送到github上，并且可以自己设定不需要上传哪些文件
