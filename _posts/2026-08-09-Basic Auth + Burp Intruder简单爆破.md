---
layout: default
title: Basic Auth + Burp Intruder简单爆破
date: 2026-08-09
---
## 爆破：Basic Auth + Burp Intruder简单爆破

### 原理（web21）

- 已知用户名 `admin`，但不知道密码；服务端使用 **HTTP Basic Authorization** 做认证。
- `Authorization: Basic xxx` 里的 `xxx` 通常是 `base64(username:password)`。
- 爆破本质：用字典批量替换认证信息，观察响应差异（常见是 **状态码 / 响应长度 / 重定向位置 / 返回内容关键字**）来定位正确密码。

### 详细操作步骤 用Burp Suite

1. 前提：已知用户名是 `admin`。
2. 浏览器设置代理，打开 Burp（Proxy -> Intercept 可开可关）。随便输入密码 `123` 尝试登录，同时用 Burp 抓包。
3. 在请求头中找到 `Authorization`（用于向服务器证明访问权限）。
4. 观察得到的值，若形如 Basic <base64>，将 <base64>`解码。
5. 解码后若得到 `admin:123` 这种格式，则确认是 Basic Auth，可对 `password` 做爆破。
6. 右键该请求：**Send to Intruder**。
7. Intruder -> Positions：选择 **Sniper**（单点爆破，一个位置多 payload）。
8. 将爆破点设置在 `Authorization` 对应的 base64 字符串处（只保留一个位置）。
9. Intruder -> Payloads：加载题目给的密码字典。因为字典只有密码，需要在前面拼上前缀 `admin:`，再整体做 base64，得到符合请求头格式的 payload.
10. Start attack。重点对比：
    - Status（例如 200 vs 401/403）
    - Length（响应长度不同）
    - Location（是否跳转到登录后页面）
    - 响应内容（是否出现 “welcome/flag/后台/退出登录” 等）
11. 找到异常项后：把对应 payload 解码得到明文密码，重新登录并获取 flag。
