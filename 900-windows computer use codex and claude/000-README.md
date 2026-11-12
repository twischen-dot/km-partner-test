---
项目: 900 · 多台 Windows 电脑本机使用 Claude Code 与 Codex
文档: 索引与总体方案
建档: 2026-10-30
作者: Claude（US2 槽位 4）
依据: 600 <伙伴A>装机（2026-09-18/19 实测）、270 <伙伴A>断网排障（2026-10-20 实测）、570 美国 VPS 项目
状态: 🟡 方案整理完成，尚未在你自己的任何一台 Windows 上按本指南完整走过一遍
---

# 900 · 多台 Windows 电脑用 Claude 与 Codex

## 一句话方案

每台 Windows 本机装 **Claude Code**（用 Claude Pro）和 **Codex CLI**（用 ChatGPT Plus），起始目录设为本机 Obsidian 库，让两个 AI 直接读写本地文件。网络由本机 **mihomo** 分流：AI 相关域名经美国 VPS 节点出境，国内系统一律直连，兜底 `MATCH,DIRECT`。

> 这套方案来自<伙伴A>电脑，原记录在 `005-Project/600-how to install bang brother windows computer/` 与 `005-Project/270-Help debug the Bang brother computer/`。本目录把它改写成**可在多台电脑上重复执行**的版本，凡是只在<伙伴A>机器上实测过、没在新机器上验证的地方，都标了「未实测」。

## 架构

```text
Windows 桌面图标「Claude」/「ChatGPT」
  └─ Windows Terminal 配置（起始目录 = 本机 Obsidian 库）
       └─ claude-local.cmd / codex-local.cmd（进程内设 HTTPS_PROXY=127.0.0.1:7890）
            └─ 本机 claude.exe / codex.exe
                 └─ mihomo（127.0.0.1:7890，SYSTEM 开机任务 + 守护循环）
                      ├─ anthropic / claude / openai / chatgpt …  → 美国 VPS 节点（<代理协议>）→ Anthropic / OpenAI
                      ├─ 公司 OA、银行、券商、微信                  → DIRECT
                      └─ 其余未知                                    → DIRECT（兜底）
本机 Obsidian 库  ← 两个 AI 按库根目录 AGENTS.md 的规矩读写
                  ← 计划任务每小时快照备份（含两个 AI 的原生记忆）
```

要点：
- **AI 程序不在美国服务器上跑。** 美国 VPS 只做出口节点。Windows 能直连 VPS，但不能直连 Anthropic / OpenAI。
- **命令行工具不读 Windows 系统代理**，所以代理写在启动脚本的进程环境里（坑 N1）。
- **文件不离开本机**，只有 AI 实际读到的内容会发给 Anthropic / OpenAI。

## 为什么选这个方案（不再重复讨论）

| 被否决的做法 | 原因（详见 600/010） |
|---|---|
| Claude 桌面 App + 系统代理 | MSIX 难装、`region_unavailable`、浏览器登录易出错；桌面版不认一年期 token |
| SSH 到服务器上用 CLI | 读不到本机 Obsidian 库；要么文件上服务器，要么从服务器反连电脑 |
| 全局代理 / TUN | 公司 OA、银行、券商被误送出境可能触发不可撤销的风控 |
| 拷贝别的机器的凭据文件 | OAuth refresh token 轮换会互相踢下线 |

## 文档地图

| 文件 | 内容 |
|---|---|
| `000-README.md` | 本页：方案、架构、开工前决定、机器登记表 |
| `010-逐台部署手册.md` | 每台机器从零到验收的步骤，每步带验收标准 |
| `020-维护排障与换号换节点.md` | 日常维护、故障分层判断、VPS 换 IP、换账号、拆远程通道 |

脚本原件（不含密钥）仍在原项目里，不复制一份，避免两处版本分叉：
- `005-Project/600-how to install bang brother windows computer/scripts/`：装机脚本（`claude-local.cmd`、`codex-local.cmd`、`mihomo-config-template.yaml`、`register-mihomo.ps1`、`mihomo-keepalive.ps1`、`set-proxy.ps1`、`run-task.ps1`、`gui.ps1`、`click.ps1`、`lid.ps1`、`claude-preseed.ps1`、`paste-keys.ps1`、`codex-login-local.ps1`、`set-claude-token.sh`、`vault-backup.ps1`、`AGENTS.md`、`AI记忆-模板.md`）。
- `005-Project/270-Help debug the Bang brother computer/scripts/20261020-original/`：排障脚本（诊断、临时放行、独立测试节点、真实桌面验收）。这些是**一次性脚本**，写死了 IP、机器名和日期，复用前必须改。

