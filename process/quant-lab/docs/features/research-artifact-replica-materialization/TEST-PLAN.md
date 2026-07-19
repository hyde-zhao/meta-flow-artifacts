---
status: "cp4-design-ready"
version: "1.2"
feature_id: "FEAT-CR172-I02"
---

# FEAT-CR172-I02 Research Artifact Replica and Materialization — TEST-PLAN

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 初始 replica/materialization fixture、negative、failure-recovery 与 permission 测试计划。 |
| 1.1 | 2026-07-18 | meta-se-critical | CP5 R1：增加 S02 verified seal digest correlation；固化 S04 16 个唯一测试 ID，重复 ID=`0`。 |
| 1.2 | 2026-07-18 | meta-se-critical | CP5 R2：增加 S04 staging sealed-bundle bytes-level verifier 复用与 tampered-seal negative；禁止绕过 S03 selection和第二 digest。 |

## 1. 测试矩阵

| Case | Story | 输入 / 故障 | 期望 |
|---|---|---|---|
| I02-P01 | S03 | valid original seal + expected release + eligible sync fixture | receipt valid；distribution pointer advance=1；research pointer change=0 |
| I02-N01 | S03 | partial/stale/unversioned/hash/manifest/seal/release mismatch | accepted/pointer advance=`0/0` for all classes |
| I02-F01 | S03 | interrupted copy / revoked authorization | partial non-distributable；previous replica preserved |
| I02-P02 | S04 | valid receipt + `4/4` match + eligible pull fixture | immutable cache + receipt + atomic pointer |
| I02-N02 | S04 | direct-NAS runtime or direct-research read | accepted=`0/2` |
| I02-N03 | S04 | any expected release/manifest/seal/content mismatch | runtime cache selection advance=`0` |
| I02-F02 | S04 | interrupted pull/materialize or revoke | staging non-runtime；previous cache preserved |
| I02-F03 | S04 | S03-selected staging bundle 的 seal bytes 被篡改 | S02 verifier seal=false；cache pointer advance=0；secondary digest=0 |
| I02-B01 | S05 | same content across research/NAS/execution roots | logical identity/hash distinct count=`1`；absolute path in hash=`0` |
| I02-A01 | S05 | only sync or only pull action approved | no permission union；downstream action executed=`0` |

## 2. 静态边界

- execution module 对 research canonical module 的 write/import edge=`0`。
- future runtime direct-NAS/research path consumers=`0`。
- S03 re-seal count=`0`；S04 seal creation count=`0`。
- S04 bypass-S03-selection/receipt-only-seal-trust/secondary-digest=`0/0/0`；S02 verifier-library calls=`1` per staging candidate。
- real NAS/mount/host path literals in tests=`0`；只使用 `tmp_path` 和 logical URI fixture。

### S04 唯一测试 ID（16/16）

`T-S04-P01`、`T-S04-P02`、`T-S04-N01`、`T-S04-N02`、`T-S04-N03`、`T-S04-N04`、`T-S04-N05`、`T-S04-N06`、`T-S04-N07`、`T-S04-N08`、`T-S04-F01`、`T-S04-F02`、`T-S04-F03`、`T-S04-B01`、`T-S04-B02`、`T-S04-B03`。unique=`16/16`，duplicate=`0`；R2 LLD 中任何 ID 都不得重复。

## 3. 场景覆盖

主要覆盖 SC-I02/N03/B02/B03/A02/F02/G02/Q02；S05 合并证明 REQ-010/011/012/013 `4/4`。

## 4. 失败路由

合同/owner 冲突→NEEDS_DESIGN_CLARIFICATION；实现缺陷→NEEDS_REWORK；真实 NAS/执行机操作或 direct-NAS runtime edge→BLOCKED。CP4 实际执行=`0`。

## 5. Gotchas

- fake replica fixture 必须保留原 seal，不能为了测试方便重新 seal。
- atomic pointer 测试必须验证失败前后的 selected version，而不只检查异常类型。
