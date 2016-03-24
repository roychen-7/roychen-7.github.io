---
layout: post
title: "Firefox下Flash的cookie问题"
description: ""
category: 
tags: ['Firefox', 'Flash', 'Cookie']
---
{% include JB/setup %}
### Firefox下Flash的cookie问题

#### 起因：在使用swfUpload控件的时候，在IE和Chrome下都正常运行，但是Firefox下就会有问题

#### 排查步骤

1. 因为我不了解Flash的机制，虽然猜测可能是各个浏览器下Flash的实现不同造成的问题，但是无从入手，只能换个方向排查。
2. 接下来，我认为可能是服务器的返回的数据确实有问题。因为Firefox下，没办法看到通过Flash来执行的请求（也可能是因为我不知道有没有这方面的插件，毕竟FF不是我的主浏览器），所以我用了Sniffer抓包看了，发现返回码不是200或者是期望中的500，而是302。
3. 因为Server上对应的action中明显没有主动的302动作，后来查看系统log得知是因为请求**没有登录**。
4. 到这里，因为我对Session的工作机制还是比较了解的，所以基本认定是请求的时候这个SessionId有问题。
5. Google（这是废话），我基本上确认这是一个前端方向的常见问题，so google之。
6. 查到的情况基本上是说Firefox的Flash不会带浏览器的cookie上去。ps：不知道这是为什么，如果是为了安全，貌似并没有用 ＝ ＝

#### 解决方法

1. 换一种验证方法，Session的机制归根结底就是拿一串字符串去和服务器上的数据做比较。只是说因为现在的浏览器都支持cookie了，所以这个SessionId一般存放在cookie里面。那我们换的这个方法是把SessionId放到url里面或者是body里面（Get和Post）。
2. 不过这样一来需要对session在做一些特殊的处理，如下：

		destroy_session(); // 结束当前Session
		session_id($GET['sessionid']); // 指定SessionId
		session_start(); // 重新开启Session
		// PHP下无法在Session开启状态下，通过指定SessionId来切换Session

3. 注：因为这个操作是在具体的方法里面做的，那么如果框架里面有权限验证之类的功能，可能就要另想办法了。

##### 越来越觉得Flash真不是一个好东西，虽然可以解决很多问题，不过各个浏览器厂商都不怎么支持的样子。FireFox下有这个问题，Chrome下面则常常崩溃。
