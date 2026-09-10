# 项目、数据与字形资产规范

## 项目目录

```text
MyFontProject/
├── project.jianzi        # JSON 项目清单
├── project.db            # SQLite 元数据
├── sources/source-001/original.jpg
├── pages/
├── glyphs/U+5929/default/{original.png,cleaned.png,glyph.svg,metadata.json}
├── glyphs/U+5929/variant-001/
├── glyphs/unmapped/
├── mappings/  fonts/  builds/  cache/  logs/
```

`project.jianzi` 至少包含 `format: "JianziProject"`、`schema_version`、`app`、`name`、`created_at`、`updated_at`。所有模式变化必须提供迁移路径；禁止静默破坏旧项目。

## 存储责任

- SQLite 保存 project、sources、pages、processing_operations、glyphs、glyph_variants、character_mappings、font_profiles、builds、tasks 等元数据。
- 文件系统保存 JPG/PNG/TIFF/SVG/TTF/OTF/WOFF2/SFD、缓存、缩略图和日志；不得把高清图、PNG 或 SVG Blob 写进 SQLite。
- Python Core 是项目数据库唯一的业务读写者。
- Source 必须记录名称、作品名、作者、年代、来源、URL、授权、版权备注、SHA256 和导入时间。导入源文件只读保留，所有处理均生成派生资产或操作记录。

## 字形不变量

每个 glyph 至少可关联 `source`、`page`、`bbox [x,y,width,height]`、`row`、`column`、`order`、`status` 与 `quality_score`。内部编码使用 Unicode code point，例如 `U+5929`。

Glyph Library 才是项目核心资产：原始裁切图、清理位图和 SVG 必须可追溯。构建出的 TTF/OTF/WOFF2 从 SVG 与 profile 可重复生成，不得成为唯一编辑源。

同一 Unicode 必须支持多个 variant：一个 default，以及任意 `variant-001` 等备用项；用户可设默认、保留、禁用或删除。未来 OpenType alternates 应建立在这一模型上。

## 非破坏性与可逆性

页面操作仅以有序参数记录形式保存，支持 Crop、Rotate、Deskew、Perspective、Gray、Levels、Threshold、Denoise、Morphology、Background Removal 等。必须支持 Undo、Redo、Reset 和 Before/After。bbox、映射和 SVG transform 同样要纳入操作历史。书法处理的 morphology 默认保守，优先保留尖锋、细线、飞白和边缘。

