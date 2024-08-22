---
author: zjmantou
title: mac终端代理
time: 2024-08-22 周四
tags:
  - 笔记
  - shell
---
# 参考链接 

[终端代理](https://www.hangge.com/blog/cache/detail_3138.html) 

```shell
//环境变量加入，1080替换成vpn的端口
alias proxy='export all_proxy=socks5://127.0.0.1:1080'

alias unproxy='unset all_proxy'
```

执行proxy，终端走代理，执行unproxy 关闭终端代理 

验证：执行`curl ipinfo.io` 

```shell
{

  "ip": "206.168.191.3",

  "city": "Phoenix",

  "region": "Arizona",

  "country": "US",

  "loc": "33.4484,-112.0740",

  "org": "AS14315 1GSERVERS, LLC",

  "postal": "85001",

  "timezone": "America/Phoenix",

  "readme": "https://ipinfo.io/missingauth"

}**%**                                                                              mantou@
```

