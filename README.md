# 手机服务器项目 · 脱敏版（可分享）

生成时间：2026-09-15

这个包**可以直接给别人，或者给其他 agent 用**——所有凭据和个人信息都已替换成占位符。

## 包里有什么

- `skill/android-napcat-astrbot/` —— 通用 skill（不限机型与系统）。放进 `~/.agents/skills/`
  （用户级，所有项目可用）或 `<项目>/.agents/skills/`（项目级）
- `deploy/` —— 本次部署用到的脚本与配置，作为「参考实现」

## 脱敏对照

| 占位符 | 原本是什么 |
|---|---|
| `<YOUR_SSH_KEY_MATERIAL>` | SSH 密钥内容与指纹 |
| `<YOUR_NATFRP_TOKEN>` | 隧道服务的访问密钥 |
| `<YOUR_CONTAINER_PASSWORD>` / `<YOUR_ASTRBOT_PASSWORD>` / `<YOUR_NAPCAT_WEBUI_TOKEN>` | 各面板口令 |
| `<YOUR_QQ_NUMBER>` / `<YOUR_DEVICE_SERIAL>` / `<DEVICE_MODEL>` | 账号与设备标识 |
| `<YOUR_PUBLIC_IP>` / `<PHONE_LAN_IP>` / `<PC_LAN_IP>` / `<GATEWAY_IP>` / `<TUNNEL_NODE_DOMAIN>` / `<TUNNEL_NODE_IP>` | 各类网络地址 |
| `<YOUR_WIFI_SSID>` | WiFi 名称 |
| `<TUNNEL_ID_*>` | 隧道 ID |

**未收录**：SSH 私钥、界面截图、安装物料（Linux Deploy APK / Ubuntu rootfs / frpc 二进制）。
前者是隐私，后者能从公开渠道重新下载（skill 里写了来源和获取方法）。

## 怎么用这份 skill

1. 把 `skill/android-napcat-astrbot/` 整个目录复制到 `~/.agents/skills/`
2. 之后 agent 处理「手机跑 QQ 机器人 / 把手机做成服务器 / 内网穿透发 WebUI」这类任务时会自动加载
3. 也可以手动强制加载：`/skill android-napcat-astrbot`

## 怎么用 deploy/ 里的脚本

它们是这次真实部署里跑通的版本，用之前把占位符换成你自己的值。重点几个：

| 文件 | 作用 |
|---|---|
| `phone-postinstall.sh` | 容器重装后一键恢复（PAM 修复、用户、SSH 加固、审计日志、rc.local、服务脚本、隧道程序） |
| `svc-watchdog` / `99-ld-watchdog.sh` | 容器层 / 手机层看门狗 |
| `napcat-svc` / `astrbot-svc` | 两个业务服务的进程管理（看门狗靠它们拉起服务） |
| `frpc-tunnel` + `frpc-tunnel.conf` | 隧道客户端包装脚本与配置 |
| `debloat.sh` / `debloat2.sh` / `d3.sh` | 去预装（分三批，用 `pm uninstall --user 0`，可回滚） |
| `tls.sh` | 容器内 TLS 终结（中转拦明文 HTTP 时用） |
| `napcat.sh` | NapCat 官方安装脚本（非 Docker 模式） |

## 已知局限（如实标注，别当成"全都验证过"）

| 位置 | 情况 |
|---|---|
| NapCat 的 OneBot 配置**界面步骤** | 只写到"在 WebUI 里配一个 OneBot 服务端、记下端口与 token"，菜单名/字段随版本变化，**未逐步验证**；但验证方法是可靠的（看 AstrBot 日志里有没有 QQ 会话消息） |
| AstrBot 的 aiocqhttp 适配器字段、**模型 provider 配置** | 同上，只给结论与验证方法，界面细节以官方文档为准 |
| Termux / proot-distro 路线 | 只覆盖 Linux Deploy；其它 Linux 用户空间方案**未验证** |
| 版本时效 | 包内脚本是 **2026-09 快照**：QQ 客户端版本、AstrBot 版本、隧道服务的规则与额度都会变 |
| `deploy/svc-watchdog` 与 `skill/.../scripts/svc-watchdog.sh` | 前者是本次部署用的「参考实现」（服务名写死），后者是「通用模板」（服务表可配）。两者都可参考，别纠结哪个"对" |
| `linux.conf` 里的 `SOURCE_PATH` | 写的是本次用的归档文件名，与你实际下载的 rootfs 文件名可能不同，按自己的改 |

## 安全前提（务必先确认）

这套东西会把服务暴露到公网。用之前确认：

- 容器 SSH **已关闭密码认证**（`sshd -T | grep -i '^passwordauthentication'` → `no`），
  否则不要发布 22 端口
- 各面板的初始密码都已改掉
- 隧道访问密钥、SSH 私钥等凭据只放在权限 600 的文件里
