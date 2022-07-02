---
typora-copy-images-to: ../images
typora-root-url: ..
---





## Kali环境配置

最近将大学时代的笔记本重装了系统，装了一个2022.2版本的kali，这里记录一下kali的环境配置，以防万一



### 换源

文件地址：

```bash
/etc/apt/sources.list
```



* 官方源

  ```bash
  deb http://http.kali.org/kali kali-rolling main non-free contrib
  deb-src http://http.kali.org/kali kali-rolling main non-free contrib
  ```

* 阿里源

  ```bash
  deb https://mirrors.aliyun.com/kali kali-rolling main non-free contrib
  deb-src https://mirrors.aliyun.com/kali kali-rolling main non-free contrib
  ```

* 中科大

  ```bash
  deb http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
  deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
  ```



### 软件更新

命令：

```bash
sudo apt-get update
```





### 安装中文输入法



* 安装fcitx输入法框架

  ```bash
  sudo apt-get install fcitx 
  ```

  设置fcitx开机自启：

  将fcitx的桌面配置文件`/usr/share/applications/fcitx.desktop`拷贝到`/etc/xdg/autostart`目录下，开机就会自动启动

* reboot

然后先介绍失败的方法，我这里kali版本是2022.2，网上成功的方法在这行不通，就是那个下载fcitx-googlepinyin的，怎么都成功不了，最后下载搜狗拼音才成功了

参考链接：https://www.icode9.com/content-3-1286056.html

直接在搜狗官网下载.deb安装包

![image-20220615111138765](/images/image-20220615111138765.png)

然后使用dpkg -i 软件包名命令安装输入法

```bash
sudo dpkg -i sogoupinyin_4.0.1.2123_amd64.deb
```

然后在fcitx设置中将搜狗拼音置顶，通过ctrl+space就可以切换中英了



### zsh配置

安装oh-my-zsh

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

设置主题为random

配置文件：`~/.zshrc`



### VScode安装

在官网下载deb安装包，然后通过dpkg -i 来安装



### typora安装

typora从1.0版本就开始收费了，这里去网上找了下它未收费前的版本

参考链接：https://www.bilibili.com/read/cv14501670

直接下.deb包，使用`dpkg -i `命令安装



### 科学上网

* ssr

  https://github.com/the0demiurge/CharlesScripts/blob/master/charles/bin/ssr

  。。。。过过过，好大的坑要踩，之前装过但是没做踩坑记录

这里直接使用我台式的小飞机，通过局域网将流量转发到主机

这里kali在网络设置中设置网络代理，将代理地址填写台式主机的局域网ip地址，端口填小飞机的端口

而主机中的小飞机打开设置，本地代理中设置允许来自局域网的连接

![image-20220614195122606](/images/image-20220614195122606.png)

如果是命令行下的科学上网，可以使用proxychains

* proxychains

  这个是一个终端命令行代理神器

  安装：

  ```bash
  sudo apt-get install proxychains
  ```

  配置：

  ```bash
  sudo vim /etc/proxychains.conf
  ```

  ![image-20220614195536390](/images/image-20220614195536390.png)

用法：

```bash
proxychains wget https://www.google.com
```





### SSH设置

SSH配置文件地址：

```bash
/etc/ssh/sshd_config
```

修改命令：

```bash
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config
```



SSH相关服务：

```bash
# 启动ssh
/etc/init.d/ssh start

# 设置为开机自启
update-rc.d ssh enable
```





### nc的配置

Kali 默认的 nc 版本是 netcat-traditional，虽然功能比较全，但是 2007 年 1 月的时候就停止更新了，所以额外安装个 ncat

```bash
apt update
apt install ncat
```

安装完成后默认的`nc`命令实际上指向变成了`ncat`，手动把`nc`命令换回`netcat-traditioanl`版本

```bash
update-alternatives --config nc
```



### 截屏工具

```bash
sudo apt-get install flameshot
```

设置快捷键alt+a为截屏

![](/images/2022-06-15_16-02.png)



### 安装docker

```bash
# 添加Docker PGP密钥
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

# 配置docker apt源 我这里用的国内阿里云的docker下载源
echo 'deb https://mirrors.aliyun.com/docker-ce/linux/debian buster stable'> /etc/apt/sources.list.d/docker.list

# 更新apt源
apt update

# 如果之前安装了docker的话 这里得卸载旧版本docker
apt remove docker docker-engine docker.io

# 安装docker
apt install docker-ce

# 查看版本
docker version
```



