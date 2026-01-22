# Claude Code Marketplace 维护规范

> 基于 [wshobson/agents](https://github.com/wshobson/agents) 项目整理
>
> **当前项目状态**:
> - 本项目是 My Claude Marketplace,一个正在建设中的Claude插件市场
> - 当前版本: v1.0.0
> - 已有插件: 1个 (design-doc-generator)
> - 本文档描述了目标架构和维护规范,用于指导插件开发

## 📋 目录

- [项目概览](#项目概览)
- [核心架构原则](#核心架构原则)
- [目录结构](#目录结构)
- [Marketplace.json 配置](#marketplacejson-配置)
- [插件开发规范](#插件开发规范)
- [Agent 编写规范](#agent-编写规范)
- [命名规范](#命名规范)
- [安装和使用](#安装和使用)
- [贡献流程](#贡献流程)
- [最佳实践](#最佳实践)

---

## 项目概览

**My Claude Marketplace** 是一个正在建设中的Claude插件市场项目,致力于提供生产级的智能自动化和多智能体编排解决方案。

**当前状态**:
- **1 个插件** - design-doc-generator (文档生成)
- **1 个专业化 Agent** - 详细设计文档生成器
- **版本**: v1.0.0

**目标架构** (参考 wshobson/agents):
- **72+ 插件** - 按功能域组织
- **108+ 专业化 Agents** - 跨领域专家代理
- **129+ Agent Skills** - 模块化知识包
- **72+ 开发工具** - 脚手架、扫描、测试、部署工具
- **23+ 分类** - 覆盖开发、基础设施、安全、语言等

### 关键特性

- **细粒度设计**：每个插件独立运行，避免不必要的上下文膨胀
- **渐进式披露**：按需加载知识，而非预先加载
- **模型分层**：根据任务复杂度使用不同模型（Opus/Sonnet/Haiku）

---

## 核心架构原则

### 1. Granular Design（细粒度设计）

**Unix 哲学**："每个插件做好一件事"

- 平均每个插件 **3-4 个组件**（符合 Anthropic 的 2-8 模式）
- 单一职责原则，避免臃肿插件
- 用户可以根据需求混搭插件

**示例**：

```
backend-development/      # 后端开发插件
├── agents/
│   ├── backend-architect.md
│   ├── graphql-architect.md
│   └── tdd-orchestrator.md
└── skills/
    └── api-design.md
```

### 2. Progressive Disclosure（渐进式披露）

Skills 使用**三层架构**优化 Token 使用：

| 层级               | 内容                   | 加载时机  |
|------------------|----------------------|-------|
| **Metadata**     | 名称、激活条件（frontmatter） | 始终加载  |
| **Instructions** | 核心指导                 | 激活时加载 |
| **Resources**    | 示例、模板                | 按需加载  |

**优势**：

- 防止"上下文污染"
- 专业知识不臃肿
- 降低 Token 成本

### 3. Model Stratification（模型分层）

根据任务复杂度选择模型：

| 模型             | Agent 数量 | 适用场景         |
|----------------|----------|--------------|
| **Opus 4.5**   | 42       | 关键架构、安全、代码审查 |
| **Sonnet 4.5** | 39       | 复杂推理、架构决策    |
| **Haiku 4.5**  | 18       | 快速确定性任务、操作   |
| **Inherit**    | 42       | 继承用户选择的模型    |

**混合编排**：平衡性能与成本效率

---

## 目录结构

### 仓库根目录

```
claude-agents/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace 配置文件
├── plugins/                      # 插件目录（72个）
│   ├── backend-development/
│   ├── frontend-mobile-development/
│   ├── python-development/
│   ├── javascript-typescript/
│   ├── kubernetes-operations/
│   ├── cloud-infrastructure/
│   └── ...
├── docs/                         # 文档目录
│   ├── plugin-reference.md
│   ├── agent-reference.md
│   ├── architecture.md
│   └── contributing.md
└── README.md
```

### 插件内部结构

每个插件目录遵循以下结构：

```
plugin-name/
├── agents/          # AI Agent 配置（.md 文件）
│   ├── agent-1.md
│   └── agent-2.md
├── commands/        # CLI 命令（.md 文件）
│   └── command-1.md
├── skills/          # 可复用能力（.md 文件）
│   └── skill-1.md
└── [其他插件文件]
```

---

## Marketplace.json 配置

### 根结构

```json
{
  "name": "claude-code-workflows",
  "owner": {
    "name": "Seth Hobson",
    "email": "seth@major7apps.com",
    "url": "https://github.com/wshobson"
  },
  "metadata": {
    "description": "Production-ready workflow orchestration for Claude Code",
    "version": "1.3.7"
  },
  "plugins": [
    // 插件数组
  ]
}
```

### 插件配置字段

每个插件条目包含以下字段：

```json
{
  "name": "backend-development",
  // 唯一标识符
  "source": "plugins/backend-development",
  // 本地路径
  "description": "Backend architecture and API design experts",
  "version": "1.2.0",
  // 语义化版本
  "author": {
    "name": "Seth Hobson",
    "url": "https://github.com/wshobson"
  },
  "homepage": "https://github.com/wshobson/agents",
  "repository": "https://github.com/wshobson/agents",
  "license": "MIT",
  // MIT 或 Apache-2.0
  "keywords": [
    // 可搜索标签
    "backend",
    "api",
    "architecture",
    "microservices"
  ],
  "category": "development",
  // 功能分组
  "strict": false,
  // 强制规则
  "commands": [
    // 命令文件路径
    "commands/api-design.md"
  ],
  "agents": [
    // Agent 定义数组
    {
      "name": "backend-architect",
      "file": "agents/backend-architect.md",
      "model": "opus",
      // opus/sonnet/haiku/inherit
      "description": "Expert backend architect"
    }
  ],
  "skills": [
    // 可选：技能模块
    "skills/api-patterns.md"
  ]
}
```

### 关键字段说明

| 字段            | 类型     | 必填 | 说明                   |
|---------------|--------|----|----------------------|
| `name`        | string | ✅  | 插件唯一标识符（kebab-case）  |
| `source`      | string | ✅  | 插件目录相对路径             |
| `description` | string | ✅  | 插件功能描述               |
| `version`     | string | ✅  | 语义化版本号（如 1.2.3）      |
| `author`      | object | ✅  | 作者信息（name + url）     |
| `license`     | string | ✅  | 开源协议（MIT/Apache-2.0） |
| `keywords`    | array  | ✅  | 搜索标签                 |
| `category`    | string | ✅  | 分类（见下表）              |
| `agents`      | array  | ✅  | Agent 配置列表           |
| `commands`    | array  | ⬜  | 命令文件路径               |
| `skills`      | array  | ⬜  | 技能文件路径               |

### 插件分类（23个）

| 分类               | 说明      | 示例插件                                             |
|------------------|---------|--------------------------------------------------|
| `development`    | 开发工具    | backend-development, frontend-mobile-development |
| `infrastructure` | 基础设施    | kubernetes-operations, cloud-infrastructure      |
| `security`       | 安全      | security-scanning, backend-api-security          |
| `operations`     | 运维      | cicd-automation, deployment-strategies           |
| `documentation`  | 文档      | code-documentation, api-documentation            |
| `testing`        | 测试      | unit-testing, test-automation                    |
| `languages`      | 语言专用    | python-development, javascript-typescript        |
| `database`       | 数据库     | database-operations, sql-nosql-expert            |
| `api`            | API     | api-development, graphql-development             |
| `ai-ml`          | AI/机器学习 | machine-learning-ops, llm-development            |
| `blockchain`     | 区块链     | blockchain-web3, smart-contracts                 |
| `payments`       | 支付      | payment-integration, fintech-development         |
| `gaming`         | 游戏      | game-development, unity-unreal-expert            |
| `marketing`      | 营销      | seo-analysis-monitoring, content-marketing       |
| `performance`    | 性能      | performance-optimization, load-testing           |
| ...              | ...     | ...                                              |

---

## 插件开发规范

### 创建新插件

1. **在 `plugins/` 目录下创建插件目录**
   ```bash
   mkdir plugins/my-plugin
   ```

2. **创建子目录结构**
   ```bash
   cd plugins/my-plugin
   mkdir agents commands skills
   ```

3. **添加组件文件**（.md 格式）
    - `agents/my-agent.md` - Agent 定义
    - `commands/my-command.md` - 命令定义
    - `skills/my-skill.md` - 技能定义

4. **更新 marketplace.json**
   在 `plugins` 数组中添加插件配置

### 插件设计原则

✅ **DO（推荐）**：

- 保持插件专注（单一职责）
- 平均 3-4 个组件
- 提供清晰的文档
- 使用语义化版本
- 添加有意义的关键字

❌ **DON'T（避免）**：

- 创建臃肿的多功能插件
- 混合不相关的功能
- 缺少版本控制
- 模糊的描述或命名

---

## Agent 编写规范

### Agent 文件结构

Agent 使用 **Markdown 格式**，包含 **YAML Frontmatter** 和内容主体。

#### 示例：backend-architect.md

```markdown
---
name: backend-architect
description: Expert backend architect specializing in scalable API design, microservices architecture, and distributed systems
model: opus
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
whenToUse: Use PROACTIVELY when creating new backend services or APIs
color: blue
---

# Backend Architect

You are an expert backend architect specializing in:

## Core Competencies

- **API Design**: REST, GraphQL, gRPC with versioning strategies
- **Microservices**: Service boundaries, inter-service communication
- **Event-Driven Systems**: Message queues, Kafka, pub/sub patterns
- **Resilience Patterns**: Circuit breakers, retries, timeouts
- **Observability**: Logging, metrics, distributed tracing

## Frameworks & Technologies

- Node.js, Python, Java, Go, C#/.NET, Ruby, Rust
- Authentication: OAuth 2.0, JWT, mTLS
- Databases: SQL, NoSQL, caching strategies

## Deliverables

Generate:

- Architecture diagrams
- API contracts (OpenAPI/Swagger)
- Service definitions
- Deployment strategies

## Workflow

1. Analyze requirements
2. Define service boundaries
3. Design API contracts
4. Plan data flow and communication
5. Document architecture decisions

---

## Example Prompts

- "Design a microservices architecture for an e-commerce platform"
- "Create an API gateway pattern with rate limiting"
- "Implement event-driven order processing system"
```

### Frontmatter 字段

| 字段            | 类型     | 必填 | 说明                                            |
|---------------|--------|----|-----------------------------------------------|
| `name`        | string | ✅  | Agent 唯一标识符（kebab-case）                       |
| `description` | string | ✅  | 简短描述（1-2 句话）                                  |
| `model`       | string | ✅  | 使用的模型：`opus`/`sonnet`/`haiku`/`inherit`       |
| `tools`       | array  | ✅  | 可用工具列表（Read, Write, Edit, Bash, Grep, Glob 等） |
| `whenToUse`   | string | ✅  | 激活条件（明确说明何时使用）                                |
| `color`       | string | ⬜  | UI 显示颜色                                       |

### Agent 描述编写原则

#### 1. **清晰的身份定位**

```markdown
You are an expert [role] specializing in [domain].
```

#### 2. **核心能力列表**

使用 Markdown 列表清晰列出：

```markdown
## Core Competencies

- **API Design**: REST, GraphQL, gRPC
- **Microservices**: Service boundaries, communication patterns
- **Security**: Authentication, authorization, encryption
```

#### 3. **技术栈说明**

明确支持的框架、语言、工具：

```markdown
## Technologies

- Languages: Python, Node.js, Go
- Frameworks: FastAPI, Express, Gin
- Databases: PostgreSQL, MongoDB, Redis
```

#### 4. **工作流程**

提供结构化的执行步骤：

```markdown
## Workflow

1. Analyze requirements
2. Design architecture
3. Implement patterns
4. Generate documentation
```

#### 5. **示例提示词**

帮助用户理解如何使用：

```markdown
## Example Prompts

- "Design RESTful API for user management"
- "Implement OAuth2 authentication flow"
```

### Model 选择指南

| Model       | 适用场景               | Agent 示例                            |
|-------------|--------------------|-------------------------------------|
| **opus**    | 关键架构决策、安全审查、复杂代码审查 | backend-architect, security-auditor |
| **sonnet**  | 复杂推理、架构设计、深度分析     | database-architect, system-designer |
| **haiku**   | 快速任务、确定性操作、简单转换    | code-formatter, syntax-checker      |
| **inherit** | 继承用户偏好、通用任务        | debugger, test-runner               |

---

## 命名规范

### 通用规则

**一律使用小写 + 连字符（kebab-case）**

```
✅ 正确：
- backend-development
- kubernetes-operations
- full-stack-orchestration
- python-pro
- api-design

❌ 错误：
- BackendDevelopment
- kubernetes_operations
- fullStackOrchestration
- pythonPro
- API-Design
```

### 插件命名

**格式**：`<domain>-<functionality>`

```
backend-development       # 后端开发
frontend-mobile-development  # 前端和移动开发
kubernetes-operations    # Kubernetes 运维
security-scanning        # 安全扫描
```

### Agent 命名

**格式**：`<role>-<specialty>` 或 `<technology>-<level>`

```
backend-architect        # 后端架构师
database-optimizer      # 数据库优化师
python-pro             # Python 专家
graphql-architect      # GraphQL 架构师
```

### Command 命名

**格式**：`<action>-<target>`

```
security-hardening     # 安全加固
code-review            # 代码审查
api-scaffold          # API 脚手架
test-generation       # 测试生成
```

### Skill 命名

**格式**：`<knowledge-area>-<aspect>`

```
api-design-patterns    # API 设计模式
testing-strategies     # 测试策略
deployment-workflows   # 部署工作流
```

---

## 安装和使用

### 添加 Marketplace

```bash
/plugin marketplace add wshobson/agents
```

这会使所有 72 个插件可用，但**不会加载到上下文中**。

### 浏览可用插件

```bash
/plugin
```

### 安装插件

```bash
# 安装单个插件
/plugin install python-development

# 安装多个插件
/plugin install javascript-typescript
/plugin install kubernetes-operations
/plugin install full-stack-orchestration
```

### 重要概念：插件 vs Agent

**关键区别**：

- 用户安装的是**插件**（包含多个 agents）
- **不是**直接安装单个 agent

**正确示例**：

```bash
✅ /plugin install javascript-typescript@claude-code-workflows
✅ /plugin install backend-development
```

**错误示例**：

```bash
❌ /plugin install typescript-pro  # typescript-pro 是 agent，不是 plugin
❌ /plugin install backend-architect
```

### 使用插件功能

#### 1. 调用全栈编排

```bash
/full-stack-orchestration:full-stack-feature "user authentication with OAuth2"
```

协调 7+ 个 agents：

- 后端架构
- 数据库设计
- 前端开发
- 测试
- 安全
- 部署

#### 2. 安全评估

```bash
/security-scanning:security-hardening --level comprehensive
```

多 agent 评估：

- SAST（静态分析）
- 依赖分析
- 代码审查

#### 3. Python 项目搭建

```bash
/python-development:python-scaffold fastapi-microservice
```

生成生产就绪的 FastAPI 项目。

### 故障排除

**问题：插件未找到**

```bash
# 确保添加 marketplace
/plugin marketplace add wshobson/agents

# 使用完整插件名
/plugin install backend-development@claude-code-workflows
```

**问题：插件未加载**

清除缓存：

```bash
rm -rf ~/.claude/plugins/cache/claude-code-workflows
rm ~/.claude/plugins/installed_plugins.json
```

---

## 贡献流程

### 添加新 Agent

1. **识别适当的插件目录**
   ```bash
   cd plugins/backend-development
   ```

2. **创建 Agent 文件**
   ```bash
   touch agents/my-new-agent.md
   ```

3. **编写 Agent 定义**
    - 添加 YAML frontmatter
    - 编写详细的指导说明
    - 提供示例用法

4. **更新 marketplace.json**
   在对应插件的 `agents` 数组中添加：
   ```json
   {
     "name": "my-new-agent",
     "file": "agents/my-new-agent.md",
     "model": "sonnet",
     "description": "Brief description"
   }
   ```

### 添加新 Skill

1. **创建 Skill 文件**
   ```bash
   touch plugins/backend-development/skills/my-skill.md
   ```

2. **使用三层架构**
   ```markdown
   ---
   name: my-skill
   description: Brief description
   ---

   # Metadata Section (Always loaded)

   ## Instructions (Loaded when activated)

   ## Resources (Loaded on demand)
   ```

3. **更新插件配置**
   在 marketplace.json 的 `skills` 数组中添加路径。

### 提交代码

1. **Fork 仓库**
2. **创建功能分支**
   ```bash
   git checkout -b feature/add-new-agent
   ```

3. **提交更改**
   ```bash
   git add .
   git commit -m "Add: new backend security agent"
   ```

4. **推送并创建 PR**
   ```bash
   git push origin feature/add-new-agent
   ```

---

## 最佳实践

### 1. 插件设计

✅ **保持专注**

- 每个插件平均 3-4 个组件
- 遵循单一职责原则

✅ **清晰的边界**

- 明确插件的功能范围
- 避免功能重叠

✅ **可组合性**

- 设计可以与其他插件协同的功能
- 提供清晰的集成接口

### 2. Agent 开发

✅ **明确的激活条件**

```markdown
whenToUse: Use PROACTIVELY when creating REST APIs
```

✅ **结构化的能力描述**

```markdown
## Core Competencies

- Capability 1: Description
- Capability 2: Description
```

✅ **实用的示例**

```markdown
## Example Prompts

- "Design authentication system"
- "Implement rate limiting"
```

### 3. 文档编写

✅ **使用 Markdown 格式**

- 清晰的标题层次
- 代码块语法高亮
- 表格展示结构化数据

✅ **提供示例**

- 配置示例
- 代码示例
- 命令示例

✅ **保持更新**

- 版本号同步
- 及时反映功能变化

### 4. 版本管理

✅ **语义化版本**

- `1.0.0` - 主版本.次版本.补丁
- 主版本：不兼容的 API 变更
- 次版本：向后兼容的功能新增
- 补丁：向后兼容的问题修正

### 5. Token 优化

✅ **渐进式披露**

- 使用三层技能架构
- 按需加载资源

✅ **精简描述**

- 简洁明了的描述
- 避免冗余信息

---

## 参考资源

- **项目主页**：https://github.com/wshobson/agents
- **插件参考**：72 个插件完整目录
- **Agent 参考**：108 个 agents 按类别分类
- **Agent Skills**：129 个专业技能文档
- **架构指南**：设计原则和模式

---

## 附录：完整插件类别列表

| 序号 | 类别                    | 插件数量 | 主要插件                                             |
|----|-----------------------|------|--------------------------------------------------|
| 1  | Development           | 4    | backend-development, frontend-mobile-development |
| 2  | Documentation         | 3    | code-documentation, api-documentation            |
| 3  | Workflows             | 2    | full-stack-orchestration, workflow-automation    |
| 4  | Testing               | 2    | unit-testing, test-automation                    |
| 5  | Quality               | 2    | comprehensive-review, code-quality               |
| 6  | AI/ML                 | 3    | machine-learning-ops, llm-development            |
| 7  | Data                  | 2    | data-engineering, data-science                   |
| 8  | Database              | 3    | database-operations, sql-nosql-expert            |
| 9  | Operations            | 4    | cicd-automation, deployment-strategies           |
| 10 | Performance           | 2    | performance-optimization, load-testing           |
| 11 | Infrastructure        | 5    | kubernetes-operations, cloud-infrastructure      |
| 12 | Security              | 4    | security-scanning, backend-api-security          |
| 13 | Python                | 1    | python-development                               |
| 14 | JavaScript/TypeScript | 1    | javascript-typescript                            |
| 15 | Java                  | 1    | java-development                                 |
| 16 | Go                    | 1    | go-development                                   |
| 17 | Rust                  | 1    | rust-development                                 |
| 18 | C#/.NET               | 1    | dotnet-development                               |
| 19 | Ruby                  | 1    | ruby-development                                 |
| 20 | Blockchain            | 2    | blockchain-web3, smart-contracts                 |
| 21 | Finance               | 1    | fintech-development                              |
| 22 | Payments              | 1    | payment-integration                              |
| 23 | Gaming                | 2    | game-development, unity-unreal-expert            |

---

**版本**: 1.0.0
**最后更新**: 2026-01-21
**基于项目**: wshobson/agents v1.3.7