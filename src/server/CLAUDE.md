# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Memobase 是一个基于用户画像的长期记忆系统，专为 LLM 应用设计。该项目采用多语言、多架构设计，支持 Python、TypeScript/JavaScript 和 Go 客户端。项目支持多种嵌入服务（OpenAI、Ollama、Jina），使用 PostgreSQL + pgvector 进行向量存储，Redis 作为缓存层。

## 开发环境设置

### 快速启动（使用 Docker Compose）

```bash
# 当前工作目录已经是 src/server
# 配置环境变量
cp .env.example .env
# 编辑 .env 文件设置必要配置（数据库密码、访问令牌等）

# 启动所有服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f
```

### 服务组件

- **API 服务**: 端口 8019 (默认)
- **PostgreSQL 数据库**: 端口 15432 (默认)
- **Redis 缓存**: 端口 16379 (默认)
- **MCP 服务**: 端口 8050
- **Inspector 管理界面**: 端口 3000 (默认)

## 常用开发命令

### Python 服务器端 (当前目录下的 `api/`)

```bash
# 进入 API 目录
cd api

# 安装依赖
pip install -e .[dev]

# 运行测试
pytest

# 运行特定测试
pytest tests/test_api.py

# 运行测试并生成覆盖率报告
pytest --cov=api tests/

# 数据库迁移
alembic upgrade head

# 创建新的数据库迁移
alembic revision --autogenerate -m "描述"
```

**注意**: 服务器需要 Python 3.12+

### 其他客户端（从当前目录的相对路径）

#### Python 客户端 (`../client/memobase`)

```bash
# 进入客户端目录
cd ../client/memobase

# 安装依赖
pip install -e .

# 运行测试
pytest tests/
```

#### TypeScript/JavaScript 客户端 (`../client/memobase-ts`)

```bash
# 进入客户端目录
cd ../client/memobase-ts

# 安装依赖
npm install

# 构建
npm run build

# 运行测试
npm test

# 代码检查
npm run lint

# 格式化代码
npm run fix
```

#### Go 客户端 (`../client/memobase-go`)

```bash
# 进入客户端目录
cd ../client/memobase-go

# 运行测试
go test ./...

# 运行特定包的测试
go test ./core

# 构建
go build ./...
```

#### MCP 服务 (`../mcp`)

```bash
# 进入 MCP 目录
cd ../mcp

# 安装依赖
pip install -e .

# 运行 MCP 服务
mcp install .
mcp run memobase-mcp
```

**注意**: MCP 服务需要 Python 3.11+

## 项目架构

### 项目整体结构（从当前目录的相对路径）

```
../                    # 项目根目录 memobase/
├── client/            # 客户端 SDK
│   ├── memobase/      # Python 客户端
│   ├── memobase-ts/   # TypeScript/JavaScript 客户端
│   └── memobase-go/   # Go 客户端
├── server/            # 服务器端（当前目录）
│   └── api/           # FastAPI 后端服务源码
├── mcp/               # Model Context Protocol 实现
├── memobase-inspector/ # Inspector 管理界面
├── docs/              # 文档
└── .github/           # GitHub 配置
```

### 当前目录结构

```
./                    # src/server (当前目录)
├── api/               # FastAPI 后端服务源码
│   ├── memobase_server/  # 主要代码
│   ├── tests/            # 测试文件
│   ├── alembic/          # 数据库迁移
│   ├── pyproject.toml    # Python 项目配置
│   └── config.yaml.example  # 配置文件模板
├── db/                # 数据库持久化目录
├── docker-compose.yml # Docker Compose 配置
├── .env.example       # 环境变量模板
├── README.md          # 服务器说明文档
└── CLAUDE.md          # 本文件
```

### 核心组件

1. **API 服务** (`api/`):
   - 基于 FastAPI 的 Python 后端（需要 Python 3.12+）
   - 使用 PostgreSQL + pgvector 进行向量存储
   - Redis 用于缓存
   - 支持 OpenAI、Ollama、Jina 等多种嵌入服务
   - 使用 SQLAlchemy 2.0+ ORM 和 Alembic 进行数据库迁移
   - 集成 OpenTelemetry 遥测和 Prometheus 指标
   - 使用 structlog 进行结构化日志记录

