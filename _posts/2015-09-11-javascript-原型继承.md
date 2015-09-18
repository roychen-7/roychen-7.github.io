---
layout: post
title: "JavaScript原型继承的区别"
description: ""
category: 
tags: ['Javascript']
---
{% include JB/setup %}
## 先贴代码

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
    super: undefined
    sub: sub
    
    New===============
    1
    2
    attr super: attrSuper
    super: super
    sub: sub

来看一下Node.js原生的继承方式（util.inherits），代码如下：

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

这个做法会导致superCtor中构造函数内的属性得不到继承，原因是Object.create根据superCtor的prototype创建的对象，而非直接使用superCtor的构造函数。
