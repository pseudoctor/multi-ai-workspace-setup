# Claude Code + Codex + Gemini CLI：Mac 用户完全配置指南

> 从零开始 · 小白友好  三工具共享 Skills · 多 Workspace 隔离 · Token 高效管理

---

## 目录

- [阅读前须知](#阅读前须知)
- [第一章：整体架构](#第一章整体架构)
- [第二章：安装三个工具](#第二章安装三个工具)
- [第三章：配置用户级](#第三章配置用户级全局基础设置)
- [第四章：配置环境变量与别名](#第四章配置环境变量与别名)
- [第五章：创建 Workspace](#第五章创建-workspace)
- [第六章：Skills 管理](#第六章skills-管理)
- [第七章：日常使用](#第七章日常使用)
- [第八章：常见问题](#第八章常见问题)
- [附录：快速参考](#附录快速参考)

---

## 阅读前须知

### 终端是什么，怎么打开

本指南所有操作都在「终端」（Terminal）里完成。终端是一个可以输入文字命令的窗口，你输入命令，电脑执行。

**打开方式：** 按 `Command + 空格`，输入 `Terminal`，回车。或者在访达 → 应用程序 → 实用工具 → 终端。

> 💡 每次打开终端，你会看到一行提示符，类似 `yourname@MacBook ~ %`，这是在等你输入命令。输完命令按回车执行。

### 路径符号说明

| 符号 | 含义 | 实际路径示例 |
|------|------|-------------|
| `~` | 你的用户主目录，即 `/Users/你的用户名/` | `/Users/yourname/` |
| `~/workspace-dev` | 主目录下的 workspace-dev 文件夹 | `/Users/yourname/workspace-dev/` |
| `~/.zshrc` | 主目录下的隐藏文件 .zshrc | `/Users/yourname/.zshrc` |

> 💡 以点（`.`）开头的文件和文件夹是隐藏的，在访达里默认看不到，但终端里可以正常访问和操作。

### 命令格式说明

所有需要你输入的命令都显示在代码块里：

```bash
echo hello
```

`#` 开头的行是注释说明，**不需要输入**，其他行复制到终端回车执行即可。

> ⚠️ 每条命令执行完，等看到提示符（`%`）重新出现再输入下一条，不要急着连续输入。

### nano 编辑器使用方法

本指南使用 `nano` 编辑文件，它是 Mac 自带的简单文字编辑器：

1. 运行 `nano 文件路径` 打开编辑器
2. 用方向键移动光标，直接输入内容
3. 编辑完成后按 `Control + X`
4. 提示是否保存，按 `Y`
5. 确认文件名，按回车

---

## 第一章：整体架构

### 1.1 三层结构

整个配置分三层，从外到内分别是：

| 层级 | 位置 | 存放内容 | 影响范围 |
|------|------|----------|----------|
| 用户级 | `~/.claude/`<br>`~/.codex/`<br>`~/.gemini/` | 跨所有场景都需要的个人偏好，内容要尽量少 | 你 Mac 上所有项目 |
| Workspace 级 | `~/workspace-xxx/.claude/`<br>`~/workspace-xxx/.codex/`<br>`~/workspace-xxx/.gemini/` | 该工作场景专用的 Skills 和规范，三工具共享 | 该 Workspace 下所有项目 |
| Project 级 | 项目文件夹根目录下 | 该项目特有的说明 | 仅该项目 |

> 💡 **Skills** 是告诉 AI 工具「如何完成某类任务」的文件。三个工具都支持同一格式的 Skills，所以可以共享一份，不用重复写三遍。

### 1.2 为什么要这样分层（Token 节省原理）

三个工具启动时会把配置内容加载进「上下文」（AI 的工作记忆）。加载的内容越多，消耗的 token 越多，费用越高，响应也可能变慢。

分层之后：
- **用户级** 只放极少量内容，每次启动消耗极少 token
- **Workspace 级** 只在对应场景下加载，写作场景不会加载开发工具，开发场景不会加载写作工具
- **Skills** 采用「用时才加载」机制：启动时只读取名称和描述（几十个 token），只有真正用到时才加载完整内容

### 1.3 目录结构总览

```
~/                                  # 你的主目录
├── .claude/                        # 用户级：Claude Code 全局配置
├── .codex/                         # 用户级：Codex 全局配置
├── .gemini/                        # 用户级：Gemini CLI 全局配置
│
├── workspace-dev/                  # 开发场景 Workspace
│   ├── .claude/
│   │   ├── CLAUDE.md               # 开发场景指令
│   │   └── skills/                 # ← Skills 真实文件放这里
│   ├── .codex/
│   │   ├── AGENTS.md
│   │   └── skills -> ../.claude/skills/   # 软链接（快捷方式）
│   ├── .gemini/
│   │   ├── GEMINI.md
│   │   └── skills -> ../.claude/skills/   # 软链接（快捷方式）
│   └── projects/
│       ├── project-a/  (.git)
│       └── project-b/  (.git)
│
├── workspace-docs/                 # 写作场景 Workspace
│   └── ...（结构相同）
│
└── workspace-other/                # 其他场景 Workspace
    └── ...（结构相同）
```

### 1.4 三工具查找机制对比

| 工具 | 指令文件 | 查找范围 | 有无 `.git` 边界限制 |
|------|----------|----------|---------------------|
| Claude Code | `CLAUDE.md` | 从当前目录一直向上，**无边界** | 无 |
| Codex | `AGENTS.md` | 全局 `~/.codex/` + 从 `.git` 根向下到当前目录 | **有** |
| Gemini CLI | `GEMINI.md` | 从当前目录向上到 `.git` 根为止 | **有** |

> ⚠️ Codex 和 Gemini CLI 不会越过 `.git` 边界向上查找。这是 Workspace 级配置需要借助环境变量解决的根本原因。

> ✅ Skills 使用 `.agents/skills/` 跨工具标准目录，三个工具均支持，同一份 `SKILL.md` 无需任何修改即可三端通用。

---

## 第二章：安装三个工具

> ⚠️ 每一步操作完请等命令执行完毕（看到 `%` 提示符重新出现），再继续下一步。

### 2.1 安装 Claude Code

**1. 安装：**

```bash
npm install -g @anthropic-ai/claude-code
```

**2. 验证安装：**

```bash
claude --version
```

**3. 获取 API Key：**

- 打开 [https://console.anthropic.com](https://console.anthropic.com)
- 登录你的 Anthropic 账号
- 点击左侧菜单「API Keys」→「Create Key」
- 复制生成的 key（以 `sk-ant-` 开头），**只显示一次，请妥善保存**

**4. 设置 API Key：**

```bash
# 将 YOUR_KEY_HERE 替换成你刚复制的真实 key
echo 'export ANTHROPIC_API_KEY="YOUR_KEY_HERE"' >> ~/.zshrc
source ~/.zshrc
```

**5. 验证 key 已生效：**

```bash
echo $ANTHROPIC_API_KEY
# 显示你的 key 说明成功
```

### 2.2 安装 Codex

**1. 安装：**

```bash
npm install -g @openai/codex
```

**2. 验证安装：**

```bash
codex --version
```

**3. 获取 OpenAI API Key：**

- 打开 [https://platform.openai.com/api-keys](https://platform.openai.com/api-keys)
- 登录你的 OpenAI 账号
- 点击「Create new secret key」
- 复制 key（以 `sk-` 开头）

**4. 设置 API Key：**

```bash
echo 'export OPENAI_API_KEY="YOUR_KEY_HERE"' >> ~/.zshrc
source ~/.zshrc
```

### 2.3 安装 Gemini CLI

**1. 安装：**

```bash
npm install -g @google/gemini-cli
```

**2. 验证安装：**

```bash
gemini --version
```

**3. 获取 Gemini API Key：**

- 打开 [https://aistudio.google.com/apikey](https://aistudio.google.com/apikey)
- 登录你的 Google 账号
- 点击「Create API key」
- 复制 key（以 `AIza` 开头）

**4. 设置 API Key：**

```bash
echo 'export GEMINI_API_KEY="YOUR_KEY_HERE"' >> ~/.zshrc
source ~/.zshrc
```

**5. 首次启动（会自动创建 `~/.gemini/` 目录）：**

```bash
gemini
# 看到欢迎界面后输入 /quit 退出即可
```

> ✅ 三个工具都安装完成后，可以继续下一章。

---

## 第三章：配置用户级（全局基础设置）

用户级配置对你 Mac 上的所有项目生效。内容要**极度精简**，只放真正每个场景都需要的偏好，其他内容留给 Workspace 级。

### 3.1 创建用户级目录

```bash
mkdir -p ~/.claude/skills
mkdir -p ~/.claude/commands
mkdir -p ~/.claude/agents
mkdir -p ~/.codex
# ~/.gemini/ 已在上一章首次启动时自动创建
```

### 3.2 创建用户级指令文件

**Claude Code 用户级指令：**

```bash
nano ~/.claude/CLAUDE.md
```

输入以下内容（根据自己喜好修改）：

```markdown
## 我的个人偏好
- 回复使用简体中文
- 代码注释使用中文
- Git 提交信息使用英文

## 注意
- 修改代码前先告诉我你打算怎么做
- 不要一次性修改太多文件
```

**Codex 用户级指令：**

```bash
nano ~/.codex/AGENTS.md
```

```markdown
## 我的个人偏好
- 回复使用简体中文
- Git 提交信息使用英文
- 修改前告知计划
```

**Gemini CLI 用户级指令：**

```bash
nano ~/.gemini/GEMINI.md
```

```markdown
## 我的个人偏好
- 回复使用简体中文
- 遵循项目现有代码风格
- 修改前告知计划
```

> ⚠️ 这三个文件每次启动工具都会**全文加载**。内容越少越好，不要把场景相关的规范写在这里。

---

## 第四章：配置环境变量与别名

Codex 和 Gemini CLI 需要通过环境变量告诉它们去哪里找 Workspace 级配置。Claude Code 不需要，它会自动向上查找。

### 4.1 编辑 ~/.zshrc

```bash
nano ~/.zshrc
```

用方向键移动到文件**最底部**，添加以下内容：

```bash
# ── AI 工具 Workspace 配置 ──────────────────────────────────────

# Codex 默认使用 workspace-dev（切换时用下面的别名）
export CODEX_HOME="$HOME/workspace-dev/.codex"

# Gemini CLI 默认使用 workspace-dev
export GEMINI_CLI_HOME="$HOME/workspace-dev"

# ── 各 Workspace 的启动别名 ─────────────────────────────────────

# workspace-dev（开发场景）
alias cdx-dev='CODEX_HOME="$HOME/workspace-dev/.codex" codex'
alias gem-dev='GEMINI_CLI_HOME="$HOME/workspace-dev" gemini'

# workspace-docs（写作场景）
alias cdx-docs='CODEX_HOME="$HOME/workspace-docs/.codex" codex'
alias gem-docs='GEMINI_CLI_HOME="$HOME/workspace-docs" gemini'

# workspace-other（其他场景）
alias cdx-other='CODEX_HOME="$HOME/workspace-other/.codex" codex'
alias gem-other='GEMINI_CLI_HOME="$HOME/workspace-other" gemini'
```

编辑完成后：`Control + X` → `Y` → 回车 保存。

### 4.2 使配置生效并验证

```bash
source ~/.zshrc

# 验证环境变量
echo $CODEX_HOME
# 预期输出：/Users/yourname/workspace-dev/.codex

echo $GEMINI_CLI_HOME
# 预期输出：/Users/yourname/workspace-dev
```

> 💡 `$HOME` 是自动代表你主目录路径的变量，等同于 `~`。这样写换电脑也不需要修改配置文件。

> 💡 Claude Code 不需要环境变量。只要在项目目录里启动 `claude`，它会自动向上查找 `.claude/` 目录，天然支持 Workspace 级配置。

---

## 第五章：创建 Workspace

本章以创建三个 Workspace 为例：`workspace-dev`（开发）、`workspace-docs`（写作）、`workspace-other`（其他）。名字可以自定义，但改了之后第四章的环境变量也要对应修改。

### 5.1 创建目录结构

```bash
# workspace-dev
mkdir -p ~/workspace-dev/.claude/skills
mkdir -p ~/workspace-dev/.claude/rules
mkdir -p ~/workspace-dev/.claude/commands
mkdir -p ~/workspace-dev/.codex
mkdir -p ~/workspace-dev/.gemini
mkdir -p ~/workspace-dev/projects

# workspace-docs
mkdir -p ~/workspace-docs/.claude/skills
mkdir -p ~/workspace-docs/.claude/rules
mkdir -p ~/workspace-docs/.codex
mkdir -p ~/workspace-docs/.gemini
mkdir -p ~/workspace-docs/projects

# workspace-other
mkdir -p ~/workspace-other/.claude/skills
mkdir -p ~/workspace-other/.claude/rules
mkdir -p ~/workspace-other/.codex
mkdir -p ~/workspace-other/.gemini
mkdir -p ~/workspace-other/projects
```

验证创建成功：

```bash
ls ~/workspace-dev/
# 预期输出：.claude  .codex  .gemini  projects
```

### 5.2 创建 Skills 软链接（最重要的步骤）

软链接相当于快捷方式。让 Codex 和 Gemini CLI 的 skills 目录都指向 Claude Code 的 skills 目录，这样只维护一份 Skills 文件，三个工具都能看到。

> ⚠️ 软链接命令的两个路径**顺序不能搞反**：第一个是「真实文件的位置」，第二个是「快捷方式放在哪里」。

```bash
# workspace-dev 的软链接
ln -s ~/workspace-dev/.claude/skills ~/workspace-dev/.codex/skills
ln -s ~/workspace-dev/.claude/skills ~/workspace-dev/.gemini/skills

# workspace-docs 的软链接
ln -s ~/workspace-docs/.claude/skills ~/workspace-docs/.codex/skills
ln -s ~/workspace-docs/.claude/skills ~/workspace-docs/.gemini/skills

# workspace-other 的软链接
ln -s ~/workspace-other/.claude/skills ~/workspace-other/.codex/skills
ln -s ~/workspace-other/.claude/skills ~/workspace-other/.gemini/skills
```

**立刻验证软链接是否正确（非常重要）：**

```bash
ls -la ~/workspace-dev/.codex/
ls -la ~/workspace-dev/.gemini/
```

你应该看到类似这样的输出：

```
lrwxr-xr-x  skills -> /Users/yourname/workspace-dev/.claude/skills
```

关键是看到 `skills ->` 后面跟着正确的路径。如果没有 `->` 符号，说明软链接创建失败，需要删掉重建：

```bash
# 删除错误的软链接
rm ~/workspace-dev/.codex/skills
# 重新创建
ln -s ~/workspace-dev/.claude/skills ~/workspace-dev/.codex/skills
```

### 5.3 创建 Workspace 级指令文件

**workspace-dev：**

```bash
nano ~/workspace-dev/.claude/CLAUDE.md
```

```markdown
## 开发场景约定
- 技术栈：TypeScript + Node.js
- 包管理器：pnpm（不要用 npm 或 yarn）
- 测试：修改代码后运行 pnpm test
- 提交前运行 pnpm lint
```

```bash
nano ~/workspace-dev/.codex/AGENTS.md
```

```markdown
## 开发场景约定
- 技术栈：TypeScript + Node.js
- 包管理器：pnpm
- 提交前运行 pnpm test 和 pnpm lint
- 不得修改 .env 文件
```

```bash
nano ~/workspace-dev/.gemini/GEMINI.md
```

```markdown
## 开发场景约定
- 技术栈：TypeScript + Node.js
- 遵循项目 .eslintrc 配置
- 修改前先阅读相关代码
```

**workspace-docs（参照上面格式，根据实际用途填写）：**

```bash
nano ~/workspace-docs/.claude/CLAUDE.md
```

```markdown
## 写作场景约定
- 面向技术读者，语言简洁直接
- 技术术语保留英文原文
- 代码示例必须可以直接运行
```

用同样方式创建 `~/workspace-docs/.codex/AGENTS.md` 和 `~/workspace-docs/.gemini/GEMINI.md`。

### 5.4 创建第一个子项目

每个 Workspace 下的子项目都独立管理 Git：

```bash
cd ~/workspace-dev/projects

mkdir project-a
cd project-a
git init
git branch -M main

# 可选：添加项目特有指令（只写这个项目特有的内容）
nano CLAUDE.md
```

项目级 `CLAUDE.md` 示例：

```markdown
## project-a 专属说明
- 这是用户认证服务
- 数据库：PostgreSQL 15
- 端口：3001
- 禁止直接查询 users 表，使用 UserService
```

同样创建 `AGENTS.md` 和 `GEMINI.md`（内容类似）。

> 💡 项目级指令文件**只写该项目特有的内容**。Workspace 级已经说明的内容不要重复，否则会被加载两次浪费 token。

---

## 第六章：Skills 管理

### 6.1 什么是 Skill

Skill 是一个告诉 AI「如何完成某类固定任务」的文件。比如你经常需要做代码审查，与其每次都描述一遍审查的标准，不如写成一个 Skill，以后 AI 会自动或手动触发它。

Skills 存在两种触发方式：
- **自动触发**：AI 根据你说的话，自动判断是否需要调用某个 Skill
- **手动触发**：你直接输入 `$skill名称` 来调用

### 6.2 SKILL.md 文件格式

```markdown
---
name: skill-name          # 必填，技能名称
description: |            # 必填，决定工具何时自动触发此 skill
  描述这个 skill 做什么，何时使用。
  越具体越好，包含关键词和触发场景。
---

# Skill 名称

## 背景
简要说明这个 skill 解决什么问题。

## 步骤
1. 第一步
2. 第二步
3. 第三步

## 注意事项
- 重要限制或边界条件
```

> ⚠️ `description` 字段是自动触发的关键。写得越具体，工具越准确地知道何时加载这个 skill。

### 6.3 创建第一个 Skill

以在 `workspace-dev` 里创建「代码审查」Skill 为例：

```bash
# 创建 skill 目录（目录名就是 skill 的名字）
mkdir -p ~/workspace-dev/.claude/skills/code-review

# 创建 SKILL.md 文件
nano ~/workspace-dev/.claude/skills/code-review/SKILL.md
```

输入以下内容：

```markdown
---
name: code-review
description: |
  对代码进行系统审查，检查代码质量、潜在 bug、安全问题、性能问题。
  当用户说「审查代码」、「review」、「检查代码」、「code review」时触发。
---

# 代码审查

## 审查维度
1. **正确性**：逻辑是否正确，边界条件是否处理
2. **安全性**：是否有注入、越权、信息泄露风险
3. **性能**：是否有明显性能问题（如循环内查询数据库）
4. **可读性**：命名是否清晰，逻辑是否易懂
5. **测试**：是否有对应的测试

## 输出格式
按严重程度分级：
- 🔴 严重（必须修复）
- 🟡 建议（最好修复）
- 🟢 优化（有空可以改）
```

### 6.4 验证 Skill 已被加载

```bash
cd ~/workspace-dev/projects/project-a

# Claude Code
claude
# 然后在对话框里输入：/skills
# 应该能看到 code-review 出现在列表里

# Codex
cdx-dev
# 启动后输入 /skills 查看

# Gemini CLI
gem-dev
# 启动后输入 /skills 查看
```

> ✅ 三个工具都应该能看到同一个 `code-review` Skill，因为它们指向的是同一个文件目录。

### 6.5 带脚本的 Skill

复杂的 Skill 可以附带可执行脚本：

```
~/workspace-dev/.claude/skills/
└── run-tests/
    ├── SKILL.md          # skill 的描述和步骤
    └── scripts/
        └── run.sh        # 实际执行的脚本
```

### 6.6 不同 Workspace 的 Skills 完全独立

```bash
# 验证隔离效果
ls ~/workspace-dev/.claude/skills/
# 应该看到：code-review（开发相关）

ls ~/workspace-docs/.claude/skills/
# 应该只看到写作相关 skill，不会看到 code-review
```

---

## 第七章：日常使用

### 7.1 启动工具

**在 workspace-dev 里工作：**

```bash
# 第一步：进入你的项目目录
cd ~/workspace-dev/projects/project-a

# 启动 Claude Code（自动找到 workspace-dev 的配置）
claude

# 启动 Codex
cdx-dev

# 启动 Gemini CLI
gem-dev
```

**切换到 workspace-docs：**

```bash
cd ~/workspace-docs/projects/docs-site

claude      # 自动加载 workspace-docs 的配置
cdx-docs    # Codex 使用 workspace-docs 配置
gem-docs    # Gemini CLI 使用 workspace-docs 配置
```

> 💡 Claude Code 完全靠当前目录自动判断使用哪个 Workspace 的配置，只要 `cd` 到正确的项目目录就行。Codex 和 Gemini CLI 需要用对应的别名启动。

### 7.2 各层配置的加载顺序

启动工具时，配置按以下顺序叠加，越靠后越优先：

| 顺序 | 工具 | 加载内容 |
|------|------|----------|
| 第 1 层（基础） | Claude Code | `~/.claude/CLAUDE.md` |
| 第 2 层 | Claude Code | `~/workspace-dev/.claude/CLAUDE.md`（自动向上查找） |
| 第 3 层（最优先） | Claude Code | `项目根目录/CLAUDE.md` |
| 第 1 层（基础） | Codex | `~/.codex/AGENTS.md` |
| 第 2 层 | Codex | `$CODEX_HOME/AGENTS.md`（别名指定的 workspace） |
| 第 3 层（最优先） | Codex | `项目根目录/AGENTS.md` |
| 第 1 层（基础） | Gemini CLI | `~/.gemini/GEMINI.md` |
| 第 2 层 | Gemini CLI | `$GEMINI_CLI_HOME/.gemini/GEMINI.md` |
| 第 3 层（最优先） | Gemini CLI | `项目根目录/GEMINI.md` |

### 7.3 手动调用 Skill

```bash
# 查看可用 Skills
/skills

# 手动调用某个 Skill
$code-review

# 自动触发：直接说你要做什么，AI 会判断是否需要调用 Skill
# 例如：「帮我看看这段代码有没有问题」→ 自动触发 code-review
```

---

## 第八章：常见问题

### Q1：/skills 列表里看不到我创建的 Skill

按顺序检查：

1. 确认 `SKILL.md` 文件里有 `name` 和 `description` 字段，且内容不为空
2. 确认软链接正确：
   ```bash
   ls -la ~/workspace-dev/.codex/
   # 应该看到：skills -> /Users/yourname/workspace-dev/.claude/skills
   ```
3. 重启工具（Skills 在每次启动时扫描，不会热加载）
4. 确认启动工具时用了正确的别名（`cdx-dev` 而不是直接 `codex`）

### Q2：Codex 或 Gemini CLI 加载了错误 Workspace 的配置

检查当前的环境变量：

```bash
echo $CODEX_HOME
echo $GEMINI_CLI_HOME
```

如果输出的路径不是你期望的 Workspace，使用别名启动，别名会临时覆盖环境变量：

```bash
cdx-docs    # 强制使用 workspace-docs 的配置启动 Codex
gem-docs    # 强制使用 workspace-docs 的配置启动 Gemini CLI
```

### Q3：修改了 ~/.zshrc 后新终端窗口没有生效

每次修改 `~/.zshrc` 后需要执行：

```bash
source ~/.zshrc
```

检查文件末尾内容是否保存正确：

```bash
tail -20 ~/.zshrc
# 应该能看到你添加的环境变量和别名
```

### Q4：软链接失效了（移动过目录或重命名过）

```bash
# 删除旧的软链接
rm ~/workspace-dev/.codex/skills
rm ~/workspace-dev/.gemini/skills

# 重新创建
ln -s ~/workspace-dev/.claude/skills ~/workspace-dev/.codex/skills
ln -s ~/workspace-dev/.claude/skills ~/workspace-dev/.gemini/skills
```

### Q5：想新增一个 Workspace

以新增 `workspace-research` 为例：

```bash
# 第一步：创建目录结构
mkdir -p ~/workspace-research/.claude/skills
mkdir -p ~/workspace-research/.claude/rules
mkdir -p ~/workspace-research/.codex
mkdir -p ~/workspace-research/.gemini
mkdir -p ~/workspace-research/projects

# 第二步：创建软链接
ln -s ~/workspace-research/.claude/skills ~/workspace-research/.codex/skills
ln -s ~/workspace-research/.claude/skills ~/workspace-research/.gemini/skills

# 第三步：创建指令文件
nano ~/workspace-research/.claude/CLAUDE.md
nano ~/workspace-research/.codex/AGENTS.md
nano ~/workspace-research/.gemini/GEMINI.md

# 第四步：在 ~/.zshrc 里添加别名
nano ~/.zshrc
# 在文件末尾添加：
# alias cdx-research='CODEX_HOME="$HOME/workspace-research/.codex" codex'
# alias gem-research='GEMINI_CLI_HOME="$HOME/workspace-research" gemini'

source ~/.zshrc
```

---

## 附录：快速参考

### 别名速查

| 别名 | 工具 | 使用哪个 Workspace |
|------|------|-------------------|
| `claude`（直接输入） | Claude Code | 自动根据当前目录判断 |
| `cdx-dev` | Codex | workspace-dev |
| `cdx-docs` | Codex | workspace-docs |
| `cdx-other` | Codex | workspace-other |
| `gem-dev` | Gemini CLI | workspace-dev |
| `gem-docs` | Gemini CLI | workspace-docs |
| `gem-other` | Gemini CLI | workspace-other |

### 关键文件位置速查

| 文件 | 位置 | 用途 |
|------|------|------|
| 用户级指令 | `~/.claude/CLAUDE.md` | Claude Code 全局偏好 |
| 用户级指令 | `~/.codex/AGENTS.md` | Codex 全局偏好 |
| 用户级指令 | `~/.gemini/GEMINI.md` | Gemini CLI 全局偏好 |
| Workspace 级指令 | `~/workspace-xxx/.claude/CLAUDE.md` | 该场景 Claude Code 规范 |
| Workspace 级指令 | `~/workspace-xxx/.codex/AGENTS.md` | 该场景 Codex 规范 |
| Workspace 级指令 | `~/workspace-xxx/.gemini/GEMINI.md` | 该场景 Gemini CLI 规范 |
| Skills（真实文件） | `~/workspace-xxx/.claude/skills/` | 三端共享的 Skills |
| Skills（软链接） | `~/workspace-xxx/.codex/skills/` | 指向上面的真实目录 |
| Skills（软链接） | `~/workspace-xxx/.gemini/skills/` | 指向上面的真实目录 |
| Shell 配置 | `~/.zshrc` | 环境变量和别名 |
| 项目级指令 | `项目根目录/CLAUDE.md` | 项目特有说明 |

### 环境变量速查

| 变量 | 工具 | 说明 |
|------|------|------|
| `ANTHROPIC_API_KEY` | Claude Code | Anthropic API 密钥，以 `sk-ant-` 开头 |
| `OPENAI_API_KEY` | Codex | OpenAI API 密钥，以 `sk-` 开头 |
| `GEMINI_API_KEY` | Gemini CLI | Google API 密钥，以 `AIza` 开头 |
| `CODEX_HOME` | Codex | 指向 Codex 配置目录（`.codex` 文件夹所在路径） |
| `GEMINI_CLI_HOME` | Gemini CLI | 指向 Workspace 根目录（`.gemini` 文件夹的父目录） |

### 资源共享总览

| 资源 | Claude Code | Codex | Gemini CLI | 共享方式 |
|------|-------------|-------|------------|----------|
| Skills | `workspace/.claude/skills/` | `workspace/.codex/skills/`（软链接） | `workspace/.gemini/skills/`（软链接） | ✅ 三端完全共享 |
| 全局指令 | `~/.claude/CLAUDE.md` | `~/.codex/AGENTS.md` | `~/.gemini/GEMINI.md` | ⚠️ 内容可参考，格式独立 |
| Workspace 指令 | 自动向上查找 | `CODEX_HOME` 指定 | `GEMINI_CLI_HOME` 指定 | ⚠️ 分别维护 |
| 项目指令 | `CLAUDE.md` | `AGENTS.md` | `GEMINI.md` | ⚠️ 分别维护 |
| MCP 配置 | `.claude/settings.json` | `.codex/config.toml` | `.gemini/settings.json` | ❌ 格式不同，分别配置 |

---

> 配置完成后：三个工具共享同一套 Skills，不同 Workspace 完全隔离，每个项目独立管理 Git，每次只加载当前场景需要的内容。
