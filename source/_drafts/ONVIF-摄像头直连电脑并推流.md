---
title: ONVIF 摄像头直连电脑并推流
tags:
---

# 从零搭建：ONVIF摄像头直连电脑并推流直播的完整方案

年初在淘宝入手了一个便宜的杂牌ONVIF摄像头，商品页面写着支持RTSP协议，心想这下可以直接连电脑用了。结果买回来才发现，说明书上的拓扑图清一色都显示要接录像机——敢情这玩意儿本来就不是设计来直连电脑的？

折腾了一整天，最终不仅成功让摄像头直连电脑，还把画面推到了直播平台。整个过程挺有代表性的，把踩过的坑和解决方案整理一下，或许能帮到遇到类似问题的朋友。

## 遇到的三大难题

买回来才意识到几个关键问题：

1. **IP地址问题**：摄像头没有固定IP，直接用网线连电脑根本ping不通
2. **无客户端支持**：厂商软件只有Windows版，Linux下完全抓瞎
3. **串流地址未知**：不知道正确的RTSP地址格式，更别说转推直播了

本来以为就是个"连上线就能用"的事儿，结果发现网络摄像头的世界水挺深。

## 方案一：让电脑当DHCP服务器

最开始的想法很简单：既然摄像头需要IP地址，我让电脑临时充当DHCP服务器不就行了？

### 1. 安装dnsmasq

```bash
sudo apt install dnsmasq -y
```

### 2. 配置DHCP服务

创建配置文件 `/etc/dnsmasq.d/camera.conf`：

```ini
port=0                    # 禁用DNS功能，避免端口冲突
interface=enp0s31f6       # 我的有线网卡名
dhcp-range=192.168.1.50,192.168.1.100,24h
```

**关键点**：需要先给网卡设置静态IP，否则dnsmasq不知道该在哪个网段分配地址：

```bash
sudo ip addr flush dev enp0s31f6
sudo ip addr add 192.168.1.1/24 dev enp0s31f6
sudo ip link set enp0s31f6 up
```

### 3. 启动并监控

```bash
sudo systemctl restart dnsmasq
journalctl -u dnsmasq -f
```

看到这条日志说明成功了：
```
DHCPACK(enp0s31f6) 192.168.1.60 00:12:34:6b:df:d7
```

测试连接：
```bash
ping 192.168.1.60
nc -vz 192.168.1.60 554  # 测试RTSP端口
```

## 方案二：通过ONVIF协议获取串流信息

连上网络后，更重要的问题是：怎么获取正确的RTSP串流地址？

### 1. 安装ONVIF客户端

```bash
pip install onvif-zeep
```

### 2. 编写Python脚本获取串流地址

```python
from onvif import ONVIFCamera
import cv2

# 摄像头参数
CAM_IP = "192.168.1.60"
PORT = 8000  # ONVIF端口，扫描发现的有效端口
USER = "admin"
PASS = "admin"

# 初始化连接
cam = ONVIFCamera(CAM_IP, PORT, USER, PASS)
media_service = cam.create_media_service()

# 获取媒体配置
profiles = media_service.GetProfiles()
profile = profiles[0]

# 构造请求获取RTSP地址
req = media_service.create_type('GetStreamUri')
req.ProfileToken = profile.token
req.StreamSetup = {
    'Stream': 'RTP-Unicast',
    'Transport': {'Protocol': 'RTSP'}
}

# 获取并打印RTSP URL
uri = media_service.GetStreamUri(req)
rtsp_url = uri.Uri
print("RTSP URL:", rtsp_url)
```

运行后得到了这样的地址：
```
rtsp://192.168.1.60:554/user=admin_password=tlJwpbo6_channel=0_stream=0&onvif=0.sdp?real_stream
```

**注意**：这个摄像头的认证方式比较特殊，不是标准的 `rtsp://user:pass@` 格式，而是把用户名密码放在URL参数里。

## 方案三：串流转直播推送

有了RTSP地址，下一步就是推送到直播平台。我用的是Mux的RTMPS服务。

### 1. 编码格式转换

最初尝试直接转发：
```bash
ffmpeg -i "rtsp://192.168.1.60:554/user=admin_password=..." -c copy -f flv rtmps://...
```

报错了：
```
Video codec hevc not compatible with flv
```

摄像头输出的是H.265编码，而FLV容器不支持H.265，需要转码成H.264。

### 2. 完整转发命令

```bash
ffmpeg -i "rtsp://192.168.1.60:554/user=admin_password=tlJwpbo6_channel=0_stream=0&onvif=0.sdp?real_stream" \
  -c:v libx264 -preset veryfast -tune zerolatency \
  -c:a aac \
  -f flv "rtmps://global-live.mux.com:443/app/b1dc2e52-bffd-afef-bb9b-ffaa4fa989be"
```

### 3. Python自动化脚本

为了稳定运行，写了个Python脚本自动处理：

```python
import subprocess
import time
from onvif import ONVIFCamera

def get_rtsp_url():
    """通过ONVIF获取最新的RTSP串流地址"""
    cam = ONVIFCamera('192.168.1.60', 8000, 'admin', 'admin')
    media = cam.create_media_service()
    profiles = media.GetProfiles()
    req = media.create_type('GetStreamUri')
    req.ProfileToken = profiles[0].token
    req.StreamSetup = {
        'Stream': 'RTP-Unicast',
        'Transport': {'Protocol': 'RTSP'}
    }
    return media.GetStreamUri(req).Uri

def start_streaming():
    """启动推流"""
    rtsp_url = get_rtsp_url()
    rtmp_url = "rtmps://global-live.mux.com:443/app/b1dc2e52-bffd-afef-bb9b-ffaa4fa989be"
    
    cmd = [
        "ffmpeg", "-i", rtsp_url,
        "-c:v", "libx264", "-preset", "veryfast", "-tune", "zerolatency",
        "-c:a", "aac",
        "-f", "flv", rtmp_url
    ]
    
    subprocess.run(cmd)

if __name__ == "__main__":
    start_streaming()
```

## 关键技术要点总结

### DHCP配置要点
- **先设静态IP**：给网卡配置静态IP，dnsmasq才能确定工作网段
- **禁用DNS**：用 `port=0` 避免端口冲突
- **查看日志**：通过日志确认摄像头IP地址分配情况

### ONVIF协议细节
- **端口扫描**：不同厂商ONVIF端口不同（80、8000、8899等）
- **功能限制**：山寨摄像头可能只支持部分ONVIF功能
- **URL格式**：部分设备使用非标准RTSP地址格式

### 编码转换关键点
- **H.265转H.264**：大多数直播平台需要H.264编码
- **实时优化**：使用 `-preset veryfast -tune zerolatency` 减少延迟
- **音频处理**：同步转换音频为AAC格式

## 后续优化方向

1. **固定IP设置**：避免每次重启后重新配置
2. **自动重连机制**：网络异常时自动恢复推流
3. **本地录像功能**：保存关键时刻视频片段
4. **多平台推送**：同时推送到多个直播平台

整个方案下来，硬件成本就是一个摄像头加一根网线，软件全是开源工具。虽然过程曲折，但最终实现了从摄像头到直播平台的完整链路，比买成品NVR方案灵活太多了。