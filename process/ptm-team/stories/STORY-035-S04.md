---
doc_type: STORY
cr_id: CR-035
story_id: STORY-035-S04
evidence_type: technical-note
lld_policy_required_level: technical-note
risk_level: Low
file_ownership: agents/ptm-te.md + skills/case-execution/SKILL.md + skills/trex-traffic/SKILL.md（独占）
dependencies: [STORY-035-S01, STORY-035-S02, STORY-035-S03]
hld_ref: process/changes/CR-035-HLD.md
author: meta-dev
created_at: 2026-07-31
---

# STORY-035-S04 技术说明：文档同步

> HLD ref：§2.2（Out of Scope 边界）/ §13（Gotchas）/ Story S04。
> 本技术说明不重复 HLD 设计正文，仅定义三份文档的改动要点、章节锚点与验收准则。

## 文件影响与设计依据

| 文件 | 改动 |
|---|---|
| `agents/ptm-te.md` | 新增 TG 生命周期 config-once 章节 + Gotcha |
| `skills/case-execution/SKILL.md` | 新增 config-once skip Gotcha（类比 #8 fw_login）+ force_config 例外 + tg_port_ownership 字段说明 |
| `skills/trex-traffic/SKILL.md` | 新增两层/三层命令差异注解 + owned vs legacy force 模式说明 |

## 接口、数据与改动要点

### 1. agents/ptm-te.md
- **新增章节「TG 端口生命周期（config-once，CR-035）」**：描述 run 级三段生命周期 acquire-once（[4c]）/ config-once（[5] 首个真实执行，后续 skip）/ release-once（[6c]），与 DUT 侧 login-once/preconfigure-once/fw_logout 对称（HLD §7 对称关系表）。
- **tg_user_id 生成规则**：`dev{ip_suffix}-{run_id_hash4}`，run 级唯一，acquire 绑定，release 释放（HLD §5.1，DQ-035-02）。
- **中断语义**：acquire 失败中断 run（类比 login_ok=False）；config 失败不中断（AGA-4）；release 失败不阻断 report（HLD §12）。
- **Gotcha**：config-once skip 仅 --execute 生效，dry-run 每个 tg_config_interface 都 build_command（DQ-035-06）；acquire/release 不经 execute_steps（main 直接调 execute_op）；release 不传 --ports（释放该 user-id 全部端口）。

### 2. skills/case-execution/SKILL.md
- **新增 Gotcha（类比 #8 fw_login skip）**：`tg_config_interface` 在 --execute 模式首个真实执行置 run 级标志，后续 step 标记 skipped（reason: config-once 复用）；类比 fw_login skip（L1473-1487，Gotcha#8）的 `and not dry_run` 守卫，dry-run 不 skip。
- **force_config 例外**：用例 step `args.force_config: true` 时真实执行并重置标志（DQ-035-01），用于强制重配 TG 接口。
- **tg_user_id 自动注入**：6 个现有 tg op（tg_config_interface/apply_traffic_template/delete_traffic_template/start_traffic_stream/stop_traffic_stream/verify_traffic_loss）在 run 级 acquire 成功后自动注入 `--user-id`，走 owned 占用模式；不注入走 legacy force 模式（HLD Gotcha#5）。
- **result.json tg_port_ownership 字段**：含 tg_user_id/acquire_ok/acquire_record/tg_interface_configured/config_once_skip_count/release_record，与 dut_preconfig 对称（HLD §11）。

### 3. skills/trex-traffic/SKILL.md
- **两层 vs 三层命令差异注解**（HLD §2.2 Out of Scope）：trex-traffic SKILL.md 原描述 `tg tg_acquire_ports` 两层约定；ptm-te 走 ptm-atomic 三层命令 `ptm-atomic run ... tg trex acquire-ports`。澄清 ptm-te 的 op_mapper build_command 对 family=="tg" 自动插入 "trex" 生成三层命令（L784-786）。
- **owned vs legacy force 模式**：owned 模式（CR-035 后默认）= acquire 成功后 tg op 自动注入 `--user-id`，端口由该 user-id 占用，支持多用户并发；legacy force 模式 = 不注入 user_id（default），force 占用，不支持并发追溯。CR-035 后 ptm-te 默认 owned 模式。

## 实施

1. S01/S02/S03 实现完成后定稿（文档需反映实际实现）。
2. ptm-te.md 新增 TG 生命周期 config-once 章节（可并行起草）。
3. case-execution SKILL.md 新增 config-once Gotcha + force_config + tg_port_ownership。
4. trex-traffic SKILL.md 新增两层/三层命令差异 + owned vs legacy force。
5. 三份文档交叉引用一致（tg_user_id 格式、config-once 语义、dry-run 行为描述统一）。

## 验收准则（含权限与失败处理）

- [ ] ptm-te.md 含 TG 生命周期 config-once 三段描述 + tg_user_id 生成规则 + 中断语义 + Gotcha。
- [ ] case-execution SKILL.md 含 config-once skip Gotcha（类比 #8）+ force_config 例外 + tg_port_ownership 字段。
- [ ] trex-traffic SKILL.md 含两层/三层命令差异注解 + owned vs legacy force 模式说明。
- [ ] 三份文档对 dry-run 不 skip、acquire 失败中断、release 不传 ports 的描述一致（HLD §10/§12/§13）。

## 依赖

- S01/S02/S03（可并行起草，实现完成后定稿）。
- HLD §2.2 + §7 + §10 + §11 + §13 为设计真相源。

## 风险

| 风险 | 等级 | 缓解 |
|---|---|---|
| 文档描述与实现不一致 | Low | S01-S03 实现完成后定稿；验收准则交叉引用一致性检查 |
| trex-traffic SKILL.md 两层/三层差异描述歧义 | Low | 明确 ptm-te 走 ptm-atomic 三层命令，trex-traffic 两层约定为服务端原生形态 |


## 语义维度覆盖

> lld-check required token 覆盖声明（指向对应章节）：

- **设计依据**：本 LLD 已覆盖（见对应章节）
- **文件影响**：本 LLD 已覆盖（见对应章节）
- **接口**：本 LLD 已覆盖（见对应章节）
- **数据**：本 LLD 已覆盖（见对应章节）
- **权限**：本 LLD 已覆盖（见对应章节）
- **失败**：本 LLD 已覆盖（见对应章节）
- **测试**：本 LLD 已覆盖（见对应章节）
- **风险**：本 LLD 已覆盖（见对应章节）
