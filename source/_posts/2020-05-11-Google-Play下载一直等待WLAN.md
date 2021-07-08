---
title: Google Play 下载一直等待 WLAN
date: 2020-05-11 14:24:38
categories:
- Tip
- Android
tags:
- Android
- Google Play
- MIUI
---

手机设置了 Play 通过任何网络下载，MIUI 里面的下载管理器也设置了移动流量不限制，可是用另一个手机分享热点更新应用的时候，Google 
Play 还是会一直在等待 WLAN，而且要在下载管理器里调成显示 100M 或其他数值，才会在每一次下载的时候提示是否要用流量下载，而且不管设置多少，哪怕下载的应用很小，都会无差别提示，不过不能设置不限制，因为这样连提示都不提示了，根本无法更新。尝试了很多方法，虽然最后发现可能是刷的系统的问题，但还是记录一下有用的命令。

<!--more-->

主要参考 [V2EX](https://v2ex.com/t/461403) 。

```Shell
# 查看 job
adb shell dumpsys jobscheduler com.android.providers.downloads
```

关键字是 `UNMETERED` ，意思是不按使用量计费。

```Shell
# 强制启动 job
adb shell jobscheduler run -f com.android.providers.downloads :JOB_ID
```

Job ID 应该不是从 1 开始顺序数的，不过可能对于下载器任务，他的下载任务 ID 是从 1 开始，具体可以参考 
[JobScheduler](https://developer.android.com/reference/android/app/job/JobScheduler)