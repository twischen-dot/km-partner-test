> 本项目说明多台 Windows、Mac 使用本机 Codex CLI，由 Mac Mini 4 集中管理配置和用量的方案；2026-10-31 形成设计，尚未完成跨平台试点。

---
## 提纲
- [[#一、结论]]
- [[#二、三层架构]]
- [[#三、三个目标如何实现]]
- [[#四、已确认的边界与待验事项]]
- [[#五、文档入口]]

---
## 一、结论
每台 Windows 或 Mac 笔记本在**本机**运行 Codex CLI，直接处理本机文档和软件。Mini4 保存统一配置、发布更新、接收各机用量摘要；美国服务器只提供模型请求所需的网络出口。笔记本外出或 Mini4 暂时不可达时，本机 Codex 仍应能工作，用量和管理状态随后补报。

不把“网络出口经过 Mini4”当成统计 token 的方法。普通代理只能看连接和字节数，不能从加密流量准确还原模型 token。按机器统计必须从各机的 Codex 用量事件采集。→ [OpenAI Docs · Codex 监控与遥测](https://learn.chatgpt.com/docs/agent-approvals-security)

## 二、三层架构
```text
Windows / Mac 笔记本（每台独有文件）
  ├─ 本机 Codex CLI：读写本机目录、运行本机命令
  ├─ 本机代理：AI 域名 → 美国节点；国内系统按现有直连规则
  ├─ 本机用量采集：机器名 + 会话/事件标识 + 时间 + token 数
  └─ 本机待发送队列：离线时保存，上线后向 Mini4 补报

Mac Mini 4（管理台）
  ├─ 配置模板、安装包版本、更新清单
  ├─ 每台设备的状态、最近上报时间、用量汇总
  └─ 经受控通道供笔记本主动拉取与上报

美国服务器（出口节点）
  └─ 固定域名指向当前 IP；机器换 IP 时只更新域名解析
```
Mini4 可以在家中内网且能访问互联网；这不意味着外出的笔记本能直接连进来。跨网访问需先验证现有 EasyTier/Tailscale 等通道，或用受控中继。笔记本采用**主动连接**，并允许离线排队，不依赖持续在线的反向管理连接。

## 三、三个目标如何实现

| 目标 | 做法 | 关键限制 |
|---|---|---|
| 管理本地文档与软件 | Codex 在本机启动，以本机目录为工作区；软件安装/卸载用该系统原生命令和权限流程 | 管理员权限、系统弹窗和 Codex 批准仍在目标机处理；不能把 Mini4 的权限视作笔记本权限 |
| 按电脑统计 token | 各机产生本机用量记录，附稳定设备 ID，Mini4 去重汇总 | 需要先实测 CLI 交互、`codex exec`、重启和离线场景；账号总用量不等于机器用量之和 |
| 美国服务器 IP 变化 | 代理节点的 `server` 使用自有固定域名；仅 IP 变更时更新 DNS | 协议、密钥、端口变化要发布新版配置；DNS 缓存和解析路径须实测 |

Codex 的 `/status` 可显示当前会话 token 用量；官方遥测的部分完成事件含 token 数，也可通过 app-server 取得线程用量更新。采用哪一种自动采集路线，先在当前 CLI 版本做小样验证，再定实现。→ [OpenAI Docs · CLI 命令](https://learn.chatgpt.com/docs/developer-commands) · [监控事件](https://learn.chatgpt.com/docs/agent-approvals-security) · [app-server](https://learn.chatgpt.com/docs/app-server)

mihomo 节点 `server` 支持域名或 IP。域名只用于定位节点，Reality 的 `servername`、公钥和 short-id 是另一组参数，不能因为换地址就随意改。→ [mihomo 文档 · Common fields](https://wiki.metacubex.one/en/config/proxies/) · [VLESS](https://wiki.metacubex.one/en/config/proxies/vless/)

## 四、已确认的边界与待验事项
**已确认的设计边界：** Codex CLI 在本机运行才能直接处理该机文件；每台机器保留自己的 ChatGPT 登录状态。换服务器 IP 不要求重登 ChatGPT；主动换 ChatGPT 账号则要逐机授权。各机份额共用同一 ChatGPT 账号的额度，机器报表只用于归因。→ [OpenAI Docs · 认证](https://learn.chatgpt.com/docs/auth?surface=cli) · [Codex 用量](https://learn.chatgpt.com/docs/pricing)

**尚未在本项目验证：** Windows 与 Mac 自动采集的 token 字段和去重口径；外出热点到 Mini4 的实际可达性；离线队列补报；固定域名在当前 Reality 节点上的解析、切换时间与真实 Codex 调用；跨平台更新流程。验收通过前，文档状态保持“方案”。

**敏感资料边界：** 本项目目录不存 `auth.json`、API key、节点 UUID/公钥/short-id、SSH 私钥或完整代理配置。上报数据默认只含计数和机器标识，不含提示词、文档正文或工具输出。Codex 遥测配置中保持 `log_user_prompt = false`，接收端也只保留必要字段。→ [OpenAI Docs · 遥测安全建议](https://learn.chatgpt.com/docs/agent-approvals-security)

## 五、文档入口
- [010-部署与运维操作指南](010-%E9%83%A8%E7%BD%B2%E4%B8%8E%E8%BF%90%E7%BB%B4%E6%93%8D%E4%BD%9C%E6%8C%87%E5%8D%97.md)：试点、逐机部署、日常统计、换 IP、更新与排障。
- [900 · Windows 本机安装 Codex 与 Claude](../900-windows%20computer%20use%20codex%20and%20claude/000-README.md)：已有 Windows 本机方案；本项目只负责多机管理层，不复制安装脚本或敏感配置。
- `300 · 网络资源`（原库引用，未包含在本测试包）：现行资源入口；服务器状态和 IP 要按检查日期重新确认。