2. **客户端 SDK**:
   - **Python**: 主要的客户端库，提供完整的 API 接口
   - **TypeScript**: 支持 npm 和 jsr 包管理，使用 Jest 测试框架
   - **Go**: 高性能的客户端实现（Go 1.22.3+）

3. **MCP 服务**: 实现 Model Context Protocol，提供标准化的内存访问接口（需要 Python 3.11+）

4. **Inspector 管理界面**: 基于 Next.js 的 Web 管理界面，用于管理和查看 Memobase 数据

### 环境变量配置

必需的环境变量（见 `.env.example`）：
- `DATABASE_NAME`: 数据库名称
- `DATABASE_USER`: 数据库用户名
- `DATABASE_PASSWORD`: 数据库密码
- `REDIS_PASSWORD`: Redis 密码
- `PROJECT_ID`: 项目 ID
- `ACCESS_TOKEN`: 访问令牌

可选配置：
- `OPENAI_API_KEY`: OpenAI API 密钥
- `OPENAI_BASE_URL`: OpenAI API 基础 URL
- `API_EXPORT_PORT`: API 服务端口（默认 8019）
- `INSPECTOR_EXPORT_PORT`: Inspector 端口（默认 3000）

### 配置文件

除了环境变量，还需要配置 `api/config.yaml`（从 `api/config.yaml.example` 复制）：
```yaml
# LLM 配置
llm_style: "openai"
llm_api_key: [API密钥]
llm_base_url: [API基础URL]
best_llm_model: [模型名称]

# 嵌入配置
enable_event_embedding: true
embedding_provider: "openai"  # 支持 openai、ollama、jina
embedding_api_key: [API密钥]
embedding_base_url: [API基础URL]
embedding_dim: 1024
embedding_model: "BAAI/bge-m3"
```

## 测试

### 测试框架
- **Python**: pytest + pytest-asyncio
- **TypeScript**: Jest
- **Go**: 标准库 testing

### 测试位置（从当前目录的相对路径）
- Python 服务器测试: `api/tests/`
- Python 客户端测试: `../client/memobase/tests/`
- TypeScript 测试: `../client/memobase-ts/tests/`
- Go 测试: 与源代码在同一包内 (`../client/memobase-go/`)

## 部署

### 生产环境部署

```bash
# 拉取最新代码
git pull

# 重新构建并启动服务
docker-compose up --build -d
```

### 开发模式

对于 Inspector 界面的开发：
```bash
cd ../memobase-inspector
pnpm install
pnpm dev
```

### 项目依赖版本

- **Python 服务器**: Python 3.12+
- **Python MCP**: Python 3.11+
- **Go 客户端**: Go 1.22.3+
- **TypeScript 客户端**: Node.js (支持 npm 和 jsr)
- **数据库**: PostgreSQL + pgvector 扩展
- **缓存**: Redis 7.4+

## 数据库操作

- **数据库迁移**: 使用 Alembic 管理数据库版本
- **向量搜索**: 使用 pgvector 扩展进行高效向量检索
- **数据持久化**: 所有数据通过 Docker volumes 持久化

## 认证和权限

- 使用 PROJECT_ID 和 ACCESS_TOKEN 进行身份验证
- API 请求需要在 header 中包含认证信息

## 监控和遥测

- 使用 OpenTelemetry 进行应用遥测
- Prometheus 指标导出
- 结构化日志（使用 structlog）

## 关键架构模式

### 用户画像系统
- 支持动态用户画像生成和管理
- 多语言支持（中文/英文）
- 画像合并和主题组织

### 事件记忆系统
- 基于嵌入的事件检索
- 支持事件标签和时间范围过滤
- 向量相似度搜索

### 认证和权限
- 基于 PROJECT_ID 和 ACCESS_TOKEN 的认证
- API 请求头部认证

## 故障排除

1. **端口冲突**: 修改 `.env` 文件中的端口配置
2. **权限问题**: `sudo chown -R $USER:$USER ./db`
3. **服务无法启动**: 查看日志 `docker-compose logs [service-name]`
4. **重新构建**: `docker-compose build --no-cache [service-name]`
5. **Python 版本问题**: 确保服务器使用 Python 3.12+，MCP 服务使用 Python 3.11+