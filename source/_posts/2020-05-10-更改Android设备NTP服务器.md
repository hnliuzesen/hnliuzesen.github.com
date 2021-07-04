---
title: 更改 Android 设备 NTP 服务器
date: 2020-05-10 18:35:44
categories:
- Tip
- Android
tags:
- Android
- NTP
---

有时候用软件在淘宝上抢购或者秒杀，总是提示当前时间与淘宝服务器差多少 ms ，那么只需要更改手机同步时间的服务器为淘宝的服务器，是不是
就能让时差减小呢？大家可以试试效果，其实只需要一行命令即可。

settings put global ntp_server ntp.aliyun.com

<!--more-->

## 使用电脑修改
使用 ADB 命令进行修改，修改后可以验证一下，如果修改成功，去设置里面关闭时间同步再开启时间就会自动同步了，同理，也可以设置其他的时间
同步服务器，服务器列表在最后。

```shell script
❯ adb shell settings put global ntp_server ntp.aliyun.com
❯ adb shell settings get global ntp_server
ntp.aliyun.com
```

## 使用手机修改
使用手机直接修改可能需要 root，这里使用的是 [Termux](https://termux.com/) 。

```shell script
$ su
:/data/data/com.termux/files/home # settings put global ntp_server ntp.aliyun.com
:/data/data/com.termux/files/home # settings get global ntp_server
ntp.aliyun.com
```

## NPT 服务器

### Public NTP Pool Time Servers
<http://support.ntp.org/bin/view/Servers/WebHome#Browsing_the_Lists>

请打开链接查看具体地区的服务器地址

| Area | HostName |
| ---- | ---- |
| Worldwide | [pool.ntp.org](pool.ntp.org) |
| Asia | [asia.pool.ntp.org](asia.pool.ntp.org) |
| Europe | [europe.pool.ntp.org](europe.pool.ntp.org) |
| North America | [north-america.pool.ntp.org](north-america.pool.ntp.org) |
| Oceania | [oceania.pool.ntp.org](oceania.pool.ntp.org) |
| South  America | [south-america.pool.ntp.org](south-america.pool.ntp.org) |

### 阿里云
<https://help.aliyun.com/document_detail/92704.html>

- ntp.aliyun.com
- ntp1.aliyun.com
- ntp2.aliyun.com
- ntp3.aliyun.com
- ntp4.aliyun.com
- ntp5.aliyun.com
- ntp6.aliyun.com
- ntp7.aliyun.com

### Google
<https://developers.google.com/time>

- time1.google.com
- time2.google.com
- time3.google.com
- time4.google.com

### CloudFlare
<https://www.cloudflare.com/time/>

- time.cloudflare.com

### Microsoft
- time.windows.com