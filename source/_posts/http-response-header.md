---
title: 安全相关的HTTP响应头
date: 2020-06-06 17:30:00
tags: HTTP Response Header
categories: TCP/IP
---

## 安全相关的HTTP响应头

### Strict-Transport-Security
简称HSTS，要求浏览器总是通过HTTPS访问该网站。

```javascript
Strict-Transport-Security "max-age=63072000; includeSubdomains;";
// includeSubDomains 可选，用来指定是否作用于子域名。
```

### X-Frame-Options
不允许以 iframe 或 frame 的形式嵌入，是为了减少点击劫持而引入的一个响应头。
```javascript
X-Frame-Options DENT;
```
- DENT：不允许被任何页面嵌套
- SAMEORIGIN：不允许被本域以外的页面嵌入
- ALLOW-FROM uri：不允许被指定的域名以外的页面嵌入


### X-XSS-Protection
用来防范XSS，目前主流浏览器都默认开启XSS保护。
```javascript
X-Xss-Protection "1; mode=block";
```
- 0：禁用XSS保护
- 1：启用XSS保护
- 1; mode=block：启用XSS保护，并在检测到XSS攻击时停止渲染页面


### X-Content-Type-Options
浏览器一般会根据Content-Type响应头来判断请求的资源类型，例如："text/html" 代表html文档，"image/png" 代表png图片， "text/css" 代表CSS样式。
如果有些资源的COntent-Type是错误的或未定义的，部分浏览器会启用MIME-sniffing来猜测该资源的类型，并解析执行。
该响应头可以禁用浏览器的Content-Type猜测行为。
```javascript
X-Content-Type-Options nosniff;
```


### X-Content-Security-Policy
定义页面可以加载哪些资源，减少XSS的发生。

```javascript
Content-Security-Policy: default-src 'self'
```
> https://imququ.com/post/content-security-policy-reference.html

- Aliexpress 使用案例
```javascript
content-security-policy-report-only: default-src * 'unsafe-eval' 'unsafe-inline'  data:;report-uri //pointman.alibaba.com/csp?app=ae_default
strict-transport-security: max-age=31536000 ; includeSubDomains
strict-transport-security: max-age=31536000
x-content-type-options: nosniff
x-frame-options: DENY
x-xss-protection: 1; mode=block
```