# BeatCut AI 技术栈建议

## 1. 文档目标

本文档基于 `memory-bank/design_document.md` 的产品边界与内部 Demo 目标，为 BeatCut AI 推荐一套最合适的技术栈。

选型原则如下：

- 简单，优先选择成熟且上手快的方案
- 健壮，优先保证长任务、视频处理和失败重试的稳定性
- 可扩展，后续能平滑升级为更完整的产品架构
- 适合内部 Demo，不引入过早复杂度

## 2. 推荐结论

BeatCut AI 内部 Demo 最合适的技术栈建议为：

- 前端：`Next.js + TypeScript + Tailwind CSS + shadcn/ui`
- 后端 API：`FastAPI`
- 异步任务：`Celery + Redis`
- 核心媒体处理：`FFmpeg`
- 音乐分析：`librosa`
- 视频分析：`PySceneDetect + OpenCV`
- 主体/人脸检测：`MediaPipe + OpenCV`，必要时补充 `YOLOv8`
- 数据存储：`PostgreSQL`
- 文件存储：本地磁盘开发，后续可切 `S3 / MinIO`
- 部署方式：`Docker Compose`
- 监控与日志：`structlog + Sentry`

这套组合的核心优点是：

- Python 一套语言即可打通 API、任务队列、音视频分析和渲染流程
- FFmpeg 生态成熟，适合做视频导出、拼接、滤镜和转码
- Celery + Redis 对长任务、重试、状态跟踪足够稳定
- Next.js 足以完成 Demo 所需的上传、状态页、预览和结果页
- 整体学习成本和实现复杂度都可控

## 3. 为什么这是最合适的方案

## 3.1 这个项目的本质不是“高并发 Web”，而是“长耗时媒体处理”

BeatCut AI 的核心难点不在普通 CRUD，而在：

- 文件上传
- 视频转码
- 音乐分析
- 视频镜头切分
- 片段评分
- 时间线生成
- 视频渲染导出

因此技术栈应优先服务“异步任务链路稳定”，而不是优先追求超轻量 API 框架或前后端分离复杂度。

## 3.2 Python 是最自然的主语言

设计文档里的核心处理能力几乎都天然偏 Python 生态：

- `librosa`
- `OpenCV`
- `PySceneDetect`
- 视觉分析模型
- FFmpeg 调度脚本

如果后端不用 Python，最终仍然要额外维护一套 Python worker，反而增加复杂度。

## 3.3 Celery 比“自己手搓后台任务”更稳

内部 Demo 虽然不大，但视频生成流程天然是长任务，且有这些需求：

- 多阶段状态更新
- 失败重试
- 中断后不影响主服务
- 能够查询任务状态
- 未来能扩展并发任务

相比自己用 `BackgroundTasks`、线程池或临时脚本硬撑，`Celery + Redis` 更稳，也更容易留出后续演进空间。

## 4. 分层技术栈建议

## 4.1 前端

### 推荐方案

- `Next.js`
- `TypeScript`
- `Tailwind CSS`
- `shadcn/ui`
- `React Hook Form`
- `Zod`
- `TanStack Query`

### 选型理由

#### Next.js

适合内部 Demo 的原因：

- 上手快
- 路由和页面组织清晰
- 适合做上传页、参数页、生成页、结果页
- 后续若需要简单服务端接口也容易补充

#### TypeScript

虽然 Demo 可以只用 JavaScript，但这里推荐直接用 TypeScript，原因是：

- 上传任务状态、参数结构、结果结构都较明确
- 能减少前后端联调阶段的低级错误
- 后续扩展参数和状态机更稳

#### Tailwind CSS

适合当前项目，因为：

- 页面数量不多
- 需要快速搭建高可用 UI
- 状态页、表单页和结果页都偏业务型布局

#### shadcn/ui

适合作为轻量组件层：

- 不重
- 可控
- 和 Tailwind 配合自然
- 适合构建上传卡片、表单、提示框、按钮、进度组件

#### React Hook Form + Zod

用于参数表单和上传校验，优势是：

- 表单状态简单
- 验证规则清晰
- 可以与后端参数结构保持一致

#### TanStack Query

用于：

- 上传后的状态查询
- 生成任务轮询
- 结果获取

比手写请求状态更稳，也更适合任务状态页面。

### 不推荐的替代方案

- 不建议用纯 HTML 模板：后续状态管理和交互会变乱
- 不建议一开始就上复杂组件库如 Ant Design：体量偏重，设计风格也未必匹配
- 不建议为 Demo 引入 Electron：当前内部 Demo 更适合先做 Web

## 4.2 后端 API

### 推荐方案

