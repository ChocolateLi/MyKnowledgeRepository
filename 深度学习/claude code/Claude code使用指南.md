# Claude code使用指南

# claude安装

1.先检查基础环境

```cmd
nvm -v
npm -v
node -v
```

2.安装claude

```cmd
npm install -g @anthropic-ai/claude-code
```

3.安装skills技能包

```cmd
npm list -g openskills -- 查看是否有openskills
npm install -g openskills --安装openskills
-- 默认安装在本地
D:\Myfile\开发项目\护士层级研发系统>openskills install anthropics/skills
Installing from: anthropics/skills
Location: project (.claude/skills)
Default install is project-local (./.claude/skills). Use --global for ~/.claude/skills.
```

# Skills

## 什么是Skills？

Skills是模块化的能力包，包含指令、脚本和资源，让Claude在需要时自动加载和使用。

**"模块化"**：Skills是一个个独立的文件夹，每个Skill做一件事。比如"生成PPT"是一个Skill，"审校文章"是另一个Skill。

**"能力包"**：每个Skill文件夹里可以包含：

- SKILL.md（核心指令文件，必需）
- scripts/（可执行脚本，可选）
- references/（参考文档，可选）
- assets/（模板和资源，可选）

**"自动加载"**：你不需要手动告诉Claude"现在用XX Skill"。Claude会根据你的任务描述，自动判断需要哪个Skill，然后加载。

一个 Skill 至少是一个文件夹

```
skill-name/
├── SKILL.md              # 主要说明（触发时加载）
├── FORMS.md              # 表单填充指南（根据需要加载）
├── reference.md          # API 参考（根据需要加载）
├── examples.md           # 使用示例（根据需要加载）
└── scripts/
    ├── analyze_form.py   # 实用脚本（执行，不加载）
    ├── fill_form.py      # 表单填充脚本
    └── validate.py       # 验证脚本
```

官方最小可用模板

```markdown
---
name: example-skill
description: 简要说明该技能的用途和适用场景
---

## 使用场景
说明在什么情况下应该使用这个 Skill。

## 执行步骤
1. 第一步要做什么
2. 第二步要做什么
3. 异常情况如何处理

## 输出要求
说明输出格式或必须包含的内容。
```

更推荐的实战模板

```markdown
---
name: security-log-analysis
description: 对安全日志进行结构化分析，判断是否存在异常行为
metadata:
  version: 1.0
  author: ailot
---

## 技能目标
明确这个 Skill 希望 Agent 达成的目标。

## 输入说明
- 支持的输入类型
- 必须包含的字段

## 执行流程
1. 识别数据类型
2. 提取关键字段
3. 进行规则或逻辑判断
4. 输出分析结论

## 输出格式
- 是否异常：
- 判断依据：
- 风险说明：
- 建议动作：

## 注意事项
- 无法确认时必须说明不确定性
- 禁止空泛总结
```



## 什么是渐进式披露？

简单说，就是**分阶段、按需加载**。

一个Skill包含很多内容：核心指令、参考文档、执行脚本、模板资源。但Claude不会一次性把所有内容都加载进上下文。它采用三层加载机制：

**第一层：元数据（Metadata）—— 总是加载**

内容：SKILL.md文件开头的YAML部分，就两个字段：name和description。

```
---name: ai-proofreading
description: 系统化降低AI检测率，增加人味。使用场景：审校文章、降低AI味、初稿完成后。
---
```

加载时机：Claude启动时就加载所有Skills的元数据。

Token成本：每个Skill大约100 tokens。就算你装了50个Skills，也就5000 tokens。

作用：让Claude知道有哪些Skills可用，什么时候该用哪个。

**第二层：指令（Instructions）—— 触发时加载**

内容：SKILL.md的主体部分，详细的操作指南。

加载时机：当用户请求匹配某个Skill的description时，Claude才加载这个Skill的完整内容。

Token成本：通常在3000-5000 tokens。

作用：告诉Claude具体怎么做。

**第三层：资源（Resources）—— 引用时加载**

内容：scripts/目录里的脚本、references/目录里的参考文档、assets/目录里的模板。

加载时机：只有当SKILL.md中的指令引用这些文件时才加载。

Token成本：几乎无限——脚本执行后只有输出进入上下文，代码本身不占Token。

作用：提供确定性的执行能力和详细的参考资料。

## Skills资源

