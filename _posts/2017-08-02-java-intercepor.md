---
layout: post
title: Spring MVC 处理器拦截器
date: 2017-8-02
categories: blog
tags: [java,拦截器]
description: 处理器拦截器的实际应用
---

### 常见应用场景
* 日记记录：记录请求信息的日志、可以进行信息统计，PV等  
* 权限检查：例如登录检测，权限控制等   
* 性能监控：得到处理器的处理时间  
* 通用行为：本文就是使用拦截器来保证app接口的安全调用  
>拦截器是AOP的一种实现

#### 区别（过滤器）：  
* 拦截器是基于Java的反射机制，而过滤器是基于函数回调；  
* 拦截器不依赖于servlet容器，而过滤器依赖于servlet容器；  
* 拦截器只对action请求起作用，而过滤器则可以对几乎所有的请求起作用；    
* 拦截器可以访问action上下文、值栈里的对象，过滤器不可以；  
* 在action生命周期中，拦截器可以多次被调用，过滤器只能在容器初始化时被调用一次。    

#### 本文应用场景  
APP 接口签名验证：主要用来验证请求来源是否合法；  

[API接口签名验证](http://www.jianshu.com/p/d47da77b6419)



##参考资料
[参考一](http://jinnianshilongnian.iteye.com/blog/1670856)
