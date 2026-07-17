---
story_id: "STORY-RA-06.1-DETECT"
canonical_story_id: "ST-RA-06.1-DETECT"
title: "变更检测、合并与版本历史（S2）"
source_cr: "CR-030"
feature: "FEAT-RA-INGESTION"
lld_policy: "full-lld"
tier: "Standard"
design_evidence_type: "full-lld"
created_by: "meta-dev"
created_at: "2026-07-16"
open_items: []
shared_fragments:
  - "shared_skill: skills/itr-ticket-ingestion/SKILL.md (with ST-RA-05.1-INGEST, ST-RA-05.2-CLEAN, ST-NRA-03)"
---

# ST-RA-06.1-DETECT: 变更检测、合并与版本历史（S2） — LLD

## 0. 工程依据与模板索引

| 来源 | 消费内容 |
|---|---|
| HLD / Feature DESIGN / Feature Matrix | S2 变更检测、冲突队列与 `full-lld` 约束 |
| Story `ST-RA-06.1-DETECT` | 输出文件、验收标准和依赖 |

本文保留原有详细章节；目标、需求、模块拆分、代码结构、数据模型、API、流程、技术细节、安全、测试、实施、风险与 DoD 分别由后续编号章节定义。

> 对应 HLD REV-03 的 S2 增量摄取 + 变更检测/合并阶段。
> 本 Story 在共享 SKILL.md 中追加 §变更检测与版本历史部分。
> 注意：本 Story 属于 Wave 2（实现需 Wave 1 完成），但 LLD 与 Wave 1 同批次产出。

## 1. 文件影响范围

| 文件 | 操作 | 所有权 | 说明 |
|---|---|---|---|
| `skills/itr-ticket-ingestion/SKILL.md` | 修改（共享写入） | F-020 | 追加 §变更检测、§合并策略、§版本历史三节。前置 Story 已创建摄取/清洗段 |
| `skills/itr-ticket-ingestion/templates/conflict-queue.yaml` | 新建 | F-020 | 人工冲突队列模板 |

**不修改的文件**: `data/schema.sql`、`data/dao.py`（只消费其写入接口）。

## 2. 接口设计

### 2.1 变更检测入口

```python
def detect_changes(new_batch: list[dict], batch_ref: str,
                   db_conn: sqlite3.Connection) -> ChangeResult:
    """
    输入: 清洗后的新批次 ticket 数组（来自 ST-RA-05.2）、batch_ref、
          数据库连接（用于查询已有 ticket）
    输出: ChangeResult {new, modified, unchanged, conflicts, field_diffs_map}
    """
```

### 2.2 ChangeResult 结构

```python
@dataclass
class ChangeResult:
    new: list[dict]            # 新增 ticket 记录
    modified: list[ModifiedRecord]  # 已变更记录
    unchanged: list[str]       # 未变化的 source_ticket_id 列表
    conflicts: list[ConflictRecord] # 冲突队列项
    stats: ChangeStats

@dataclass
class ModifiedRecord:
    source_ticket_id: str
    ticket_id: int             # 数据库中已有记录的 id
    previous_status: str | None
    field_diffs: dict          # {field_name: {old: value, new: value}}
    ticket_record: dict        # 清洗后的完整 record（用于 upsert）

@dataclass
class ConflictRecord:
    source_ticket_id: str | None   # None 表示无稳定 ID
    reason: str                # 'missing_id' | 'semantic_conflict' | 'irreconcilable'
    new_record: dict           # 新批次中的记录
    existing_record: dict | None   # 数据库中已有记录（如存在）
    suggested_resolution: str  # 'manual_review' | 'keep_existing' | 'keep_new'
    created_at: str            # ISO8601

@dataclass
class ChangeStats:
    total_input: int
    count_new: int
    count_modified: int
    count_unchanged: int
    count_conflict: int
```

### 2.3 冲突队列模板

