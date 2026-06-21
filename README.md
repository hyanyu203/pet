# 宠物领养系统 (Pet Adoption Platform)

一个面向**领养人、救助机构、平台管理员**三类角色的宠物领养全流程平台，覆盖宠物发布与审核、领养申请、电子协议签署、领养后回访追踪、论坛社区、消息通知与数据统计。

技术栈：Spring Boot + MyBatis + MySQL + Redis + MinIO（后端） / Vue 3 + Vite + Element Plus + Pinia（前端）。

> 📖 **整体架构与业务逻辑分析见 [项目架构总览.md](./项目架构总览.md)**。
> 本仓库含**两套后端**：单体 `petback/`（功能完整、当前主版本）与微服务版 `petcloud/`（Spring Cloud Alibaba，按业务边界拆分），二者共用同一个 `pet_adoption` 库。前端默认对接单体。

---

## 功能模块

| 角色 | 主要功能 |
|---|---|
| 领养人 (adopter) | 浏览/搜索宠物、收藏、提交领养申请、签署电子协议、查看领养后回访、论坛发帖评论、消息通知 |
| 救助机构 (organization) | 发布与管理宠物、审核领养申请、管理协议、发起并跟踪领养后回访 |
| 管理员 (admin) | 用户管理、宠物审核、申请/协议管理、举报处理、数据统计与导出（Excel/PDF/Word）、系统配置 |

其他特性：基于用户行为的宠物推荐、登录失败锁定、JWT 鉴权、**注册邮箱验证码（Redis + SMTP）**、图片对象存储（MinIO）、WebSocket 实时消息。

---

## 技术栈

**后端 `petback/`**
- Spring Boot 2.7.14，Java 8
- MyBatis（XML mapper）+ MySQL 8（HikariCP 连接池）
- Spring Security + JWT（jjwt）鉴权，BCrypt 密码加密（兼容历史 MD5 平滑升级）
- Redis（缓存）、MinIO（图片/文件存储）
- Apache POI（Excel/Word 导出）、WebSocket（实时消息）

**前端 `petfront/`**
- Vue 3 + Vite 4
- Element Plus（UI）、Pinia（状态）、Vue Router、Axios

---

## 目录结构

```
pet/
├── petback/        # Spring Boot 后端
│   └── src/main/
│       ├── java/com/xm/pet/
│       │   ├── controller/   # 按角色分包：admin / organization / adopter / user / tracking
│       │   ├── service/      # 业务接口 + impl 实现
│       │   ├── mapper/       # MyBatis Mapper 接口
│       │   ├── pojo/         # 实体
│       │   ├── config/       # Security / CORS / Redis / MinIO / WebSocket
│       │   └── utils/        # JWT / 密码 / MinIO / 算法等工具
│       └── resources/
│           ├── application.yml
│           └── mapper/*.xml  # MyBatis SQL 映射
├── petfront/       # Vue 3 前端
│   └── src/
│       ├── views/      # 页面（按角色分目录）
│       ├── components/ # 复用组件
│       ├── api/        # 接口封装（统一走 api/request.js）
│       ├── stores/     # Pinia store
│       └── router/     # 路由与守卫
├── petcloud/       # 后端微服务版（Spring Cloud Alibaba：Nacos+Gateway+Feign+Sentinel）
│   ├── pet-common / pet-gateway / pet-{user,pet,adoption,forum,message,statistics}-service
│   ├── nacos-config/   # Nacos 配置中心配置（8 个 dataId + dev-all.yaml 总览）
│   ├── docker-compose.yml  # 一键拉起 MySQL+Nacos+Redis+6 服务+网关
│   └── README.md
├── pet_adoption.sql   # 数据库结构 + 演示数据（19 张表，MySQL 8 / utf8mb4）
├── 项目架构总览.md     # ⭐ 整体架构 + 业务逻辑分析（先看这个）
└── README.md
```

---

## 环境要求

| 组件 | 版本 |
|---|---|
| JDK | 1.8 |
| Maven | 3.6+ |
| Node.js | 16+（建议 18+） |
| MySQL | 8.0+ |
| Redis | 5+ |
| MinIO | 任意近期版本 |

---

## 快速开始

### 1. 初始化数据库

```bash
# 创建数据库并导入结构与演示数据
mysql -u root -p -e "CREATE DATABASE pet_adoption DEFAULT CHARSET utf8mb4 COLLATE utf8mb4_0900_ai_ci;"
mysql -u root -p pet_adoption < pet_adoption.sql
```

### 2. 启动依赖中间件

