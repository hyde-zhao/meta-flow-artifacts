# PTM Team 测试覆盖矩阵

> 版本：v1.0 · 更新：2026-06-08 · 覆盖：ptm-tde（已交付）+ 其余 Agent（规划中）

---

## 追溯矩阵

### ptm-tde

| 场景 ID | 需求 | Story | LLD | CP6 | CP7 | 状态 |
|---|---|---|---|---|---|---|
| SCN-TDE-001 | REQ-001~003 | STORY-011-01 | STORY-011-01-LLD | CP6-STORY-011-01 PASS | CP7-CR-011 PASS | ✅ verified |
| SCN-TDE-002 | REQ-004~006 | STORY-011-02/03 | STORY-011-02/03-LLD | CP6-STORY-011-02/03 PASS | CP7-CR-011 PASS | ✅ verified |
| SCN-TDE-003 | REQ-007~012 | STORY-012-03 + 016-01 + 017-01 | 3 LLD | 3 CP6 PASS | 3 CP7 PASS | ✅ verified |
| SCN-TDE-004 | REQ-013~016 | STORY-012-04 | STORY-012-04-LLD | CP6-STORY-012-04 PASS | CP7-CR-012 PASS | ✅ verified |
| SCN-TDE-005 | REQ-017~019 | STORY-012-05 | STORY-012-05-LLD | CP6-STORY-012-05 PASS | CP7-CR-012 PASS | ✅ verified |
| SCN-TDE-006 | REQ-020~024 | STORY-013-01/04 | 2 LLD | 2 CP6 PASS | CP7-CR-013 PASS | ✅ verified |
| SCN-TDE-007 | REQ-025~026 | STORY-013-02/03 | 2 LLD | 2 CP6 PASS | CP7-CR-013 PASS | ✅ verified |
| SCN-TDE-008 | REQ-027 | STORY-011-04 | STORY-011-04-LLD | CP6-STORY-011-04 PASS | CP7-CR-011 PASS | ✅ verified |
| SCN-TDE-009 | REQ-028 | STORY-012-07 | STORY-012-07-LLD | CP6-STORY-012-07 PASS | CP7-CR-012 PASS | ✅ verified |
| SCN-TDE-010 | NFR-01~03 | 全部 Story | 自测覆盖 | CP6 自测 PASS | CP7 PASS | ✅ verified |
| SCN-TDE-011 | REQ-029 | CR-015 fast-lane | — | CP6-CR-015 PASS | CP7-CR-015 PASS | ✅ verified |

### 其余 Agent（规划中）

| 场景 ID | Agent | 需求 | Story | 状态 |
|---|---|---|---|---|
| SCN-TE-001~003 | ptm-te | 待定义 | 待拆分 | ⬜ planned |
| SCN-TAE-001~003 | ptm-tae | 待定义 | 待拆分 | 🔄 in-progress |
| SCN-TM-001 | ptm-tm | 待定义 | 待拆分 | ⬜ planned |
| SCN-TSE-001 | ptm-tse | 待定义 | 待拆分 | ⬜ planned |
| SCN-QA-001 | ptm-qa | 待定义 | 待拆分 | ⬜ planned |
| SCN-CROSS-001~003 | 跨 Agent | 待定义 | 待拆分 | ⬜ planned |

---

## 覆盖统计

| Agent | 场景数 | 已验证 | 规划中 | 验证率 |
|---|---|---|---|---|
| ptm-tde | 11 | 11 | 0 | 100% |
| ptm-te | 3 | 0 | 3 | 0% |
| ptm-tae | 3 | 0 | 3 | 0% |
| ptm-tm | 1 | 0 | 1 | 0% |
| ptm-tse | 1 | 0 | 1 | 0% |
| ptm-qa | 1 | 0 | 1 | 0% |
| 跨 Agent | 3 | 0 | 3 | 0% |
| **合计** | **22** | **11** | **11** | **50%** |

---

## 验证证据索引

| 类型 | 路径 | 覆盖范围 |
|---|---|---|
| CP7 全局 | `process/checks/CP7-CR-011-global-VERIFICATION-DONE.md` | CR-011（4 Stories） |
| CP7 全局 | `process/checks/CP7-CR-012-global-VERIFICATION-DONE.md` | CR-012（8 Stories） |
| CP7 全局 | `process/checks/CP7-CR-013-global-VERIFICATION-DONE.md` | CR-013（4 Stories） |
| CP7 全局 | `process/checks/CP7-CR-016-global-VERIFICATION-DONE.md` | CR-016 |
| CP7 全局 | `process/checks/CP7-CR-017-global-VERIFICATION-DONE.md` | CR-017 |
| CP7 单点 | `process/checks/CP7-CR-015-ask-user-question-VERIFICATION-DONE.md` | CR-015 |
| 实现证据 | `docs/ptm-tde/IMPLEMENTATION.md` | 高风险 Story 汇总 |

---

*本矩阵覆盖 ptm-team 全部 6 个 Agent。ptm-tde 已 100% 验证通过，其余 Agent 待启动。*
