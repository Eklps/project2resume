---
name: project-to-resume
description: 从项目源码+用户信息自动生成完整 HTML 格式简历。扫描代码仓库，提炼技术亮点，采集个人信息，用 PSR 法输出 A4 排版的专业简历 HTML 文件，可直接打印投递。
---

# Skill: Project-to-Resume

## 元数据

| 属性 | 值 |
|------|-----|
| 名称 | project-to-resume |
| 版本 | 2.1.0 |
| 作者 | CodeArts |
| 标签 | resume, career, project-analysis, job-hunting, html |
| 语言支持 | zh-CN |

## 描述

从项目源码 + 用户个人信息，自动生成完整的 **HTML 格式简历**。扫描代码仓库 → 提炼技术亮点 → 采集个人信息 → 用 PSR 法输出 A4 排版的专业简历 HTML 文件，可直接打印投递。

## 触发条件

### 关键词触发
- "生成简历"
- "简历生成"
- "项目简历"
- "写简历"
- "resume"
- "项目经历"
- "技术亮点"
- "HTML简历"

### 文件触发
当仓库包含以下文件时可自动建议：
- README.md
- package.json / pom.xml / go.mod / Cargo.toml
- src/ 或 app/ 目录

### 显式调用
```
/skill project-to-resume [options]
```

## 输入参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `target_role` | string | `"backend"` | 目标岗位：frontend / backend / fullstack / custom |
| `language` | string | `"zh"` | 输出语言：zh / en |
| `output_mode` | string | `"html"` | 输出模式：html(完整HTML简历) / project(仅项目经历MD) / full(完整简历MD) |
| `output_path` | string | `"docs/resume.html"` | 输出路径 |
| `highlight_techs` | string[] | `[]` | 重点突出的技术关键词 |
| `max_highlights` | number | `4` | 每个项目最多技术亮点条数 |
| `include_metrics` | boolean | `true` | 是否推断量化指标（标注[需验证]） |
| `project_dirs` | string[] | `["."]` | 要扫描的项目目录列表（支持多项目） |

## 依赖能力

此 Skill 需要以下 Agent 能力：

- ✅ 文件读取 (Read) — 读取源码和配置文件
- ✅ 目录扫描 (Glob/ListDirectory) — 扫描项目结构
- ✅ 内容搜索 (Grep) — 搜索技术关键词
- ✅ 命令执行 (Bash) — 统计代码行数等
- ✅ 文件写入 (Write) — 输出 HTML 简历文件
- ✅ 用户交互 (Chat) — 采集用户个人信息
- ⚠️ Git 历史 (可选) — 分析提交记录

## 工作流程

### 阶段 0: 用户信息采集（首次运行或信息更新）

**目标**: 收集用户个人信息，为完整简历输出做准备。

> **此阶段在 `output_mode=html` 时为必需**，其他模式可跳过。

#### 0.1 检查已有信息

读取 `user-profile/profile.json`（skill 目录下）：
- **如果存在**：展示已有信息摘要给用户，询问是否需要修改
- **如果不存在**：进入信息采集流程

#### 0.2 采集交互

按以下顺序询问用户（每个问题一次对话，避免信息过载）：

```
📝 为了生成完整的 HTML 简历，我需要采集一些个人信息。
   这些信息将保存在本地，后续生成时可直接复用。

1️⃣ 基础信息
   - 姓名：
   - 电话：
   - 邮箱：
   - 微信（可选）：
   - GitHub 地址（可选）：

2️⃣ 求职意向
   - 目标岗位（如"Java 后端开发工程师"）：
   - 到岗时间（如"随时到岗"）：

3️⃣ 教育背景
   - 学校名称：
   - 学校标签（如 985/211/双一流，用逗号分隔，可选）：
   - 专业：
   - 学历（本科/硕士/博士）：
   - 起止时间（如"2023.09 - 2027.06"）：

4️⃣ 证书与荣誉（可选）
   - 如"CET-4、CET-6、XX大赛XX奖"

5️⃣ 照片路径（可选）
   - 提供照片的本地绝对路径

6️⃣ 额外技能补充（可选）
   - 除了项目中扫描到的技能外，是否有需要额外列出的技能？
   - 格式：类别:描述（如"AI应用开发：熟悉LangChain4j..."）
```

