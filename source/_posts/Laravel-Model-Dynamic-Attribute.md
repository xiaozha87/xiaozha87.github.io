---
title: Laravel模型中的动态属性
date: 2018-08-30 14:50:22
tags:
- Laravel
category: 
- Laravel
---

setxxAttribute 在设置(sql: insert update) 的时候 会将$obj->xx = 'value'的时候, 操作数据库之前 自动转化一下

getxxAttribute 在获取xx属性的时候  $obj->xx 会转化

```php
    //获取之前首字母大写
    public function getFirstNameAttribute($value)
    {
        return ucfirst($value);
    }
    //存入之前的改变
    public function setFirstNameAttribute($value)
    {
        $this->attributes['first_name'] = strtolower($value);
    }
```
<font color="#dd0000">注意：使用驼峰格式命名</font>
<!-- more -->

# 一、引入
一对一关联是很基本的关联。例如一个User模型也许会对应一个Phone。要定义这种关联，我们必须将phone方法放置于User模型上。phone方法应该要返回基类Eloquent上的hasOne方法的结果：
```php
<?php
namespace App; 
use Illuminate\Database\Eloquent\Model; 
class User extends Model 
{ 
	/** * 获取与指定用户互相关联的电话纪录。 */ 
	public function phone() 
	{ 
		return $this->hasOne('App\Phone'); 
	} 
}
```
传到hasOne方法里的第一个参数是关联模型的类名称。定义好关联之后，我们就可以使用Eloquent的动态属性来获取关联纪录。动态属性让你能够访问关联函数，就像他们是在模型中定义的属性：
```php
$phone = User::find(1)->phone;
```
但这里只讲了动态属性的最简单的一种形式，也就是调用的属性不存在，但存在同名的方法时，则会调用同名的方法，返回的类型是collection类型（Eloquent的集合）。下文让我们走一遍Laravel的源代码看看还有其他几种不同种类的动态属性。
# 二、Laravel源代码trace
## 1、对于动态属性疑问的产生
在项目的blade里用到了一个方法，但是user的model里并不存在同名的avatar_src()方法，但是存在一个getAvatarSrcAttribute()名字有点像的方法，当时就觉得很懵逼，看代码的确是调用了这个方法，但不知是如何关联起来的，所以想搞明白这里面的逻辑究竟是怎么回事。
```php
<a href="#"><img src="{{ Auth::user()->avatar_src }}" alt=""></a>
```
## 2、__get()
那么问题来了，如何追溯？这里需要的一个预备知识是关于PHP的魔术方法__get()，当读取不可访问属性的值时，__get()会被调用。所以决定从这个方法开始进行追溯。具体的方法是在PhpStorm里打开user模型的代码，在菜单栏选择Navigate-File Structure，弹出的框子里勾选Show inherited members，英文输入状态下输入get可以找到我们想要的方法，点进去可以看到__get()方法源代码如下：
```php
/** 
* Dynamically retrieve attributes on the model. 
* @param string $key 
* @return mixed 
*/ 
public function __get($key) 
{ 
	return $this->getAttribute($key); 
}
```
## 3、getAttribute($key)
```php
/** 
* Get an attribute from the model. 
* @param string $key 
* @return mixed
 */ 
 public function getAttribute($key) 
 { 
	 if (array_key_exists($key, $this->attributes) || $this->hasGetMutator($key)) { 
		return $this->getAttributeValue($key); 
	 } 
	 return $this->getRelationValue($key); 
 }
```
第一个if的左半边，如果这个model有这个attribute那么就直接返回，没什么可说的。
第一个if的右半边mutator是变异体的意思事实上处理了本节开头的疑问，看一下源代码：
```php
/** 
* Determine if a get mutator exists for an attribute. 
* @param string $key 
* @return bool 
*/ 
public function hasGetMutator($key) 
{ 
	return method_exists($this, 'get'.Str::studly($key).'Attribute'); 
}
```
本方法的作用是判断所调用的这个不存在的属性是否存在“按照一定格式变形的类似名字的方法”。所谓的“一定格式”可以参考Studly caps命名法，对应的源代码：
```php
 /** 
 * Convert a value to studly caps case.
 * @param string $value 
 * @return string 
 */ 
 public static function studly($value) 
 { 
	 $key = $value; 
	 if (isset(static::$studlyCache[$key])) 
	 { 
		return static::$studlyCache[$key]; 
	 } 
	 $value = ucwords(str_replace(['-', '_'], ' ', $value)); 
	 return static::$studlyCache[$key] = str_replace(' ', '', $value); 
 }
```
注意到经studly caps处理过的-和_都会被去掉。再回到hasGetMutator($key)这个方法，我们可以看到Laravel会尝试去寻找名字形似getStudlyCapsNameAttribute()的方法，如果有的话则会在getAttribute($key)里返回相关的值。第一小节提到的例子对应的方法名我们可以知道当调用这个不存在的属性avatar_src时，Laravel会尝试调用getAvatarSrcAttribute()这个方法，看了下代码果然是存在这个方法的，开始的疑问解决啦~
## 4、getRelationValue($key)

回到getAttribute($key)这个方法，如果在第一个if里没有返回则会调用getRelationValue($key)这个方法，源代码如下：
```php
/** 
* Get a relationship. 
* @param string $key 
* @return mixed 
*/ 
public function getRelationValue($key) 
{ 
	// If the key already exists in the relationships array, it just means the 
	// relationship has already been loaded, so we'll just return it out of 
	// here because there is no need to query within the relations twice. 
	if ($this->relationLoaded($key)) 
	{ 
		return $this->relations[$key]; 
	} 
	// If the "attribute" exists as a method on the model, we will just assume 
	// it is a relationship and will load and return results from the query 
	// and hydrate the relationship's value on the "relationships" array. 
	if (method_exists($this, $key)) 
	{ 
		return $this->getRelationshipFromMethod($key); 
	} 
}
```
第一个if注释写得很清楚了，第二个if就是判断是否存在和所调用属性同名的方法，如果存在则调用getRelationshipFromMethod($key)方法。
## 5、getRelationshipFromMethod($method)

这个方法比较关键，我们看一下源代码：
```php
/** 
* Get a relationship value from a method. 
* @param string $method 
* @return mixed * 
* @throws \LogicException 
*/ 
protected function getRelationshipFromMethod($method) 
{ 
	$relations = $this->$method(); 
	if (! $relations instanceof Relation) 
	{ 
		throw new LogicException('Relationship method must return an object of type ' .'Illuminate\Database\Eloquent\Relations\Relation'); 
	} 
	$this->setRelation($method, $results = $relations->getResults()); 
	return $results; 
}
```
注意if语句块那里的判断，意味着与属性同名的方法的返回类型必须是Relation类型或者是它的子类，例如hasMany等。所以如果要另外做处理，返回的类型不为Relation的话可以参考第四小节那样的命名法构造相关方法名。另外，setRelation那一行的意思是将没有加载的relation进行加载，那么下次需要时就可以在getRelationValue($key)的第一个if中即返回需要的结果。还有值得注意的是此方法最后的返回值返回的$results是Collection类型，也就是说如果调用不存在的动态属性后返回的是Collection类型，而如果我们直接调用方法返回的则是Relation类型，可以在其上构造查询进一步处理，而再调用getResults()后才能再获得Collection类型的返回值。


