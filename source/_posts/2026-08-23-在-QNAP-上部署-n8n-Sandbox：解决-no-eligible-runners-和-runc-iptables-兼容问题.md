---
title: 在 QNAP 上部署 n8n Sandbox：解决 no eligible runners 和 runc / iptables 兼容问题
date: 2026-08-23 17:00:34
description: "记录在 QNAP NAS 上部署 n8n Sandbox 时排查 no eligible runners 错误，并通过替换 runc 与调整 Docker 网络配置解决内核和 iptables 兼容问题。"
categories:
  - Systems and Operations
  - Linux
tags:
  - n8n
  - QNAP
  - Docker
  - runc
  - iptables
---

最近在 NAS 上更新了 n8n 的版本，发现多了一个 [AI Assistant](https://docs.n8n.io/deploy/host-n8n/configure-n8n/set-up-ai-assistant) 
功能。本来升级 n8n 就是想看看有没有什么 MCP 能让 AI 来帮我维护工作流，现在 n8n 自己支持了，真实太方便了，结果在配置 
[n8n Sandbox](https://docs.n8n.io/deploy/host-n8n/configure-n8n/set-up-ai-assistant#id-2.-self-host-the-sandbox-manually-advanced) 
时遇到了不少问题。  

<!--more-->

第一个问题是 Sandbox Service 会通过一个 Runner 在 Docker 中创建用于执行代码的 Sandbox 容器，官方 Linux 环境推荐使用 `sysbox-runc` 
来实现 [Docker in Docker (DinD)](https://www.docker.com/resources/docker-in-docker-containerized-ci-workflows-dockercon-2023/)，
但是 QNAP 的 Container Station 不支持 Sysbox。

因为是自己用，对隔离性没什么要求，所以就想参考 macOS 的运行方式，直接使用 `privileged` 模式运行 `runner-dind`。

结果部署完成后 `sandbox-api` 和 `runner` 都能正常启动，但是在 n8n 中测试 Sandbox Service 时一直提示：

```text
The service couldn't complete the test. Check its status and your settings, then try again.
```

查看 `sandbox-api` 日志发现：

```text
create sandbox failed: no eligible runners
```

没有符合要求的 runner，于是去检查 Runner。

## 检查 Runner

先尝试从 n8n 容器中测试 API：

```bash
wget -S -O- http://sandbox-api:8080/healthz
```

结果正常返回：

```text
HTTP/1.1 200 OK
```

Runner 的 Health Check 也正常：

```bash
wget -S -O- http://sandbox-runner-1:8080/healthz
```

同样是正常返回 `200 OK`。

然后去查看 Runner 的日志：

```text
runner registration stream established
runner registration heartbeat sent
sandbox image ready
```

和 API 的日志：

```text
runner registered runner_id=runner-1 ... healthy=false
```

runner 正常运行，API 也能收到 Runner 注册，说明 n8n、API、Runner 之间的 Docker 网络、mTLS 和 gRPC 注册应该都没有问题，但是 Runner 
一直都没有变成可调度状态。

## 手动创建 Sandbox

n8n 页面上的错误信息比较少，所以尝试在 n8n 容器中直接调用 Sandbox API 创建一个 Sandbox：

```bash
wget -S -O- \
  --post-data='{}' \
  --header='Content-Type: application/json' \
  --header='X-Api-Key: [API_KEY]' \
  http://sandbox-api:8080/sandboxes
```

API 返回了 `500 Internal Server Error`，日志中终于出现了详细的错误：

```text
create sandbox failed: runner control create ...

Error response from daemon:
failed to create task for container:
failed to create shim task:
OCI runtime create failed:
runc create failed:
unable to start container process:
error during container init:
error creating device nodes:
update new c device inode /dev/null file mode:
fchmodat2 AT_EMPTY_PATH: no such file or directory
```

所以问题不是出在 n8n 或 Sandbox Service，而是 Runner 内部的 Docker 无法创建容器。

进入 Runner 容器内部：

```bash
docker exec -it n8n-sandbox-runner-1 sh
```

直接运行一个 Alpine 镜像试试：

```bash
docker run --rm alpine:latest echo hello
```

果然出现了完全相同的错误：

```text
runc create failed:
unable to start container process:
error during container init:
error creating device nodes:
update new c device inode /dev/null file mode:
fchmodat2 AT_EMPTY_PATH: no such file or directory
```

Runner 中的版本是：

```text
Docker version 29.3.1
runc version 1.3.4
```

我的宿主机是威联通的 NAS，内核版本是：

```text
5.10.60-qnap
```

## runc 的 fchmodat2 问题

搜索 runc 的历史提交后，找到了一个比较关键的 [Commit](https://github.com/opencontainers/runc/commit/01de9d65dc72f67b256ef03f9bfb795a2bf143b4)：

```text
01de9d65dc72f67b256ef03f9bfb795a2bf143b4
rootfs: avoid using os.Create for new device inodes
```

这个修改是一次针对 CVE 的安全修复。

之前创建 `/dev/null` 这些设备节点后，会直接通过路径修改权限：

```go
os.Chmod(dest, fileMode)
```

修改后会先用 `O_PATH` 打开刚创建的设备节点，然后再通过文件描述符直接修改权限：

```go
unix.Fchmodat(int(f.Fd()), "", mode, unix.AT_EMPTY_PATH)
```

因为 `fchmodat2` 的 `AT_EMPTY_PATH` 是 [Linux 6.6 才支持](https://www.man7.org/linux/man-pages/man2/fchmodat.2.html)的，所以 
runc 同时实现了老内核的兼容逻辑。如果调用返回 `EINVAL` 或者 `EOPNOTSUPP` 就会退回通过 `/proc/self/fd` 修改权限。结果 QNAP 的 
`5.10.60-qnap` 内核返回的却是：

```text
ENOENT
No such file or directory
```

刚好不在 runc 的 fallback 判断范围内，所以 runc 没有进入兼容逻辑，直接认为操作失败。

首先想到的方案是降回修复漏洞之前的 runc 版本，但 QNAP 用的是自己修改过的内核，直接按 Linux 5.10 内核的行为来判断不可靠，要先验证一下旧方案是否能用。

## 验证旧方式能否使用

如果 QNAP 连旧的 `/proc/self/fd` 方法也不支持，那么降级 runc 也没有意义。

先测试一下普通文件：

```bash
tmp=$(mktemp)

exec 9<> "$tmp"

chmod 600 /proc/self/fd/9

stat "$tmp"
```

权限能正常修改成 `0600`。

再测试一种更接近 runc 实际行为的情况，使用 `O_PATH` 打开 `/dev/null`：

```python
import os

fd = os.open("/dev/null", os.O_PATH)

print("fd =", fd)
print("target =", os.readlink(f"/proc/self/fd/{fd}"))

try:
    os.chmod(f"/proc/self/fd/{fd}", 0o666)
    print("chmod via /proc/self/fd: OK")
except Exception as e:
    print("chmod via /proc/self/fd: FAILED:", repr(e))

os.close(fd)
```

结果：

```text
fd = 3
target = /dev/null
chmod via /proc/self/fd: OK
```

说明 QNAP 的内核支持旧的 `/proc/self/fd` 方式，问题只是新的 `fchmodat2 + AT_EMPTY_PATH` 调用返回了 runc 预期之外的错误。

查询了 runc 的提交记录后发现，这次设备节点的安全修复包含在了 `1.3.3` 中，所以先尝试退回修改前的 `1.3.2`。

## 自定义 Runner 镜像

为了尽可能少改动 Compose file，没有直接降级 Docker 和 Sandbox Service，而是继续使用官方 Runner，只替换掉里面的 runc。

Dockerfile：

```dockerfile
FROM ghcr.io/n8n-io/n8n-sandbox-service-runner-dind:latest

RUN apk add --no-cache curl \
    && curl -fL \
      https://github.com/opencontainers/runc/releases/download/v1.3.2/runc.amd64 \
      -o /usr/local/bin/runc \
    && chmod 0755 /usr/local/bin/runc \
    && runc --version
```

构建后发布到自己的 Docker Hub：

```bash
docker build \
  -t hnliuzesen/n8n-sandbox-runner:29.3.1-runc1.3.2 \
  .

docker push hnliuzesen/n8n-sandbox-runner:29.3.1-runc1.3.2
```

然后将 Compose 中原来的 Runner：

```yaml
image: ghcr.io/n8n-io/n8n-sandbox-service-runner-dind:latest
```

替换为：

```yaml
image: hnliuzesen/n8n-sandbox-runner:29.3.1-runc1.3.2
```

重新启动后确认版本：

```text
runc version 1.3.2
Docker version 29.3.1
```

再次执行：

```bash
docker run --rm alpine:latest echo hello
```

本来以为就要成功了，结果又出现了一个新的错误：

```text
failed to set up container networking:
failed to create endpoint ...

Unable to enable DIRECT ACCESS FILTERING - DROP rule:

iptables --wait -t raw -A PREROUTING ...

iptables v1.8.11 (legacy):
can't initialize iptables table `raw':
Table does not exist
```

错误从 runc 创建 `/dev/null` 变成了 Docker 创建网络，说明前面的 runc 问题已经解决。

## Docker 29 的 raw table 问题

Docker 29 在创建 Bridge 网络 Endpoint 时会添加 Direct Access Filtering 规则：

```text
iptables -t raw -A PREROUTING ...
```

但是 QNAP 的 iptables 竟然不是标准的四表五链，没有提供 `raw` table，因此 Docker 无法创建网络。

因为只是自己在内网用的 Sandbox 环境，可以允许直接访问容器已发布端口，所以给 Runner 内部的 Docker daemon 增加配置：

```json
{
  "allow-direct-routing": true
}
```

重新启动 Runner 后再次测试：

```bash
docker run --rm alpine:latest echo hello
```

这次终于正常返回：

```text
hello
```

测试通过，把这个配置也写进自定义镜像：

```dockerfile
FROM ghcr.io/n8n-io/n8n-sandbox-service-runner-dind:latest

RUN apk add --no-cache curl \
    && curl -fL \
      https://github.com/opencontainers/runc/releases/download/v1.3.2/runc.amd64 \
      -o /usr/local/bin/runc \
    && chmod 0755 /usr/local/bin/runc \
    && mkdir -p /etc/docker \
    && printf '%s\n' \
      '{' \
      '  "allow-direct-routing": true' \
      '}' \
      > /etc/docker/daemon.json
```

## 最终测试

这里有个坑。重新启动 Sandbox Service 后，Runner 日志中会先看到：

```text
runner registration stream established
runner registration heartbeat sent
```

Sandbox 镜像准备完成后显示：

```text
sandbox image ready
```

Runner 才会变成可用状态。

之前有一次在 `sandbox image ready` 之前就点击了 n8n 的测试，这时 API 仍然会返回：

```text
no eligible runners
```

这是 Runner 还未准备完成，只需要等镜像准备完成后重新调用：

```bash
wget -S -O- \
  --post-data='{}' \
  --header='Content-Type: application/json' \
  --header='X-Api-Key: [API_KEY]' \
  http://sandbox-api:8080/sandboxes
```

如果可以正常创建 Sandbox，n8n 中的 Sandbox Service 测试应该就可以通过了。

最后成功解决了遇到的两个 QNAP 内核兼容性问题：
- QNAP `fchmodat2` 返回 `ENOENT`
- QNAP 内核 iptables 没有 raw table

其中 runc 1.3.2 是在相关[设备节点安全问题](https://nvd.nist.gov/vuln/detail/CVE-2025-52565)修复之前的版本，所以这个方案更适合用于受控环境。如果你的 
Sandbox 需要执行不可信用户提交的代码，请使用官方推荐的 Sysbox 等隔离方案。
