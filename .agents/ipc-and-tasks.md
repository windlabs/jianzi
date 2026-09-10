# Worker IPC 与任务规范

## 生命周期

开发时以 `python -m jianzi_core.worker` 启动；发布版最终由打包的 `jianzi-core` 承载。Worker 是单个常驻进程：

```text
App Start → Worker Start → Handshake → Ready → Requests → Graceful Shutdown
```

禁止“每处理一个字形启动一个 Python 进程”。Rust 负责启动、存活检测、终止、stdin/stdout 管道和错误转译。

## JSON Lines 协议

stdin/stdout 使用 UTF-8 JSON Lines，一行一个 JSON。每个请求必须有唯一 `id`、`method` 与对象 `params`；每个响应必须回传同一 `id`，并仅使用成功 `data` 或失败 `error`。

```json
{"id":"req-000001","method":"glyph.vectorize","params":{"project_id":"abc","glyph_id":"glyph-001"}}
```

```json
{"id":"req-000001","success":false,"error":{"code":"POTRACE_NOT_FOUND","message":"Potrace executable was not found.","recoverable":true,"details":{}}}
```

错误必须机器可读且可恢复性明确；普通 UI 展示友好文案，调试模式才允许展示 traceback。

## 方法命名空间

仅使用：`system.*`、`project.*`、`source.*`、`page.*`、`glyph.*`、`mapping.*`、`font.*`、`build.*`、`task.*`。新增方法需在公共 schema 和 `docs/ipc-protocol.md` 中登记。

启动期必须提供 `system.ping`、`system.info` 与 `system.check_dependencies`。依赖检查至少覆盖 Python、OpenCV、Pillow、fontTools、Potrace、FontForge，并能说明缺失依赖会禁用的能力。

## Task Queue

页面批处理、切字、批量清理、批量矢量化、字体构建、验证和 subset 都必须异步入队。合法状态是 `Pending`、`Running`、`Completed`、`Failed`、`Cancelled`。UI 必须可查看进度、取消、重试与错误详情。

进度事件采用 `task.progress`，包含 `task_id`、`current`、`total`、`progress`、`message`。任务实现不得阻塞 UI 或丢失可诊断的失败原因。