```yaml
# skills/itr-ticket-ingestion/templates/conflict-queue.yaml
batch_ref: "string"
generated_at: "ISO8601"
conflicts:
  - index: 0
    source_ticket_id: null | "string"
    reason: "missing_id | semantic_conflict | irreconcilable"
    new_record_summary:
      product: "string"
      title: "string"
      severity: "string"
    existing_record_summary:
      source_ticket_id: "string"
      first_seen_at: "string"
      last_seen_at: "string"
      status: "string"
    field_conflicts:
      - field: "status"
        existing: "已关闭"
        new: "重新打开"
    suggested_resolution: "manual_review | keep_existing | keep_new"
    reviewer_decision: ""   # 人工回填
    resolution_timestamp: ""
```

### 2.4 字段级 diff 算法

```python
def compute_field_diff(existing: dict, incoming: dict,
                       exclude_fields: set = None) -> dict:
    """
    逐字段比较已有记录与新记录。
    返回: {field_name: {old: value, new: value}}
    排除字段默认: {'first_seen_at', 'created_at', 'last_seen_at',
                  'quality_flag', 'raw_json'}
    """
    exclude_fields = exclude_fields or {
        'first_seen_at', 'created_at',   # 由系统维护
        'last_seen_at',                   # 在 upsert 时统一更新
        'quality_flag',                   # 由清洗流程独立管理
        'raw_json'                        # 每次拉取独立保存
    }
    diffs = {}
    for key in incoming:
        if key in exclude_fields:
            continue
        old_val = existing.get(key)
        new_val = incoming.get(key)
        if old_val != new_val:
            diffs[key] = {'old': old_val, 'new': new_val}
    return diffs
```

## 3. 数据模型

### 3.1 变更类型枚举

| change_type | 含义 | 触发条件 | 版本行为 |
|---|---|---|---|
| `new` | 新的 source_ticket_id | 数据库中无匹配记录 | 创建 ticket(version=1) + ticket_version(1) + change_history |
| `modified` | 字段值有变更 | field_diffs 非空 | upsert ticket + 新增 ticket_version + change_history |
| `unchanged` | 已有记录无变化 | field_diffs 为空 | 不更新 ticket，不新增 ticket_version，仅 insert change_history |
| `conflict` | 无法自动合并 | 无稳定 ID 或语义冲突 | 进入 conflict_queue，不写入 ticket |

### 3.2 冲突解决枚举

| resolution | 含义 | 触发条件 | 后续动作 |
|---|---|---|---|
| `auto_merged` | 自动合并（字段级非冲突变更） | field_diffs 中无冲突字段 | 正常 upsert |
| `manual_queue` | 进入人工处理队列 | 无 source_ticket_id 或语义冲突 | 记录在 conflict-queue.yaml，等待人工 reviewer |
| `rejected` | 人工拒绝合并 | reviewer 决定保留旧版本 | 不写入新记录，change_history 标记 |

### 3.3 语义冲突判定规则

| 场景 | 判定为冲突？ | 理由 |
|---|---|---|
| status 从 "已关闭" 变为 "重新打开" | **是** | status 变化可能是合法的工作流，也可能是 ITR 数据错误；需要人工判断是否接受新状态 |
| severity 从 "严重" 变为 "致命" | **是** | 严重度升降级需人工确认是否为真实变化 |
| product 字段变更 | **是** | product 是核心归属字段，变更意味着问题被重新分类 |
| title/description 微小文本差异 | **否** | 可自动合并，文本级 diff 保留在 field_diffs |
| 仅有时间字段变化（openeddate/resolveddate） | **否** | 可自动合并 |
| module 字段从具体值变为空 | **是** | 信息丢失，需人工判断 |

**冲突字段白名单**: `status`, `severity`, `product` — 这些字段的任何非空变更都视为语义冲突，进入 manual_queue。

