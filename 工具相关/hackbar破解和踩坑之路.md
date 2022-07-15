---
typora-copy-images-to: ../images
typora-root-url: ..
---



##  hackbar破解和踩坑之路

### hackbar破解

作为一个倔强的白嫖怪(主要是没钱)，用到的工具都是基本都是破解版。

hackbar应该是web渗透的一个十分好用的插件，现在最新版本为2.31。它的用于检查license的代码比较简单，所以破解起来就比较简单，破解的具体步骤我参考了这篇文章[Firefox-Hackbar-2.2.9-学习版](https://fengwenhua.top/index.php/archives/36/)

我就不在重复破解步骤了，主要是这个大佬破解的版本加上了他微信公众号的广告，我看着不是很喜欢，就自己动手把它给剁了，删除掉相关的html代码就可以了。



### hackbar的bug

我在使用hackbar最新版是发现了个问题，如果post数据中包含submit=Submit这个字段的话，再点击Execute按钮时hackbar就不会发送请求。这个bug很奇怪，我曾经一度认为我是不是在破解时哪个地方删多了代码，导致这个问题，但显然不是。

打开firefox的插件调试功能，先按F12打开开发者工具，然后按F1打开设置，勾选上启用浏览器页面及附加组件的调试工具箱，然后就可以愉快的进行调试了

![image-20210626153739934](/images/image-20210626153739934.png)



对hackbar进行测试，在post数据中加入submit=Submit字段，点击Execute按钮，这时候调试器就会捕获到hackbar抛出的一个异常，如下所示：

![image-20210626154056713](/images/image-20210626154056713.png)

这个异常提示form.submit属性不是一个函数，通过控制台可以发现form.submit是form表单的一个元素，form原始的submit方法就会被覆盖，所以hackbar通过form.submit()来提交表单时就会抛出异常

![image-20210626154843671](/images/image-20210626154843671.png)

这里看下hackbar对POST数据提交的代码：

```js
    if(method === 'GET'){
        let code = 'const url = "'+ encodeURIComponent(url) +'";';
        code += 'window.location.href = decodeURIComponent(url);';
        chrome.devtools.inspectedWindow.eval(code, function(result, isException){
            setTimeout( () => { currentFocusField.focus() }, 100 );
        });
    }else{
        let code = 'var post_data = "' + encodeURIComponent(JSON.stringify(post_data)) + '"; var url = "' + encodeURIComponent(url) + '";';
        code+= 'var fields = JSON.parse(decodeURIComponent(post_data));';
        code+= 'const form = document.createElement("form");';
        code+= 'form.setAttribute("method", "post");';
        code+= 'form.setAttribute("action", decodeURIComponent(url));';
        code+= 'fields.forEach(function(f) { var input = document.createElement("input"); input.setAttribute("type", "hidden"); input.setAttribute("name", f[\'name\']); input.setAttribute("value", f[\'value\']); form.appendChild(input); });';
        code+= 'document.body.appendChild(form);'
        code+= 'form.submit();';
        exec(code)
    }
```

它表单的提交是通过form.submit()方法来提交，这也不是不行，只要发送的post数据不带submit=Submit就可以正常发送了，但是我就是想带怎么办呢，将`form.submit();`改为`HTMLFormElement.prototype.submit.call(form);`就可以正常提交了。

后面的步骤就是将插件打包，上传到开发者中心获取签名等步骤





### REFERENCE

* https://fengwenhua.top/index.php/archives/36/
* https://trackjs.com/blog/when-form-submit-is-not-a-function/