> 服务器只读镜像只同步 Markdown/txt，`.ps1`、`.cmd`、`.yaml` 只在 Mac 主库里有，经 `mini4-run` 或 `air-run` 读取。

## 开工前对每台机器先定五件事

在 600 项目中，这五件事是边做边冒出来的，导致方案推翻了三次。你自己的电脑多数答案会不同，**每台单独定，写进下面的登记表**。

1. **这台机器给谁用？** 你自己 / 家人 / 朋友或客户。决定后面几项的默认值。
2. **AI 是否要读写本机文件？** 一般是要，那就必须本机安装。
3. **装好后留不留远程管理通道？** 给别人的电脑，默认不留（600 的约束）；你自己的电脑可以留，但范围要小，见 010 第 1 步。
4. **这台机器常访问哪些国内系统？** 公司 OA、BI、券商、银行、网盘等，先写进直连名单。
5. **用哪个账号、谁续费、什么时候到期？** 见下节。

## 账号策略（多台电脑最容易出问题的地方）

| | Claude（Pro） | ChatGPT（Plus，Codex 使用） |
|---|---|---|
| 登录方式 | 每台机器各自 `claude setup-token` 生成一枚一年期 token，存到本机 `%USERPROFILE%\.claude\oauth-token` | 每台机器各自 `codex login --device-auth`，手机上完成设备码授权，结果在 `%USERPROFILE%\.codex\auth.json` |
| 多台共用一个账号 | 可以，但额度共用；**每台一枚独立 token**，以便某台机器丢失时单独吊销（未在多台同号场景实测） | 可以，额度共用；每台各做一次设备码登录（未在多台同号场景实测） |
| 绝对不要 | 把 token 文件从一台拷到另一台；在聊天里贴 token；用 `claude auth logout` | 拷贝 `auth.json`；用 `codex logout`（可能吊销服务端 token） |
| 换号 | 重新 setup-token，覆盖 token 文件；再到 claude.ai 吊销旧 token | 重新设备码登录，覆盖 `auth.json` |

- 给别人用的机器：用**他自己的** Pro/Plus，一人一号。<伙伴A>那台到 09-25 还在用 <他人账号> 的 token，这就是过渡方案没收尾的教训。
- Pro 到期时报错的特征是「organization has disabled」，这表示订阅过期，不是封号。

## 机器登记表（每装一台填一行）

VPS 换 IP 后要按这张表逐台更新客户端（270 的根因就是漏更新了<伙伴A>的电脑）。表里**不写**节点 UUID、公钥、token、私钥。

| 机器名 | 使用者 | Windows 用户 | 库路径 | Claude 账号 / token 生成日 | ChatGPT 账号 | 节点及 IP（填写日期） | 远程通道 | 装机日期 | 最近验收 |
|---|---|---|---|---|---|---|---|---|---|
| <目标机主机名> | <伙伴A> | <内部标识> | `D:\<伙伴A>的知识库` | <他人账号> token（09-25 仍未换） | <电脑用户名>（09-25 仍未换） | <代理节点> → <美国VPS IP>（10-20） | 局域网 SSH，限来源；10-20 临时规则已撤 | 2026-09-19 | 2026-10-20 两个 AI 都回复 NETWORK_OK |
| （新机器） | | | | | | | | | |

## 当前已知节点信息（有时间边界）

- AI 流量节点：`<代理节点名>`，<代理协议>，在第一台美国 VPS 上。2026-10-17 IP 从 `<美国VPS旧IP>` 改为 `<美国VPS IP>`（570 项目 017、018）。
- 节点参数（UUID、public-key、short-id）**永远不写进知识库**。从 Mac 本地的私有文件取，例如 `~/Downloads/la-qqg-node-20260926/la-qqg.yaml`，使用前确认 server 字段已是新 IP。
- <伙伴A>的配置里还有一个东京节点（<备用节点名>），用于 Google 等其他境外流量。你自己的机器如果没有别的节点，两组都指向 <代理节点> 即可。
