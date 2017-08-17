---
layout: post
title: Spring MVC 处理器拦截器
date: 2017-8-02
categories: blog
tags: [java,拦截器,接口验证]
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
主要有这几个问题需要考虑：  
1. 请求参数是否被篡改；  
2. 请求来源是否合法；  
3. 请求是否具有唯一性；  
本文采用了主流的通信安全解决方案--参数签名方式  

#### 签名  
生成签名的步骤：        
* 约定服务双方加密密钥appsecret  
* 将所有请求参数按字母顺序排序，并拼接成字符串A  
* 将字符串A与appsecret拼接并用md5加密  
>项目中的签名    
将参数放入签名中可以有效避免参数被篡改    
// 1.获取签名  
    String pdAppSign = request.getHeader("PdAppSign");  
    // 1.1 诺签名为null 则为非法请求  
    if (pdAppSign == null || pdAppSign.equals("")) {  
      return false;  
    }  
    // 2.判断请求时间有效性  
    String reqdata = request.getParameter("reqdata");  
    JSONObject req = JSONObject.parseObject(reqdata);  
    long nowTime = System.currentTimeMillis();  
    long appTime = req.getLong("t");  
    System.out.println("hahahahahahaah"+Math.abs(nowTime - appTime));  
    if (Math.abs(nowTime - appTime) > 10000) {  
      return false;  
    }  
    // 3.验证签名  
    Map<String, String> valueMap = new TreeMap<String, String>();  
    for (String key : req.keySet()) {  
      String keyStr = (String) key;  
      Object keyValue = req.get(keyStr);  
      valueMap.put(keyStr, (String) keyValue);  
    }  
    StringBuffer sb = new StringBuffer();  
    // 加密签名  
    for (Map.Entry<String, String> entry : valueMap.entrySet()) {  
      entry.getValue());    
      sb.append(entry.getValue());  
    }  
    String appSign = StringUtil.md5(sb.toString() + Config.APPKEY);
    if (!pdAppSign.equals(appSign)) {  
      return false;  
    }  


在请求中加入时间戳可以保证请求的唯一性，时间间隔越长，链接的有效期就越长，唯一性越差。  
[API接口签名验证](http://www.jianshu.com/p/d47da77b6419)

应用请求将签名放在请求头中，服务器端响应请求时先验证请求的合法性。  


#### 拦截器配置  
* xml 配置
> <mvc:interceptors>  
		<bean class="com.bluemobi.log.interceptor.ControlInterceptor" />    
		<mvc:interceptor>  
			<mvc:mapping path="/hhapp/login" />  
			<mvc:mapping path="/productForApp/test" />    
			<bean class="com.bluemobi.controller.app.AppInterceptor" />    
		</mvc:interceptor>    
</mvc:interceptors>  

* 拦截器  
>  public class AppInterceptor implements HandlerInterceptor {  
	public void afterCompletion(HttpServletRequest arg0,
			HttpServletResponse arg1, Object arg2, Exception arg3)
			throws Exception {  
	}  
	public void postHandle(HttpServletRequest arg0, HttpServletResponse arg1,
			Object arg2, ModelAndView arg3) throws Exception {  
	}  
	public boolean preHandle(HttpServletRequest request,HttpServletResponse response, Object handle) throws Exception {  
}  





##参考资料
[参考一 拦截器](http://jinnianshilongnian.iteye.com/blog/1670856)
