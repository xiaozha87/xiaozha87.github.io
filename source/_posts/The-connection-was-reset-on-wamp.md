---
title: The connection was reset on wamp
date: 2018-11-13 11:33:36
tags:
- PHP
- Wamp
category:
- PHP
- Wamp
---


I have installed wamp server on my pc.Then I have installed a zend application. I put it in www directory. When I access that directory by localhost it just shows
`The connection was reset`.This problem could not fix by increasing timeouts and memory limits.
`Os:window 7(x64), Apache : 2.5, PHP:5.5.12, Mysql : 5.6.17`

### 解决方法：
Add the following to the end of httpd.conf to increase the Apache stack size to 8MB.

```bash
<IfModule mpm_winnt_module>
   ThreadStackSize 8388608 #用来控制堆栈大小
</IfModule>
```