**可自动合并字段**: `title`, `description`, `module`, `openeddate`, `resolveddate`, `root_cause`, `test_missed_analysis`, `test_missed_phase`, `improvement_measures`

### 3.4 ticket_version 写入规范

```python
def build_version_record(ticket_id: int, existing_record: dict,
                         field_diffs: dict, batch_ref: str) -> dict:
    """
    构建 ticket_version 写入字典。
    version 号由调用方从 DAO 获取（get_next_version）。
    field_diffs 序列化为 JSON string。
    previous_status 从 existing_record 中提取。
    """
    return {
        'ticket_id': ticket_id,
        'version': get_next_version(conn, ticket_id),
        'previous_status': existing_record.get('status'),
        'field_diffs': json.dumps(field_diffs, ensure_ascii=False),
        'batch_ref': batch_ref
    }
```

## 4. 核心流程

### 4.1 S2 增量摄取主流程

```
── 输入 ──
new_batch: list[dict]  (来自 ST-RA-05.2 的 CleanResult.cleaned)
batch_ref: str         (batch_id)
          │
          ▼
┌─ 1. 加载已有数据 ──────────────────────────────────────┐
│  existing_map = {}                                      │
│  FOR each record in new_batch:                          │
│    sid = record['source_ticket_id']                     │
│    IF sid IS NOT None:                                  │
│      existing = get_ticket_by_source_id(conn, sid)      │
│      existing_map[sid] = existing                       │
└────────────────────────────────────────────────────────┘
          │
          ▼
┌─ 2. 逐条变更检测 ──────────────────────────────────────┐
│  new_list, modified_list, unchanged_list, conflict_list │
│  = [], [], [], []                                       │
│                                                         │
│  FOR each record in new_batch:                          │
│    sid = record.get('source_ticket_id')                 │
│                                                         │
│    a. 无 stable ID:                                     │
│       IF sid IS None OR sid == '':                      │
│         conflict_list.append(ConflictRecord(            │
│           reason='missing_id'                           │
│         ))                                              │
│         CONTINUE                                        │
│                                                         │
│    b. 匹配已有记录:                                     │
│       existing = existing_map.get(sid)                  │
│       IF existing IS None:                              │
│         new_list.append(record)                         │
│         CONTINUE                                        │
│                                                         │
│    c. 计算 diff:                                         │
│       diffs = compute_field_diff(existing, record)      │
│       IF diffs IS EMPTY:                                │
│         unchanged_list.append(sid)                      │
│         CONTINUE                                        │
│                                                         │
│    d. 冲突判定:                                         │
│       conflict_fields = check_semantic_conflict(diffs)  │
│       IF conflict_fields IS NOT EMPTY:                  │
│         conflict_list.append(ConflictRecord(            │
│           reason='semantic_conflict',                   │
│           field_conflicts=conflict_fields               │
│         ))                                              │
│         CONTINUE                                        │
│                                                         │
│    e. 自动合并:                                         │
│       modified_list.append(ModifiedRecord(              │
│         ticket_id=existing['id'],                       │
│         previous_status=existing.get('status'),         │
│         field_diffs=diffs,                              │
│         ticket_record=record                            │
│       ))                                                │
└────────────────────────────────────────────────────────┘
          │
          ▼
┌─ 3. 写入 SQLite（事务包裹）───────────────────────────┐
│  begin_transaction(conn)                               │
│                                                         │
│  FOR each record in new_list:                          │
│    ticket_id = insert_ticket(conn, record)              │
│    insert_ticket_version(conn, ticket_id, 1,            │
│      None, "{}", batch_ref)                             │
│    insert_change_history(conn, ticket_id, batch_ref,    │
│      'new', None, 'auto_merged')                        │
│                                                         │
│  FOR each record in unchanged_list:                    │
│    ticket_id = get_ticket_by_source_id(conn, sid).id    │
│    insert_change_history(conn, ticket_id, batch_ref,    │
│      'unchanged', None, None)                           │
│                                                         │
│  FOR each mod in modified_list:                        │
│    diff_fields = json.dumps(mod.field_diffs)            │
│    affected = json.dumps(list(mod.field_diffs.keys()))  │
│    upsert_ticket(conn, mod.ticket_record,               │
│                  mod.field_diffs)                       │
│    version = get_next_version(conn, mod.ticket_id)      │
│    insert_ticket_version(conn, mod.ticket_id, version,  │
│      mod.previous_status, diff_fields, batch_ref)       │
│    insert_change_history(conn, mod.ticket_id,           │
│      batch_ref, 'modified', affected, 'auto_merged')   │
│                                                         │
│  FOR each conflict in conflict_list:                    │
│    # 冲突不写入 ticket 表；先创建受控队列条目获取 ref  │
│    conflict_ref = write_conflict_queue(conflict)         │
│    insert_change_history(conn, conflict.ticket_id_or_none,│
│      batch_ref, 'conflict', None, 'manual_queue', conflict_ref)│
│                                                         │
│  commit(conn)                                          │
│  异常 → rollback(conn)                                 │
└────────────────────────────────────────────────────────┘
          │
          ▼
┌─ 4. 生成冲突队列文件 ─────────────────────────────────┐
│  IF conflict_list IS NOT EMPTY:                        │
│    写入 conflict-queue.yaml 到输出目录                  │
└────────────────────────────────────────────────────────┘
          │
          ▼
── 输出 ──
ChangeResult {
    stats: {total, new, modified, unchanged, conflict},
    conflict_queue_ref
}
```

