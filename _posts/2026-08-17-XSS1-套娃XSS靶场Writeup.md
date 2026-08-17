---
layout: default
title: "XSS1 - 套娃XSS靶场Writeup"
date: 2026-08-16
categories: [Writeup]
tags: [XSS, Web安全]
---

# XSS1

## 一、概述

本次挑战为渐进式 XSS 靶场，共 6 关。

考点从最基础的反射型 XSS，到 JS 上下文逃逸、DOM XSS、javascript: 伪协议利用，最后到 AngularJS 客户端模板注入（CSTI）沙箱逃逸，全面覆盖。

## 二、逐关解题思路

题目已经给出了提示，过关目标为利用 XSS 漏洞在页面执行 `alert()` 函数，后续查看源代码注意这个。

### Level 1：反射型 XSS —— 无过滤的直接注入

**分析：**

页面通过 URL 参数 `username` 接收输入，并直接回显到页面上。没有任何过滤、转义、编码。

所以我们直接构造 payload：
?username=alert(1)

回车后通关。

**原理：**

服务端把用户输入原封不动地拼接到 HTML 响应中，浏览器解析时把 `<script>` 当作真正的脚本标签执行。这是最经典的反射型 XSS。

### Level 2：遭遇转义 —— 从 HTML 上下文切换到 JS 上下文

**分析：**

同样有 `username` 参数，但这次输入 `<img src=x onerror=alert(1)>` 后，页面显示的是乱码。

**失败原因分析：**

查看网页源码可以发现关键代码：

`escape()` 函数把 `<`、`>`、`"` 全部转成了 `%xx` 编码。浏览器看到 `%3Cscript%3E` 只会当纯文本显示，HTML 标签注入这条路被堵死了。

但是注意看源代码，我们的输入被放在了一对 `' '` 里面，这是天无绝人之路！

众所周知，`escape()` 会转义 `< > "`，但不会转义单引号 `'`。既然不能跳出 HTML，那就在 JS 内部逃逸。

构造 payload：
?username=';alert(1)//

注入后的变化是这样的：

// 原始
var username = 'xss';

// 注入后
var username = '';       // ① 单引号闭合，变量赋值为空
alert(1);                // ② 恶意代码，正常执行
//';                     // ③ // 注释掉残留的引号，防止语法报错
回车之后就可以通过这一关了。

### Level 3：DOM XSS —— 去掉 escape 后

**场景分析：**

查看源码。

注意这一段：

document.getElementById('ccc').innerHTML = "Welcome " + username;

我们看出 escape() 被去掉了！输入不再被编码，< > 会原样进入 DOM。

所以直接复现第一关的思路，构造 payload：

?username=<script>alert(1)</script>

但是题目过滤了 script 关键字……简直离谱。

所以我们换一种方式，构造：

?username=<img src=x onerror=alert(1)>

回车后通关。

### Level 4：javascript: 伪协议 —— 定向执行代码

**场景分析：**

查看源代码，发现页面跳转的控制代码。

查阅资料后我们可以发现这段代码里面的逻辑：

escape() 只作用于页面显示文本，而真正执行跳转的 location.href = jumpUrl 用的是原始参数值。

而 显示安全 ≠ 执行安全。

所以可以利用这一点构造 payload：

?jumpUrl=javascript:alert(1)

等倒计时结束就通关啦！

### Level 5：表单 action 劫持 —— 伪协议的变体

**场景分析：**

页面是表单提交页，所以我们查看源码。

阅读代码后我们发现：

表单的 action 属性完全由 URL 参数 action 控制，且存在 autosubmit 自动提交开关。

这似曾相识的感觉，这一点与 Level 4 同源，在表单提交时会被浏览器当作 JS 执行。

区别只是触发载体从 location.href 换成了 form.action。

所以我们就可以仿照这构造 payload：

?autosubmit=1&action=javascript:alert(1)

回车通关！

### Level 6：AngularJS 沙箱逃逸 —— 客户端模板注入（CSTI）
**分析：**

页面使用 AngularJS 1.4.6，输入 {{1+1}} 输出 2，确认存在模板注入。

接下来是错误示范：

第一次尝试：

?username={{constructor.constructor('alert(1)')()}}

页面无反应。原因：AngularJS 1.3+ 的沙箱机制禁止表达式直接访问 constructor、prototype 等敏感属性。

第二次尝试：

?username={{constructor.constructor('alert(1)')}}

漏了最后的调用括号 ()，Function('alert(1)') 只是创建了函数，没有执行。

最终 Payload（沙箱逃逸）：

{{'a'.constructor.prototype.charAt=[].join;  $ eval('x=1} } };alert(1)//');}}

通关得到 flag！
