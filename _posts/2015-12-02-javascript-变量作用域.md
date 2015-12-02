---
layout: post
title: "Javascript 变量作用域"
description: ""
category: 
tags: ['Javascript']
---
{% include JB/setup %}
### 在ECMAScript6之前，Javascript中，变量的作用域只和函数有关。 

在Javascript这门语言中，有时我们会混淆“代码块”和“作用域”这两个概念。

####首先，我们可以明确的是，“代码块” != “作用域”。

也就是说，在一个代码块（例如：if for）中定义的变量，在这个代码块执行结束之后，还是会在当前作用域中找到的。如下代码：

	for (var i in [1, 2]) {
		var a = 1;	
	}

	console.log(a); // 1

这里可能会造成我们混淆的原因是，有时候for或者if内部的代码不执行，那么我们将拿到一个undefined。这会使得我们误认为，a在for或者if代码块结束后就被销毁了。如下代码：
	
	if (false) {
		var a = 2;		
	}

	console.log(a); // undefined

####其次，函数才是决定Javascript中，变量生命周期的因素。

当一个函数被调用结束后，vm会做退栈的操作，那么相应的在这个函数栈里面的变量等等会一并被销毁。

	function a () {
		var b = 2;	
	}

	a();
	console.log(b); // 报错，b未定义

####再次，还是要提一下不加var的变量申明。

上面的结论都是建立于通过var关键字申明的变量，一旦直接申明变量，例如：

	function a() {
		b = 2;
	}

	a();
	console.log(b); // 2

此时，解释器会沿着作用域向上找b的申明，最终发现直到顶层作用域也没有找到这个变量申明的地方，所以对解释器来说，无法知道这个变量是在哪里申明的，就默认将它放到了顶层作用域中。

####最后，因为Javascript的函数式特性，这里还是要粗浅的说一下闭包。

当一个函数被赋值给一个变量的时候，该函数的上下文会被保存下来（不被释放、回收）。

	function fn() {
		var count = 0;
		return function fn2() {
			return ++count;	
		}
	}

	var  a = fn();
	a();
	console.log(a());  // 2

这里，因为Javascript的functional特性，我们可以将函数fn2作为返回值给到变量a。在做这个赋值操作的同时，fn2的上下文fn1不会被释放（因为fn2执行时，fn1内的变量会被用到），所以count这个变量被保存了下来。这个例子只能体现2层关系，接下来有一个更加直观的例子：

	function fn1() {
		var count = 0;
		return function fn2() {
			return function fn3() {
				return ++count;
			}
		}
	}

	a = fn1();
	var c = a();
	c();
	console.log(c()); // 2

这个例子中，fn3从父亲的父亲fn1中获取到了count，并对其值进行修改。

#### 以上例子都是为了说明变量在Javascript中的生命周期，最后有一些自己的经验：

#####1. 在代码块（if，for）中会做修改，并且会被后续代码用到的变量，还是在相应代码块之前做申明比较好，以免代码块不执行时，后续使用报错。

#####2. 在业务逻辑中，不要使用不必要的闭包设计（前后端都是），这样可能会带来很大限度上的内存浪费，以至于造成不可复现的异常问题。
