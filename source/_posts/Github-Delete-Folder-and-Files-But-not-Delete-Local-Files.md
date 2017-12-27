---
title: 'git删除文件夹/文件,但不删除本地文件'
date: 2017-12-23 15:28:15
tags:
- Github
category: Github
---

**实质就是删除缓冲区里的文件，然后再提交给服务器端。**
1.首先进入要删除的文件夹或文件的根目录下，如E:\Hexo
2.执行下面的语句”directory-name”是相对于本地根目录下的文件夹/文件路径
```github
git rm -r --cached directory-name
git commit -m 'Remove  directory "directory-name"'
git push origin master
```
`git pull --rebase origin master` 这句的意思是：把github上最新的文件下载下来。