#### 0.3 保存信息

将采集到的信息序列化为 JSON，保存到 `user-profile/profile.json`。
文件模板参见 `user-profile/profile-template.json`。

---

### 阶段 1: 项目扫描（事实收集）

**目标**: 收集项目的客观事实数据，不做任何主观推断。

> 如果 `project_dirs` 包含多个目录，则对每个目录分别执行以下步骤。

#### 1.1 项目元信息

读取以下文件获取基础信息：

| 文件 | 提取内容 |
|------|----------|
| `README.md` | 项目描述、功能列表、架构说明 |
| `package.json` / `pom.xml` / `go.mod` | 技术栈、依赖版本 |
| `docker-compose.yml` / `Dockerfile` | 部署方式、中间件组件 |
| `.github/workflows/` / `Jenkinsfile` | CI/CD 配置 |
| `nginx.conf` / `application.yml` | 运行配置 |

#### 1.2 架构信息

- 扫描目录结构，识别分层模式（MVC / DDD / 微服务等）
- 定位核心入口文件
- 识别模块划分方式

#### 1.3 技术深度指标

通过命令和搜索收集可量化数据：

```bash
# 代码行数
find . -name "*.java" | xargs wc -l  # 或对应语言
# API 端点
grep -r "@GetMapping\|@PostMapping\|@RequestMapping" --include="*.java" -c
# 数据表
grep -r "CREATE TABLE\|@Table\|@Entity" --include="*.java" --include="*.sql" -c
# 测试文件
find . -path "*/test*" -name "*Test*.java" | wc -l
```

#### 1.4 技术亮点线索

搜索以下关键模式，作为技术亮点的线索：

| 搜索目标 | 对应亮点方向 |
|----------|-------------|
| Redis / Redisson / Jedis | 缓存策略 / 分布式锁 |
| RabbitMQ / Kafka / RocketMQ | 异步处理 / 消息解耦 |
| Lua 脚本 / `EVAL` | 原子操作 / 高级缓存 |
| `@Transactional` / 分布式事务 | 事务管理 |
| JWT / OAuth / Spring Security | 安全认证 |
| WebSocket / SSE | 实时通信 |
| Elasticsearch | 搜索引擎 |
| Docker / K8s / Compose | 容器化部署 |
| Nginx / 网关 | 反向代理 / 负载均衡 |
| Swagger / OpenAPI | API 文档化 |
| AOP / 切面 / 拦截器 | 横切关注点 |
| 限流 / 熔断 / Sentinel | 高可用 |
| LangChain / RAG / Agent | AI 应用开发 |
| Pgvector / 向量检索 | 向量数据库 |
| MCP / 工具调用 | Agent 工具集成 |

### 阶段 2: 智能提炼（分析推理）

**目标**: 将技术事实转化为简历语言。

#### 2.1 项目定位

用一句话概括项目：`{领域}领域的{产品形态}，解决{核心问题}`

#### 2.2 核心职责提炼

从代码模块中推断职责，规则：
- 动词开头（设计、实现、优化、架构、重构）
- 包含技术方案
- 尽量量化

#### 2.3 技术亮点 — PSR 法

每条技术亮点必须包含：

```
P (Problem)  : 面临什么技术挑战？
S (Solution) : 采用什么技术方案？
R (Result)   : 达到什么技术效果？（可量化）
```

从代码中推断来源：
- P: 从代码注释、TODO、架构选型中识别
- S: 从实际实现代码中提取
- R: 从测试结果、配置参数中推断，不确定的标注 `[需验证]`

#### 2.4 技术栈标签化

按领域分类技术栈：

```
后端: Spring Boot / MyBatis-Plus / ...
前端: Vue.js / React / ...
中间件: Redis / RabbitMQ / ...
数据库: MySQL / MongoDB / ...
DevOps: Docker / Nginx / GitHub Actions / ...
```

#### 2.5 专业技能汇总（html 模式专用）

从所有扫描的项目中汇总技术能力，按类别生成专业技能描述：

