# Feedback CR137

## 观察信号

| 信号 | 触发阈值 | 分流 |
|---|---|---|
| registry entry 字段不足以支撑 batch/report | 任意真实离线批量需求 | 启动 Offline Batch Run / report follow-up。 |
| blocked diagnostic entry 不够定位 | 用户无法判断 blocked 原因 | registry UX / diagnostics follow-up。 |
| cr-tracking warning 仍出现 | 用户要求零 warning | `FU-CR137-001` 候选。 |
| 用户要求真实 QMT/MiniQMT runtime | 任意 runtime 诉求 | 独立 runtime authorization gate；不得并入 offline runner CR。 |

## 后续候选

| 候选 ID | 标题 | 状态 | 启动条件 |
|---|---|---|---|
| FU-CR137-001 | CR tracking warning cleanup | candidate | 用户要求零 warning。 |
| FU-CR137-002 | Offline Batch Run | candidate | 继续 offline runner roadmap。 |
| RA-CR137-001 | Online readonly runtime authorization gate | candidate | 用户明确授权 runtime 验证讨论。 |

## 不授权范围

runtime/NAS/QMT/credentials/provider/lake/catalog/trading/Git write 均保持 deferred，需独立 CR 和人工门禁。