### 4.2 语义冲突判定子流程

```python
SEMANTIC_CONFLICT_FIELDS = {'status', 'severity', 'product'}

def check_semantic_conflict(field_diffs: dict) -> dict:
    """筛选出需要人工判断的冲突字段。"""
    conflicts = {}
    for field, diff in field_diffs.items():
        if field in SEMANTIC_CONFLICT_FIELDS:
            # status/severity/product 的任何变更都是语义冲突
            if diff['new'] is not None and diff['new'] != '':
                conflicts[field] = diff
        elif field == 'module':
            # module 从非空变为空 = 信息丢失 → 冲突
            if diff['old'] and not diff['new']:
                conflicts[field] = diff
    return conflicts
```

## 5. 状态机

### 5.1 变更记录生命周期

```
新批次到来
    │
    ├─ 无 source_ticket_id ──► conflict (manual_queue) ──► reviewer 决策 ──┬─► rejected
    │                                                                       └─► 手工创建 ticket
    │
    ├─ 首次出现 ──► new (auto_merged) ──► ticket(version=1)
    │
    ├─ 无变化 ──► unchanged ──► ticket 不变，仅 change_history 记录
    │
    ├─ 字段变更 + 语义冲突 ──► conflict (manual_queue) ──► reviewer 决策 ──┬─► auto_merged（批准合并）
    │                                                                       └─► rejected（拒绝变更）
    │
    └─ 字段变更 + 无冲突 ──► modified (auto_merged) ──► ticket 更新 + ticket_version(N+1)
```

### 5.2 ticket 版本状态视角

```
version=1 ──► S1 首次摄取
    │
    ├─ S2 无变化 → version 不变
    ├─ S2 自动合 ──► version=2
    │   ├─ S2 字段变更 → version=3
    │   └─ S2 无变化 → version 仍为 2
    └─ S2 冲突 → version 不变，conflict_queue 等待处理
```

## 6. 错误处理与降级

