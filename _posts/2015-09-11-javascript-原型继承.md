---
layout: post
title: "JavaScript原型继承的区别"
description: ""
category: 
tags: ['Javascript']
---
{% include JB/setup %}
## 先说剧情概要

众所周知，Javascript是一门函数式编程语言，函数在Javascript中属于一等公民。同时，不像别的一些函数式编程语言，Javascript本身缺少一些面向对象的特性，比方在ES6之前，缺少关键字Class，更加别提extends、interface之类的了。所以，Javascript本身没有“类”和“继承”的概念，但我们在使用中会因此受到一些在代码组织方面的困扰，而大家会使用一些Javascript特有的方式来实现意义上的“类”和“继承”。

在ES6的标准出台之前，在Javascript中实现继承的方式有两种，一种是基于原型链的继承（我们接下来主要讨论的内容），另一种是基于函数的混入式继承（我们这里不会讨论）。

在基于原型链的继承方式下，也有两种实现，一种是 B.prototype = Object.create(A.prototype)，另一种是 B.prototype = new A()。因为实现的方式不同，结果上也有细微差异，接下来请看代码：

## 现在上贴代码

    // Prototype
    // Object Create
    console.log("Object Create===============");
    gen(function(superClass, subClass) {
        subClass.prototype = Object.create(superClass.prototype);
    });

    // New
    console.log("New===============");
    gen(function(superClass, subClass) {
        subClass.prototype = new superClass;
    });
    
    function gen (inheritFunc) {
        // Define super Class
        var superClass = function() { this.super = 'super'; };
        superClass.prototype.func1 = function() {console.log(1);}
        superClass.prototype.func2 = function() {console.log(2);}
        superClass.prototype.attrSuper = 'attrSuper';
    
        // Define sub Class
        var subClass = function() { this.sub = 'sub' };
        subClass.prototype.func1 = function() {console.log(11);}
        inheritFunc(superClass, subClass);
     
        // Tester
        var ins = new subClass();
    
        ins.func1();
        ins.func2();
     
        console.log('attr super: ' + ins.attrSuper);
        console.log('super: ' + ins.super);
        console.log('sub: ' + ins.sub);
    }

Object.create 的做法只针对 Super Class的原型，而不会调用到Super Class的构造方法，所以Super Class内部的this不会生效。

结果

    Object Create===============
    1
    2
    attr super: attrSuper
    super: undefined // Super Class构造函数内部定义的属性不会得到继承
    sub: sub
    
    New===============
    1
    2
    attr super: attrSuper
    super: super
    sub: sub

那么Node.js原生的继承方式（util.inherits）是怎样的呢？我们来看一下，代码如下：

    exports.inherits = function(ctor, superCtor) {
        ctor.super_ = superCtor;
        ctor.prototype = Object.create(superCtor.prototype, {
            constructor: {
            value: ctor,
            enumerable: false,
            writable: true,
            configurable: true
          }
        });
    };

这个做法就如之前说的，会导致superCtor中构造函数内的属性得不到继承，个人觉得这种做法限制javascript本身的一些特性，不是很好，不过各位可以根据自己的实际需求来评判是用原生的还是采用new的方式。
