---
title: flush()用法以及ob_flush()和flush()的区别实例详解
date: 2019-01-12 10:13:55
tags:
- PHP
- flush
category:
- PHP
---

flush()函数将当前为止程序的所有输出发送到用户的浏览器。
```php
<?php 
for ($i=10; $i>0; $i--) { 
	echo $i; 
	flush(); 
	sleep(1); 
} 
?>
```
按照php手册里的说法 
该函数将当前为止程序的所有输出发送到用户的浏览器。 
上面的这段代码，应该隔一秒钟输出一次$i。但是实际中却不一定是这样。有可能是等了10秒钟后，所有的输出同时呈现出来。 
<!-- more -->
好，我们来改一下这段代码，改成 
```php
<?php 
ob_end_clean();//修改部分 
for ($i=10; $i>0; $i--) { 
	echo $i; 
	flush(); 
	sleep(1); 
} 
?>
```
嘿，加了这一句ob_end_clean();,居然就OK了。实际上，我们把ob_end_clean()换成ob_end_flush()也一样OK。 
我再来改一改。 
```php
<?php 
for ($i=10; $i>0; $i--) { 
	echo $i; 
	ob_flush();//修改部分 
	flush(); 
	sleep(1); 
} 
?>
```
运行一下，是不是发现$i也隔一秒输出一次了？这是为什么呢？ 
别急，我们来看看php.ini。 
打开php.ini,搜索output_buffering，我们会看到类似这样的设置 output_buffering = 4096。正如它的名字output_buffering一样，这个设置的作用就是把输出缓冲一下，缓冲大小为4096bytes. 
在我们的第一段代码里，之所以没有按预期的输出，正是因为这个output_buffering把那些输出都缓冲了。没达到4096bytes或者脚本结束，输出是不会被发送出去的。 
而第二段代码中的ob_end_clean()和ob_end_flush()的作用，就是终止缓冲。这样就不用等到有4096bytes的缓冲之后才被发送出去了。 
第三段代码中，用了一句ob_flush(),它的作用就是把缓冲的数据发送出去，但是并不会终止缓冲，所以它必须在每次flush()前使用。 
如果不想使用ob_end_clean(),ob_end_flush()和ob_flush()，我们就必须把php.ini里的 output_buffering设得足够小，例如设为0。需要注意的是，如果你打算在脚本中使用ini_set(” output_buffering”,”0″)来设置，那么请停下来吧，这种方法是不行的。因为在脚本一开始的时候，缓冲设置就已经被载入，然后缓冲就开始了。 
可能你会问了，既然ob_flush()是把缓冲的数据发送出去，那么为什么还需要用flush()???直接用下面这段代码不行吗？？ 
```php
<?php 
for ($i=10; $i>0; $i--) { 
	echo $i; 
	ob_flush(); 
	sleep(1); 
} 
?>
```
请注意ob_flush()和flush()的区别。前者是把数据从PHP的缓冲中释放出来，后者是把不在缓冲中的或者说是被释放出来的数据发送到浏览器。所以当缓冲存在的时候，我们必须ob_flush()和flush()同时使用。 
那是不是flush()在这里就是不可缺少的呢？不是的，我们还有另外一种方法，使得当有数据输出的时候，马上被发送到浏览器。下面这两段代码就是不需要使用flush()了。（当你把output_buffering设为0的时候，连ob_flush()和ob_end_clean()都不需要了） 
```php
<?php 
ob_implicit_flush(true); 
for ($i=10; $i>0; $i--) { 
	echo $i; 
	ob_flush(); 
	sleep(1); 
} 
?>
```
```php
<?php 
ob_end_clean(); 
ob_implicit_flush(true); 
for ($i=10; $i>0; $i--) { 
	echo $i; 
	sleep(1); 
} 
?>
```
请注意看上面的ob_implicit_flush(true)，这个函数强制每当有输出的时候，即刻把输出发送到浏览器。这样就不需要每次输出（echo）后，都用flush()来发送到浏览器了。 
以上所诉可能在某些浏览器中不成立。因为浏览器也有自己的规则。我是用Firefox1.5,IE6,opera8.5来测试的。其中opera就不能正常输出，因为它有一个规则，就是不遇到一个HTML标签，就绝对不输出，除非到脚本结束。而FireFox和IE还算比较正常的。 
最后附上一段非常有趣的代码,作者为PuTTYshell。在一个脚本周期里，每次输出，都会把前一次的输出覆盖掉。 
以下代码只在firefox下可用，其他浏览器并不支持multipart/x-mixed-replace的Content-Type. 
```php
<?php 
header('Content-type: multipart/x-mixed-replace;boundary=endofsection'); 
print "\n--endofsection\n"; 
$pmt = array("-", "\\", "|", "/" ); 
for( $i = 0; $i <10; $i ++ ){ 
	sleep(1); 
	print "Content-type: text/plain\n\n"; 
	print "Part $i\t".$pmt[$i % 4]; 
	print "--endofsection\n"; 
	ob_flush(); 
	flush(); 
} 
print "Content-type: text/plain\n\n"; 
print "The end\n"; 
print "--endofsection--\n"; 
?>
```