- `FastAPI`
- `Pydantic`
- `Uvicorn`
- `SQLAlchemy 2.0`

### 选型理由

#### FastAPI

是最适合这个项目的后端入口框架，原因如下：

- Python 生态适配最好
- 请求模型、响应模型定义直观
- 文件上传支持好
- 后续拆任务接口、状态接口、结果接口都自然
- 自动生成 OpenAPI，利于前后端联调

#### Pydantic

用于定义：

- 上传响应结构
- 参数配置结构
- 任务状态结构
- 结果元数据结构

这样能显著减少接口对接歧义。

#### SQLAlchemy 2.0

用于管理任务、素材、导出记录等表结构，理由是：

- 成熟稳定
- 与 FastAPI 配合成熟
- 后续切换数据库压力小

## 4.3 异步任务系统

### 推荐方案

- `Celery`
- `Redis`

### 任务拆分建议

建议拆成以下任务阶段：

1. 素材预处理
2. 音乐分析
3. 视频镜头切分
4. 片段评分
5. 时间线规划
6. 渲染导出
7. 清理中间文件

### 选型理由

#### Celery

适合长耗时视频任务的原因：

- 成熟
- 重试机制完善
- 可拆分任务链
- 可记录进度
- Worker 可与 API 服务隔离

#### Redis

在当前阶段最合适作为：

- Celery broker
- Celery backend
- 临时任务状态缓存

优点是简单、成熟、部署成本低。

### 为什么不推荐更复杂的方案

- 不建议一开始就上 Kafka：明显过重
- 不建议一开始就上 Temporal：很强，但对当前 Demo 学习和接入成本过高
- 不建议只靠数据库轮询充当任务队列：实现简单但可靠性和可维护性差

## 4.4 数据存储

### 推荐方案

- `PostgreSQL`

### 存储内容

- 素材记录
- 生成任务记录
- 参数快照
- 输出文件元数据
- 状态流转记录

### 选型理由

虽然内部 Demo 用 SQLite 也能跑，但这里仍建议直接用 PostgreSQL，原因是：

- 多进程和异步任务场景更稳
- 后续更容易扩展
- 对任务状态、素材记录和日志结构更友好

### 是否可以先用 SQLite

可以作为本地个人开发临时方案，但不建议作为团队 Demo 环境默认方案。

如果你希望最省事：

- 本地开发：`SQLite`
- 团队联调和 Demo 环境：`PostgreSQL`

## 4.5 文件存储

### 推荐方案

- 本地开发：项目本地文件夹
- Demo 环境：`MinIO` 或 `S3`

### 建议目录结构

```text
storage/
  uploads/
    videos/
    bgm/
  jobs/
    {job_id}/
      clips/
      analysis/
      render/
  outputs/
```

### 选型理由

视频、音频和中间产物都比较大，不适合直接存数据库。文件存储应与元数据存储分离。

如果是最初开发阶段，本地目录即可；如果进入多人联调或部署环境，建议切到 `MinIO`，因为它：

- 和 S3 接口兼容
- 部署简单
- 对媒体文件管理更清晰

## 4.6 核心媒体处理

### 推荐方案

- `FFmpeg`
- `ffprobe`

### 主要职责

- 素材转码和标准化
- 抽帧
- 音视频时长探测
- 裁切、拼接、缩放
- 加滤镜和基础特效
- 最终导出 mp4

### 选型理由

FFmpeg 是这个项目里最关键、也最不可替代的基础能力。原因很直接：

- 处理链稳定
- 视频能力完整
- 与 Python 集成成熟
- 能覆盖大多数 MVP 特效需求

### 使用建议

- 所有素材先统一转为内部处理格式
- 通过 `ffprobe` 获取可靠元数据
- 不要把复杂滤镜逻辑散落在多个地方，建议统一封装一个 render service

## 4.7 音乐分析

### 推荐方案

- `librosa`
- 必要时补充 `numpy` / `scipy`

### 主要职责

- beat 检测
- BPM 估计
- downbeat 近似推断
- 音乐能量曲线分析
- 段落边界辅助识别

### 选型理由

对于内部 Demo，`librosa` 足够支撑：

- 节拍检测
- 音乐时序特征提取
- 能量变化分析

它不是最工业化的音乐结构分析工具，但对当前目标已经足够，且接入成本低。

## 4.8 视频分析

### 推荐方案

- `PySceneDetect`
- `OpenCV`
- `MediaPipe`
- 可选：`Ultralytics YOLOv8`

### 模块分工建议

#### PySceneDetect

负责：

- 镜头切分
- 候选片段边界生成

#### OpenCV

负责：

- 抽帧
- 模糊度检测
- 亮度与颜色变化分析
- 基础运动估计

#### MediaPipe

