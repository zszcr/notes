---
typora-copy-images-to: ../images
typora-root-url: ..

---



## github的token用法

很久没更新gitbook的内容了，今天提交时提示我要用token

![image-20220615162653635](/images/image-20220615162653635.png)

原因是github不再使用密码方式验证身份，现在使用个人token



### 生成个人token

官方文档：https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token

1. 进入个人账户的setting
2. 点解Developer settings
3. 选择Personal access tokens
4. 点击Generate new token
5. 为创建的token添加描述
6. 选择token的有效时间，可以选择永不过期
7. 为token赋予权限，如果从命令行操作仓库，至少选中repo
8. 点击Generate token，生成后的内容要先复制下来，因为离开这个页面后就看不到token的明文了



### token的使用

#### windows下使用token

使用 `https` 的方式拉取或者推送代码，每次都需要手动输入用户名和 `personal access token`，为了方便，可以使用 windows 中的凭据管理器保存相关配置，如下图所示：

![image-20220615175853985](/images/image-20220615175853985.png)

选择修改git凭据，将密码修改为token就行了



#### 命令行使用token

使用Git Credential Manager Core (GCM Core) 记住token

```shell
git push
Username: 你的用户名
Password: 你的token
# 记住token
git config credential.helper store
```







### Reference

* https://segmentfault.com/a/1190000040544939
* https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token
* https://cli.github.com/manual/gh_auth_login