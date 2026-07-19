---
status: ready-for-human-gate
version: "1.0"
release_artifact_profile: full
release_decision: READY_WITH_RISK
migration: none
created_at: "2026-07-19T01:47:00+08:00"
---

# CR-172 PATH-I Migration

## 1. 迁移结论

| 项目 | 内容 |
|---|---|
| 是否需要迁移 | no |
| 是否自动迁移 | N/A |
| 是否保留兼容路径 | yes；现有 runner/lineage/runtime 未替换 |
| 是否可逆 | repository-local changes 可回滚；无数据迁移 |
| CP8 fact_diff 迁移影响 | NOT_APPLICABLE |

## 2. 兼容性判断表

| 对象 | 是否变化 | 兼容性 | 需要迁移 | 验证方式 | 回滚方式 |
|---|---|---|---|---|---|
| `STATE.md` / machine state schema | no | N/A | no | CP8 packet/Result | N/A |
| 模板字段 | no | N/A | no | scoped evidence | N/A |
| 配置格式 | no | N/A | no | config changes=0 | N/A |
| 安装路径 | no | N/A | no | install surface=N/A | N/A |
| Agent frontmatter | no | N/A | no | delivery agents out-of-scope | N/A |
| Skill 输出格式 | no | N/A | no | delivery skills out-of-scope | N/A |
| 命令参数 | no | N/A | no | public CLI change=0 | N/A |
| 数据存储结构 | no real migration | repository contract additive | no | real writes=0 | repository revert only |
| existing runner/lineage | no | compatible | no | planned/actual diff=`0/0` | N/A |
| public C1 projection | no | deferred | no now | claim remains false | future CR |

## 3. 迁移步骤

| Step | 操作 | 前置条件 | 验证 | 回退 |
|---:|---|---|---|---|
| 1 | N/A：不执行历史目录、manifest、state、catalog pointer、NAS 或 execution cache 迁移 | real operations remain unauthorized | zero-operation evidence | N/A |

## 4. N/A 说明

| 项目 | 原因 | 后续触发条件 |
|---|---|---|
| runtime/state migration | 本 CR 只交付 repository contracts | future runtime implementation changes state/schema |
| data migration | 没有真实数据读写或历史 backfill | approved data activation/migration CR |
| installation migration | 没有 installer/path change | future packaging/install change |
| public C1 migration | `public_c1_projection_ready=false` | approved projection compatibility CR |

`migration=none` 不表示 future activation 无迁移风险；它只描述当前 PATH-I repository-local slice。
