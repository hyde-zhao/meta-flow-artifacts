---
story_id: "ST-WT-004"
title: "修复 clean-clone rule 与 cache guardrail"
story_slug: "clean-clone-guardrail"
lld_version: "1.0"
tier: "L"
status: "approved"
confirmed: true
created_by: "host-orchestrator-inline/meta-dev"
created_at: "2026-07-14"
confirmed_by: "user"
confirmed_at: "2026-07-15T12:05:17Z"
feature_design_refs: ["process/docs/features/cr047-delivery/DESIGN.md", "process/docs/features/cr047-delivery/TEST-PLAN.md", "process/docs/features/cr047-delivery/TASKS.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["guardrail", "installer-contract", "package-security"]}
open_items: 0
---

# LLD: ST-WT-004 — Clean Clone Guardrail

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator-inline/meta-dev | 冻结 tracked canonical rule 与三类 cache severity。 |

## 0. 工程依据与上游设计依据

工程依据为 ADR-WT-004、Feature delivery design、当前 `.gitignore`、`delivery/rules/AGENTS.md`、generated 根 wrapper 与 `check_delivery_guardrails.py`。clean archive 缺 ignored 根 wrapper 时仍必须自洽。

## 1. 目标

以 tracked `delivery/rules/AGENTS.md` 作为共享 canonical，根 `AGENTS.md` 仅为 generated wrapper；clean Git archive guardrail exit=0。同时把 cache 区分为 tracked、package input、ignored-local，避免测试后本地 ignored cache 永久阻断。

## 2. 需求（Functional / Non-Functional）

- guardrail 不再要求 ignored 根 wrapper 是 clean clone 必需源；安装/漂移检查从 tracked canonical 派生。
- tracked cache/pyc 与 package-input cache 100% BLOCK；ignored-local cache 为 warning/cleanup hint。
- package-input 判定优先于 gitignore；不得用“被忽略”绕过打包风险。
- clean archive、wrapper present/absent/drift、三类 cache 都有 deterministic fixture。
- 不清理用户文件、不遍历 backup/quant-lab、不放宽其他 guardrail。

## 3. 模块拆分与职责

| 模块 | 职责 |
|---|---|
| `scripts/check_delivery_guardrails.py` | canonical rule discovery、wrapper drift、cache classification |
| `.gitignore` | 明确 generated wrapper/local cache 语义，不决定 package severity |
| `delivery/rules/AGENTS.md` | tracked canonical project rules |
| `delivery/scripts/install.py` | 继续从 canonical 生成 wrapper；共享只读/最小接线 |
| `tests/test_cr047_delivery_gate.py` | clean archive 与 cache/wrapper fixtures |

## 4. 代码结构与文件影响范围

修改 guardrail、`.gitignore`，必要时最小同步 installer/rule comment；创建 delivery gate 测试。不得把个人 Memory Policy 提交进 canonical；不得把所有 ignored cache 无条件放行。

## 5. 数据模型与持久化设计

`RuleSource(kind=tracked-canonical|generated-wrapper, path, tracked, required_in_archive)`；`CacheFinding(path, git_tracked, package_input, ignored, severity, remediation)`。precedence 为 `tracked BLOCK > package_input BLOCK > ignored_local WARN > none`。无新增持久化 truth，输出由 guardrail/CP result 保存。

## 6. API / Interface 设计

`resolve_rule_source(project_root) -> RuleSource`；`classify_cache(path, git_view, package_manifest) -> finding`；现有 guardrail CLI 汇总 blocker/warning并保留退出码。installer 仍是 wrapper writer；guardrail 不写 wrapper或删除 cache。

## 7. 核心处理流程

1. 从 tracked delivery contract定位 canonical rule。
2. 若 wrapper 存在，验证 managed block/hash；缺失不阻断 clean archive。
3. 从 Git tracked 集与实际 package input 集判断 cache。
4. 按 precedence 分类，BLOCKER 令退出非零，ignored-only 输出 warning。
5. 继续运行既有 delivery 路径/内容/危险命令检查，不短路隐藏其他 finding。

## 8. 技术细节与设计细节

clean archive fixture 使用 `git archive HEAD` 等价 tracked file 清单构建临时树。package input 必须来自 installer/build manifest或同一文件选择函数，不凭目录名猜测。wrapper drift 只有 wrapper 存在时检查；canonical 缺失始终 blocker。

## 9. 安全与性能设计

不执行生成文件中的命令，不删除 cache；路径分类限制在 project root。Git/package 查询各一次并缓存集合，复杂度 `O(files)`。危险命令、权限边界和 canonical hash 既有规则必须保持。

## 10. 测试设计

| 场景 | 预期 |
|---|---|
| clean archive 无根 wrapper | guardrail exit 0 |
| canonical rule 缺失 | blocker |
| wrapper 正确/漂移 | pass/drift finding |
| tracked `__pycache__`/pyc | blocker |
| ignored 但属于 package input | blocker |
| ignored-local 且不打包 | warning、非永久 blocker |
| 其他 guardrail 负例 | 仍被拒绝 |
| 本地文件 hash 前后 | 不变 |

## 11. 实施步骤

1. `TASK-WT-004-01`：调整 canonical discovery、wrapper optional/generated 语义与 `.gitignore` 注释。
2. `TASK-WT-004-02`：实现 package-first cache classifier，补 clean archive/三类 cache/drift fixture。
3. 运行定向 pytest与 clean archive guardrail；不执行清理命令。

## 12. 风险、难点与预研建议

风险是 generated wrapper缺失掩盖 canonical 缺失、ignored package cache被误放行、clean fixture读取工作树。通过 tracked canonical 必需项、package precedence、archive-only fixture缓解。若 package manifest不存在，cache保持 blocker并回设计澄清。

## 13. 回滚与发布策略

若 clean clone行为回归，回滚 classifier接线但保留 fixtures；不恢复“ignored wrapper 必需”矛盾。发布与 installer contract同步，保持 wrapper可生成但非 canonical。

## 14. DoD（Definition of Done）

- [ ] 0–14 章节完整、`open_items=0`。
- [ ] clean archive guardrail exit=0，tracked canonical count=1。
- [ ] tracked/package cache拒绝率=100%，ignored-local有 warning且不永久阻断。
- [ ] 其他 guardrail 负例无放宽，用户文件删除次数=0。
- [ ] CP5 未批准前 `confirmed=false`，不开始实现。

## 人工确认区

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-15T12:05:17Z
- 修改意见：
- 风险接受项：inline fallback；CP7/CP8 风险上限。
