---
layout: post
title: "FileUpload"
description: ""
category: 
tags: ['Javascript']
---
{% include JB/setup %}
## 文件上传

### 原生的HTML

	<input type="file">

这是最原始的一种文件上传方法，利用原生的html规范，来上传文件。这种方式的优点就是简单，只要浏览器对html标准支持的足够好，就不会有任何问题和差异化。同样，这种方式的缺点也很明显：

1. 由于浏览器安全性的限制，不能做到文件预览（ie除外）。
2. 没有进度条，无法知道文件上传的速度和进度。

以上两点，就已经基本无法复合现代web的设计需求了。

### 基于Flash的文件上传

这种方式是当下最流行的一种文件上传方式，既支持ie、firefox、chrome等等主流浏览器，又能做到有进度条、有预览等友善的交互需求。基于这种方式的文件上传，还有很多知名的库，比方swfUpload、Uploadify等等。这些库几乎都支持beforeUpload、uploading、onComplete、onFailed等等方法，通过flash提供一些基础功能给到js，以便适合各种各样的业务场景。

但Flash毕竟是Flash，对很多只会Javascript的程序员来说，Flash一直是一个黑盒。当出现一些问题的时候，或者我们希望能够更加个性化定制一下上传控件的时候，可能就是一件很头疼的事情了。除此之外，Flash在各个浏览器下的表现还是有差异的，这里说一个我遇到的坑：FireFox下，基于swfUpload的文件上传。

在ie和chrome中，使用swfUpload来做文件上传，Flash在做HTTP请求的时候，会将当前的cookie从浏览器中带到Server，从而实现了一个会话的保持。可是在FireFox下，Flash的cookie貌似和浏览器的是分开的，从Flash中发出的请求，会带一个新的cookie值到server，这样就没办法保持会话和用户的登录身份了，所以，这里需要一些“特殊处理”。

首先，我们要了解session的原理，它是通过一个埋在cookie中的字符串，来验证用户的。那么理论上，只要我们知道这个字符串，并且能把它告诉服务器，那么依旧可以达到用户身份验证的效果。

其次，就是我们需要在请求中加入这个值。这里，我们利用到了，Flash提供的一些基础配置：目标url。在初始化对象时，将cookie中session的值拼接在url中，类似：

	new FileUpload({
		url: '/file/upload?session_no=' + $.cookie('session_id')	
	});

最后，服务器获取到session_no的值后，设置到session里面，再重启session即可生效，php代码如下：

	<?php

	session_write_close();
	session_id($_GET['session_no']);
	session_start();

	?>

这样，我们既支持了FireFox下，文件上传时的用户登录态，也没有留下什么漏洞。

ps: 再补充几句，因为FireFox还没有可以看到Flash中网络请求的功能（可能是我自己不知道，FF用的比较少），我还是通过别的工具，抓包知道服务器返回了302“请登录”的页面。

### HTML5

此处留个坑，等我研究完再说。
