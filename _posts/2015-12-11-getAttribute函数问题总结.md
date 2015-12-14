---
layout: post
title: "getAttribute函数问题总结"
description: ""
category: 
tags: ['Javascript']
---
{% include JB/setup %}
### 起因

最近在做一个文件上传的功能，要根据一个select中内容的变化，来决定要上传哪些文件。也就是说：

	// 伪代码
	$('select').change(function() {
		var html = '一组上传空间';
		$('#upload-container').html(html);
	});

对，大概就是这样的一个小功能点。在Chrome和FireFox下跑的都是ok的，但是一旦到了ie下，就会报一个著名问题：**“缺少对象”**。

补充说明一下，jQuery版本是1.11.3，上传控件是SWFUpload

### 排查过程

因为渲染的逻辑是通过一个匿名函数传进去的，根据函数栈的调用顺序，最终报出错误的地方其实是change这个函数被调用的地方。由于经验不足，当时我是从最初报error的地方一点点console.log出来的，过程大概如下：

	.change()
	.html()
	.cleanData()
	.acceptData()
	getAttribute()

可以看到，这是jQuery在.html()方法中，想要清理对象的时候产生的问题。jQuery会对原来DOM下所有元素做一个事件缓存的卸载。当它找到<object>（Flash）这个DOM元素的时候，发生了这个错误。同时，当找到getAttribute这个函数的时候，我内心几乎是崩溃的，因为这个函数是Javascript原生的函数，如果这个函数有问题，那就说明每个浏览器在执行这个方法的时候，表现不一样。最终，说明要避开这个问题，我要么做兼容（因为在jQuery内部，所以没法做兼容），要么绕过去。

### 现象

先撇开业务问题不说，我在遇到这个问题后，在3大浏览器中（Chrome、Firefox、IE），做了一些对比：

#### 其一：console.log(document.getElementById('SWFUpload_0')): 
	
1. Chrome: 能找到对应的function，不过随后就会报一个错误：*Uncaught TypeError: Cannot use 'in' operator to search for 'rawScopes' in undefined*
2. Firefox: 直接找到了<object>这个对象，并没有报错误
3. IE: 直接找到了HTML的DOM对象，也没有报错误

#### 其二：console.log(document.getElementById('SWFUpload_0').getAttribute('classid'))（jQuery中真正用到的方法）: 
	
1. Chrome: null
2. Firefox: null
3. IE: 报错：*缺少对象*

从这里比较中可以看出，Firefox是表现最好的，返回我们期望的值，并且不会报错。IE是最耿直的，能拿到就拿，一旦不行就告诉你“缺少对象”（花式虐狗啊），Chrome则是最矫情的，一旦遇到Object（Flash）就报错，但为了让开发者尽量少受困扰，则特殊处理了getAttribute函数，使其不会报错。

### 总结

##### Flash在网页技术中一直是很有争议的一个技术，它既能提高用户体验，图片上传、跨域等等，还得到了各大浏览器不错的兼容，但也会有行为不一致带来的困扰和安全性上的问题（xss）。我也没什么很好的建议，只是总结一个自己遇到的问题，然后告诉大家尽量慎用Flash，同时期待H5时代的到来。
