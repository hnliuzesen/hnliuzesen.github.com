---
title: Redis、Valkey 性能测试
date: 2024-11-29 17:16:24
categories:
  - Develop
  - Database
tags:
  - Redis
  - Valkey
---

最近看到新闻，[Redis 在试图掌管相关的开源仓库](https://www.infoq.cn/article/IEJLgTB9AayJAhOw9dzr?utm_campaign=geek_search&utm_content=geek_search&utm_medium=geek_search&utm_source=geek_search&utm_term=geek_search)，有很多人都提到了转投
[Valkey](https://valkey.io/) 阵营，查询了一下，网上说 Valkey 有很多优化，特别是多线程方面的，但是搜不到具体的性能测试对比，于是自建了 
Redis 和 Valkey 使用 [redis-Benchmark](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/benchmarks/) 
测试了一下。

<!--more-->

## 测试环境
使用的机器是 Xeon ® Platinum 8269CY 8 核 16G 内存。  
vm.overcommit_memory 设置为 1 总是允许过度分配内存。  
关闭了透明大页。  
开启了 14G swap。  

## 测试工具
使用 Redis-Benchmark，使用 Unix socket 连接，1024 个连接（默认 50），每个指令测试 1024000 次（默认 100000）。  
```shell
# 测试 Redis
redis-benchmark -a [password] -s /run/redis/redis.sock -c 1024 -n 1024000 --csv | tee redis_1024_sock.csv
# 测试 Valkey
redis-benchmark -a [password] -s /run/valkey.sock -c 1024 -n 1024000 --csv | tee valkey_1024_sock.csv
```

## 测试结果
### 资源占用
Redis 测试时监控
{% asset_img redis_1024_sock.webp "Redis 测试时监控" %}


Valkey 测试时监控  
{% asset_img valkey_1024_sock.webp "Valkey 测试时监控" %}

### 性能对比
<table style="width: 100%; overflow-x: auto; display: block; white-space: nowrap; border: 1px solid black; border-collapse: collapse;">
    <tr>
        <td style="border: 1px solid black;"></td>
        <td colspan="3" style="border: 1px solid black; text-align: center;">rps</td>
        <td colspan="3" style="border: 1px solid black; text-align: center;">avg_latency_ms</td>
        <td colspan="3" style="border: 1px solid black; text-align: center;">min_latency_ms</td>
        <td colspan="3" style="border: 1px solid black; text-align: center;">p50_latency_ms</td>
        <td colspan="3" style="border: 1px solid black; text-align: center;">p95_latency_ms</td>
        <td colspan="3" style="border: 1px solid black; text-align: center;">p99_latency_ms</td>
        <td colspan="3" style="border: 1px solid black; text-align: center;">max_latency_ms</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;"></td>
        <td style="border: 1px solid black; text-align: center;">Redis</td>
        <td style="border: 1px solid black; text-align: center;">ValKey</td>
        <td style="border: 1px solid black; text-align: center;">对比</td>
        <td style="border: 1px solid black; text-align: center;">Redis</td>
        <td style="border: 1px solid black; text-align: center;">ValKey</td>
        <td style="border: 1px solid black; text-align: center;">对比</td>
        <td style="border: 1px solid black; text-align: center;">Redis</td>
        <td style="border: 1px solid black; text-align: center;">ValKey</td>
        <td style="border: 1px solid black; text-align: center;">对比</td>
        <td style="border: 1px solid black; text-align: center;">Redis</td>
        <td style="border: 1px solid black; text-align: center;">ValKey</td>
        <td style="border: 1px solid black; text-align: center;">对比</td>
        <td style="border: 1px solid black; text-align: center;">Redis</td>
        <td style="border: 1px solid black; text-align: center;">ValKey</td>
        <td style="border: 1px solid black; text-align: center;">对比</td>
        <td style="border: 1px solid black; text-align: center;">Redis</td>
        <td style="border: 1px solid black; text-align: center;">ValKey</td>
        <td style="border: 1px solid black; text-align: center;">对比</td>
        <td style="border: 1px solid black; text-align: center;">Redis</td>
        <td style="border: 1px solid black; text-align: center;">ValKey</td>
        <td style="border: 1px solid black; text-align: center;">对比</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">PING_INLINE</td>
        <td style="border: 1px solid black; text-align: right;">101026.05</td>
        <td style="border: 1px solid black; text-align: right;">105567.02</td>
        <td style="border: 1px solid black; text-align: right;">4.49%</td>
        <td style="border: 1px solid black; text-align: right;">5.079</td>
        <td style="border: 1px solid black; text-align: right;">4.864</td>
        <td style="border: 1px solid black; text-align: right;">-4.23%</td>
        <td style="border: 1px solid black; text-align: right;">2.72</td>
        <td style="border: 1px solid black; text-align: right;">2.072</td>
        <td style="border: 1px solid black; text-align: right;">-23.82%</td>
        <td style="border: 1px solid black; text-align: right;">5.055</td>
        <td style="border: 1px solid black; text-align: right;">4.839</td>
        <td style="border: 1px solid black; text-align: right;">-4.27%</td>
        <td style="border: 1px solid black; text-align: right;">5.391</td>
        <td style="border: 1px solid black; text-align: right;">5.119</td>
        <td style="border: 1px solid black; text-align: right;">-5.05%</td>
        <td style="border: 1px solid black; text-align: right;">5.599</td>
        <td style="border: 1px solid black; text-align: right;">5.319</td>
        <td style="border: 1px solid black; text-align: right;">-5.00%</td>
        <td style="border: 1px solid black; text-align: right;">25.807</td>
        <td style="border: 1px solid black; text-align: right;">33.183</td>
        <td style="border: 1px solid black; text-align: right;">28.58%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">PING_MBULK</td>
        <td style="border: 1px solid black; text-align: right;">101637.72</td>
        <td style="border: 1px solid black; text-align: right;">105894.52</td>
        <td style="border: 1px solid black; text-align: right;">4.19%</td>
        <td style="border: 1px solid black; text-align: right;">5.043</td>
        <td style="border: 1px solid black; text-align: right;">4.844</td>
        <td style="border: 1px solid black; text-align: right;">-3.95%</td>
        <td style="border: 1px solid black; text-align: right;">2.272</td>
        <td style="border: 1px solid black; text-align: right;">2.224</td>
        <td style="border: 1px solid black; text-align: right;">-2.11%</td>
        <td style="border: 1px solid black; text-align: right;">5.031</td>
        <td style="border: 1px solid black; text-align: right;">4.823</td>
        <td style="border: 1px solid black; text-align: right;">-4.13%</td>
        <td style="border: 1px solid black; text-align: right;">5.367</td>
        <td style="border: 1px solid black; text-align: right;">5.127</td>
        <td style="border: 1px solid black; text-align: right;">-4.47%</td>
        <td style="border: 1px solid black; text-align: right;">5.583</td>
        <td style="border: 1px solid black; text-align: right;">5.335</td>
        <td style="border: 1px solid black; text-align: right;">-4.44%</td>
        <td style="border: 1px solid black; text-align: right;">11.175</td>
        <td style="border: 1px solid black; text-align: right;">21.439</td>
        <td style="border: 1px solid black; text-align: right;">91.85%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">SET</td>
        <td style="border: 1px solid black; text-align: right;">100747.74</td>
        <td style="border: 1px solid black; text-align: right;">103257.04</td>
        <td style="border: 1px solid black; text-align: right;">2.49%</td>
        <td style="border: 1px solid black; text-align: right;">5.089</td>
        <td style="border: 1px solid black; text-align: right;">4.972</td>
        <td style="border: 1px solid black; text-align: right;">-2.30%</td>
        <td style="border: 1px solid black; text-align: right;">3</td>
        <td style="border: 1px solid black; text-align: right;">2.12</td>
        <td style="border: 1px solid black; text-align: right;">-29.33%</td>
        <td style="border: 1px solid black; text-align: right;">5.079</td>
        <td style="border: 1px solid black; text-align: right;">4.935</td>
        <td style="border: 1px solid black; text-align: right;">-2.84%</td>
        <td style="border: 1px solid black; text-align: right;">5.415</td>
        <td style="border: 1px solid black; text-align: right;">5.263</td>
        <td style="border: 1px solid black; text-align: right;">-2.81%</td>
        <td style="border: 1px solid black; text-align: right;">5.591</td>
        <td style="border: 1px solid black; text-align: right;">5.455</td>
        <td style="border: 1px solid black; text-align: right;">-2.43%</td>
        <td style="border: 1px solid black; text-align: right;">12.631</td>
        <td style="border: 1px solid black; text-align: right;">25.727</td>
        <td style="border: 1px solid black; text-align: right;">103.68%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">GET</td>
        <td style="border: 1px solid black; text-align: right;">101809.51</td>
        <td style="border: 1px solid black; text-align: right;">103906.65</td>
        <td style="border: 1px solid black; text-align: right;">2.06%</td>
        <td style="border: 1px solid black; text-align: right;">5.035</td>
        <td style="border: 1px solid black; text-align: right;">4.939</td>
        <td style="border: 1px solid black; text-align: right;">-1.91%</td>
        <td style="border: 1px solid black; text-align: right;">1.96</td>
        <td style="border: 1px solid black; text-align: right;">2.848</td>
        <td style="border: 1px solid black; text-align: right;">45.31%</td>
        <td style="border: 1px solid black; text-align: right;">5.023</td>
        <td style="border: 1px solid black; text-align: right;">4.903</td>
        <td style="border: 1px solid black; text-align: right;">-2.39%</td>
        <td style="border: 1px solid black; text-align: right;">5.375</td>
        <td style="border: 1px solid black; text-align: right;">5.215</td>
        <td style="border: 1px solid black; text-align: right;">-2.98%</td>
        <td style="border: 1px solid black; text-align: right;">5.599</td>
        <td style="border: 1px solid black; text-align: right;">5.375</td>
        <td style="border: 1px solid black; text-align: right;">-4.00%</td>
        <td style="border: 1px solid black; text-align: right;">11.783</td>
        <td style="border: 1px solid black; text-align: right;">25.503</td>
        <td style="border: 1px solid black; text-align: right;">116.44%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">INCR</td>
        <td style="border: 1px solid black; text-align: right;">102073.37</td>
        <td style="border: 1px solid black; text-align: right;">103517.99</td>
        <td style="border: 1px solid black; text-align: right;">1.42%</td>
        <td style="border: 1px solid black; text-align: right;">5.022</td>
        <td style="border: 1px solid black; text-align: right;">4.958</td>
        <td style="border: 1px solid black; text-align: right;">-1.27%</td>
        <td style="border: 1px solid black; text-align: right;">2.072</td>
        <td style="border: 1px solid black; text-align: right;">2.64</td>
        <td style="border: 1px solid black; text-align: right;">27.41%</td>
        <td style="border: 1px solid black; text-align: right;">5.015</td>
        <td style="border: 1px solid black; text-align: right;">4.935</td>
        <td style="border: 1px solid black; text-align: right;">-1.60%</td>
        <td style="border: 1px solid black; text-align: right;">5.343</td>
        <td style="border: 1px solid black; text-align: right;">5.231</td>
        <td style="border: 1px solid black; text-align: right;">-2.10%</td>
        <td style="border: 1px solid black; text-align: right;">5.551</td>
        <td style="border: 1px solid black; text-align: right;">5.463</td>
        <td style="border: 1px solid black; text-align: right;">-1.59%</td>
        <td style="border: 1px solid black; text-align: right;">12.135</td>
        <td style="border: 1px solid black; text-align: right;">21.839</td>
        <td style="border: 1px solid black; text-align: right;">79.97%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">LPUSH</td>
        <td style="border: 1px solid black; text-align: right;">100688.3</td>
        <td style="border: 1px solid black; text-align: right;">103612.26</td>
        <td style="border: 1px solid black; text-align: right;">2.90%</td>
        <td style="border: 1px solid black; text-align: right;">5.092</td>
        <td style="border: 1px solid black; text-align: right;">4.953</td>
        <td style="border: 1px solid black; text-align: right;">-2.73%</td>
        <td style="border: 1px solid black; text-align: right;">2.8</td>
        <td style="border: 1px solid black; text-align: right;">2.656</td>
        <td style="border: 1px solid black; text-align: right;">-5.14%</td>
        <td style="border: 1px solid black; text-align: right;">5.079</td>
        <td style="border: 1px solid black; text-align: right;">4.919</td>
        <td style="border: 1px solid black; text-align: right;">-3.15%</td>
        <td style="border: 1px solid black; text-align: right;">5.415</td>
        <td style="border: 1px solid black; text-align: right;">5.223</td>
        <td style="border: 1px solid black; text-align: right;">-3.55%</td>
        <td style="border: 1px solid black; text-align: right;">5.599</td>
        <td style="border: 1px solid black; text-align: right;">5.407</td>
        <td style="border: 1px solid black; text-align: right;">-3.43%</td>
        <td style="border: 1px solid black; text-align: right;">12.855</td>
        <td style="border: 1px solid black; text-align: right;">25.999</td>
        <td style="border: 1px solid black; text-align: right;">102.25%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">RPUSH</td>
        <td style="border: 1px solid black; text-align: right;">102708.12</td>
        <td style="border: 1px solid black; text-align: right;">104160.3</td>
        <td style="border: 1px solid black; text-align: right;">1.41%</td>
        <td style="border: 1px solid black; text-align: right;">4.988</td>
        <td style="border: 1px solid black; text-align: right;">4.927</td>
        <td style="border: 1px solid black; text-align: right;">-1.22%</td>
        <td style="border: 1px solid black; text-align: right;">2.08</td>
        <td style="border: 1px solid black; text-align: right;">2.44</td>
        <td style="border: 1px solid black; text-align: right;">17.31%</td>
        <td style="border: 1px solid black; text-align: right;">4.975</td>
        <td style="border: 1px solid black; text-align: right;">4.911</td>
        <td style="border: 1px solid black; text-align: right;">-1.29%</td>
        <td style="border: 1px solid black; text-align: right;">5.319</td>
        <td style="border: 1px solid black; text-align: right;">5.207</td>
        <td style="border: 1px solid black; text-align: right;">-2.11%</td>
        <td style="border: 1px solid black; text-align: right;">5.527</td>
        <td style="border: 1px solid black; text-align: right;">5.407</td>
        <td style="border: 1px solid black; text-align: right;">-2.17%</td>
        <td style="border: 1px solid black; text-align: right;">12.039</td>
        <td style="border: 1px solid black; text-align: right;">22.703</td>
        <td style="border: 1px solid black; text-align: right;">88.58%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">LPOP</td>
        <td style="border: 1px solid black; text-align: right;">101890.55</td>
        <td style="border: 1px solid black; text-align: right;">104319.48</td>
        <td style="border: 1px solid black; text-align: right;">2.38%</td>
        <td style="border: 1px solid black; text-align: right;">5.035</td>
        <td style="border: 1px solid black; text-align: right;">4.92</td>
        <td style="border: 1px solid black; text-align: right;">-2.28%</td>
        <td style="border: 1px solid black; text-align: right;">2.848</td>
        <td style="border: 1px solid black; text-align: right;">2.816</td>
        <td style="border: 1px solid black; text-align: right;">-1.12%</td>
        <td style="border: 1px solid black; text-align: right;">5.023</td>
        <td style="border: 1px solid black; text-align: right;">4.895</td>
        <td style="border: 1px solid black; text-align: right;">-2.55%</td>
        <td style="border: 1px solid black; text-align: right;">5.319</td>
        <td style="border: 1px solid black; text-align: right;">5.191</td>
        <td style="border: 1px solid black; text-align: right;">-2.41%</td>
        <td style="border: 1px solid black; text-align: right;">5.543</td>
        <td style="border: 1px solid black; text-align: right;">5.415</td>
        <td style="border: 1px solid black; text-align: right;">-2.31%</td>
        <td style="border: 1px solid black; text-align: right;">13.287</td>
        <td style="border: 1px solid black; text-align: right;">24.063</td>
        <td style="border: 1px solid black; text-align: right;">81.10%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">RPOP</td>
        <td style="border: 1px solid black; text-align: right;">101105.84</td>
        <td style="border: 1px solid black; text-align: right;">104724.89</td>
        <td style="border: 1px solid black; text-align: right;">3.58%</td>
        <td style="border: 1px solid black; text-align: right;">5.072</td>
        <td style="border: 1px solid black; text-align: right;">4.901</td>
        <td style="border: 1px solid black; text-align: right;">-3.37%</td>
        <td style="border: 1px solid black; text-align: right;">2.688</td>
        <td style="border: 1px solid black; text-align: right;">2.784</td>
        <td style="border: 1px solid black; text-align: right;">3.57%</td>
        <td style="border: 1px solid black; text-align: right;">5.055</td>
        <td style="border: 1px solid black; text-align: right;">4.871</td>
        <td style="border: 1px solid black; text-align: right;">-3.64%</td>
        <td style="border: 1px solid black; text-align: right;">5.335</td>
        <td style="border: 1px solid black; text-align: right;">5.151</td>
        <td style="border: 1px solid black; text-align: right;">-3.45%</td>
        <td style="border: 1px solid black; text-align: right;">5.519</td>
        <td style="border: 1px solid black; text-align: right;">5.311</td>
        <td style="border: 1px solid black; text-align: right;">-3.77%</td>
        <td style="border: 1px solid black; text-align: right;">14.807</td>
        <td style="border: 1px solid black; text-align: right;">26.495</td>
        <td style="border: 1px solid black; text-align: right;">78.94%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">SADD</td>
        <td style="border: 1px solid black; text-align: right;">101860.13</td>
        <td style="border: 1px solid black; text-align: right;">105556.12</td>
        <td style="border: 1px solid black; text-align: right;">3.63%</td>
        <td style="border: 1px solid black; text-align: right;">5.032</td>
        <td style="border: 1px solid black; text-align: right;">4.862</td>
        <td style="border: 1px solid black; text-align: right;">-3.38%</td>
        <td style="border: 1px solid black; text-align: right;">2.408</td>
        <td style="border: 1px solid black; text-align: right;">2.736</td>
        <td style="border: 1px solid black; text-align: right;">13.62%</td>
        <td style="border: 1px solid black; text-align: right;">5.023</td>
        <td style="border: 1px solid black; text-align: right;">4.831</td>
        <td style="border: 1px solid black; text-align: right;">-3.82%</td>
        <td style="border: 1px solid black; text-align: right;">5.335</td>
        <td style="border: 1px solid black; text-align: right;">5.111</td>
        <td style="border: 1px solid black; text-align: right;">-4.20%</td>
        <td style="border: 1px solid black; text-align: right;">5.527</td>
        <td style="border: 1px solid black; text-align: right;">5.399</td>
        <td style="border: 1px solid black; text-align: right;">-2.32%</td>
        <td style="border: 1px solid black; text-align: right;">11.895</td>
        <td style="border: 1px solid black; text-align: right;">24.687</td>
        <td style="border: 1px solid black; text-align: right;">107.54%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">HSET</td>
        <td style="border: 1px solid black; text-align: right;">102297.7</td>
        <td style="border: 1px solid black; text-align: right;">105839.79</td>
        <td style="border: 1px solid black; text-align: right;">3.46%</td>
        <td style="border: 1px solid black; text-align: right;">5.012</td>
        <td style="border: 1px solid black; text-align: right;">4.851</td>
        <td style="border: 1px solid black; text-align: right;">-3.21%</td>
        <td style="border: 1px solid black; text-align: right;">2.984</td>
        <td style="border: 1px solid black; text-align: right;">1.984</td>
        <td style="border: 1px solid black; text-align: right;">-33.51%</td>
        <td style="border: 1px solid black; text-align: right;">5.007</td>
        <td style="border: 1px solid black; text-align: right;">4.823</td>
        <td style="border: 1px solid black; text-align: right;">-3.67%</td>
        <td style="border: 1px solid black; text-align: right;">5.295</td>
        <td style="border: 1px solid black; text-align: right;">5.119</td>
        <td style="border: 1px solid black; text-align: right;">-3.32%</td>
        <td style="border: 1px solid black; text-align: right;">5.495</td>
        <td style="border: 1px solid black; text-align: right;">5.351</td>
        <td style="border: 1px solid black; text-align: right;">-2.62%</td>
        <td style="border: 1px solid black; text-align: right;">13.111</td>
        <td style="border: 1px solid black; text-align: right;">26.959</td>
        <td style="border: 1px solid black; text-align: right;">105.62%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">SPOP</td>
        <td style="border: 1px solid black; text-align: right;">102862.88</td>
        <td style="border: 1px solid black; text-align: right;">105414.87</td>
        <td style="border: 1px solid black; text-align: right;">2.48%</td>
        <td style="border: 1px solid black; text-align: right;">4.984</td>
        <td style="border: 1px solid black; text-align: right;">4.868</td>
        <td style="border: 1px solid black; text-align: right;">-2.33%</td>
        <td style="border: 1px solid black; text-align: right;">2.32</td>
        <td style="border: 1px solid black; text-align: right;">2.568</td>
        <td style="border: 1px solid black; text-align: right;">10.69%</td>
        <td style="border: 1px solid black; text-align: right;">4.975</td>
        <td style="border: 1px solid black; text-align: right;">4.839</td>
        <td style="border: 1px solid black; text-align: right;">-2.73%</td>
        <td style="border: 1px solid black; text-align: right;">5.271</td>
        <td style="border: 1px solid black; text-align: right;">5.127</td>
        <td style="border: 1px solid black; text-align: right;">-2.73%</td>
        <td style="border: 1px solid black; text-align: right;">5.447</td>
        <td style="border: 1px solid black; text-align: right;">5.351</td>
        <td style="border: 1px solid black; text-align: right;">-1.76%</td>
        <td style="border: 1px solid black; text-align: right;">13.495</td>
        <td style="border: 1px solid black; text-align: right;">23.919</td>
        <td style="border: 1px solid black; text-align: right;">77.24%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">ZADD</td>
        <td style="border: 1px solid black; text-align: right;">101728.59</td>
        <td style="border: 1px solid black; text-align: right;">105556.12</td>
        <td style="border: 1px solid black; text-align: right;">3.76%</td>
        <td style="border: 1px solid black; text-align: right;">5.04</td>
        <td style="border: 1px solid black; text-align: right;">4.863</td>
        <td style="border: 1px solid black; text-align: right;">-3.51%</td>
        <td style="border: 1px solid black; text-align: right;">3.088</td>
        <td style="border: 1px solid black; text-align: right;">2.768</td>
        <td style="border: 1px solid black; text-align: right;">-10.36%</td>
        <td style="border: 1px solid black; text-align: right;">5.023</td>
        <td style="border: 1px solid black; text-align: right;">4.839</td>
        <td style="border: 1px solid black; text-align: right;">-3.66%</td>
        <td style="border: 1px solid black; text-align: right;">5.375</td>
        <td style="border: 1px solid black; text-align: right;">5.151</td>
        <td style="border: 1px solid black; text-align: right;">-4.17%</td>
        <td style="border: 1px solid black; text-align: right;">5.639</td>
        <td style="border: 1px solid black; text-align: right;">5.399</td>
        <td style="border: 1px solid black; text-align: right;">-4.26%</td>
        <td style="border: 1px solid black; text-align: right;">13.143</td>
        <td style="border: 1px solid black; text-align: right;">20.751</td>
        <td style="border: 1px solid black; text-align: right;">57.89%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">ZPOPMIN</td>
        <td style="border: 1px solid black; text-align: right;">101335.98</td>
        <td style="border: 1px solid black; text-align: right;">105469.16</td>
        <td style="border: 1px solid black; text-align: right;">4.08%</td>
        <td style="border: 1px solid black; text-align: right;">5.059</td>
        <td style="border: 1px solid black; text-align: right;">4.867</td>
        <td style="border: 1px solid black; text-align: right;">-3.80%</td>
        <td style="border: 1px solid black; text-align: right;">2.944</td>
        <td style="border: 1px solid black; text-align: right;">2.416</td>
        <td style="border: 1px solid black; text-align: right;">-17.93%</td>
        <td style="border: 1px solid black; text-align: right;">5.055</td>
        <td style="border: 1px solid black; text-align: right;">4.839</td>
        <td style="border: 1px solid black; text-align: right;">-4.27%</td>
        <td style="border: 1px solid black; text-align: right;">5.407</td>
        <td style="border: 1px solid black; text-align: right;">5.111</td>
        <td style="border: 1px solid black; text-align: right;">-5.47%</td>
        <td style="border: 1px solid black; text-align: right;">5.663</td>
        <td style="border: 1px solid black; text-align: right;">5.327</td>
        <td style="border: 1px solid black; text-align: right;">-5.93%</td>
        <td style="border: 1px solid black; text-align: right;">11.199</td>
        <td style="border: 1px solid black; text-align: right;">25.615</td>
        <td style="border: 1px solid black; text-align: right;">128.73%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">LPUSH (needed to benchmark LRANGE)</td>
        <td style="border: 1px solid black; text-align: right;">100777.48</td>
        <td style="border: 1px solid black; text-align: right;">104139.12</td>
        <td style="border: 1px solid black; text-align: right;">3.34%</td>
        <td style="border: 1px solid black; text-align: right;">5.088</td>
        <td style="border: 1px solid black; text-align: right;">4.928</td>
        <td style="border: 1px solid black; text-align: right;">-3.14%</td>
        <td style="border: 1px solid black; text-align: right;">2.528</td>
        <td style="border: 1px solid black; text-align: right;">2.64</td>
        <td style="border: 1px solid black; text-align: right;">4.43%</td>
        <td style="border: 1px solid black; text-align: right;">5.071</td>
        <td style="border: 1px solid black; text-align: right;">4.903</td>
        <td style="border: 1px solid black; text-align: right;">-3.31%</td>
        <td style="border: 1px solid black; text-align: right;">5.399</td>
        <td style="border: 1px solid black; text-align: right;">5.215</td>
        <td style="border: 1px solid black; text-align: right;">-3.41%</td>
        <td style="border: 1px solid black; text-align: right;">5.599</td>
        <td style="border: 1px solid black; text-align: right;">5.431</td>
        <td style="border: 1px solid black; text-align: right;">-3.00%</td>
        <td style="border: 1px solid black; text-align: right;">13.487</td>
        <td style="border: 1px solid black; text-align: right;">25.615</td>
        <td style="border: 1px solid black; text-align: right;">89.92%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">LRANGE_100 (first 100 elements)</td>
        <td style="border: 1px solid black; text-align: right;">57206.71</td>
        <td style="border: 1px solid black; text-align: right;">58601.35</td>
        <td style="border: 1px solid black; text-align: right;">2.44%</td>
        <td style="border: 1px solid black; text-align: right;">8.958</td>
        <td style="border: 1px solid black; text-align: right;">8.746</td>
        <td style="border: 1px solid black; text-align: right;">-2.37%</td>
        <td style="border: 1px solid black; text-align: right;">4.816</td>
        <td style="border: 1px solid black; text-align: right;">4.896</td>
        <td style="border: 1px solid black; text-align: right;">1.66%</td>
        <td style="border: 1px solid black; text-align: right;">8.911</td>
        <td style="border: 1px solid black; text-align: right;">8.735</td>
        <td style="border: 1px solid black; text-align: right;">-1.98%</td>
        <td style="border: 1px solid black; text-align: right;">9.471</td>
        <td style="border: 1px solid black; text-align: right;">9.839</td>
        <td style="border: 1px solid black; text-align: right;">3.89%</td>
        <td style="border: 1px solid black; text-align: right;">9.783</td>
        <td style="border: 1px solid black; text-align: right;">10.319</td>
        <td style="border: 1px solid black; text-align: right;">5.48%</td>
        <td style="border: 1px solid black; text-align: right;">25.615</td>
        <td style="border: 1px solid black; text-align: right;">35.775</td>
        <td style="border: 1px solid black; text-align: right;">39.66%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">LRANGE_300 (first 300 elements)</td>
        <td style="border: 1px solid black; text-align: right;">22433.02</td>
        <td style="border: 1px solid black; text-align: right;">22429.58</td>
        <td style="border: 1px solid black; text-align: right;">-0.02%</td>
        <td style="border: 1px solid black; text-align: right;">22.825</td>
        <td style="border: 1px solid black; text-align: right;">22.825</td>
        <td style="border: 1px solid black; text-align: right;">0.00%</td>
        <td style="border: 1px solid black; text-align: right;">5.08</td>
        <td style="border: 1px solid black; text-align: right;">4.912</td>
        <td style="border: 1px solid black; text-align: right;">-3.31%</td>
        <td style="border: 1px solid black; text-align: right;">22.831</td>
        <td style="border: 1px solid black; text-align: right;">22.831</td>
        <td style="border: 1px solid black; text-align: right;">0.00%</td>
        <td style="border: 1px solid black; text-align: right;">26.367</td>
        <td style="border: 1px solid black; text-align: right;">27.391</td>
        <td style="border: 1px solid black; text-align: right;">3.88%</td>
        <td style="border: 1px solid black; text-align: right;">27.471</td>
        <td style="border: 1px solid black; text-align: right;">28.271</td>
        <td style="border: 1px solid black; text-align: right;">2.91%</td>
        <td style="border: 1px solid black; text-align: right;">58.687</td>
        <td style="border: 1px solid black; text-align: right;">63.199</td>
        <td style="border: 1px solid black; text-align: right;">7.69%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">LRANGE_500 (first 500 elements)</td>
        <td style="border: 1px solid black; text-align: right;">14708.84</td>
        <td style="border: 1px solid black; text-align: right;">14761</td>
        <td style="border: 1px solid black; text-align: right;">0.35%</td>
        <td style="border: 1px solid black; text-align: right;">34.813</td>
        <td style="border: 1px solid black; text-align: right;">34.682</td>
        <td style="border: 1px solid black; text-align: right;">-0.38%</td>
        <td style="border: 1px solid black; text-align: right;">4.928</td>
        <td style="border: 1px solid black; text-align: right;">4.976</td>
        <td style="border: 1px solid black; text-align: right;">0.97%</td>
        <td style="border: 1px solid black; text-align: right;">34.815</td>
        <td style="border: 1px solid black; text-align: right;">34.687</td>
        <td style="border: 1px solid black; text-align: right;">-0.37%</td>
        <td style="border: 1px solid black; text-align: right;">39.711</td>
        <td style="border: 1px solid black; text-align: right;">42.143</td>
        <td style="border: 1px solid black; text-align: right;">6.12%</td>
        <td style="border: 1px solid black; text-align: right;">41.439</td>
        <td style="border: 1px solid black; text-align: right;">43.775</td>
        <td style="border: 1px solid black; text-align: right;">5.64%</td>
        <td style="border: 1px solid black; text-align: right;">98.111</td>
        <td style="border: 1px solid black; text-align: right;">98.879</td>
        <td style="border: 1px solid black; text-align: right;">0.78%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">LRANGE_600 (first 600 elements)</td>
        <td style="border: 1px solid black; text-align: right;">12634.49</td>
        <td style="border: 1px solid black; text-align: right;">12635.27</td>
        <td style="border: 1px solid black; text-align: right;">0.01%</td>
        <td style="border: 1px solid black; text-align: right;">40.527</td>
        <td style="border: 1px solid black; text-align: right;">40.517</td>
        <td style="border: 1px solid black; text-align: right;">-0.02%</td>
        <td style="border: 1px solid black; text-align: right;">4.944</td>
        <td style="border: 1px solid black; text-align: right;">4.952</td>
        <td style="border: 1px solid black; text-align: right;">0.16%</td>
        <td style="border: 1px solid black; text-align: right;">40.511</td>
        <td style="border: 1px solid black; text-align: right;">40.511</td>
        <td style="border: 1px solid black; text-align: right;">0.00%</td>
        <td style="border: 1px solid black; text-align: right;">42.719</td>
        <td style="border: 1px solid black; text-align: right;">49.439</td>
        <td style="border: 1px solid black; text-align: right;">15.73%</td>
        <td style="border: 1px solid black; text-align: right;">44.383</td>
        <td style="border: 1px solid black; text-align: right;">52.383</td>
        <td style="border: 1px solid black; text-align: right;">18.02%</td>
        <td style="border: 1px solid black; text-align: right;">110.911</td>
        <td style="border: 1px solid black; text-align: right;">110.335</td>
        <td style="border: 1px solid black; text-align: right;">-0.52%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">MSET (10 keys)</td>
        <td style="border: 1px solid black; text-align: right;">100727.91</td>
        <td style="border: 1px solid black; text-align: right;">103142.62</td>
        <td style="border: 1px solid black; text-align: right;">2.40%</td>
        <td style="border: 1px solid black; text-align: right;">5.1</td>
        <td style="border: 1px solid black; text-align: right;">4.985</td>
        <td style="border: 1px solid black; text-align: right;">-2.25%</td>
        <td style="border: 1px solid black; text-align: right;">2.552</td>
        <td style="border: 1px solid black; text-align: right;">2.6</td>
        <td style="border: 1px solid black; text-align: right;">1.88%</td>
        <td style="border: 1px solid black; text-align: right;">5.063</td>
        <td style="border: 1px solid black; text-align: right;">4.927</td>
        <td style="border: 1px solid black; text-align: right;">-2.69%</td>
        <td style="border: 1px solid black; text-align: right;">5.391</td>
        <td style="border: 1px solid black; text-align: right;">5.287</td>
        <td style="border: 1px solid black; text-align: right;">-1.93%</td>
        <td style="border: 1px solid black; text-align: right;">5.655</td>
        <td style="border: 1px solid black; text-align: right;">5.991</td>
        <td style="border: 1px solid black; text-align: right;">5.94%</td>
        <td style="border: 1px solid black; text-align: right;">17.599</td>
        <td style="border: 1px solid black; text-align: right;">26.623</td>
        <td style="border: 1px solid black; text-align: right;">51.28%</td>
    </tr>
    <tr>
        <td style="border: 1px solid black;">XADD</td>
        <td style="border: 1px solid black; text-align: right;">100926.48</td>
        <td style="border: 1px solid black; text-align: right;">103727.72</td>
        <td style="border: 1px solid black; text-align: right;">2.78%</td>
        <td style="border: 1px solid black; text-align: right;">5.083</td>
        <td style="border: 1px solid black; text-align: right;">4.95</td>
        <td style="border: 1px solid black; text-align: right;">-2.62%</td>
        <td style="border: 1px solid black; text-align: right;">2.424</td>
        <td style="border: 1px solid black; text-align: right;">2.816</td>
        <td style="border: 1px solid black; text-align: right;">16.17%</td>
        <td style="border: 1px solid black; text-align: right;">5.071</td>
        <td style="border: 1px solid black; text-align: right;">4.911</td>
        <td style="border: 1px solid black; text-align: right;">-3.16%</td>
        <td style="border: 1px solid black; text-align: right;">5.391</td>
        <td style="border: 1px solid black; text-align: right;">5.263</td>
        <td style="border: 1px solid black; text-align: right;">-2.37%</td>
        <td style="border: 1px solid black; text-align: right;">5.575</td>
        <td style="border: 1px solid black; text-align: right;">5.775</td>
        <td style="border: 1px solid black; text-align: right;">3.59%</td>
        <td style="border: 1px solid black; text-align: right;">13.615</td>
        <td style="border: 1px solid black; text-align: right;">26.351</td>
        <td style="border: 1px solid black; text-align: right;">93.54%</td>
    </tr>
</table>

## 结论
对比下来，差距不大，针对不同场景也各有优劣，最大延迟上面 Valkey 表现普遍不如 Redis，其他指标上 Valkey 略胜一筹，说明 Valkey 表现波动比
Reids 要大，这点从硬件资源监控也能发现 Valkey 的 CPU 使用上不如 Redis 平稳。  
针对 redis-Benchmark 的一些局限性，后面计划再用 [resp-benchmark](https://github.com/tair-opensource/resp-benchmark)
测试看看会不会有差异。
