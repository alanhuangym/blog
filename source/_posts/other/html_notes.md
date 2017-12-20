---
title: HTML学习笔记
date: 2017-9-19
tags:
- html
categories:
- html
---

### HTML

使用小写

`<br />` 是空行



浏览器会自动地在段落、标题的前后添加空行。（`<p> <h1>-<h6>`是块级元素）



避免使用`<font>`等标签了，使用style

```
<h1>Look! Styles and colors</h1>

<p style="font-family:verdana;color:red">
This text is in Verdana and red</p>

<p style="font-family:times;color:green">
This text is in Times and green</p>

<p style="font-size:30px">This text is 30 pixels high</p>
```



header 中用meta重定向

```
<head>
<meta http-equiv="Content-Type" content="text/html; charset=gb2312" />
<meta http-equiv="Refresh" content="5;url=http://www.w3school.com.cn" />
</head>
```



小于号< `&lt;`  大于号>`&gt; `空格最好用 ` &nbsp;`



上传超过一个文件

```
<form action="/example/html5/demo_form.asp" method="get">
选择图片：<input type="file" name="img" multiple="multiple" />
<input type="submit" />
</form>
<p>请尝试在浏览文件时选取一个以上的文件。</p>
```

