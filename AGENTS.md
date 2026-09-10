# Jianzi（见字）项目规则

本文件是本仓库所有贡献者与自动化代理的最高优先级项目约定。详细、可维护的规格位于 [`.agents/`](.agents/README.md)。如本文件与其他项目文档冲突，以本文件为准；如产品需求变更，应先同步更新本文件及相关 `.agents` 文档。

## 产品边界

- 正式名称：**见字 Jianzi**；副标题：**Open-source Calligraphy Font Studio**。
- 产品目标：`Calligraphy / manuscript image → Glyph Library → OpenType font`。
- 不把 Jianzi 做成通用图片编辑器或 FontForge 的完整替代品。
- 首要价值是可靠、可视化、可人工校正的半自动工作流；不得以不可靠的全自动 OCR 替代原文顺序映射。
- 默认 local-first、offline-capable。不得在未取得明确授权时上传原始作品、字形或字体。
- 不得绑定某一种书体、某一个字体构建后端或某个外部命令行工具。

## 架构硬约束

```text
Vue UI → Pinia Store → Frontend Service → Tauri IPC → Rust Worker Manager
                                                        → Python Core → SQLite / assets
```

- Desktop 必须使用 Tauri 2、Vue 3、TypeScript、Vite、Pinia、Vue Router。
- Rust 只处理系统能力、文件系统、桌面能力、Python Worker 生命周期与 IPC 转发；不得实现图像处理、字体业务、Unicode 映射或 Metrics 算法。
- Python 包名固定为 `jianzi_core`，要求 Python 3.11+，可脱离 GUI 运行、测试和调用 CLI。
- Vue 组件不得直接调用 Tauri `invoke`；必须经由 Store 和 Service。
- 业务数据库只能由 Python Core 读写；Vue 和 Rust 不得直接修改项目 SQLite 数据。
- Rust 与 Python Worker 使用 UTF-8 JSON Lines（stdin/stdout），一请求一响应，且必须保留关联 `id`。
- Worker 常驻：应用启动、握手、就绪、处理任务、优雅退出。禁止按单个字形反复启动 Python 进程。

## 数据与资产硬约束

- SQLite 仅保存元数据；原图、PNG、SVG、字体、缓存、日志等大资产保存在项目文件系统中。
- 原始导入资料永远不可被处理流程覆盖。
- 核心资产是 Glyph Library，不是 TTF；每个字形应可追溯 `original.png → cleaned.png → glyph.svg`。
- SVG 是主要、可编辑、可重新导入的字形中间格式；TTF/OTF/WOFF2 是可重复生成的 Build Artifact。
- 内部字符标识统一使用 Unicode code point（例如 `U+5929`）。同一 Unicode 从第一版数据模型起就必须支持多个 variant 和默认 variant。
- 自动结果必须允许人工修正；处理操作必须尽量可重复、可记录、可撤销。

## 实施优先级

严格按下列顺序实施，不跨越通信与数据基础设施直接开发字体业务：

1. Phase 0：架构、项目格式、IPC、字体管线文档。
2. Phase 1：Monorepo、Tauri/Vue、Python Core、常驻 Worker 与 `system.ping`。
3. Phase 2：项目创建/打开、`project.jianzi`、SQLite、目录结构、最近项目。
4. Phase 3：图片导入、预览、缩放/平移、基础非破坏性页面处理。
5. Phase 4：网格切字、手工 bbox、阅读顺序。
6. Phase 5：粘贴原文、顺序 Unicode 映射、校正、Glyph Gallery。
7. Phase 6：清理、标准化、批处理。
8. Phase 7：Potrace 适配、SVG 生成与导入。
9. Phase 8：Font Profile、FontForge、TTF、fontTools 验证。

V0.1 只需完成 `Image → Glyph`；V0.3 必须完成 8 字 fixture 的 `Image → Glyph → Unicode → SVG → TTF` 闭环。

## 工程质量与验收

- TypeScript 必须开启 `strict`，使用 ESLint、Prettier、Vitest。
- Python 使用类型标注、Ruff、Black、pytest；Rust 使用 rustfmt、clippy、cargo test。
- 不得硬编码机器路径或外部工具路径；依赖通过 Adapter 与 Dependency Checker 管理。某一个可选依赖缺失不得让应用整体无法启动。
- 长任务必须进入可取消、可重试的 Task Queue，状态仅可为 `Pending`、`Running`、`Completed`、`Failed`、`Cancelled`；不得阻塞 UI。
- 大型项目必须采用缩略图缓存、惰性加载、分页或虚拟滚动，不得一次载入全部原图。
- 示例和测试素材只能使用自制、程序生成或明确 CC0 的素材；不得提交版权不明的书法扫描件或商业字体图片。
- 每个阶段完成后运行与该阶段相称的测试，并修复可复现错误后再继续。

## 品牌、许可与沟通

- UI 正式品牌使用“见字 Jianzi”；产品名、Python 包、CLI、项目格式和 bundle ID 的规范见 `.agents/branding-and-licensing.md`。
- 软件代码采用 MIT License；输入素材与生成字体的版权/许可由用户负责确认，软件不自动授予商业使用权。
- 中文优先，同时为国际化、深浅主题、高 DPI 与 Windows/macOS/Linux 预留。第一阶段以 Windows 11 稳定性为重点。

