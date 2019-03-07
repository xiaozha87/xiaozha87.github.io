---
title: 移动端如何让页面强制横屏
date: 2019-03-07 11:31:55
tags:
- CSS3
- Javascript
category: 
- CSS3 $ Javascript
---
### Html页面结构如下： ###
```html
<body class="webpBack">
  <div id="print">
      <p>lol</p>  
   </div>
</body>
```
<!-- more -->
### CSS样式 ###
```css
@media screen and (orientation: portrait) {
      html{
         width : 100% ;
         height : 100% ;
          background-color: white ;
          overflow : hidden;
      }
      body{
          width : 100% ;
         height : 100% ;
         background-color: red ;
          overflow : hidden;
      }
      #print{
         position : absolute ;
         background-color: yellow ;
      }
} 
@media screen and (orientation: landscape) {
       html{
         width : 100% ;
         height : 100% ;
         background-color: white ;
      } 
       body{
          width : 100% ;
         height : 100% ;
         background-color: white ;
      }
           #print{
            position : absolute ;
            top : 0 ; 
            left : 0 ;
            width : 100% ;
            height : 100% ;
            background-color: yellow ;
         }
}
#print p{
        margin: auto ;
        margin-top : 20px ;
        text-align: center;
      }
```
把print这个div在竖屏模式下横过来，横屏状态下不变。所以在portrait下，没定义它的宽高。会通过下面的js来补。
```Javascript
  var width = document.documentElement.clientWidth;
  var height =  document.documentElement.clientHeight;
  if( width < height ){
      console.log(width + " " + height);
      $print =  $('#print');
      $print.width(height);
       $print.height(width);
      $print.css('top',  (height-width)/2 );
      $print.css('left',  0-(height-width)/2 );
      $print.css('transform' , 'rotate(90deg)');
       $print.css('transform-origin' , '50% 50%');
 } 
```
在这里我们先取得了屏幕内可用区域的宽高，然后根据宽高的关系来判断是横屏还是竖屏。如果是竖屏，就把print这个div的宽高设置下，对齐，然后旋转。
最后，这么做带来的后果是，如果用户手机的旋转屏幕按钮开着，那么当手机横过来之后，会造成一定的悲剧。解决办法如下:
```Javascript
 var evt = "onorientationchange" in window ? "orientationchange" : "resize";
    window.addEventListener(evt, function() {
        console.log(evt);
        var width = document.documentElement.clientWidth;
         var height =  document.documentElement.clientHeight;
          $print =  $('#print');
         if( width > height ){
           
            $print.width(width);
            $print.height(height);
            $print.css('top',  0 );
            $print.css('left',  0 );
            $print.css('transform' , 'none');
            $print.css('transform-origin' , '50% 50%');
         }
         else{
            $print.width(height);
            $print.height(width);
            $print.css('top',  (height-width)/2 );
            $print.css('left',  0-(height-width)/2 );
            $print.css('transform' , 'rotate(90deg)');
            $print.css('transform-origin' , '50% 50%');
         }
        
    }, false);
```
[Demo](https://link.jianshu.com/?t=http://www.chubao.cn/s/godness/index.html)

