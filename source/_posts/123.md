---
title: Travis CI自动构建hexo博客可能遇到的坑
date: 2017-12-25 11:18:32
tags:
- Travis
category:
- Travis
---
### 坑1:travis CI构建一直提示github账户授权失败 ###
psersonal token问题，重新产生，并使用travis whoami判断token有效之后再配置travis CI environment variable
### 坑2:travis CI构建一直提示hexo-renderer-sass错误 ###
在本地deploy并没有发生此问题，在travis vm中出现此问题，解决方式是在.travis.yml中增加
```
npm install hexo-renderer-sass --save
```

<!-- more -->

### 坑3:travis CI自动构建部署之后，博客页面空白，什么也没有 ###
将主题换回默认的landscape则可以正常显示内容。则锁定是next theme配置问题，check发现themes/next 中的内容被ignore了，并没有push到raw branch.
解决方法有二：
- 使用.gitmodules，该方法会直接将next theme repository import进来，这样的好处是可以使用最新的next theme，坏处是没法客制化自己的主题配置文件
```
[submodule "next"]
    path = themes/next
    url = https://github.com/iissnan/hexo-theme-next
```
- 删除themes/next的.git和.gitignore，然后就可以讲themes/next的内容push到repository中了

### Others ###
1.在.travis.yml中将node_modules添加到cache中，可以加快构建速度
```
cache:
  directories:
    - node_modules
```
2.如果想在github的README.md显示构建成功与否的标示，可以修改README.md：
```
[build-info](https://travis-ci.org/userName/repoName.svg)
```
