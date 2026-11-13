# hk-watch-location

accessory-location 的 iPhone USB/iAP2 定位数据传输测试。

执行脚本会自动下载组件包并按 CPU 架构选择对应的 lib 包（主包含 `accessory-location` 脚本与源码、`identity` MFi 证书；lib 包按平台独立打包），校验 SHA-256 后解压，并从 `lib/` 离线安装依赖，无需再克隆仓库或解析 APK。

## 用法

macOS（在仓库根目录执行；仅用于获批测试设备）：

```sh
./hk-watch-location/hk-watch-location_sh run <测试纬度> <测试经度>   # 完整流程：环境检查 → 下载组件包 → 离线装依赖 → 发送
./hk-watch-location/hk-watch-location_sh send <测试纬度> <测试经度>  # 只发送（跳过安装步骤）
./hk-watch-location/hk-watch-location_sh stop                        # 停止发送，恢复手机真实定位
```

Windows PowerShell（在仓库根目录执行；仅用于获批测试设备）：

```powershell
powershell -ExecutionPolicy Bypass -File .\hk-watch-location\hk-watch-location.ps1 run <测试纬度> <测试经度>
powershell -ExecutionPolicy Bypass -File .\hk-watch-location\hk-watch-location.ps1 send <测试纬度> <测试经度>
powershell -ExecutionPolicy Bypass -File .\hk-watch-location\hk-watch-location.ps1 stop
```

## 组件包

主包 `hk-watch-location-bundle.zip`（所有平台通用）：

| 目录 | 内容 |
| --- | --- |
| `accessory-location/` | iAP2 定位客户端源码、`tools/local_mfi_v3_helper.py` 等脚本 |
| `identity/` | 共享实验性 MFi 身份（`certificate.p7b` / `identity.pk8`） |

平台 lib 包（按 CPU 架构独立打包，脚本自动选择）

## 前提

- macOS：已安装 uv（无则脚本自动安装）
- Windows：需要 Apple Devices 应用（Microsoft Store）或官网版 iTunes 提供 usbmuxd（本地端口 27015）。脚本检测到缺失时自动通过 winget 安装（官网渠道优先、Store 渠道兜底），并启动 Apple Mobile Device Service；无 winget 或安装失败时提示手动安装
- iPhone 通过 USB 连接，已解锁并点击"信任此电脑"
- Windows 若连接异常，可重启电脑后重新运行脚本（Apple Devices 安装后通常需重启完成服务注册）

## 说明与限制

本工具用于自有设备或明确获批的测试设备，在受控环境中验证 accessory-location 的数据传输。它会发送 GGA/RMC 模拟坐标；这不是设备的真实 GPS 位置，也不是 Apple 提供的授权定位测试接口，设备或应用可能会暂时收到模拟位置。


设备需解锁并信任连接的电脑。发送阶段持续运行（每轮 900 秒无缝循环），按 `Ctrl+C` 可随时手动停止，累计 30 分钟后自动终止；停止后自动输出结果解读，随后运行 `stop`，断开 USB，并在设备上确认定位已恢复。日志中的坐标包确认只表示传输送达，不表示系统或应用采纳，也不构成任何服务资格证明。
