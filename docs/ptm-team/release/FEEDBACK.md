# ptm-tde v1.0 发布后反馈计划

> 版本：v1.0 · 更新：2026-06-08

---

## 观察信号

| 信号 | 说明 | 触发阈值 | 处理方式 |
|---|---|---|---|
| 安装失败 | `ptm-team install claude --agent ptm-tde` 报错 | 任意一次 | 检查 DEPLOY-CHECKLIST，修复安装脚本 |
| Skill 调用异常 | Agent 执行中 Skill 返回格式错误 | 同一 Skill ≥3 次 | 回溯 LLD → 修复 → 重跑 CP6/CP7 |
| 人工检查点卡顿 | 用户在 Gate 处反复修改 | 同一 Gate ≥5 轮 | 优化提示词和选项设计 |
| atomic-ops CLI 不可用 | CLI 查询返回错误 | 任意一次 | 检查 CLI 安装和版本兼容 |
| 因子库加载失败 | m-analyzer Step 1.5 报错 | 任意一次 | 检查 YAML schema 和文件路径 |

---

## 反馈渠道

| 渠道 | 用途 | 响应时间 |
|---|---|---|
| 直接对话 | 使用中遇到的问题 | 即时 |
| CR 流程 | 功能增强和 Bug 修复 | 按 CR 优先级 |
| follow-up tracking | 非紧急改进项 | 按台账跟踪 |

---

## 反馈分流

```
用户反馈
  ├── Bug（功能异常） → 创建 CR → 修复 → CP6/CP7 → 发布
  ├── Enhancement（功能增强） → follow-up candidate → 评估优先级 → CR
  ├── Question（使用问题） → 更新 USER-MANUAL 或 FAQ
  └── Invalid（误报/已修复） → 记录关闭
```

---

## 版本迭代触发条件

| 条件 | 动作 |
|---|---|
| P0 Bug | 立即创建 CR，hotfix |
| P1 Bug ≥ 3 | 创建 CR，纳入下一版本 |
| 新 Feature 请求 ≥ 2 | 评估优先级，进入 backlog |
| atomic-ops 新增 op | 更新 aliases 覆盖 |
| 新因子库上线 | 更新 resource/ 目录 |

---

## 当前跟踪项

| ID | 类型 | 跟踪文件 | 状态 |
|---|---|---|---|
| CR-011-T-01 | Enhancement | `process/changes/CR-011-FOLLOW-UP-TRACKING-2026-06-02.md` | candidate |
| CR-011-T-02 | Enhancement | 同上 | candidate |
| CR-015-T-01 | Enhancement | `process/changes/CR-015-FOLLOW-UP-TRACKING-2026-06-04.md` | candidate |
| atomic-ops aliases | Enhancement | — | spike_candidate |

---

*本计划随 ptm-tde 实际使用反馈动态更新。*

---

## 修订记录

| 日期 | 变更 | 处理人 |
|------|------|--------|
| 2026-07-06 | §观察信号"安装失败"行安装命令对齐 install.py 实际 CLI（补 platform 位置参数 `claude` + `--agent ptm-tde`）。 | host-orchestrator |