确保本地（或远程）已启动 MySQL、Redis、MinIO，并在 MinIO 中创建名为 `pet-adoption` 的 bucket。

### 3. 配置后端环境变量

后端 `application.yml` 全部通过环境变量注入，**无硬编码凭据**。生产环境务必覆盖以下变量：

| 变量 | 说明 | 默认值 |
|---|---|---|
| `DB_URL` | 数据库连接 | `jdbc:mysql://localhost:3306/pet_adoption?...` |
| `DB_USERNAME` / `DB_PASSWORD` | 数据库账号密码 | `root` / 空 |
| `REDIS_HOST` / `REDIS_PORT` / `REDIS_PASSWORD` | Redis | `localhost` / `6379` / 空 |
| `MINIO_ENDPOINT` / `MINIO_ACCESS_KEY` / `MINIO_SECRET_KEY` / `MINIO_BUCKET` | MinIO | `http://localhost:9005` / `minioadmin` / `minioadmin` / `pet-adoption` |
| `JWT_SECRET` | JWT 签名密钥（**生产必须修改**） | 占位字符串 |
| `JWT_EXPIRATION` | Token 有效期（毫秒） | `86400000`（24h） |

### 4. 启动后端

```bash
cd petback
mvn spring-boot:run           # 默认端口 8082
```

### 5. 启动前端

```bash
cd petfront
npm install
npm run dev                   # 默认 Vite 端口
```

---

## 演示账号

导入演示数据后可直接登录（**密码统一为 `123456`**）：

| 角色 | 用户名 |
|---|---|
| 管理员 | `admin01` |
| 救助机构 | `org01` ~ `org05` |
| 领养人 | `adopter01` ~ `adopter20` |

> 演示数据中的密码以 MD5 存储，首次登录成功后会被后端自动平滑升级为 BCrypt。

---

## 数据库说明

- 共 19 张表，引擎 InnoDB，字符集 `utf8mb4`，排序规则统一为 `utf8mb4_0900_ai_ci`。
- 含 32 个外键约束，核心表（user / pet / adoption_application / adoption_agreement / forum_* / tracking_* 等）均建有针对查询场景的复合索引。

---

## 安全提示

- 演示数据使用占位手机号、`example.com` 邮箱，**不含真实个人信息**。
- 默认 `JWT_SECRET`、MinIO `minioadmin` 等仅供本地开发，生产环境务必通过环境变量替换。
- 真实凭据请放入 `.env` 或 `application-local.yml`（已在 `.gitignore` 中忽略，不会提交）。

---

## 注册邮箱验证码

注册需先获取邮箱验证码（单体与微服务均已实现）：
- `POST /api/user/send-email-code` `{ "email": "..." }` —— 6 位码存 Redis 5 分钟、同邮箱 60 秒限一次
- `POST /api/user/register` —— 请求体多带 `"emailCode": "123456"`

需配置 SMTP：环境变量 `MAIL_USERNAME`（发件邮箱）、`MAIL_PASSWORD`（邮箱**授权码**，非登录密码）。详见 `邮箱验证码方案.md`。

---

## 微服务版（petcloud）

Spring Cloud Alibaba 实现，6 个业务服务 + 网关 + 公共库，共用单库 `pet_adoption`。一键启动：

```bash
cd petcloud
docker compose up --build -d     # MySQL + Nacos + Redis + 配置自动导入 + 6 服务 + 网关
# 入口：网关 http://localhost:8080  |  Nacos 控制台 http://localhost:8848/nacos
```

各服务接口文档：`http://localhost:8081~8086/swagger-ui.html`。详见 `petcloud/README.md`、`微服务拆分方案.md`。

---

## 文档

| 文档 | 内容 |
|---|---|
| [项目架构总览.md](./项目架构总览.md) | ⭐ 整体架构 + 业务逻辑分析 |
| [重构方案.md](./重构方案.md) | 单体重构计划与进展 |
| [JDK8兼容性报告.md](./JDK8兼容性报告.md) | Java 21→1.8 降级 |
| [SQL分析报告.md](./SQL分析报告.md) / [数据库优化方案.md](./数据库优化方案.md) | 数据库分析与优化 |
| [微服务拆分方案.md](./微服务拆分方案.md) | Spring Cloud Alibaba 拆分设计 |
| [邮箱验证码方案.md](./邮箱验证码方案.md) | 注册邮箱验证码方案 |

---

## 开发规范

本仓库附带 `CLAUDE.md`，为使用 AI 辅助编码时的协作规范（精简改动、避免过度抽象等），非项目功能说明。
