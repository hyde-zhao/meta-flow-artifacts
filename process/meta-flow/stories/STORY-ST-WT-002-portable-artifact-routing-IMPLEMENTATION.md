---
story_id: "ST-WT-002"
cr_id: "CR-047"
stage: "CP6"
status: "implemented"
execution_mode: "inline-fallback"
implemented_at: "2026-07-15"
---

# ST-WT-002 Implementation

## 实现对象

- `meta_flow/workspace/routing.py`：重连时保留既有 `created_at`，元数据字节未变化时不重写；继续保存锚点与相对路由。
- `delivery/rules/DIRECTORY-CONTRACT.yaml` / `.md`：meta-flow self-dev 内部文档唯一 canonical 为 `process/docs/**`，根仓只保留明确列出的 public docs。
- `tests/test_cr047_artifact_routing.py`：覆盖重复 link 字节幂等、不同父目录 relocation、regular-path fail-closed 与 canonical copy 数。

## 契约映射

`<project-root>/process` 只路由到 `<artifact-root>/process/<project-name>`；route metadata 是 artifact 侧身份记录。production 项目仍优先消费自身 README/docs 约定，不继承 meta-flow self-dev 特例。

## 验证与边界

路由定向回归通过；重复 link 不再仅因时间戳制造 artifact dirty。regular path 仍明确阻断，不静默迁移。未读取或修改 prelink backup、quant-lab 与外部运行环境。

## 设计差异与交接

无设计差异、无新路由层；CP7 需复验 routed workspace 与 clean staging 两种形态。
