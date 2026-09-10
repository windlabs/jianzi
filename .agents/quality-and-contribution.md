# 质量、测试、依赖与贡献规范

## 工具链

- TypeScript：strict、ESLint、Prettier、Vitest。
- Python：Python 3.11+、typing、Ruff、Black、pytest；核心依赖为 OpenCV、Pillow、NumPy、Pydantic、fontTools、Typer、Rich。
- Rust：rustfmt、clippy、cargo test。
- CI 最低覆盖前端 lint/typecheck/test、Python lint/test、Rust fmt/clippy/test；后续加字体管线集成测试。

## 测试策略

Python 测试覆盖 project、segmentation、mapping、Unicode、cleaning、vectorization、metrics、font build 与 validation。字体集成 fixture 使用程序生成或明确 CC0 的 2×4 小图，流程须验证最终 TTF 的 cmap 含对应 code points。

代码应保持模块职责单一、类型明确、无全局业务状态、无机器路径和魔法数字。关键图像/字体算法应解释其参数与单位。大型字库须实施缩略图缓存、惰性加载和虚拟化/分页。

## 可选依赖与降级

Potrace 与 FontForge 是外部工具，均需独立 Adapter 与清晰的 dependency check。缺少 FontForge 时，页面、切字、映射与 SVG 流程仍须可用，只禁用构建功能并返回可恢复错误。不得通过重新实现成熟字体基础设施来绕过后端抽象。

## 日志、隐私与诊断

日志按 desktop/core/font-build 分类。导出诊断包默认不得包含原始作品、用户字形图片或最终字体，除非用户明确选择包含。所有应用行为遵循 local-first；任何未来云功能都必须让用户明确授权。

