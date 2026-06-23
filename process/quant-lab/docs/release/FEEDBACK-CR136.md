# Feedback CR136

## 观察信号

| 信号 | 触发阈值 | 分流 |
|---|---|---|
| valid 历史 bundle 被拒绝 | 任意可复现样本 | 检查是否真实损坏；必要时开 compatibility follow-up。 |
| CLI validate 错误信息不够定位 | 用户无法判断缺失/损坏文件 | 后续 UX 改进 CR。 |
| cr-tracking warning 仍出现 | 用户要求零 warning | `FU-CR136-001` 候选。 |

## 后续候选

| 候选 ID | 标题 | 状态 | 启动条件 |
|---|---|---|---|
| FU-CR136-001 | CR tracking warning cleanup | candidate | 用户要求零 warning。 |
| FU-CR136-002 | Offline runner run registry | candidate | CR136 关闭后继续增强离线运行管理。 |
| FU-CR136-003 | Offline batch run | candidate | run registry 或同等索引基础就绪后。 |

## 不授权范围

runtime/NAS/QMT/credentials/provider/lake/catalog/trading/Git write 均保持 deferred，需独立 CR 和人工门禁。
