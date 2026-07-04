# Feedback CR076

## 观察信号

| 信号 | 触发条件 | 后续处理 |
|---|---|---|
| 需要真实 data/reports 访问 | 用户明确要求读取、列出、复制、恢复、比对或删除 | 创建独立 runtime_authorization / execution CR |
| 需要旧根退役 | 用户要求删除、移动或重命名 legacy root | 启动 CR077 或新 destructive gate |
| 需要远端治理 | 用户要求 Git remote / branch / default branch 操作 | 启动 CR078 |
| 需要 CR046 恢复 | 用户明确恢复 QMT/MiniQMT framework verification | 恢复 CR046，不通过 CR076 自动推进 |
| 发现 policy 台账不一致 | CR tracking 或 human-gate 校验失败 | 回滚 process ledger 并重新发起门禁 |

## 反馈入口

在后续对话中明确说明目标 CR 或操作类型。未明确授权前，默认保持 no-data-touch。
