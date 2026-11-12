# 方案索引

## Mac 存储故障检查与修复 (`hdisk-check`)

- **问题**：Mac 出现 APFS 警告或疑似磁盘故障时，如何先保护数据、有序诊断并安全修复
- **适用场景**：macOS / APFS、内置 SSD；Apple Silicon 实测为主
- **技术栈**：diskutil、fsck_apfs、smartctl、恢复模式
- **状态 / 最近验证**：verified / 2026-11-08
- **原文**：[710-How to check the Hdisk error/000-README.md](710-How%20to%20check%20the%20Hdisk%20error/000-README.md)
- **边界**：外接盘、硬盘盒、供电与接口场景未实测；案例中的磁盘编号与恢复模式步骤不可照抄

## 局域网 Windows/Mac 接入与长期维护 (`lan-access-maintain`)

- **问题**：如何为新 Windows/Mac 电脑建立 SSH 管理通道、部署软件并长期维护
- **适用场景**：同一局域网内的 Windows 与 Mac 混合环境
- **技术栈**：OpenSSH、公钥认证、PowerShell、屏幕共享
- **状态 / 最近验证**：verified（局域网）/ 2026-11-05
- **原文**：[880-how to control the local windows and mac/000-README.md](880-how%20to%20control%20the%20local%20windows%20and%20mac/000-README.md)
- **边界**：SSH 成功不等于 GUI 可用；重启、断网及跨网恢复有缺口

## Windows 本机部署 Codex 与 Claude (`windows-codex-claude`)

- **问题**：在 Windows 电脑上安装、配置和维护 Codex 与 Claude Code
- **适用场景**：Windows 10/11，单人单机使用
- **技术栈**：Codex CLI、Claude Code、代理配置、launch 脚本
- **状态 / 最近验证**：verified（单机）/ 2026-10-20
- **原文**：[900-windows computer use codex and claude/000-README.md](900-windows%20computer%20use%20codex%20and%20claude/000-README.md)
- **边界**：来自单台机器的实测；新机器完整复现未验证；账号一人一号，不可复制凭据

## Mac mini 集中管理多台 Windows Codex (`mini4-fleet-manage`)

- **问题**：用一台 Mac mini 作为管理端，集中管理多台 Windows 上的 Codex 用量与配置
- **适用场景**：小规模多机管理；异地主动连接架构参考
- **技术栈**：SSH 跳板、配置下发、用量上报
- **状态 / 最近验证**：试点中 / 2026-11-01（零用量手动上报）
- **原文**：[830-mac mini4 management windows codex/000-README.md](830-mac%20mini4%20management%20windows%20codex/000-README.md)
- **边界**：自动上报、非零用量、跨平台与跨网均未验收；属设计参考而非成熟方案
