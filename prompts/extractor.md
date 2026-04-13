# Extractor Prompt: 项目信息提取框架

## 目的

本提示词定义了从项目源码中提取结构化信息的完整流程和数据模型。是阶段 1（项目扫描）的执行指南。

---

## 提取数据模型

以下是需要收集的完整数据结构：

```yaml
project_info:
  # === 基础信息 ===
  name: ""                    # 项目名称（从 README 或包名）
  description: ""             # 一句话描述
  repo_url: ""                # 仓库地址（如有）
  
  # === 技术栈 ===
  tech_stack:
    language: ""              # 主语言
    framework: ""             # 核心框架
    backend: []               # 后端技术
    frontend: []              # 前端技术
    middleware: []             # 中间件（Redis/MQ/ES 等）
    database: []              # 数据库
    devops: []                # DevOps 工具
    testing: []               # 测试框架
  
  # === 架构信息 ===
  architecture:
    pattern: ""               # 架构模式（MVC/DDD/微服务/单体）
    layers: []                # 分层结构
    modules: []               # 模块列表
    entry_points: []          # 入口文件
  
  # === 量化指标 ===
  metrics:
    code_lines: 0             # 核心代码行数
    api_count: 0              # API 端点数量
    table_count: 0            # 数据表数量
    test_count: 0             # 测试文件数量
    module_count: 0           # 模块数量
    dependency_count: 0       # 依赖项数量
  
  # === 技术亮点线索 ===
  highlight_clues: []         # 列表，每项包含 { pattern, files, category }
  
  # === 功能列表 ===
  features: []                # 从 README 提取的功能列表
```

---

## 提取步骤

### Step 1: 读取项目元信息

**目标文件与提取内容**：

| 文件 | 要提取的字段 | 提取方法 |
|------|-------------|----------|
| `README.md` | name, description, features | 读取标题和功能列表部分 |
| `package.json` | name, dependencies, devDependencies, scripts | JSON 解析 |
| `pom.xml` | groupId, artifactId, dependencies | XML 节点提取 |
| `go.mod` | module, require | 文本解析 |
| `Cargo.toml` | [package].name, [dependencies] | TOML 解析 |
| `build.gradle` | dependencies | Groovy/Kotlin DSL |
| `requirements.txt` / `Pipfile` | 依赖列表 | 逐行读取 |

**技术栈分类规则**：

```
后端框架判定:
  - Spring Boot / Spring Cloud → Java 后端
  - Express / Koa / NestJS → Node.js 后端
  - Django / Flask / FastAPI → Python 后端
  - Gin / Echo / Fiber → Go 后端
  - Actix / Axum → Rust 后端

前端框架判定:
  - Vue.js / Nuxt → Vue 生态
  - React / Next.js → React 生态
  - Angular → Angular 生态
  - uni-app / Taro → 跨端框架

中间件判定:
  - spring-boot-starter-data-redis / ioredis → Redis
  - spring-rabbit / amqplib → RabbitMQ
  - spring-kafka / kafkajs → Kafka
  - spring-boot-starter-data-elasticsearch → Elasticsearch
  - spring-boot-starter-websocket / socket.io → WebSocket

数据库判定:
  - mysql-connector / mysql2 → MySQL
  - mongodb / mongoose → MongoDB
  - postgresql / pg → PostgreSQL

DevOps 判定:
  - Dockerfile → Docker
  - docker-compose.yml → Docker Compose
  - .github/workflows/ → GitHub Actions
  - Jenkinsfile → Jenkins
  - nginx.conf → Nginx
```

### 跨平台工具说明

> **重要**：本 Skill 的命令示例提供了 bash 和 PowerShell 两种写法。
> 但在 Agent 环境中（如 Antigravity / Gemini Code Assist），**应优先使用内置工具**而非 shell 命令：
>
> | 需求 | 推荐使用 |
> |------|----------|
> | 目录扫描 | `list_dir` 工具 |
> | 关键词搜索 | `grep_search` 工具 |
> | 文件读取 | `view_file` 工具 |
> | 代码行数统计 | `run_command` + 对应 shell 命令 |
>
> 仅在内置工具无法满足时（如复杂正则统计），才 fallback 到 shell 命令。

