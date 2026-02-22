---
layout: post
title: '基于LeanCloud实现留言板'
subtitle: '网页留言板'
date: 2020-05-14
categories: tech
author: yucol
cover: 'https://images.unsplash.com/photo-1529322365446-6efd62aed02e?w=1600&q=900'
cover_author: 'inma santiago'
cover_author_link: 'https://unsplash.com/@inmasantiago'

tags: 前端
music-id: 35307030
---


> 学习教程[https://www.jianshu.com/p/9880042a6658](https://www.jianshu.com/p/9880042a6658)

> 请勿滥用他人的密钥，建议自行创建一个LeanCloud账号。

> 送佛不如送到西，因为这个写的太丑了，所以我也不会用他，推荐使用[Valine](https://valine.js.org/)插件，插件是基于LeanCloud搭建的，可以认为是这个留言板的Plus版。我博客中使用的就是这个。 

效果可以访问网址：[https://test.jmbaozi.top/message/](https://test.jmbaozi.top/message/) 查看或下载[源代码](https://github.com/JMbaozi/absorb/tree/master/program/%E5%89%8D%E7%AB%AF%E5%AD%A6%E4%B9%A0/message)本地浏览。

LeanCloud官网：[https://www.leancloud.cn/](https://www.leancloud.cn/)

[数据存储开发指南 · JavaScript](https://leancloud.cn/docs/leanstorage_guide-js.html)

按照官网流程创建好应用，找到自己的密钥，用于初始化：
```js
AV.init({
    appId: APP_ID,
    appKey: APP_KEY
});
````
在初始化时需要引入av-min.js(其他版本请自行查阅):
```html
<script src="//cdn1.lncld.net/static/js/3.5.0/av-min.js"></script>
```

根据实际情况设置输入框，比如昵称、邮箱、网址和留言框等：
```html
<form id="postMessageForm" style="width: 700px; margin: 50px auto;">
    <input placeholder="昵称" type="text" name="name">
    <input placeholder="邮箱" type="text" name="email">
    <input placeholder="个人网站" type="text" name="site">
    <br>
    <textarea id="content" placeholder="欢迎留言(oﾟvﾟ)ノ" name="content"></textarea>
    <br>
    <input id="submit" type="submit" value="提交">
</form>
```
用户输入后将输入的内容保存到LeanCloud中：
```js
myForm.addEventListener('submit', function(e) {
    e.preventDefault()
    let content = myForm.querySelector('textarea[name=content]').value
    let name = myForm.querySelector('input[name=name]').value
    let email = myForm.querySelector('input[name=email]').value
    let site = myForm.querySelector('input[name=site]').value
    var Message = AV.Object.extend('Message');
    var message = new Message();
    message.save({
        'name':name,
        'email':email,
        'site':site,
        'content':content
    })
})
```
为了实现留言显示到网页中，可以将留言框内的内容输出到```<div>```中的```<li>```或```<P>```标签中。

```html
<div class="messagediv">
    <div id="messageList"></div>
</div>
<script>
var query = new AV.Query('Message');
query.find()
    .then(
        function(messages) {
            let array = messages.map((item) => item.attributes)
            array.forEach((item) => {
                let p = document.createElement('p')
                p.innerText =`${item.name}：${item.content}`
                let messageList = document.querySelector('#messageList')
                messageList.appendChild(p)
            })
        }
    )
</script>
```
这样存在的问题是，在留言后需要重新刷新网页才能显示，所以设置一下在留言后自动显示：
```js
myForm.addEventListener('submit', function(e) {
    e.preventDefault()
    let content = myForm.querySelector('textarea[name=content]').value
    let name = myForm.querySelector('input[name=name]').value
    let email = myForm.querySelector('input[name=email]').value
    let site = myForm.querySelector('input[name=site]').value
    var Message = AV.Object.extend('Message');
    var message = new Message();
    message.save({
        'name':name,
        'email':email,
        'site':site,
        'content':content
    }).then(function(object) { //obiect为存入的数据的相关信息
        let p = document.createElement('p')
        p.innerText =`${object.attributes.name}：${object.attributes.content}        ${object.attributes.time}`
        let messageList = document.querySelector('#messageList')
        messageList.appendChild(p)
        myForm.querySelector('textarea[name=content]').value=''
    })
})
```

这时，就基本完成了一个网页留言板，为了美观，可以设置CSS样式，或者添加其他内容，因为我是为了完成作业，并没有做什么过多的美化，所以以实用为主。

### 总代码

#### HTML

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="//cdn1.lncld.net/static/js/3.5.0/av-min.js"></script>
    <link rel="stylesheet" href="css/message.css">
</head>
<body background="img/memphis-colorful.png">
    <section class="message">
        <center>
            <h2 id="title">留言板</h2>
        </center>
        <form id="postMessageForm" style="width: 700px; margin: 50px auto;">
            <input placeholder="昵称" type="text" name="name">
            <input placeholder="邮箱" type="text" name="email">
            <input placeholder="个人网站" type="text" name="site">
            <br>
            <textarea id="content" placeholder="欢迎留言(oﾟvﾟ)ノ" name="content"></textarea>
            <br>
            <input id="submit" type="submit" value="提交">
        </form>
    </section>
    <div class="messagediv">
        <div id="messageList"></div>
    </div>
</body>
<script src="js/message.js"></script>
</html>
```

#### JS

```js
// 初始化
AV.init({
    appId: APP_ID,
    appKey: APP_KEY
});

var query = new AV.Query('Message');
query.find()
    .then(
        function(messages) {
            let array = messages.map((item) => item.attributes)
            array.forEach((item) => {
                let p = document.createElement('p')
                p.innerText =`${item.name}：${item.content}        ${item.time}`
                let messageList = document.querySelector('#messageList')
                messageList.appendChild(p)
            })
        }
    )

let myForm = document.querySelector('#postMessageForm')

myForm.addEventListener('submit', function(e) {
    e.preventDefault()
    let content = myForm.querySelector('textarea[name=content]').value
    let name = myForm.querySelector('input[name=name]').value
    let email = myForm.querySelector('input[name=email]').value
    let site = myForm.querySelector('input[name=site]').value
    var myDate = new Date();
    var time = myDate.toLocaleString( );
    var Message = AV.Object.extend('Message');
    var message = new Message();
    message.save({
        'name':name,
        'email':email,
        'time':time,
        'site':site,
        'content':content
    }).then(function(object) { //obiect为存入的数据的相关信息
        let p = document.createElement('p')
        p.innerText =`${object.attributes.name}：${object.attributes.content}        ${object.attributes.time}`
        let messageList = document.querySelector('#messageList')
        messageList.appendChild(p)
        myForm.querySelector('textarea[name=content]').value=''
    })
})

```

#### CSS

```css
#title{
    top: 50px;
    font-size: 33px;
    color: skyblue;
}
.messagediv{
    text-align: center;
}
.messagediv p{
    list-style: none;
    font-size: 30px;
    color: skyblue;
    border: 1px solid transparent;
    border-color: slategray;
}
#content{
    width: 630px;
    height: 300px;
    font-size: 15px;
}
#submit{
    width: 80px;
    height: 40px;
    background-color: #ffffff;
    border: 1px solid transparent;
    border-color: turquoise;
    cursor:pointer;
    transition: all 0.5s ease;
}
#submit:hover{
    background: turquoise;
    box-shadow:5px 20px 20px #999999;
}
#submit:focus{
    outline: none;
}
#header input{
    width: 200px;
}
```

![](https://lz.sinaimg.cn/orj1080/ebeef3aaly3ges68ax7enj20zd1b5gs5.jpg)