---
layout: post
title: "Javascript 抛出异常"
description: ""
category: 
tags: ['Javascript']
---
{% include JB/setup %}
## 异常用途

一般情况下，在Web应用开发中（尤其是Node.js），我们不鼓励在业务代码里面抛出异常，但是一旦开始编写一些抽象的模块，抛出异常就变的比较常用。一般我们用抛出异常的方式处理以下情况：

1. 初始化参数有问题，导致这个模块无法正常工作。比方，一个Config模块，接收ConfigFile路径为参数，但是当这个路径不存在时，整个模块无法工作，这时应该抛出异常。
2. 同步逻辑中遇到模块内部无法处理的结果。不过Javascript中，在会发生这种情况的前提下，都会使用callback去接收结果。

## 编写异常

### Javascript中，抛出的异常可以是各种类型

数字：

    throw 1;

字符串：

    throw 'error';

对象：

    throw new Error();

甚至是日期：

    throw new Date();

### 但是，我们更加推崇使用对象（自定义对象）来描述异常

原因在于，对象可以更加充分的描述异常，让内部和外部的程序都清楚的了解这个异常。

附上代码：

    var util = require('util');

    try {
        throw new typeException(100010, '%s is an unexpected type', 'Number');
    } catch (e) {
        console.log(e);
    }
  
    function baseException(code, message) {
        this.code = code;
        this.message = message;
    }
  
    function typeException(code, message, type) {
        baseException.call(this, code, message);
        this.message = util.format(message, type);
        this.type = type;
    }

上述代码通过function定义了异常对象，并且使用了this对象，实现了异常类型的继承。

这样的做法，使得异常之间建立了关系，使得异常更加容易统一的维护。这样，在一个比较大模块中，使用者和编写者都可以比较直观的知道具体报错的原因，对模块本身的可读性和可维护性是一个很大的提升。
  
以上代码可以参见：[链接](https://github.com/roychen-7/javascript-test/blob/master/throw/index.js)