### Step 2: 扫描目录结构

**推荐方式（Agent 内置工具）**：
```
使用 list_dir 工具扫描项目根目录，递归查看子目录结构。
自动排除 node_modules、.git、target、dist、.idea、__pycache__ 等目录。
```

**备选方式（Shell 命令）**：

```bash
# Linux/macOS
find . -type d \
  -not -path "*/node_modules/*" \
  -not -path "*/.git/*" \
  -not -path "*/target/*" \
  -not -path "*/dist/*" \
  -not -path "*/.idea/*" \
  -not -path "*/__pycache__/*" \
  | head -50
```

```powershell
# Windows PowerShell
Get-ChildItem -Recurse -Directory |
  Where-Object { $_.FullName -notmatch 'node_modules|\.git|target|dist|\.idea|__pycache__' } |
  Select-Object -First 50 -ExpandProperty FullName
```

**架构模式识别规则**：

| 目录特征 | 推断模式 |
|----------|----------|
| `controller/ service/ dao/` 或 `mapper/` | MVC 分层 |
| `domain/ application/ infrastructure/` | DDD 领域驱动 |
| `api-gateway/ service-*/ common/` | 微服务架构 |
| `src/main/ src/test/` | Maven 标准结构 |
| `components/ pages/ store/` | 前端 SPA |
| `cmd/ internal/ pkg/` | Go 标准布局 |

### Step 3: 收集量化指标

> 优先使用 `grep_search` 内置工具进行搜索统计，无需执行 shell 命令。
> 如需精确的行数统计，可 fallback 到 shell 命令。

**API 端点统计**：

使用 `grep_search` 工具（推荐）：
```
# Java (Spring) — 搜索 API 注解
Query: "@(Get|Post|Put|Delete|Request)Mapping"  IsRegex: true  Includes: ["*.java"]

# Node.js — 搜索路由定义
Query: "router\.(get|post|put|delete)"  IsRegex: true  Includes: ["*.js", "*.ts"]

# Python — 搜索路由装饰器
Query: "@(app|router)\.(route|get|post|put|delete)"  IsRegex: true  Includes: ["*.py"]

# Go — 搜索路由注册
Query: "\.(GET|POST|PUT|DELETE|Handle)"  IsRegex: true  Includes: ["*.go"]
```

备选 shell 命令：
```bash
# Linux/macOS
grep -rn "@GetMapping\|@PostMapping\|@PutMapping\|@DeleteMapping\|@RequestMapping" \
  --include="*.java" | wc -l
```
```powershell
# Windows PowerShell
(Get-ChildItem -Recurse -Include *.java | Select-String -Pattern '@(Get|Post|Put|Delete|Request)Mapping').Count
```

**数据表统计**：

使用 `grep_search` 工具（推荐）：
```
# 从 SQL 文件
Query: "CREATE TABLE"  Includes: ["*.sql"]

# 从 JPA Entity
Query: "@(Entity|Table)"  IsRegex: true  Includes: ["*.java"]

# 从 ORM Model
Query: "class.*(Model|Entity)|tableName"  IsRegex: true  Includes: ["*.py", "*.js", "*.ts"]
```

**代码行数统计**（核心代码）：

> 此项建议使用 shell 命令，内置工具不适合做行数统计。

```bash
# Linux/macOS
find . -name "*.java" -o -name "*.js" -o -name "*.ts" -o -name "*.py" -o -name "*.go" |
  xargs grep -cv "^\s*$\|^\s*//\|^\s*\*\|^\s*#" | tail -1
```
```powershell
# Windows PowerShell
(Get-ChildItem -Recurse -Include *.java,*.js,*.ts,*.py,*.go |
  Get-Content | Where-Object { $_ -match '\S' -and $_ -notmatch '^\s*(//|\*|#)' }).Count
```

### Step 4: 搜索技术亮点线索

> 以下搜索推荐使用 `grep_search` 内置工具，每条 Query 作为一次工具调用。
> 设置 `MatchPerLine: true` 可查看匹配行内容和上下文。

**搜索清单**（按类别）：

#### 缓存与性能

