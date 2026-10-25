# Socool 浏览器 · SocoolAI

Socool 是内置 AI 智能体的浏览器，支持 Windows、macOS、UOS / deepin，另有安卓手机 App。
内置的 **SocoolAI（小酷AI）** 可以总结、翻译网页，替你操作网页、按规则审批待办工单，生成 Word / Excel / PPT，
运行定时任务和后台任务，并通过 MCP 连接外部系统，通过 Agent Skills 安装各种 skill。

这个仓库用来发布安装包、SocoolAI 模块更新和智能体广场。

## 下载

到 **[Releases → 最新版本](https://github.com/SocoolBrowser/SocoolAI/releases/latest)** 下载：

| 设备 | 文件 |
|---|---|
| Windows 10（1903 及以上）/ 11 | `Socool-…-win64-setup.exe`（安装版），或 `…-win64-portable.zip`（免安装便携版） |
| 旧版 Windows 10（打开时提示“找不到 icu.dll”） | `Socool-2.x.122-win64-setup.exe` |
| Mac（Apple 芯片：M1 / M2 / M3 / M4） | `Socool-…-macOS-AppleSilicon.dmg` |
| Mac（Intel 芯片，macOS 13 及以上） | `Socool-…-macOS-Intel.dmg` |
| Mac（Intel 芯片，macOS 11 Big Sur / 12 Monterey） | `Socool-…-macOS-Intel-macOS11.dmg` |
| UOS / deepin（ARM） | `socool_…_arm64.deb` |
| 安卓手机（SocoolAI App） | `SocoolAI-….apk` |

Mac 第一次打开时如果提示“无法验证开发者”：在“访达”中按住 Control 点 Socool，选“打开”。

## 主要功能

- **SocoolAI 智能体**：网页总结、翻译、网页操作（真实鼠标键盘、框架和网页组件、上传、截图）、工单处理、编程助手、办公文档
- **后台任务和定时任务**：几个任务同时进行；每天、工作日、每周定时运行任何智能体
- **MCP**：连接数据库、企业网盘、项目管理、OA 接口等外部系统
- **智能体广场**：按 [Agent Skills](https://agentskills.io) 开放标准安装、分享 skill
- **手机 App**：同一 Wi-Fi 下用手机向电脑上的 SocoolAI 提问、操作网页、互传文件
- 支持国内外大模型（DeepSeek、通义千问、豆包、Kimi、智谱、混元、OpenAI、Claude、Gemini）和本地模型（Ollama、LM Studio），API Key 只保存在本机

## 开发 skill 和智能体

见 [SocoolAI 智能体与 skill 开发规范](docs/socoolai-dev-guide.md)。想把自己的 skill 放进智能体广场，欢迎提交 [Issue](https://github.com/SocoolBrowser/SocoolAI/issues)。

## 这个仓库的 Releases

| Release | 内容 |
|---|---|
| `v2.5.134` 等 | Socool 各平台安装包 |
| `ai-module` | SocoolAI 模块更新（官方签名，Socool 中 设置 → 更新 自动检查） |
| `plaza` | 智能体广场：目录 `index.json` 和官方签名的 skill |
