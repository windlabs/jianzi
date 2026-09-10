# 架构与仓库规范

## 固定技术栈

```text
apps/desktop     Tauri 2 + Vue 3 + TypeScript + Vite + Pinia + Vue Router
apps/desktop/src-tauri
                 Rust：系统能力、Worker 生命周期与 IPC
python           Python 3.11+ 的 jianzi_core：图像、字形、字体业务
packages/shared-types
                 前端共享 API 类型
packages/ui      可复用 UI 原语（可在需要时创建）
```

推荐仓库结构：

```text
Jianzi/
├── apps/desktop/{src,src-tauri}
├── python/{src/jianzi_core,tests}
├── packages/{shared-types,ui}
├── docs/
├── examples/  scripts/  fixtures/
└── .agents/
```

Python Core 中，`api` 处理协议分发，`models` 定义 Pydantic 模型，`project` 管理项目和迁移；图像、切割、清理、映射、矢量化、glyph、fonts 和 validation 各自独立成模块。不要把业务塞入 `worker.py` 或单一服务文件。

## 单向边界

```text
Component → Store → Service → Tauri command → Rust manager → Python Worker
                                                        ↘ system integrations
Python Core → project repository → SQLite metadata + project assets
```

- UI 状态按领域拆分：`appStore`、`projectStore`、`sourceStore`、`pageStore`、`glyphStore`、`mappingStore`、`fontStore`、`taskStore`、`settingsStore`。
- Frontend services 按 `worker`、`project`、`source`、`page`、`glyph`、`mapping`、`font`、`build`、`task` 拆分。
- Rust command 只做参数校验、生命周期管理和转发，不承载领域规则。
- Python CLI 与 Worker 调用同一服务层，不能复制业务逻辑。
- Python 的 `FontBuildBackend`、`Vectorizer`、`Segmenter`、`OCREngine` 都用抽象接口隔离具体实现。

## 阶段门槛

Phase 0 先完成 `docs/architecture.md`、`project-format.md`、`ipc-protocol.md`、`font-pipeline.md`。Phase 1 的唯一业务验收是 UI 经 Tauri 到常驻 Python Worker 的 `system.ping` 往返成功，响应包含 `name: Jianzi Core` 与版本。该链路未稳定前，不开始字体业务。

外部工具用 Adapter 隔离：Potrace、FontForge 缺失时，依赖检查必须报告可恢复问题，并保留项目、页面、切字、映射与 SVG 相关可用能力。