```
Query: "RedisTemplate|StringRedisTemplate|@Cacheable|ioredis|redis\.createClient"  IsRegex: true
Includes: ["*.java", "*.js", "*.ts", "*.py"]

Query: "BloomFilter|布隆过滤器|bloomfilter"  IsRegex: true
Includes: ["*.java", "*.js", "*.ts"]

Query: "caffeine|LoadingCache|Guava.*Cache|lru-cache"  IsRegex: true
Includes: ["*.java", "*.js", "*.ts"]
```

#### 消息队列

```
Query: "RabbitTemplate|@RabbitListener|amqp|KafkaTemplate|@KafkaListener"  IsRegex: true
Includes: ["*.java", "*.js", "*.ts"]
```

#### 并发控制

```
Query: "RLock|Redisson|分布式锁|SETNX|setIfAbsent"  IsRegex: true
Includes: ["*.java", "*.js", "*.ts"]

Query: "EVAL|EVALSHA|\.lua\b"  IsRegex: true
Includes: ["*.java", "*.js", "*.lua"]

Query: "@Transactional|事务|Seata|TCC"  IsRegex: true
Includes: ["*.java"]
```

#### 安全认证

```
Query: "JWT|JwtUtil|jsonwebtoken|SecurityConfig|OAuth|Spring Security"  IsRegex: true
Includes: ["*.java", "*.js", "*.ts"]
```

#### 实时通信

```
Query: "WebSocket|SockJS|STOMP|socket\.io|ServerSentEvent|SSE"  IsRegex: true
Includes: ["*.java", "*.js", "*.ts"]
```

#### 搜索引擎

```
Query: "ElasticsearchRestTemplate|@Document|elasticsearch|elastic"  IsRegex: true
Includes: ["*.java", "*.js", "*.ts"]
```

#### 高可用

```
Query: "RateLimiter|Sentinel|Hystrix|CircuitBreaker|限流|熔断"  IsRegex: true
Includes: ["*.java", "*.js", "*.ts"]
```

#### AOP / 横切

```
Query: "@Aspect|@Around|@Before|@After|拦截器|Interceptor"  IsRegex: true
Includes: ["*.java", "*.js", "*.ts"]
```

#### AI / Agent / RAG

> ⚠️ 注意：Agent、Chain、Workflow 等通用词容易产生大量误报。
> 以下关键词已做精确化处理，优先搜索框架特有的类名和注解。

```
# LLM 框架（高置信度）
Query: "LangChain|langchain4j|ChatModel|AiService|@SystemMessage|ChatLanguageModel"  IsRegex: true
Includes: ["*.java", "*.py", "*.js", "*.ts"]

# RAG / 向量检索（高置信度）
Query: "EmbeddingModel|EmbeddingStore|VectorStore|Pgvector|pgvector|ContentRetriever"  IsRegex: true
Includes: ["*.java", "*.py", "*.js", "*.ts"]

# MCP / 工具调用（高置信度）
Query: "McpServer|McpClient|ToolProvider|@Tool|FunctionCall|ToolSpecification"  IsRegex: true
Includes: ["*.java", "*.py", "*.js", "*.ts"]

# Agent 工作流（中置信度 — 需人工复核结果）
Query: "AgentExecutor|ToolExecutor|AiAgent|ReActAgent|AgentWorkflow"  IsRegex: true
Includes: ["*.java", "*.py", "*.js", "*.ts"]
```

---

## Step 5: 专业技能汇总提取（HTML 模式专用）

当 `output_mode=html` 时，在所有项目扫描完成后，需要额外执行专业技能汇总。

### 5.1 汇总逻辑

遍历所有已扫描项目的 `project_info`，按以下规则聚合技能：

```yaml
skill_summary:
  categories:
    - category: "AI 应用开发"
      techs: ["LangChain4j", "RAG", "Pgvector", "Agent", "MCP"]
      depth_clues: ["从 highlight_clues 中提取相关深度线索"]
    - category: "基础与架构"
      techs: ["Java", "JVM", "JUC", "DDD", "设计模式"]
      depth_clues: []
    - category: "数据库与缓存"
      techs: ["MySQL", "Redis", "Lua"]
      depth_clues: []
    - category: "框架应用与原理"
      techs: ["Spring Boot", "MyBatis-Plus", "IoC/DI", "AOP"]
      depth_clues: []
    - category: "DevOps 与工程化"
      techs: ["Docker", "Nginx", "CI/CD"]
      depth_clues: []
```