| 错误场景 | 处理策略 | 降级行为 |
|---|---|---|
| source_ticket_id 缺失 | 进入 conflict_queue（reason='missing_id'） | 该条记录不写入 ticket，批次中其他记录正常处理 |
| status/severity/product 变更 | 进入 conflict_queue（reason='semantic_conflict'） | 该条记录不更新，保留旧版本 |
| module 字段从有值变为空 | 进入 conflict_queue | 该条记录不更新 |
| 数据库查询异常（已有 ticket 查询失败） | 记录错误 → 该条按 'new' 处理（首次） | 可能导致重复 ticket，但 source_ticket_id UNIQUE 约束会捕获 |
| ticket_version 写入 UNIQUE 冲突 | 事务回滚 | 批次不保存，检查 version 号生成逻辑 |
| 事务提交失败（磁盘满等） | 事务回滚 | 批次全部不保存，已有数据安全 |
| 冲突队列文件写入失败 | 终止该批次事务并保留错误摘要；不得声称已持久化冲突 | 无可审计 `conflict_ref` 时不得提交 change_history，避免产生无法定位的冲突记录 |

**降级原则**：
- 不静默合并冲突项
- 无稳定 ID 是阻断条件，不自动生成 ID
- 其余记录独立处理：一条记录的冲突不影响其他记录的正常写入

## 7. 测试设计

### 7.1 Fixture 计划

| Fixture | 内容 | 覆盖场景 |
|---|---|---|
| `fixtures/db/pre_seeded.db` | 预置 5 条 ticket（version=1）的 SQLite 数据库 | 全部 S2 测试 |
| `fixtures/s2/new_tickets.json` | 3 条新 source_ticket_id | T-DET-01 |
| `fixtures/s2/unchanged_tickets.json` | 与已有记录完全相同的记录 | T-DET-02 |
| `fixtures/s2/modified_fields.json` | title/description 有差异（非冲突字段） | T-DET-03 |
| `fixtures/s2/status_changed.json` | status 从 "已关闭" 变为 "已解决" | T-DET-04 |
| `fixtures/s2/missing_id.json` | source_ticket_id 为空的记录 | T-DET-05 |
| `fixtures/s2/mixed_batch.json` | 含 new + modified + unchanged + conflict 的混合批次 | T-DET-06 |

### 7.2 测试用例与覆盖矩阵

| ID | 场景 | 类型 | 输入 | 预期结果 | 对应 AC |
|---|---|---|---|---|---|
| T-DET-01 | 新 ticket 标记为 new | positive | 新的 source_ticket_id | change_type='new', version=1 | AC-1 |
| T-DET-02 | 已有 ticket 无变化 | positive | 与已有记录完全相同的记录 | change_type='unchanged', ticket 不变, version 不递增 | AC-2 |
| T-DET-03 | 字段变更自动合并 | positive | title 修改 | change_type='modified', field_diffs 含 title, version 递增 | AC-3 |
| T-DET-04 | status 变更进入冲突 | boundary | status 从 "已关闭" → "已解决" | change_type='conflict', resolution='manual_queue' | AC-4 |
| T-DET-05 | 无 stable ID 进入冲突并留痕 | negative | source_ticket_id 缺失 | `change_history.ticket_id=NULL` + `conflict_ref`，队列含 reason='missing_id' | AC-4 |
| T-DET-14 | 冲突队列写入失败不提交 | negative | mock queue 写入失败 | 整批事务回滚；无 change_history 假留痕 | AC-4 |
| T-DET-06 | 混合批次独立处理 | boundary | 多类型混合 | new/modified 正常写入，unchanged 记录，conflict 进队列 | AC-1..4 |
| T-DET-07 | field_diffs JSON 序列化正确 | positive | 多字段变更 | field_diffs 为合法 JSON，包含 {field: {old, new}} | AC-3 |
| T-DET-08 | version 号递增正确 | positive | 连续 3 次 modified | version=2, 3, 4 | AC-3 |
| T-DET-09 | change_history 含 type + resolution | positive | 各种变更类型 | 每条 change_history 记录的 type 和 resolution 字段正确 | AC-5 |
| T-DET-10 | 事务回滚后数据不变 | negative | 模拟写入异常 | ticket 数、version 数、change_history 数均不变 | 隐式 |
| T-DET-11 | product 变更进入冲突 | boundary | product 从 "TGFW" → "TGW" | conflict (semantic_conflict) | AC-4 |
| T-DET-12 | severity 变更进入冲突 | boundary | severity 从 "一般" → "致命" | conflict (semantic_conflict) | AC-4 |
| T-DET-13 | conflict-queue.yaml 生成 | positive | 冲突记录存在 | 输出冲突队列文件 | AC-4（隐式） |