负责：

- 人脸检测
- 人体关键点或主体辅助判断

#### YOLOv8

只在需要增强主体识别时引入，不作为 MVP 默认依赖。

### 为什么这样分

因为设计文档强调“主体明显优先”，而不是高度复杂的语义理解。对 Demo 而言：

- `PySceneDetect` 足够做镜头边界
- `OpenCV + MediaPipe` 足够做基础可用的主体和质量判断
- 没必要一开始就引入更重的视频理解模型

## 4.9 日志、监控与错误追踪

### 推荐方案

- `structlog`
- `Sentry`

### 选型理由

这个项目的问题很多会发生在长链路任务中，例如：

- 某一段素材损坏
- ffmpeg 渲染失败
- 某个 worker 崩溃
- 某个中间步骤超时

因此必须有比 `print` 更可靠的日志方案。

#### structlog

适合输出结构化日志，方便按 `job_id` 跟踪整条任务链路。

#### Sentry

适合收集：

- API 异常
- Worker 异常
- 前端运行时异常

对于内部 Demo，这能大幅降低排查成本。

## 4.10 测试

### 推荐方案

- 后端：`pytest`
- 前端：`Playwright`
- 接口测试：`httpx + pytest`

### 测试重点

- 上传校验
- 任务状态流转
- 素材不足时的降级逻辑
- 音乐过短时的提示逻辑
- 结果页重新生成流程

### 选型理由

当前产品最容易出问题的不是复杂业务规则，而是流程串联。测试应优先覆盖主流程稳定性，而不是追求过细单元测试数量。

## 4.11 部署

### 推荐方案

- `Docker Compose`

### 服务组成

- `frontend`
- `api`
- `worker`
- `redis`
- `postgres`
- 可选：`minio`

### 选型理由

内部 Demo 阶段不需要 Kubernetes。`Docker Compose` 已足够满足：

- 本地开发
- 团队联调
- 内部演示环境部署

它的优点是：

- 上手快
- 配置直观
- 便于统一依赖环境
- 对 FFmpeg 这类系统依赖更友好

## 5. 推荐目录结构

```text
beatcut-ai/
  apps/
    web/
    api/
    worker/
  packages/
    shared/
    media-core/
  storage/
  docker/
  scripts/
  docs/
```

### 目录说明

- `apps/web`：Next.js 前端
- `apps/api`：FastAPI 服务
- `apps/worker`：Celery Worker
- `packages/shared`：共享 schema、常量、任务状态定义
- `packages/media-core`：音视频分析与渲染核心逻辑封装
- `storage`：本地开发期文件存储
- `docker`：部署配置
- `scripts`：辅助脚本

## 6. 不推荐的技术路径

以下方案不是不能做，而是对当前项目“不够划算”。

### 6.1 Node.js 作为主后端 + Python 作为分析子进程

不推荐原因：

- 双语言维护成本更高
- 媒体分析主链路仍在 Python
- 接口层和任务层割裂

### 6.2 只用 FastAPI 同步处理全部任务

不推荐原因：

- 视频任务阻塞严重
- 状态页体验差
- 容易出现请求超时
- 无法优雅重试

### 6.3 一开始就上微服务

不推荐原因：

- 当前模块虽然多，但边界还在快速变化
- 过早拆服务会提升联调成本
- 对内部 Demo 没有必要

### 6.4 一开始就上重模型视频理解

不推荐原因：

- 成本高
- 推理慢
- 工程复杂度明显上升
- 当前产品目标并不需要那么强的语义能力

## 7. 最终推荐版本

如果只保留一套最简但健壮的落地方案，我推荐：

### 前端

- `Next.js`
- `TypeScript`
- `Tailwind CSS`
- `shadcn/ui`
- `TanStack Query`

### 后端

- `FastAPI`
- `Pydantic`
- `SQLAlchemy`

### 任务队列

- `Celery`
- `Redis`

### 数据与存储

- `PostgreSQL`
- 本地文件存储起步，后续切 `MinIO`

### 媒体处理

- `FFmpeg`
- `librosa`
- `PySceneDetect`
- `OpenCV`
- `MediaPipe`

### 运维与稳定性

- `Docker Compose`
- `structlog`
- `Sentry`

## 8. 一句话总结

对 BeatCut AI 这个内部 Demo 来说，最合适的技术栈不是“最潮”的，而是“最能稳定跑完整条自动剪辑链路”的。

因此推荐采用：

`Next.js + FastAPI + Celery + Redis + PostgreSQL + FFmpeg + librosa + PySceneDetect + OpenCV`

这套方案简单、成熟、工程风险低，也最适合从内部 Demo 平滑演进到下一阶段产品。
