# OpenClaw 常用操作指南

这份指南总结了日常使用和维护 OpenClaw Gateway 时的常用命令和操作流程。

## 1. 服务管理 (Gateway)

Gateway 是 OpenClaw 的核心服务，负责处理所有连接和消息路由。

- **启动 Gateway (前台运行)**
  ```bash
  openclaw gateway run
  ```
- **启动 Gateway (后台运行/守护进程)**
  通常建议使用 `nohup` 或 `systemd`。
  ```bash
  nohup openclaw gateway run --bind loopback --port 18789 --force > /tmp/openclaw-gateway.log 2>&1 &
  ```
- **停止 Gateway**
  ```bash
  pkill -9 -f openclaw-gateway
  ```
- **查看 Gateway 状态**
  ```bash
  openclaw status
  # 或者查看更详细的探针状态
  openclaw status --deep
  ```

## 2. 设备配对与安全 (Device Pairing)

当通过公网（如 Cloudflare Tunnel, Nginx 反代）访问 Control UI 时，出于安全考虑，OpenClaw 要求对新设备（浏览器）进行手动授权。

- **查看所有设备状态 (包含待审批的请求)**

  ```bash
  openclaw devices list
  ```

  _在输出中寻找 `Pending` 列表，记录下 `Request` ID。_

- **批准设备配对请求**

  ```bash
  openclaw devices approve <Request-ID>
  ```

  _例如：`openclaw devices approve ba211854-776a-46b0-96cb-bc203062d568`_

- **移除已配对的设备**
  ```bash
  openclaw devices remove <Device-ID>
  ```

## 3. 配置管理 (Config)

OpenClaw 的配置可以通过 CLI 动态修改，无需手动编辑 JSON 文件。

- **查看当前配置**
  ```bash
  openclaw config get
  ```
- **查看特定配置项**
  ```bash
  openclaw config get gateway.controlUi.allowedOrigins
  ```
- **设置配置项**

  ```bash
  # 允许特定的域名访问 Control UI (解决跨域问题)
  openclaw config set gateway.controlUi.allowedOrigins '["https://clawd.biubiuniu.com"]'

  # 设置 Gateway 绑定的 IP (loopback, lan, auto)
  openclaw config set gateway.bind loopback
  ```

## 4. 频道与消息 (Channels)

管理连接到 OpenClaw 的各种聊天平台（如 Discord, Telegram, WhatsApp 等）。

- **查看所有频道状态**
  ```bash
  openclaw channels status
  ```
- **频道配对 (针对需要扫码或验证的频道，如 WhatsApp/Telegram)**
  ```bash
  openclaw pairing list <channel_name>
  openclaw pairing approve <channel_name> <code>
  ```

## 5. 故障排查 (Troubleshooting)

- **运行诊断检查**

  ```bash
  openclaw doctor
  ```

  _这个命令会检查配置、依赖和常见问题，并给出修复建议。_

- **查看日志**
  如果 Gateway 是后台运行的，查看你重定向的日志文件：
  ```bash
  tail -f /tmp/openclaw-gateway.log
  ```

## 常见问题场景

**场景：通过外网域名打开 Control UI，提示 "pairing required"**

1. 确保你使用的是 `https://`（HTTP 下浏览器无法生成设备密钥）。
2. 确保域名已加入 `allowedOrigins` (`openclaw config get gateway.controlUi.allowedOrigins`)。
3. 运行 `openclaw devices list` 找到 Pending 的请求。
4. 运行 `openclaw devices approve <Request-ID>` 批准请求。
5. 刷新浏览器页面。