#### docker换源

配置文件：`/etc/docker/daemon.json`

加中国源，可以多加几个：

```
{
  "registry-mirrors": ["https://bytkgxyr.mirror.aliyuncs.com","https://registry.docker-cn.com","http://hub-mirror.c.163.com"]
}
```

然后重启 Docker

```sh
service docker restart
```



#### 安装docker-compose

最新发行的版本地址：https://github.com/docker/compose/releases

运行以下命令以下载 Docker Compose 的当前稳定版本：

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.6.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

将可执行权限应用于二进制文件：

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

创建软链：

```bash
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

测试是否安装成功：

```bash
┌──(n1rvana㉿kali)-[~]
└─$ docker-compose --version
Docker Compose version v2.6.0
```



### Firefox ESR

#### 汉化

这个是kali自带的浏览器，它默认是英文的，可以通过命令行安装汉化包

```bash
apt install firefox-esr-l10n-zh-cn
```



#### 安装hackbar

破解方法参照之前写的文章，装完后要关闭更新





### 替换burpsuite

将社区版burpsuite替换为专业版的，这里我选的版本是2022.5.1

1. 安装jdk

   ```bash
   #确定java版本
   java --version
   #确定jdk安装路径
   update-alternatives --config java
   # 默认位置为/usr/lib/jvm/
   
   # 将下载好的JDK包解压至/usr/lib/jvm/
   tar zvxf jdk-17_linux-x64_bin.tar.gz 
   mv ./jdk-17.0.3.1/ /usr/lib/jvm/
   
   # 更新kali可以使用的java版本
   update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk-17.0.3.1/bin/java 777
   
   # 修改java版本，选择jdk-17.0.3.1
   update-alternatives --config java
   
   # 确认java版本
   Java --version
   
   ```

2. 下载burpsuite和注册机

   注册机地址：https://github.com/h3110w0r1d-y/BurpLoaderKeygen/releases

   burpsuite地址：https://portswigger.net/burp/Releases

   ```bash
   #将下载的文件都拷贝到/usr/bin目录下
   sudo cp -r burpsuite_pro_v2022.5.1.jar BurpLoaderKeygen.jar /usr/bin
   ```

3. 运行注册机激活burpsuite

   ```bash
   sudo java -jar BurpLoaderKeygen.jar
   ```

   常规激活操作，参考：https://blog.csdn.net/zw05011/article/details/122459723

   这里之所以用sudo来操作，是因为普通用户状态下它的`_JAVA_OPTIONS`环境变量会导致运行失败

   错误如下所示：

   ```bash
   ☁  ~  java -jar BurpLoaderKeygen.jar 
   Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
   Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: Index 1 out of bounds for length 1
   	at com.h3110w0r1d.burploaderkeygen.KeygenForm.getJavaVersion(KeygenForm.java:60)
   	at com.h3110w0r1d.burploaderkeygen.KeygenForm.GetCMD(KeygenForm.java:103)
   	at com.h3110w0r1d.burploaderkeygen.KeygenForm.main(KeygenForm.java:268)
   ```

   它这里显示KeygenForm.java:60发生了数组下标溢出，它在获取Java版本时产生了溢出，我猜测是java版本号太长？

   我这里使用的java版本如下：

   ```
   java 17.0.3.1 2022-04-22 LTS
   Java(TM) SE Runtime Environment (build 17.0.3.1+2-LTS-6)
   Java HotSpot(TM) 64-Bit Server VM (build 17.0.3.1+2-LTS-6, mixed mode, sharing)
   ```

   我淦，真的是java版本问题，我这里切回kali自带java就行了

4. 设置启动快捷键

   先删除burpsuite社区版

   ```bash
   cd /usr/bin
   sudo rm -rf burpsuite
   ```

   然后新建burpsuite，burpsuite内容如下

   ```bash
   sudo vim burpsuite
   
   #!/bin/sh
   java -jar BurpLoaderKeygen.jar
   ```

   增加权限

   ```bash
   sudo chmod +x burpsuite
   ```

   这就基本搞定了，我这里还没有搞桌面快捷键......在终端输入burpsuite就可以用了

   

   

   

   

   