### 5.2 技能分类规则

| 发现的技术 | 归入类别 |
|-----------|---------|
| LangChain, RAG, 向量检索, Agent, MCP | AI 应用开发 |
| Java 核心, JVM, JUC, 设计模式, DDD | 基础与架构 |
| MySQL, Redis, MongoDB, ES, Lua | 数据库与缓存 |
| Spring Boot, MyBatis, Spring Security | 框架应用与原理 |
| Docker, K8s, Nginx, CI/CD, 监控 | DevOps 与工程化 |
| Vue, React, 小程序, 跨端 | 前端开发 |

### 5.3 描述生成规则

每条技能描述要求：
- **不是简单罗列技术名词**，要体现使用深度
- 包含"熟悉/熟练/深入理解"等程度词
- 结合项目中实际使用的场景进行描述
- 长度适中（一行以内，简洁有力）

**示例**：
```
AI 应用开发：熟悉 LangChain4j 开发框架，熟悉 Agent 核心工作流的编排引擎设计；
深入理解 RAG 架构，熟练使用 PostgreSQL+Pgvector 进行向量检索。

数据库与缓存：深入理解 MySQL；深度应用 Redis，除常规缓存方案外，能熟练运用
高级数据结构与 Lua 脚本解决 Feed 流推送、海量统计等复杂场景挑战。
```

### 5.4 合并用户自定义技能

读取 `user-profile/profile.json` 中的 `custom_skills` 字段：
- 如果用户提供了自定义技能条目，将其合并到对应类别中
- 如果类别不存在，新增类别
- 用户自定义内容优先级高于自动生成

---

## 输出格式

提取完成后，以 YAML 格式输出结构化数据，供后续阶段使用。
对于未找到的字段，设为 `null` 或 `0`，不做推断。

**示例**：

```yaml
project_info:
  name: "campus-market"
  description: "面向高校用户的二手商品交易平台"
  tech_stack:
    language: "Java"
    framework: "Spring Boot 3.x"
    backend: ["Spring Boot 3.2", "MyBatis-Plus 3.5", "Spring Security"]
    frontend: ["Vue 3", "Vant 4"]
    middleware: ["Redis 7.x", "RabbitMQ 3.x"]
    database: ["MySQL 8.0"]
    devops: ["Docker", "Docker Compose", "GitHub Actions", "Nginx"]
    testing: ["JUnit 5", "Mockito"]
  architecture:
    pattern: "MVC 分层"
    layers: ["Controller", "Service", "Mapper", "Entity"]
    modules: ["user", "product", "order", "chat", "common"]
    entry_points: ["src/main/java/com/.../Application.java"]
  metrics:
    code_lines: 12500
    api_count: 45
    table_count: 12
    test_count: 23
    module_count: 5
    dependency_count: 28
  highlight_clues:
    - pattern: "Redis + Lua 库存扣减"
      files: ["SeckillService.java", "stock-deduct.lua"]
      category: "高并发"
    - pattern: "RabbitMQ 异步下单"
      files: ["OrderMQConsumer.java", "SeckillService.java"]
      category: "消息队列"
    - pattern: "JWT + Spring Security"
      files: ["JwtUtil.java", "SecurityConfig.java"]
      category: "安全认证"
  features:
    - "商品发布与搜索"
    - "即时通讯"
    - "在线交易"
    - "秒杀抢购"

# 多项目场景下的汇总数据（HTML 模式）
skill_summary:
  categories:
    - category: "基础与架构"
      description: "扎实的 Java 核心基础，深入理解 JVM 原理与 JUC 并发编程；具备良好的工程化思维，熟练应用 DDD 架构思想。"
    - category: "数据库与缓存"
      description: "深入理解 MySQL；深度应用 Redis，能熟练运用高级数据结构与 Lua 脚本解决复杂场景挑战。"
    - category: "框架应用与原理"
      description: "熟练使用 Spring Boot 体系，深度掌握 IoC/DI、AOP 及常用注解背后的实现逻辑。"
```
