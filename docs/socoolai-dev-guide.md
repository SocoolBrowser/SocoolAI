# SocoolAI 智能体与 skill 开发规范

版本 2.1（适用于 Socool 2.5 及以上，本体接口版本 3）

SocoolAI（中文名：小酷AI）的第三方扩展遵循 **[Agent Skills 开放标准](https://agentskills.io/specification)**。按这份规范写的 skill：

- 在 Socool 里可以作为智能体、工单技能，或任何智能体都能按需使用的知识和流程；
- 在其它支持 Agent Skills 的工具（Claude、各类 AI 编程工具等）里也能直接使用；
- 网上现成的 skill 也能直接装进 Socool。

这份规范面向两类人：**第三方开发者**（编写、签名、分享 skill），**Socool 维护者**（维护智能体广场、发布 SocoolAI 模块）。

---

## 1. 基本概念

### 1.1 skill 是什么

一个 skill 就是一个文件夹，至少包含一个 `SKILL.md`：

```
mail-check/
├── SKILL.md          # 必须：YAML 头（name、description……）+ Markdown 说明
├── references/       # 可选：参考资料，需要时才读取
├── scripts/          # 可选：可以运行的脚本
├── assets/           # 可选：模板、数据等
└── socoolai-signature.json   # 可选：SocoolAI 签名（见第 7 节）
```

AI 使用 skill 的方式是 **渐进加载**：平时只看每个 skill 的名称和说明；遇到相关任务时读取完整的 `SKILL.md`；说明里提到参考文件时再去读那个文件。所以可以装很多 skill，而不会占用太多上下文。

### 1.2 在 SocoolAI 中的三种用法

| 用法 | 怎么声明 | 效果 |
|---|---|---|
| **普通 skill**（默认） | 什么都不用加 | 所有智能体都能看到它的名称和说明，遇到相关任务时用 `skill_open` 读取并照做。例如“商务邮件写作”“公司报销制度” |
| **智能体** | `metadata` 中 `socoolai-agent: "true"` | 智能体菜单中多一个智能体，提示词就是 `SKILL.md` 的正文。例如“邮件漏处理检查”“合同审阅” |
| **工单技能** | `metadata` 中 `socoolai-kind: "ticket"` | 设置 → 工单技能中多一个技能，由“📋 工单处理”智能体按规则审批待办 |

### 1.3 skill 和 SocoolAI 模块的区别

| | skill | SocoolAI 模块（`.sam`） |
|---|---|---|
| 内容 | 说明文字、参考资料、可选的脚本 | SocoolAI 自己的程序代码（界面、工具、读网页的方法） |
| 谁能发布 | 任何人 | 只有 Socool 官方 |
| 签名 | 可以不签名（显示“未验证来源”），也可以用开发者自己的钥匙签名；官方审核后由官方签名 | **必须官方签名**，否则不安装 |
| 安装方式 | 设置 → 广场 | 设置 → 更新 |

skill 的说明文字不是代码，智能体能做的事完全由 SocoolAI 已有的工具决定，并受用户在“权限与偏好”中的确认设置约束。skill 自带的脚本是代码，所以默认**不运行**，见第 6 节。

### 1.4 信任等级

| 标记 | 含义 |
|---|---|
| ✅ 官方 | 用 Socool 官方钥匙签名，内容经过官方审核 |
| 🔏 开发者签名 | 用开发者自己的钥匙签名：确认文件没被改过；以后的更新必须用同一把钥匙签名，否则警告“可能不是同一个作者” |
| ⚠️ 未验证来源 | 没有签名（网上大部分 skill 都是这样）。安装前必须勾选“我已查看上面的内容” |
| ❌ 签名不正确 | 签名后文件被改过，**拒绝安装** |

---

## 2. 快速开始：五分钟做一个 skill

1. 新建文件夹 `weekly-report`，在里面新建 `SKILL.md`：

```markdown
---
name: weekly-report
description: 把零散的工作记录整理成格式规范的周报、日报或月报。用户要写周报、日报、月报、工作总结时使用。
metadata:
  socoolai-agent: "true"
  socoolai-name: "周报助手"
  socoolai-icon: "🗓️"
  socoolai-starters: "把下面的记录整理成周报：|帮我写一份月度工作总结"
  version: "1.0.0"
  author: "示例公司 IT 部"
---
# 周报助手

你是周报助手。把用户给出的工作记录整理成：一、本周完成；二、问题与风险；三、下周计划。
不编造记录中没有的内容；信息不足时在最后列出需要用户补充的问题。
```

2. 在 Socool 中打开小酷AI → 设置 → **广场** → **从文件夹安装**，选择这个文件夹。
3. 在智能体菜单中选“周报助手”试用。修改后把 `version` 改大，再安装一次就是更新。
4. 分享：把文件夹压缩成 zip 发给同事；或者签名后再发（第 7 节）；或者提交到智能体广场（第 8 节）。

也可以先在 SocoolAI 中创建智能体、工单技能，调好后用 **广场 → 导出为 skill** 生成 zip。

---

## 3. SKILL.md 格式

`SKILL.md` 由 YAML 头（两行 `---` 之间）和 Markdown 正文组成，文件用 UTF-8 编码。

### 3.1 标准字段（Agent Skills 规范）

| 字段 | 必填 | 说明 |
|---|---|---|
| `name` | 是 | 1–64 个字符，只能用小写英文字母、数字和连字符 `-`；不能以连字符开头或结尾，不能有连续的 `--`；**必须和文件夹名一致**。例如 `mail-check` |
| `description` | 是 | 1–1024 字。写清**做什么**和**什么时候用**，带上用户可能说的关键词。AI 靠它决定什么时候使用这个 skill |
| `license` | | 许可证名称，或包内许可证文件的名字 |
| `compatibility` | | 运行环境要求（最多 500 字），例如“需要 Python 3 和 pandas”。安装时会显示给用户 |
| `allowed-tools` | | 用空格分隔的工具名，声明这个 skill 要用的工具（见第 4 节）。在 SocoolAI 中它只是**申请的权限**：安装时显示给用户，不会因此跳过任何确认 |
| `metadata` | | 键值表（值都是字符串）。SocoolAI 的扩展字段都写在这里（3.2 节） |

### 3.2 SocoolAI 扩展字段（写在 `metadata` 中）

其它工具会忽略这些字段，不影响 skill 的通用性。所有值都写成带引号的字符串。

| 字段 | 说明 |
|---|---|
| `socoolai-agent` | `"true"`：作为智能体安装，出现在智能体菜单中 |
| `socoolai-kind` | `"ticket"`：作为工单技能安装（正文格式见 5.3 节） |
| `socoolai-name` | 显示名称（最多 30 字），例如 `"邮件漏处理检查"`。不写时用正文的第一个 `# 标题`，再没有就用 `name` |
| `socoolai-icon` | 一个表情符号 |
| `socoolai-starters` | 智能体的快捷问题，用 `|` 分隔，最多 6 个 |
| `socoolai-auto-page` | `"true"`：每次提问都自动附上当前网页的正文（例如“合同审阅”） |
| `socoolai-max-per-run` | 工单技能每次最多处理的条数（1–200，默认 20） |
| `socoolai-category` | 智能体广场中的分类，例如 `"办公"`、`"写作"`、`"工单技能"` |
| `version` | 版本号，例如 `"1.0.0"`。更新时必须变大，广场据此提示“更新” |
| `author` | 作者或单位 |

### 3.3 正文

正文是给 AI 看的说明，没有格式限制。建议：

- 第一行用 `# 标题`；
- 先写目标，再写分步骤的做法，最后写输出格式；
- 正文控制在 500 行以内（大约 5000 个 token），详细资料放到 `references/` 中，用相对路径引用，例如 `见 [模板](references/templates.md)`。引用只放一层，不要让参考文件再引用别的文件。

---

## 4. 可以使用的工具

在 `allowed-tools` 中声明，在正文中可以直接写工具名，告诉 AI 什么时候用哪个。

| 工具 | 作用 | 用户看到的权限 |
|---|---|---|
| `page_read` | 读取当前标签页：网址、标题、正文、可操作元素（`[编号] 类型 文字`）。会读出同源框架和网页组件里的内容；只有图标的元素显示为它的提示文字，例如 `[已回复]`；横排的一行读成 `图标 | 发件人 | 标题 | 时间` | 读取网页 |
| `page_reload` | 刷新当前网页 | 刷新网页 |
| `page_click` / `page_type` / `page_select` / `page_scroll` | 点击（真实鼠标）、输入、选择下拉框、滚动 | 操作网页 |
| `page_navigate` / `page_back` | 打开网址、返回 | 操作网页 |
| `page_tabs` / `page_switch_tab` / `page_close_tab` | 查看、切换、关闭标签页（点击后打开新标签页时会自动切换过去） | 操作网页 |
| `page_upload` | 把工作目录中的文件上传到网页 | 操作网页 |
| `page_screenshot` | 网页截图（用户的模型勾选了“能看图”时才有） | 操作网页 |
| `skill_get` / `ticket_decide` | 读取工单技能；处理每条工单前记录决定（通过 / 驳回 / 交给人工） | 工单技能 |
| `list_files` / `read_file` / `search_files` / `write_file` / `edit_file` / `run_command` | 工作目录中的文件和命令 | 读写文件、运行命令（高风险） |
| `office_create` | 生成 Word / Excel / PowerPoint 文件（docx 用 Markdown；xlsx 用二维数组，支持公式；pptx 用页面列表或 Markdown） | 生成文档 |
| `delegate_tasks` | 把任务拆成最多 5 个子任务，在后台并行处理后汇总 | 后台任务 |
| `task_create` | 创建定时任务（用户确认后保存） | 后台任务 |
| `mcp_<服务器>_<工具>` | 用户添加的 MCP 服务器提供的工具（名称因用户而异，不要写进 `allowed-tools`，在说明中按用途描述即可） | 由用户在 设置 → MCP 中授权 |
| `skill_open` | 读取已安装 skill 的 `SKILL.md` 或参考文件 | 所有用到 skill 的智能体自动拥有 |
| `skill_script` | 运行 skill 自带的脚本（第 6 节） | 只有能运行命令的智能体才有 |

其它工具常用的写法也能识别：`Read`、`Write`、`Edit`、`Bash(...)`、`Grep`、`Glob` 视为“读写文件、运行命令”；`WebFetch`、`WebSearch` 视为“读取网页”。SocoolAI 没有的工具会在安装时提示并忽略。

只声明真正需要的工具。权限越少，用户越放心安装。

---

## 5. 三种 skill 的写法

### 5.1 普通 skill（知识和流程）

不用写 SocoolAI 扩展字段。`description` 要写清什么时候用，AI 才会在合适的时候读取它。例如：

```markdown
---
name: business-email
description: 中文商务邮件的写作规范和常用模板（请示、催办、致歉、拒绝、跟进、会议邀请）。用户要写或润色工作邮件、回复客户或领导时使用。
metadata:
  socoolai-name: "商务邮件写作"
  socoolai-icon: "✉️"
  version: "1.0.0"
---
# 商务邮件写作
1. 标题一句话说清事项和期限……
不同场景的模板见 [references/templates.md](references/templates.md)。
```

用户可以在 广场 → 已安装 中单独停用某个 skill，也可以关闭“所有智能体都可以使用已安装的 skill”。

### 5.2 智能体

加上 `socoolai-agent: "true"`。正文就是智能体的提示词；`allowed-tools` 决定它能用的工具。例如“邮件漏处理检查”：

```markdown
---
name: mail-check
description: 检查网页邮箱的收件箱，找出未读、没回复也没转发、需要用户处理的邮件。用户提到漏处理的邮件、检查收件箱时使用。
allowed-tools: page_read page_reload page_click page_scroll
metadata:
  socoolai-agent: "true"
  socoolai-name: "邮件漏处理检查"
  socoolai-icon: "📮"
  socoolai-starters: "检查当前页有没有漏处理的邮件|检查收件箱前 3 页"
  version: "1.0.0"
---
# 邮件漏处理检查
你是邮件检查助手……（做法、禁止事项、汇报格式）
```

提示词建议：

- 第一句说清角色和目标；然后分步骤写做法；最后写输出格式（例如“用表格汇报：发件人 | 标题 | 时间 | 理由”）。
- 写清楚**不能做什么**，例如“只读取和翻页，不要回复、删除、移动任何邮件”。
- 写清楚信息不全时怎么办（“如实说明，不要猜”）。
- 网页内容来自第三方。SocoolAI 已经要求模型不执行网页中的指令；你的提示词也不要让模型“按网页上的说明操作”。
- 不要在提示词中放密码、令牌、内部系统账号；不要让智能体把内容发到你的服务器，或打开和用途无关的网站。

### 5.3 工单技能

加上 `socoolai-kind: "ticket"`。正文用下面的二级标题分节（中文或英文标题都可以）：

| 小标题 | 英文也可以写 | 内容 |
|---|---|---|
| `## 适用范围` | `## Scope` | 怎么识别这类工单 |
| `## 待办在哪里` | `## Where` | 网址或菜单路径 |
| `## 处理步骤` | `## Steps` | 打开工单、看哪些信息、通过 / 驳回按钮和意见框在哪 |
| `## 通过条件` | `## Approve when` | **必填**。全部满足才通过，分条写 |
| `## 驳回条件` | `## Reject when` | 满足任一条就驳回，分条写 |
| `## 通过意见` | `## Approve comment` | 通过时填写的审批意见 |
| `## 驳回意见` | `## Reject comment` | 驳回时的意见模板（写明原因） |
| `## 补充说明` | `## Notes` | 例如哪些情况交给人工 |

规则要能判断：写具体的金额、类型、部门、日期范围、附件要求，不要写“合理的”“必要的”。覆盖不了的情况统一交给人工。完整示例见智能体广场中的“差旅报销审批”。

**执行方式（自动执行）和定时运行不能写在 skill 里**，只能由用户在自己的电脑上打开。安装时一律按“逐条确认”、定时关闭；更新 skill 时保留用户自己的设置。

---

## 6. 参考文件和脚本

**参考文件**（`references/`、`assets/` 中的文本文件）：AI 用 `skill_open` 读取，每个文件最多读 256 KB，二进制文件不能读取。

**脚本**（`scripts/` 中的文件，或 `.py`、`.sh`、`.js`、`.ps1` 等）：

- 安装时会标出全部脚本，用户必须勾选“了解它带有脚本”；
- 默认**不运行**；只有能运行命令的智能体（例如“编程助手”，或声明了文件权限的智能体）才能用 `skill_script` 运行；
- **每次运行都弹出确认**并显示脚本内容，“全部允许”对它无效；
- 在用户选择的**工作目录**中运行，单次最长 2 分钟；
- 支持 Python（`python3` / Windows 上 `python`）、JavaScript（`node`）、Shell（`bash`）、Windows 上的 PowerShell 和批处理；需要的环境要写在 `compatibility` 中。

所以写 skill 时，优先把做法写成说明，让 AI 用已有的工具完成；只在确实需要时才用脚本。

---

## 7. 签名（推荐）

签名证明两件事：文件没被改过；更新来自同一个作者。工具是仓库中的 `scripts/socoolai-sign.py`，只需要 Python 3。

```bash
# 1. 生成你自己的钥匙（只做一次；私钥文件妥善备份，不要发给别人、不要上传）
python3 socoolai-sign.py keygen --out my-signing.key

# 2. 签名（在 skill 文件夹中生成 socoolai-signature.json；之后改了任何文件都要重新签名）
python3 socoolai-sign.py skill mail-check --key my-signing.key

# 3. 检查
python3 socoolai-sign.py verify mail-check
```

签名文件 `socoolai-signature.json` 的格式：

```json
{ "alg": "ed25519", "publicKey": "<公钥 base64>", "sig": "<签名 base64>" }
```

签名的内容是：

```
socoolai-skill
<sha256>  <相对路径>
<sha256>  <相对路径>
...
```

第一行是 `socoolai-skill`，后面每个文件一行：小写十六进制的 SHA-256、两个空格、用 `/` 分隔的相对路径。文件按路径排序，不包括签名文件本身、符号链接、`.DS_Store` 和 `__MACOSX`。算法是 Ed25519（RFC 8032）。

其它工具会忽略这个文件，所以签过名的 skill 照样能在其它工具里使用。

把你的**公钥**和**钥匙编号**（keygen 时显示）公布在主页上，方便用户核对。私钥丢了就不能再用同一身份发布更新，用户更新时会看到“签名者不同”的警告。

---

## 8. 安装来源和智能体广场

用户可以从这些地方安装：

- **智能体广场**（设置 → 广场）；
- **zip 文件**（`.zip`、`.skill`），可以包含一个或多个 skill；包含多个时让用户选择；
- **文件夹**（里面有 `SKILL.md`）；
- **网址**：zip 的地址、`SKILL.md` 的地址，或者 **GitHub 地址**：
  - 仓库：`https://github.com/<用户>/<仓库>`
  - 目录：`https://github.com/<用户>/<仓库>/tree/<分支>/<目录>`
  - 文件：`https://github.com/<用户>/<仓库>/blob/<分支>/<目录>/SKILL.md`

  SocoolAI 会下载整个仓库，找出对应的 skill。

**智能体广场**是一个目录文件 `index.json`，加上官方签名的 skill（每个一个 zip），发布在公开仓库 [SocoolBrowser/SocoolAI](https://github.com/SocoolBrowser/SocoolAI) 的 Releases（`plaza`）中。

```json
{
  "format": "socoolai-plaza",
  "updated": "2026-10-25 20:00",
  "skills": [
    { "name": "mail-check", "title": "邮件漏处理检查", "description": "…", "version": "1.0.0",
      "author": "Socool", "icon": "📮", "category": "办公", "url": "mail-check.zip" }
  ]
}
```

`url` 可以是相对 `index.json` 的路径，也可以是完整网址。目录本身不签名，信任等级以下载的 skill 的签名为准。

**公司内部广场**：把 `index.json` 和 skill 的 zip 放到内网服务器上，员工在 广场 → 广场地址 → 修改 中填写内网地址。

**提交到官方广场**：把 skill（zip 或 GitHub 地址）发给 Socool 维护者，或在 [SocoolBrowser/SocoolAI](https://github.com/SocoolBrowser/SocoolAI/issues) 提交 Issue。维护者审核以下几点：

1. 格式符合 Agent Skills 规范：可以用官方工具检查，`skills-ref validate ./my-skill`；
2. 内容：没有诱导泄露数据，没有和用途无关的网站，权限够用不多给，脚本可读、可解释；
3. 用“从文件夹安装”实际试用。

审核通过后，维护者把 skill 放进源码仓库的 `plaza/skills/<name>/`，运行发布流程（选“只发布广场”），由官方钥匙签名，在广场中显示 ✅ 官方。

---

## 9. 维护者一次性设置

1. **生成官方签名钥匙**（在自己的电脑上，只做一次）：

   ```bash
   python3 scripts/socoolai-sign.py keygen --out ~/socoolai-official.key
   ```

   - 把显示的**公钥**（一行 base64）写入 `packaging/socoolai-signing.pub`，提交到仓库。之后编译的 Socool 都内置这个公钥。
   - 把显示的**私钥**那一行添加到源码仓库的 Settings → Secrets and variables → Actions → New repository secret，名称为 `SOCOOLAI_SIGNING_KEY`。
   - 私钥文件离线备份，不要上传到任何仓库（`.gitignore` 已经排除了 `*.key`）。

2. **发布仓库**：公开仓库 `SocoolBrowser/SocoolAI`，放安装包、SocoolAI 模块和智能体广场。源码仓库可以继续私有。

3. **发布令牌**：GitHub → Settings → Developer settings → Fine-grained tokens → Generate new token：
   - 用发布仓库所在的账号创建；Repository access 只选 `SocoolBrowser/SocoolAI`；
   - Permissions → Contents：Read and write。

   把令牌添加到源码仓库的 Secrets，名称为 `UPDATES_REPO_TOKEN`。

4. 在源码仓库的 Variables 中添加 `SOCOOLAI_UPDATES_REPO`，值为 `SocoolBrowser/SocoolAI`（安装包、模块、广场都发布到这里）。

5. 运行一次“发布小酷AI 模块和智能体广场”流程，选“模块和广场”。

国内访问 GitHub 慢时，可以把发布仓库的 Releases 文件同步到 Gitee 或公司服务器，在 设置 → 广场 / 更新 中把地址改成镜像地址。镜像改不了签名过的内容。

---

## 10. 本体接口版本和 SocoolAI 模块（官方）

**本体接口版本**：

| 接口版本 | Socool | 新增 |
|---|---|---|
| 1 | 2.3 | 读网页、点击、输入、跳转、文件、命令、工单技能 |
| 2 | 2.4 | 真实鼠标键盘、读取框架和组件、图标提示、标签页、上传、截图、刷新、SocoolAI 模块、Agent Skills |
| 3 | 2.5 | 后台任务和并行子任务、定时任务、MCP、生成 Word / Excel / PPT |

skill 需要较新的 Socool 时，在 `compatibility` 中写明，例如 `"需要 Socool 2.4 及以上"`。

**SocoolAI 模块**就是 `resources/pages/` 中的 `assistant.html`、`assistant.js`、`assistant.css`、`qrcode.js`、`module.json`。只改这几个文件时，可以作为模块更新发布，用户不用重装 Socool。

- `module.json` 中的 `version` 必须比用户已有的版本大：可能是 Socool 安装包自带的版本，也可能是已经更新过的版本。`minHost` 是需要的本体接口版本。
- 新增了 C++ 中的 Host 操作（例如新的 `page.*`）时，要把 `src/aimodule.h` 中的 `kHostApi` 加 1，模块的 `minHost` 也改成这个值。这种更新必须先发布新的 Socool 安装包。
- 模块文件 `.sam` 的结构：
  - 第一行是 JSON 头：`{"format":"socoolai-module","version","minHost","sha256","size","keyId","sig"}`；
  - 后面是 Qt 资源包：rcc 格式 2，由 `scripts/socoolai_rcc.py` 生成，不需要安装 Qt。
  - 签名的内容是 `"socoolai-module\n{version}\n{minHost}\n{sha256}\n{size}"`。
- Socool 启动时验证签名、版本和接口版本，通过后把资源包注册到 `qrc:/ai/`；任何一项不通过就用软件自带的版本。用户也可以在 设置 → 更新 中“恢复自带版本”。

发布步骤：

1. 修改 `resources/pages/` 中的文件，把 `module.json` 的 `version` 改大，提交；
2. GitHub Actions → “发布小酷AI 模块和智能体广场” → 选“只发布模块”，填写更新说明；
3. 用户在 设置 → 更新 → 检查更新 中安装，几秒钟就生效。

---

## 11. 安装时强制执行的安全规则

无论 skill 里怎么写，安装时都会：

- 显示它需要的全部权限、`SKILL.md` 中提到的网站、全部文件（标出脚本），并且 `SKILL.md` 全文可以查看；
- 签名不正确的拒绝安装；未签名、换了签名者、带脚本的，必须勾选确认；
- 工单技能一律按“逐条确认”安装、定时关闭；
- 智能体的编号自动加前缀，不能覆盖内置智能体，也不会和用户自己的智能体冲突；
- 文件路径不能超出 skill 文件夹；zip 中的符号链接、加密文件、超大文件（单个 20 MB、合计 150 MB）都会拒绝；
- 不导入、也不导出 API Key、模型设置、对话历史、处理记录；
- 卸载时删除 skill 文件夹以及它带来的智能体、工单技能（处理记录保留）。

---

## 12. 常见问题

**网上下载的 skill 能直接用吗？** 能。按 Agent Skills 规范制作的 skill 都能安装，作为普通 skill 被所有智能体按需使用。其中的说明和参考资料可以直接用；脚本要在能运行命令的智能体中、经你确认后才运行，并且需要电脑上有对应的运行环境。

**为 SocoolAI 写的 skill 在其它工具里能用吗？** 能。SocoolAI 的扩展都写在 `metadata` 和签名文件里，其它工具会忽略。只用到 `page_*` 等 SocoolAI 工具的 skill，在没有浏览器操作能力的工具里只能当作说明来读。

**没有签名的 skill 安全吗？** 说明文字不是代码，智能体只能用 SocoolAI 已有的工具，并受你的确认设置约束。主要风险是说明诱导 AI 做你不想要的事，例如把网页内容发到某个网站。所以安装前会列出权限和提到的网站，内容全部可以查看。只安装信任来源的 skill；涉及审批、付款的场景，保持“逐条确认”。

**我改了已安装的智能体，更新 skill 会覆盖吗？** 会覆盖提示词和工具，保留你为它选的模型；工单技能保留执行方式、上限和定时。想长期保留自己的修改，可以导出成自己的 skill（换一个 `name`）。

**企业内部怎么统一分发？** 用自己的钥匙签名，搭一个内部广场（第 8 节），让员工把广场地址改成内网地址。