1. **自动汇总**：遍历所有项目的技术栈和亮点，按类别聚合
2. **合并用户补充**：将 `profile.json` 中 `custom_skills` 的内容合并
3. **生成描述**：每条技能用 `类别 + 描述` 格式，描述需体现深度而非罗列

技能分类参考：
- AI 应用开发
- 基础与架构
- 数据库与缓存
- 框架应用与原理
- DevOps 与工程化
- Agent 应用

### 阶段 3: 简历生成

**目标**: 按模板输出结构化简历内容。

根据 `output_mode` 参数选择模板：
- `html` → 使用 `templates/resume.html`，输出完整 HTML 简历文件
- `project` → 使用 `templates/project-section.md`
- `full` → 使用 `templates/full-resume.md`

#### 3.1 HTML 模式生成规则

**读取模板文件** `templates/resume.html`，理解其 HTML 结构和 CSS 样式。

**填充数据**时严格遵循以下规则：

1. **个人信息区**：从 `profile.json` 读取，填充 header 部分
   - 姓名 → `.name`
   - 电话/邮箱/微信 → `.contact-info` 的 `copy-item` 按钮
   - 求职意向 + 到岗时间
   - 照片（如有路径）→ `.photo-box img`
   - 无微信/无照片时，删除对应的 HTML 块

2. **教育背景区**：从 `profile.json` 读取
   - 学校名 + badges（每个 badge 一个 `<span class="badge">`）
   - 专业 + 时间
   - 证书（如有）

3. **项目经历区**：从阶段 2 的分析结果生成
   - 每个项目一组 `<h3> + .tech-stack-row + <ul>` 结构
   - 技术栈用 `<span class="tech-tag">` 标签
   - 亮点用 `<li><strong>标题</strong>：描述</li>` 格式
   - 按时间倒序排列（最近的在前）
   - 如有 GitHub 链接，在项目名旁加 `.repo-link`

4. **专业技能区**：从阶段 2.5 的汇总结果生成
   - 每条技能一个 `<li><span class="bold-key">类别：</span>描述</li>`

**输出完整 HTML 文件**到 `output_path`（默认 `docs/resume.html`）。

#### 3.2 排版约束

- 内容必须适合 A4 一页纸（210mm × 296mm）
- 如内容过长，优先精简项目亮点条目数（从 4 条减为 3 条或 2 条）
- 技术栈标签选择最重要的（不超过 8 个/项目）
- 专业技能控制在 4-6 条

### 阶段 4: 质量润色 + 岗位适配

**目标**: 提升简历质量，适配目标岗位。

#### 4.1 岗位适配

根据 `target_role` 调整描述侧重：

| 岗位 | 侧重方向 |
|------|----------|
| frontend | UI 框架、组件化、性能优化、响应式 |
| backend | 架构设计、并发处理、数据库/缓存优化 |
| fullstack | 前后端均衡、系统集成能力 |

#### 4.2 语言润色

- 弱动词 → 强动词（编写→设计，做了→架构，用了→引入）
- 补充量化数据（标注 `[需验证]` 的保留提示）
- 术语规范化（统一大小写、全称/缩写）

#### 4.3 质量检查

执行 `prompts/quality-check.md` 中的检查清单。

#### 4.4 推断数据确认

生成 HTML 后，汇总所有标注了 `[需验证]` 的条目，以列表形式呈现给用户：

```
📋 以下数据为推断值，请逐条确认或修正：

1. "QPS 提升约 5 倍 [需验证]" → 请提供真实压测数据，或确认保留推断值
2. "接口 RT 从 800ms 降至 120ms [需验证]" → 请确认实际数据
3. ...

回复格式：
- 确认：直接回复 "确认" 或 "全部确认"
- 修正：回复 "1. QPS 提升约 3 倍（JMeter 实测）"
- 删除：回复 "删除 2"（移除该条量化数据）
```

用户确认后，**批量移除 `[需验证]` 标注**，更新 HTML 文件。

## 输出规范

### 文件位置
```
docs/resume.html    # HTML 完整简历（默认）
docs/resume-zh.md   # 中文 Markdown 版（可选）
docs/resume-en.md   # 英文 Markdown 版（可选）
```

### HTML 模式必需章节