### 7.3 测试执行方式

- 使用临时 SQLite 数据库（`:memory:` 或 `tmp_path`）
- 预置 fixture 数据后验证变更检测结果
- 冲突队列文件验证使用 `tmp_path`

## 8. 安全与权限

| 安全规则 | 实现方式 |
|---|---|
| 不静默覆盖历史数据 | ticket 主表 upsert 更新最新状态；ticket_version 保留旧版本；change_history 记录每次变更 |
| 冲突不自动合并 | 语义冲突字段（status/severity/product）的任何变更进入 manual_queue |
| 无稳定 ID 不伪造 | 直接进入 conflict_queue，不生成随机 ID |
| 数据完整性 | 整个批次使用事务包裹，异常回滚 |

## 9. 实施步骤

### 切片 1：变更检测核心逻辑（TASK-ING-06.1-01, 02）
1. 实现 `compute_field_diff()`：字段级比较
2. 实现 `check_semantic_conflict()`：冲突判定
3. 实现 `detect_changes()` 主函数（匹配 + 分类）
4. 测试：T-DET-01..06

### 切片 2：版本化写入（TASK-ING-06.1-03, 04）
1. 实现 `process_new_tickets()`：INSERT ticket + version=1 + change_history
2. 实现 `process_modified_tickets()`：upsert + ticket_version + change_history
3. 实现 `process_unchanged_tickets()`：仅 change_history 记录
4. 测试：T-DET-07, T-DET-08, T-DET-09

### 切片 3：冲突队列（TASK-ING-06.1-05, 06）
1. 创建 `conflict-queue.yaml` 模板
2. 实现冲突队列文件生成
3. 测试：T-DET-04, T-DET-05, T-DET-11, T-DET-12, T-DET-13

### 切片 4：事务 + 异常处理（TASK-ING-06.1-07）
1. 实现整体事务包裹（批量写入在一个事务中完成或全部回滚）
2. 异常路径处理
3. 测试：T-DET-10

### 验证汇总
- 13 个测试用例全部通过
- conflict-queue.yaml 模板结构合法
- 与 ST-RA-INGEST-DB DAO 的接口调用正确

## 10. 回滚策略

| 变更类型 | 回滚方式 |
|---|---|
| 变更检测逻辑错误（误判冲突） | 修正冲突判定规则 → 通过 change_history 识别受影响记录 → 重新处理 |
| UPSERT 错误覆盖数据 | 从 ticket_version 历史恢复 → 回写 ticket 表 |
| 事务异常导致部分写入 | SQLite 事务保证原子性：异常时自动回滚，无需手动恢复 |
| 冲突队列模板格式错误 | 修正模板 → 重新生成 |

**关键回滚保障**：
- `ticket_version` 表保留每次变更前后的完整状态（field_diffs 含 old/new）
- `change_history` 表记录每次摄取的变更类型和结果
- 任何时间点都可以从 version 历史重建 ticket 状态

## 11. 平台差异检查

| 检查项 | Codex | Claude Code | Qoder | 说明 |
|---|---|---|---|---|
| SQLite UPSERT 语法（`INSERT ... ON CONFLICT`） | SQLite ≥3.24 | 同左 | 同左 | 与 ST-RA-INGEST-DB 相同，无差异 |
| JSON 序列化 | `json.dumps` | 同左 | 同左 | 无差异 |
| 集合/字典操作 | Python 标准库 | 同左 | 同左 | 无差异 |

