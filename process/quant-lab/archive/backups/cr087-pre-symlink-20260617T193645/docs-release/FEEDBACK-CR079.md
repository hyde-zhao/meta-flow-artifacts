# CR079 Feedback

## 观察信号

| 信号 | 触发条件 | 分流 |
|---|---|---|
| target root 缺少 `reports/` / `data/` 影响日常工作 | 用户需要在 authoritative root 运行依赖本地数据或报告的任务 | 启动 `CR079-restore-candidate` |
| legacy retained assets 可能不完整或过期 | 用户需要证明内容质量、日期范围或完整性 | 启动内容校验或 NAS compare CR |
| 需要从 provider 重建数据 | local / NAS 均不足 | 启动 `CR079-provider-rebuild-candidate` |
| 需要旧根退役 | target root 资产恢复并验证完成 | 启动 CR077 |

## 反馈边界

FEEDBACK 只记录输入来源，不表示候选 CR 已启动。任何复制、恢复、删除、内容读取、凭据读取、provider/lake/catalog、remote Git 或 runtime 操作都需要独立授权。