1. **个人信息头部**（姓名、联系方式、求职意向、照片）
2. **教育背景**（学校 + 标签 + 专业 + 时间 + 证书）
3. **项目经历**（每个项目：名称+角色+时间、技术栈标签、PSR 亮点列表）
4. **专业技能**（按类别分组的技能描述）

### 质量标准

- ✅ 技术栈来自依赖文件，准确无误
- ✅ 每条亮点有 PSR 结构
- ✅ 至少 50% 条目含量化数据
- ✅ 动词有力、术语规范
- ✅ HTML 排版适合 A4 打印
- ✅ 个人信息来自用户填写，非编造
- ❌ 无臆测：推断的数据标注 `[需验证]`
- ❌ 无功能罗列：每条都有技术深度

## 使用示例

### 基础用法（默认 HTML 模式）
```
用户: 帮我根据这个项目生成简历
Agent: [激活 project-to-resume skill]
      → 检查 profile.json
      → 采集/确认用户信息
      → 扫描项目 → 生成 HTML 简历
```

### 多项目简历
```
用户: 帮我根据 ProjectA 和 ProjectB 两个项目生成完整简历
Agent: [激活 skill，project_dirs=["path/to/ProjectA", "path/to/ProjectB"]]
      → 分别扫描两个项目
      → 合并生成完整 HTML 简历
```

### 指定岗位
```
用户: 帮我生成面向后端岗位的 HTML 简历
Agent: [激活 skill，target_role=backend, output_mode=html]
```

### 兼容旧模式（仅项目描述 Markdown）
```
用户: 只需要生成项目经历的 Markdown 描述
Agent: [激活 skill，output_mode=project]
```

### 更新个人信息
```
用户: 更新我的简历信息
Agent: → 读取现有 profile.json，展示摘要
      → 询问用户哪些需要修改
      → 更新保存
```

## 限制与约束

1. **仅基于事实**: 所有技术栈描述必须来自仓库真实文件
2. **诚实标注**: 推断的量化数据必须标注 `[需验证]`
3. **不编造经历**: 不会虚构未在代码中体现的技术使用
4. **个人信息来源**: 必须从 `profile.json` 读取或询问用户，不编造
5. **时间范围**: 无法准确推断项目时间，使用占位符 `{start_date} - {end_date}`
6. **一页纸原则**: HTML 简历内容须控制在 A4 一页纸范围内

## 错误处理

| 场景 | 处理方式 |
|------|----------|
| 仓库为空 | 提示"仓库无内容，无法生成" |
| 缺少依赖文件 | 降级为手动输入技术栈 |
| 无法识别语言 | 列出文件类型，请求用户确认 |
| 技术亮点不足 | 提示用户补充，至少生成基础描述 |
| 项目过于简单 | 提醒用户是否需要扩充功能描述 |
| profile.json 缺失 | 进入信息采集流程，引导用户填写 |
| 内容超出一页 | 自动精简亮点条目数和技术标签数 |

## 版本历史

### v2.1.0 (2026-04-13)
- **规范化**: 添加标准 YAML frontmatter（name + description）
- **跨平台适配**: extractor.md 所有搜索命令改为 Agent 内置工具（grep_search / list_dir）优先，保留 bash + PowerShell 双备选
- **精确化**: AI/Agent/RAG 搜索关键词去除通用词（Agent/Chain/Workflow），改为框架特有类名，标注置信度
- **新增 Step 8**: 推断数据确认步骤 — 生成后引导用户逐条确认 `[需验证]` 标注
- **修复**: profile-template.json `custom_skills` 默认值从空对象改为空数组
- **新增示例**: examples/sample-resume.html 完整可预览示例简历

### v2.0.0 (2026-04-12)
- **重大升级**: 支持完整 HTML 格式简历生成
- 新增用户信息采集与持久化（profile.json）
- 新增 HTML 简历模板（仿照专业简历排版）
- 支持多项目目录扫描
- 增加专业技能自动汇总
- 支持照片、联系方式复制等交互效果
- 保留原有 Markdown 输出模式兼容

### v1.0.0 (2026-04-12)
- 初始版本
- 支持单项目扫描
- 支持中英文输出
- PSR 法技术亮点提炼
- 岗位适配（前端/后端/全栈）
- 项目经历 + 完整简历双模式

## 许可证

MIT License
