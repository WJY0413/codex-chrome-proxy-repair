# Codex Chrome Proxy Repair

一个面向 Codex 的排障 skill，覆盖 Windows Chrome/CUA 连接的两类独立故障：

- CUA 子进程没有继承正确代理，导致网络请求持续超时。
- Windows 工作目录为 junction，原生权限配置路径检查误判为符号链接。

核心原则：从使用者的实际进程、Windows 代理设置和正在运行的代理客户端发现正确代理；不写死 IP 或端口。TUN 不是唯一方案。权限错误不能靠绕过安全检查解决。

## 安装和使用

把 `skills/codex-chrome-proxy-repair` 文件夹复制到当前 Codex home 的 `skills` 目录（优先使用 `CODEX_HOME`，未设置时为用户目录下 `.codex`）。若目标已存在，先比较并备份，不要直接覆盖。

在 Codex 中输入：

> 使用 $codex-chrome-proxy-repair 排查 Chrome 连接失败，发现本机实际代理，修复后验证页面读取和可逆点击。

[阅读 skill](skills/codex-chrome-proxy-repair/SKILL.md) · [技术说明与公开线索](skills/codex-chrome-proxy-repair/references/repair-details.md)

## 验证范围

2026-10-09 的 Windows 实例已通过官方 Chrome 工具完成标签枚举、原失败页面读取、菜单展开和恢复。启动目录调整是针对原生路径判断缺陷的 workaround；运行时更新可能覆盖本地补丁。本仓库不提供自动修改权限、批量杀进程或一键套用私人代理配置的脚本，也不保证修复所有连接故障。

这是社区排障经验，不是 OpenAI 官方产品。公开包只含通用操作指南，不含实际使用者的代理地址、凭据、私人目录或浏览记录。
