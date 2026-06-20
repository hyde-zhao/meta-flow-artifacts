---
story_id: CR101-S01-target-taxonomy-manifest-contract
change_id: CR-101
title: Target taxonomy and manifest contract realignment
status: verified
lld_policy: full-lld
wave_id: CR101-W1-CONTRACT
depends_on: []
owner: host-orchestrator
story_lld: process/stories/CR101-S01-target-taxonomy-manifest-contract-LLD.md
cp5_check: process/checks/CP5-CR101-S01-target-taxonomy-manifest-contract-LLD-IMPLEMENTABILITY.md
cp6_check: process/checks/CP6-CR101-S01-target-taxonomy-manifest-contract-CODING-DONE.md
cp7_check: process/checks/CP7-CR101-S01-target-taxonomy-manifest-contract-VERIFICATION-DONE.md
implementation_evidence: process/stories/CR101-S01-target-taxonomy-manifest-contract-IMPLEMENTATION.md
---

# CR101-S01 Target taxonomy and manifest contract realignment

## 用户价值

消除 `miniqmt_runner` 被误当作策略交付 target 的历史混淆，建立 `delivery_targets[]` 与 `execution_adapters[]` 双边界，让后续 checker、fixture、runner 和 evidence 都有同一 contract。

## 范围

- 修改 manifest / adapter payload 设计，当前唯一 implemented delivery target 为 `qmt_terminal_direct`。
- 保留 future target schema slot，但不实现 Goldminer / generic Python runtime。
- MiniQMT 只作为 `execution_adapters[]` 的 gateway / broker adapter contract。

## 验收标准

| AC | 描述 |
|---|---|
| S01-AC-01 | manifest contract 明确 `delivery_targets[]` 与 `execution_adapters[]`，且当前 implemented target 仅 `qmt_terminal_direct`。 |
| S01-AC-02 | 新 manifest / fixture 不再把 `miniqmt_runner` 作为 delivery target。 |
| S01-AC-03 | 所有 authorization flags 默认且必须为 false。 |
| S01-AC-04 | runner core 不直接 import QMT / MiniQMT / XtQuant SDK。 |

## 文件所有权

| 类型 | 文件 |
|---|---|
| primary | `trading/strategy_runner/package_loader.py` |
| shared | manifest fixture、CR101 tests、docs/qmt CR101 HLD follow-up |
| forbidden | `.env`、凭据、真实 NAS、runtime launcher、QMT/MiniQMT/XtQuant SDK runtime import |

## 测试设计

- manifest schema positive / negative unit tests。
- authorization false flags tests。
- `miniqmt_runner` delivery target negative fixture。
- import boundary static check 或等价单测。
