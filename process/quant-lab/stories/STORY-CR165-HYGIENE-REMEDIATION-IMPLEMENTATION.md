# CR-165 Workflow and Artifact Hygiene Implementation

## 实现前置检查

- process workspace route health：PASS。
- 用户授权：主会话内联实施；禁止子 agent。
- 产品、真实数据、runtime、交易、发布、远端写入：不在范围。

## 实现对象与契约映射

| 对象 | 契约 | 实现与验证 |
|---|---|---|
| CR tracking checker / fixture | JSON 为机器真相源；legacy YAML 可兼容；结构化状态按字段读取 | `scripts/legacy/cr/check_cr_tracking_consistency.py`、`tests/meta_flow/test_cr_tracking_consistency.py`；10 tests PASS |
| Design surface | 根目录只保留 canonical authority；历史快照归档可追溯 | 32 份快照迁入 `process/archive/design-cr-docs/`，索引 v1.1；3 tests PASS |
| Artifact hygiene | CR164 closed bundle、CR165 active bundle、归档迁移必须显式分类；陌生文件仍阻断 | `scripts/check_process_artifact_hygiene.py`、19 tests PASS、unclassified=0 |
| Test taxonomy | 测试必须位于 domain 目录且不用 CR 文件名 | 7 个 lineage tests 移至 `tests/research/`；PROVENANCE 同步；264 tests PASS |
| CR lifecycle | 无效占位路径和生命周期枚举不得产生 ERROR | CR159 path=N/A、CR164 relationship、CR010/018/031 lifecycle 规范化；checker OK |

## 最小实现切片

1. tracking fixture/checker：局部 10/10 PASS。
2. design archive：design surface 3/3 PASS。
3. artifact classification：19/19 PASS，真实工作区 unclassified=0。
4. test taxonomy：264/264 PASS。
5. split-domain full regression：1,952/1,952 unique tests PASS。

## 平台与安全边界

平台适配 N/A；本 CR 仅修改本地 Python guardrail、测试与 process metadata。没有读取凭据、真实 lake/NAS/provider，没有 runtime、交易、发布或 Git remote write。

## 实现交接

QA 应重点复核：分类规则仍对未知文件 fail closed、归档索引存在、PROVENANCE 与实际测试一一对应、CR tracking 为 OK。剩余 warning 是历史候选未索引或 legacy status 摘要差异，不是本轮 ERROR。