官方资源：[anthropics/skills](https://github.com/anthropics/skills)

官方文档：[agentskills.io](https://code.claude.com/docs/zh-CN/overview)

社区资源（包含：TDD、调试、代码审查、计划执行等）：[obra/superpowers](https://github.com/obra/superpowers)

SkillsMP：[Agent Skills 市场](https://skillsmp.com)

## Skills安装

### 插件安装

```markdown
/plugin marketplace add anthropics/skills

/plugin install document-skills@anthropic-agent-skills # 文档技能包，可以处理 Excel、Word、PPT、PDF 等文档。
/plugin install example-skills@anthropic-agent-skills # 示例技能包 ，可以处理技能创建、MCP 构建、视觉设计、算法艺术、网页测试、Slack 动图制作、主题样式等。
```

### 自定义skills

请使用 /skill-creator 技能，帮我把下面的提示词，创建为一个数据库设计的skill，名为 database-design:

-Role: 数据库架构设计专家和软件系统规划师

-Background: 用户正在开发一个.net 6项目，业务流程已经明确，需要将业务流程转化为数据库模式，以便更好地存储和管理数据，确保后端的功能实现和数据的高效处理。

-Profile: 你是一位经验丰富的数据库架构设计专家，对数据库设计理论和实践有着深入的理解。同时，你也是软件系统规划师，能够从整体上把握业务流程与数据库设计之间的关系，擅长将复杂的业务逻辑转化为高效、可扩展的数据库结构。

-Skills: 你精通数据库设计原则，如范式理论、关系数据库建模、数据完整性约束等。具备将业务需求转化为数据库模型的能力，能够根据业务流程设计出合理的表结构、字段类型和关系。熟悉主流数据库管理系统（如 Sqlserver、oralce、mysql 等）的特性和优化方法。

-Goals:    1、根据用户提供的业务流程或者需求分析书，分析业务实体及其属性。    2、确定实体之间的关系，设计出符合业务需求的数据库表结构。    3、为每个表设计合适的字段类型和约束，确保数据的完整性和一致性。    4、提供数据库模式设计的详细文档，包括表结构、关系图和必要的说明。

-Constraints: 数据库设计应符合第三范式，确保数据的最小冗余和高一致性。同时，要考虑到小程序的性能和扩展性，避免过度设计或设计不足。

-OutputFormat: 数据库模式设计文档，包括表结构描述、字段类型、主键和外键定义、关系图以及设计说明。

-Workflow:    1、深入分析业务流程，识别业务实体及其属性。    2、确定实体之间的关系，如一对一、一对多或多对多关系。    3、根据实体和关系设计数据库表结构，为每个表定义字段类型、主键和外键。    4、绘制数据库关系图，清晰展示表之间的关系。    5、编写数据库模式设计文档，详细说明设计思路和各表的作用。

## Skills设计的5个最佳实践

### 实践1：Description决定一切

description是Skill最重要的字段。它决定了：

- Claude什么时候会想到这个Skill
- Claude能否准确匹配用户意图

**写好description的公式**：

```
做什么（核心功能）+ 什么时候用（触发场景）+ 触发关键词
```

**好的例子**：

```
description: | Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when user mentions PDFs, forms, or document extraction.
```

- 核心功能：提取文本、表格，填写表单，合并文档
- 触发场景：处理PDF文件时
- 触发关键词：PDF、forms、document extraction

**坏的例子**：

```
description: Process PDF files
```

太简短，Claude不知道什么场景该用。

### 实践2：单一职责，每个Skill只做一件事

一个Skill不要做太多事情。

**原因**：

1. description难写。功能越多，触发越不准确
2. Token浪费。用户只需要其中一个功能，却加载全部内容
3. 难维护。改一个功能可能影响其他功能

**我的做法**：

一个Skill对应一个明确的任务：

- 选题生成
- AI味审校
- 图片配图
- 长文转X

而不是：

- 文章创作全流程（太大了）

### 实践3：渐进式披露，核心内容放SKILL.md，详细内容放references/

SKILL.md应该简洁，包含核心流程和最常用的指令。

详细的参考资料、边界情况、深入解释，放到references/目录下。

**结构示例**：

```markdown
# SKILL.md

## 快速流程
1. 第一步
2. 第二步
3. 第三步

## 常见场景
- 场景A：做法
- 场景B：做法
## 详细参考
- 更多细节见：[DETAILED_GUIDE.md](references/DETAILED_GUIDE.md)
- 边界情况见：[EDGE_CASES.md](references/EDGE_CASES.md)
```

这样，基础任务只加载SKILL.md（3000 tokens）。

只有遇到复杂情况，才加载references/（额外5000 tokens）。

### 实践4：脚本优于生成代码

如果一个任务可以用脚本完成，就写成脚本。

**原因**：

1. 确定性：脚本是测试过的，每次执行结果一致。让Claude现场生成代码，可能有bug。
2. Token效率：脚本代码不进入上下文，只有执行结果进入。
3. 可复用：脚本写一次，到处可用。

**我的做法**：

"图片配图与上传"Skill里，上传图片到图床的逻辑写成了Python脚本。

Claude只需要执行：`python scripts/upload_image.py image.png`

不需要每次都生成上传代码。

### 实践5：从简单开始，逐步迭代

不要一开始就想着写完美的Skill。

从最小可行版本开始：

1. 写一个简单的SKILL.md
2. 用几次，发现问题
3. 添加遗漏的规则
4. 添加常见错误的处理
5. 逐步完善

我的"AI味审校"Skill，最初只有20行。用了一个月，根据实际遇到的问题，逐步扩展到300行。

# Skills vs MCP vs Subagent

## 三者关系

MCP让Claude能碰到外部系统。Skills告诉Claude碰到之后怎么用。Subagent是派一个人出去干活。

**MCP（Model Context Protocol）**

MCP是什么？一个连接协议。它让Claude能够访问外部系统：数据库、API、文件系统、各种SaaS服务。

你可以把MCP想象成"给Claude发工具"。

比如GitHub MCP，让Claude能够读取仓库、创建PR、管理Issues。Notion MCP，让Claude能够读写Notion页面。

MCP的核心价值是**连接**。它解决的问题是"Claude能访问什么数据"。

**Skills**

Skills是什么？使用手册。它告诉Claude拿到数据之后怎么用。

比如你用GitHub MCP连接了仓库，Claude能读代码了。但"怎么做代码审查"——检查哪些方面、用什么标准、输出什么格式——这些是Skills的工作。

你可以把Skills想象成"教Claude怎么用工具"。

Skills的核心价值是**程序化知识**。它解决的问题是"Claude应该怎么做"。

**Subagent**

Subagent是什么？派出去干活的人。

当你让Claude Code派一个Subagent去做任务时，Claude会新开一个独立的对话会话。这个Subagent有自己的上下文窗口、自己的系统提示、自己的工具权限。它干完活，把结果带回来。

你可以把Subagent想象成"派一个助手出去"。

Subagent的核心价值是**并行执行和上下文隔离**。它解决的问题是"怎么处理复杂的多步骤任务"。

![](assets/skills关系图.jpg)

## 什么时候用哪个？

**用MCP**：当你需要连接外部系统。

- 查询数据库
- 调用第三方API
- 读写Notion、Jira、GitHub等

**用Skills**：当你有重复性的工作流程。

- 代码审查流程
- 文章审校流程
- 报告生成流程
- 任何"每次都要说一遍"的规则

**用Subagent**：当任务复杂、需要并行执行。

- 审查整个代码仓库（耗时长）
- 同时处理多个独立任务
- 需要防止上下文污染

# Cursor + Claude Code

## 组合使用

先使用cursor分析项目，然后叫Claude Code实现功能。

```mardown
这是我们医院自己搭建的后端项目框架护士层级管理系统，用于管理护士层级。项目基于.net框架，按“接口层 / 服务层 / 仓储层 / 模型层 / 通用工具层”分层组织，是一个.NET WebAPI多项目解决方案和前后端分离设计。
## 分析任务指令
请帮我深度分析这个项目，我需要为其添加相应的接口功能，请按照以下结构进行系统性分析：
### 1.项目架构深度分析
请详细分析一下关键架构要素：
- 入口定位：找到项目的主启动文件和命令行接口实现
- web服务架构：识别需要使用什么框架可以实现业务接口
- 前后端分析模式：分析前端与后端.net 服务的通信机制
- 路由系统：映射所有HTTP端点和API路由结构
- 会话管理：当前是否有任何用户状态或会话处理机制
- Docker集成：容器化部署对系统的影响
### 2.技术栈兼容性分析评估
重点关注以下技术组合：
- .net后端框架识别：确定使用的具体web框架和版本
- 依赖包分析：检查.sln相应的依赖包分析
- 数据库使用情况：是否已有数据库集成，使用何种ORM框架
- 前端通信协议：REST API、WebSocket或其他通信方式
- 配置管理：appsettings.json和Program.cs使用模式
请开始深度分析，并为后续的接口实现提供详实的技术基础。
```

## 三阶段工作流

[三阶段工作流](https://github.com/geekoe/workflow3)