**结论**: 无平台差异处理项。

## 12. 与相邻模块的集成契约

### 12.1 对 ST-RA-05.2-CLEAN（上游）的输入契约

| 输入 | 格式 | 要求 |
|---|---|---|
| `new_batch` | `list[dict]` | CleanResult.cleaned：清洗后、已映射、含 quality_flag 的 ticket 字典 |

### 12.2 对 ST-RA-INGEST-DB（数据写入）的接口契约

| 操作 | DAO 方法 | 附加 |
|---|---|---|
| 插入新 ticket | `insert_ticket()` | — |
| 更新已有 ticket | `upsert_ticket()` | 传入 field_diffs |
| 插入版本记录 | `insert_ticket_version()` | version 号通过 `get_next_version()` 获取 |
| 插入变更记录 | `insert_change_history()` | change_type 和 resolution 必填 |
| 事务管理 | `begin_transaction()` / `commit()` / `rollback()` | 整个批次包裹在单个事务中 |

### 12.3 对 FEAT-RA-ANALYSIS（ST-RA-06.2-REFRESH，下游）的交付契约

| 数据 | 格式 | 消费方式 |
|---|---|---|
| `ChangeResult` | `new` + `modified` + `unchanged` + `conflicts` + `stats` | 决定哪些 ticket 需要增量重算 |
| `ticket_version` 历史 | 通过 DAO 查询 | 环比/同比分析时消费版本历史 |
| `conflict_queue` | `conflict-queue.yaml` | 人工 reviewer 处理 |

### 12.4 对人工 reviewer 的契约

| 承诺 | 实现 |
|---|---|
| 冲突可见 | `conflict-queue.yaml` 列出每个冲突的源、现有记录、差异字段和建议方案 |
| 决策可追溯 | reviewer 回填 `reviewer_decision` + `resolution_timestamp` |
| 不自动覆盖 | 冲突项在 reviewer 决策前不写入 ticket 表 |

## 13. 开放项与假设

### 开放项

| ID | 描述 | 状态 | 影响 |
|---|---|---|---|
| O-DET-01 | 冲突字段白名单 `{status, severity, product}` 是否完整 | OPEN | 可能需要追加更多语义冲突字段 |
| O-DET-02 | 大规模批次（>1000 条）的变更检测性能 | OPEN（延后至性能 CR） | 可能需要批量查询优化 |

### 假设

| ID | 假设 | 依据 | 风险 |
|---|---|---|---|
| A-DET-01 | source_ticket_id 在不同批次间保持稳定 | ITR 工单 ID 是持久化唯一标识 | 低 |
| A-DET-02 | 同批次内不会有同一 source_ticket_id 出现两次（已在 ST-RA-05.2 去重） | 清洗流程的批次内去重 | 低 |
| A-DET-03 | 自动合并的字段变更不会导致语义不一致 | 非冲突字段是可安全合并的文本/元数据 | 中（需持续观察） |
| A-DET-04 | `first_seen_at` 只在首次 INSERT 时写入，后续不变 | HLD 数据模型定义 | 低 |
| A-DET-05 | `quality_flag` 由清洗流程独立管理，变更检测不修改 | ST-RA-05.2-CLEAN 的职责 | 低 |

## 14. LLD 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | meta-dev | 初始 LLD：变更检测 + 合并 + 版本历史 + 14 章节 |
| 1.1 | 2026-07-16 | host-orchestrator | CP5 Round 4：无稳定 ID 冲突先生成 `conflict_ref`，再以可空 ticket_id 写入 change_history；队列写入失败改为批次回滚，补充负向测试。 |
| 1.2 | 2026-07-16 | host-orchestrator | CP5 B12：迁移为当前 full-lld 证据兼容格式，增加 canonical Story ID 与 §0 工程依据/章节索引；不改变原设计契约。 |
