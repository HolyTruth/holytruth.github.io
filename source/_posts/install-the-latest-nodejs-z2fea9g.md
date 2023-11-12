---
title: 安装最新nodejs
date: '2023-11-11 13:08:28'
updated: '2023-11-12 09:57:33'
permalink: /post/install-the-latest-nodejs-z2fea9g.html
comments: true
toc: true
---

# 安装最新nodejs

在 ubuntu22.04 上使用`apt install nodejs`​安装时，只能安装12.21版本的nodejs，安装最新的nodejs需要另寻他法。

遵循：[https://deb.nodesource.com/](https://deb.nodesource.com/)

```sh
~$ sudo apt-get update && sudo apt-get install -y ca-certificates curl gnupg
~$ curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
~$ NODE_MAJOR=20
~$ echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list
~$ sudo apt-get update && sudo apt-get install nodejs -y
```

‍
