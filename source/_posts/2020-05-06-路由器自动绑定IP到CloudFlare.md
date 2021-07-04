---
title: 路由器自动绑定 IP 到 CloudFlare
date: 2020-05-06 17:14:14
comments: true
categories:
- Tip
- WEB
tags:
- domain
- CloudFlare
- router
---

想要不在家的时候远程控制家里的路由器，如果拨号运营商给分配了公网 IP 地址，可以使用路由器中自带的 DDNS，但是路由里面自带的 DDNS 只
有 IPv4 的配置，而且不支持自己申请的独立域名。现在基本上拿不到公网的 IPv4 地址，不过运营商给的 IPv6 地址是可以外网访问的。正好自
己有域名，只要能够动态的在 DNS 中加一条 AAAA 记录就可以实现远程访问了。

<!--more-->

## 设置路由器
这里路由器使用的固件是 [Padavan](https://opt.cn2qq.com/padavan/) ，进入路由器的管理页面，一般是 [192.168.123.1](http://192.168.123.1) ，
左侧 `高级设置` `防火墙` ，在右侧 `通用设置` 中 `从外网访问路由器服务` 下面 `允许从互联网设置 router` 打开、`互联网设置通信端口` 
设置 80，如果想要远程访问 SSH，下面的开关也要打开。

{% asset_img firewall.webp 防火墙配置 %}

然后将 `网络地图` `外部网络状态` 中的 `IPv6 地址: WAN` 拷贝到浏览器，或者发送到手机使用流量访问试试，这里需要注意的是，如果有公网
 IPv4 地址，可以直接使用地址加端口访问，如 192.168.123.1:8080 ，如果是 IPv6 地址，需要使用中括号将 IPv6 地址括住后再加端口号才
能直接访问，如 [240e:33c:2601:7be0:940e:e6cf:c212:80bf]:8080 。

{% asset_img network.webp IP 地址 %}

如果在外网可以通过 IP 访问，就可以进行域名绑定了。

## 命令获取 IP
如果上面 IP 可以通过外网访问，就可以进行绑定了，首先 SSH 连上路由后测试一下命令，可以成功执行了再加入到开机脚本中。

先要获取 IP 地址，SSH 登录后输入 `ifconfig` 查看一下要绑定的 IP 地址是哪个网卡的，在哪一行。如图，我的路由器外网 IP 地址是在 ppp0 
里，这里我要获取的是 IPv6 地址，也就是第三行。

{% asset_img ifconfig.webp ifconfig %}

通过管道命令 `grep -A 2 ppp0` 获取 ppp0 行和后面两行

```shell script
[A3004NS /home/root]# ifconfig | grep -A 2 ppp0
ppp0      Link encap:Point-to-Point Protocol
          inet addr:10.16.102.62  P-t-P:10.16.0.1  Mask:255.255.255.255
          inet6 addr: 240e:33c:2601:a451:815c:89d1:1fda:a1bf/64 Scope:Global
```

再过滤一下 inet6

```shell script
[A3004NS /home/root]# ifconfig | grep -A 2 ppp0 | grep inet6
          inet6 addr: 240e:33c:2601:a451:815c:89d1:1fda:a1bf/64 Scope:Global
```

这一行被空格分为了4份，使用 `awk` 命令获取第三段内容

```shell script
[A3004NS /home/root]# ifconfig | grep -A 2 ppp0 | grep inet6 | awk '{print $3}'
240e:33c:2601:a451:815c:89d1:1fda:a1bf/64
```

最后用 `cut` 命令将结果以 / 分割取前面的地址

```shell script
[A3004NS /home/root]# ifconfig | grep -A 2 ppp0 | grep inet6 | awk '{print $3}' | cut -d '/' -f 0
240e:33c:2601:a451:815c:89d1:1fda:a1bf
```

成功取到地址后将其赋值给变量 `ipv6add`

```shell script
ipv6add=`ifconfig | grep -A 2 ppp0 | grep inet6 | awk '{print $3}' | cut -d '/' -f 1`
```

然后就可以通过 CloudFlare API 发送请求进行绑定了，这里需要注意的是 CF API 不支持免费域名，如 .cf .ga .gq .ml .tk

## 命令绑定 IP 与域名

要使用的命令是 [Update DNS Record](https://api.cloudflare.com/#dns-records-for-a-zone-update-dns-record) ，首先需要获取自己
的 API Key，在 https://dash.cloudflare.com/:profile_identifier/profile/api-tokens 页面，点击 Global API Key 获取，然后可以
用 PostMan 试一下，这里有可以方便直接导入的所有 
[CloudFlare API](https://documenter.getpostman.com/view/10394726/SzYbxHEm?version=latest) 。


如果自己添加请求，请求类型 `PUT`，地址是 https://api.cloudflare.com/client/v4/zones/:zone_identifier/dns_records/:identifier
需要在 Header 中添加 `X-Auth-Key` 就是刚才获取的 Global API Key ，和 `X-Auth-Email` 就是 CloudFlare 账户邮箱地址。在请求的 
Body 中填上。请求 URL 中的 Zone ID 是在 CloudFlare 中点击具体管理的域名，在概述页面右下角，区域 ID 就是 Zone ID ，DNS ID 则需
要使用 [List DNS Records](https://api.cloudflare.com/#dns-records-for-a-zone-list-dns-records) 来查询。查询方式和下面请求
类似，或者直接使用页面上给出的命令进行查询。这个只需要查一次，不会变，查询命令也不需要写入路由器脚本中。

```json
    {
        "type": "AAAA",
        "name": ":your_domain",
        "content": ":IPv6_address",
        "ttl": 120
    }
```

如果是 IPv4 地址，`type` 使用 `"A"` ，`name` 使用要绑定的域名，`content` 内容填 IP 地址，不过如果是 IPv4 可以直接使用 Padavan 
的 DDNS，这里是 IPv4 不是公网 IP ，需要绑定 IPv6 地址，才需要这么麻烦的。

如果请求成功会返回一个 `"success": true` 的 response ，则说明 API 是通的，可以使用命令来请求了。

{% asset_img postman.webp PostMan %}

命令也先在路由上用 SSH 测试一下比较保险，这里需要注意的是，请求体里需要用到前面保存的 shell 变量，不能直接写 `"${ipv6add}"` ，会
被当成字符串处理，需要写 `"'"${ipv6add}"'"`

```shell script
curl -X PUT "https://api.cloudflare.com/client/v4/zones/:zone_identifier/dns_records/:identifier" \
    -H "X-Auth-Email: :your_cloudflare_email" \
    -H "X-Auth-Key: :global_api_key" \
    -H "Content-Type: application/json" \
    --data '{"type":"AAAA","name":":your_domain","content":"'"${ipv6add}"'","ttl":120,"proxied":false}'
```

可以正常返回，并且 CloudFlare 控制台中记录也被更新，通过域名也能访问路由，就可以将命令写入路由器脚本中了。最后完整的命令如下，不要
直接复制，用上面测试成功的命令拼接成自己的命令。

```shell script
ipv6add=`ifconfig | grep -A 2 ppp0 | grep inet6 | awk '{print $3}' | cut -d '/' -f 1`
logger -t "[CloudFlare]" "IPv6 address: ${ipv6add}"
api_result=`curl -X PUT "https://api.cloudflare.com/client/v4/zones/:zone_identifier/dns_records/:identifier" \
    -H "X-Auth-Email: :your_cloudflare_email" \
    -H "X-Auth-Key: :global_api_key" \
    -H "Content-Type: application/json" \
    --data '{"type":"AAAA","name":":your_domain","content":"'"${ipv6add}"'","ttl":120,"proxied":false}'`
```

## 为路由器添加脚本
将上面的命令添加到路由器设置页面的 `高级设置` `自定义设置` 下面的 `脚本` 选项卡中的 `在路由器启动后执行:` 或者 `在 WAN 上行/下行
启动后执行:` 。具体取决于能不能获取到正确的 IP 地址，上面命令中的 logger 会在日志中打印出 IP 地址，如果是可以访问的 IP 地址，则
设置应该成功。

{% asset_img set_script.webp Script %}

如果两个地方都不能够获取到 IP 地址，则可以添加一个定时任务，每小时取执行一次脚本，这样就需要将命令存为 shell script 。如下：

```shell script
#!/bin/bash
### [CloudFlare]
ipv6add=`ifconfig | grep -A 2 ppp0 | grep inet6 | awk '{print $3}' | cut -d '/' -f 1`
logger -t "[CloudFlare]" "IPv6 address: ${ipv6add}"
api_result=`curl -X PUT "https://api.cloudflare.com/client/v4/zones/:zone_identifier/dns_records/:identifier" \
    -H "X-Auth-Email: :your_cloudflare_email" \
    -H "X-Auth-Key: :global_api_key" \
    -H "Content-Type: application/json" \
    --data '{"type":"AAAA","name":":your_domain","content":"'"${ipv6add}"'","ttl":120,"proxied":false}'`
```

然后在路由器的 `高级设置` `系统管理` 下面的 `服务` 选项卡中，有一项 `计划任务 (Crontab)` ，在里面添加一个 Cron 定时任务即可。

{% asset_img set_cron.webp Cron %}

但要注意有的固件脚本可能不会被保存，也就是断电重启后会丢失。