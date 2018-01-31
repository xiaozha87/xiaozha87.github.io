---
title: PHP防止跨域提交表单[非 http_referer 验证]
date: 2018-01-31 09:00:11
tags:
- PHP
- Cross Domain
category:
- PHP
---
PHP开发中跨域表单提交解决办法，网上很多办法都是这样写的，通过判断HTTP_REFERER来验证来源，这个等于没做，别人既然都知道跨域提交了，随便写一段代码就模拟出referer，而你写的这一段验证等于0，下面介绍一个不通过使用验证码的手段来验证跨域提交,使用 session+token 验证

我们通过使用 session+token 凭证来设置一段 form 的唯一性，为每一个form都生成一个随机并加密的 token 验证数据，当完成提交，我们就销毁掉这个 token 从而达到form的有效性只有一次

<!-- more -->

1.编辑 form.php，开启session,并产生一个加密token保存到 session 中，并在form表单中加入一个隐藏域用来提交token
```PHP
<?php
session_start();
$token = md5(mt_rand());
$_SESSION['token'] = $token;
?>
<form action="form.php" method="post">
<input type="text" name="name">
<input type="hidden" name="token" value="<?php echo $token?>">
<input type="submit" value="submit">
</form>
```

2.编辑 post.php, 首先验证 token 是否合法，合法则处理请求，并销毁token，不合法则提示 bad request

```php
<?php
if(isset($_POST['token'])){
	if($_POST['token'] == $_SESSION['token']){
	   echo $_POST['name'];
	   unset($_SESSION['token']);
	}else{
	   echo 'bad request<br>';
	}
}
```

