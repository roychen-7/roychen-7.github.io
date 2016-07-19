---
layout: post
title: "Javascript-多重继承"
description: ""
category: 
tags: ['Javascript']
---
{% include JB/setup %}
## Javascript 多重继承

在Javascript中通过原型链实现继承，我们已经在之前的[文章](http://roychen-7.github.io/2015/09/11/javascript-%E5%8E%9F%E5%9E%8B%E7%BB%A7%E6%89%BF)中大致提过了，这里大概讲一讲多重继承的实现方法。

### 一言不合线上代码：

	// Abstract
	function Person(name) {
	    this.name = name;
	    this.sayName = () => { console.log(this.name); }
	}
	
	function Animal(age) {
		this.age = age;
		this.sayAge = () => { console.log(this.age); }
	}
		 
	// Multiple inherit
	function Man(name, age) {
		Person.call(this, name);
		Animal.call(this, age);
	}
			
	var roy = new Man('Roy', 25);
	roy.sayName();
	roy.sayAge();


### 无疑这种通过将this对象传递，实现在this上绑定fn的做法是最优雅的多重继承实现方法。

不过在思考到这种写法的过程中，我第一个想法是把function一个个通过循环的方式复制到某个函数的this上来实现多重继承，大致如下：

	function Person(name, age) {
		this.name = name;
		this.age = age;

		this.sayName = () => { console.log(this.name); }
		this.sayAge = () => { console.log(this.age); }
	}

	function Man(name, age) {
		var _parent = new Person(name, age);

		// Mixin
		for (var key in _parent) {
			if (!this[key]) {
				this[key] = _parent[key];
			}
		}
	}

	var roy = new Man('Roy', 25);

	console.log("ROY==========");
	roy.sayName();
	roy.sayAge();
       
可这种循环的写法的逼格就比较一般了。
