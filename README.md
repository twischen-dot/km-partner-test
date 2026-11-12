# KM 测试包（公开化预演 · 只读）

> 版本：rehearsal-20261112 · 冻结日期：2026-11-12
> 性质：按公开发布标准构建的**脱敏只读测试包**，供一位真实合作伙伴做检索闭环测试。
> 本包不是原始知识库；不包含脚本、日志、JSON 证据和图片附件。

## 包含范围

| 项目 | 主题 | 状态 |
|---|---|---|
| 710-How to check the Hdisk error | Mac 存储故障检查与修复（APFS 实录） | 已验证（内置 SSD 案例） |
| 830-mac mini4 management windows codex | Mac mini 集中管理多台 Windows Codex | 试点中，跨网未验证 |
| 880-how to control the local windows and mac | 局域网 Windows/Mac 接入与长期维护 | 局域网已验证，跨网未验证 |
| 900-windows computer use codex and claude | Windows 本机部署 Codex 与 Claude | 单机已验证，新机复现未验证 |

## 阅读方式

1. 先读根目录 `AGENTS.md`（检索协议）和 `INDEX.md`（方案索引）。
2. 各项目 `000-README.md` 为入口；占位符（如 `<伙伴A>`、`<目标机>`、`<公司域名1>`）为脱敏替换，请勿推断还原。
3. 标注「未包含在本测试包」的引用指向原库脚本或证据附件，不在本包内。
4. 命中方案 ≠ 可直接实施；先按 `AGENTS.md` 的答复格式做适配检查，经方案维护者确认后再动手。

## 版本与完整性

文件级 SHA-256 清单见 `MANIFEST.txt`。引用本包内容时请注明版本号 `rehearsal-20261112`。
