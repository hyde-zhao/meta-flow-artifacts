# BGP4+ D-Data 等价类与边界值分析 — LC-BPG-004-MD5：MD5 密码边界

> 生成时间: 2026-06-12 | 设计 Skill: data-design
> 上游输入: logic-cases.md §LC-BPG-004, ppdcs-annotation.md §M1, directory-structure.md
> PPDCS: LC-BPG-004 标注为 S-State + D-Data；本文件仅覆盖 D-Data 维度（MD5 密码数据因子）。
> 注意: 本文件为用户额外指定的 MD5 密码边界专项分析。

---

## 一、输入对齐

| 输入来源 | 关键信息 |
|---------|---------|
| design-plan.md | 缺省 |
| design-planner-reasoning.md | 缺省 |
| logic-cases.md | LC-BPG-004: MD5 开关 + MD5 密码，因子-取值表含密码边界值 |
| ppdcs-annotation.md | M1 辅 S-State（MD5 启用/禁用状态切换） |
| directory-structure.md | 四级=配置管理, 五级=BPG配置, SR-007e |
| 用户直接输入 | FAC-BGP-MD5-PASSWORD(≤32字符,无空格,拒绝<>) |

### 拓扑绑定旁路

MD5 密码为邻居配置的数据字段，不直接涉及端口物化。

---

## 二、Factor Catalog

| factor_id | 数据项 | 值域摘要 | data_type | source_ref | fact_status | confirmation_gap_refs |
|-----------|-------|---------|-----------|------------|-------------|------------------------|
| FAC-BGP-MD5-PASSWORD | MD5 认证密码 | ≤32 字符，无空格，拒绝 <> | string | LC-BPG-004, SR-007e, TP-M1-018, TP-Q-SEC-002 | confirmed | — |

---

## 三、Factor 取值定义

### FAC-BGP-MD5-PASSWORD（MD5 认证密码）

| 属性 | 值 |
|------|---|
| 数据类型 | string |
| 值域 | 1~32 字符 |
| 允许字符 | 可打印 ASCII（字母、数字、特殊符号），不含空格 |
| 拒绝字符 | 空格（任何位置）、`<`、`>` |
| 有效典型值 | `TestPass123`（字母+数字，13字符） |
| 有效典型值 | `!@#$%^&*()`（纯特殊符号，10字符） |
| 有效边界值 | 1 个字符（最小长度）：如 `A`、`1` |
| | 32 个字符（最大长度） |
| 有效典型值 | `Abc` 与 `abc`（大小写不同，视为不同密码） |
| 无效典型值 | `""`（空串，长度 0） |
| | 33 字符（超长） |
| | `abc 123`（含空格） |
| | `pass<word>`（含 <） |
| | `pass>word`（含 >） |
| | ` pass`（前导空格） |
| | `pass `（尾随空格） |
| 默认值 | 无（仅当 authEnable=true 时必填） |

**密码约束来源**：SR-007e 规定 `≤32字符,无空格`；TP-Q-SEC-002 验证注入字符 `<>` 被拒绝。

---

## 四、等价类与边界值分析

### FAC-BGP-MD5-PASSWORD

| value_class_id | 取值/范围 | class_type | boundary_role | 预期结果 | source_refs | fact_status |
|:---|---------|:---:|:---:|---------|-------------|:---:|
| EVP-PWD-01 | `TestPass123` | valid-typical | typical | 接受，邻居 MD5 认证成功 | TP-M1-016 | confirmed |
| EVP-PWD-02 | `!@#$%^&*()` | valid-typical | typical | 接受（特殊符号合法） | TP-M1-018 | confirmed |
| EVP-PWD-03 | `Abc123` | valid-typical | typical | 接受（大小写混合） | TP-M1-018 | confirmed |
| EVP-PWD-04 | `a` | valid-boundary | min-on | 接受（1 字符） | inferred | confirmed |
| BVP-PWD-MIN | `A` | valid-boundary | min-on | 接受（1 字符大写） | inferred | confirmed |
| BVP-PWD-MAX | 32 字符（如 `aaaa...a` x32） | valid-boundary | max-on | 接受 | SR-007e | confirmed |
| EVP-PWD-05 | `Abc` vs `abc` | valid-typical | case-sensitive | 大小写不同即视为不同密码 | TP-M1-018 | confirmed |
| IVP-PWD-EMPTY | `""` (空串) | invalid-boundary | min-off | 拒绝（长度=0） | TP-M1-018 | confirmed |
| IVP-PWD-LONG | 33 字符 | invalid-boundary | max-off | 拒绝（超长） | TP-M1-018 | confirmed |
| IVP-PWD-SPACE | `abc 123` | invalid | space-embedded | 拒绝（含空格） | TP-M1-018 | confirmed |
| IVP-PWD-LT | `pass<word` | invalid | forbidden-char-lt | 拒绝（含 <） | TP-Q-SEC-002 | confirmed |
| IVP-PWD-GT | `pass>word` | invalid | forbidden-char-gt | 拒绝（含 >） | TP-Q-SEC-002 | confirmed |
| IVP-PWD-LEAD-SPACE | ` pass` | invalid | leading-space | 拒绝（前导空格） | inferred | confirmed |
| IVP-PWD-TRAIL-SPACE | `pass ` | invalid | trailing-space | 拒绝（尾随空格） | inferred | confirmed |
| IVP-PWD-NULL | `null` / 缺失 | invalid | empty | 拒绝（MD5 启用时必填） | inferred | confirmed |
| IVP-PWD-CTRL | `pass\tword` (含控制字符) | invalid | control-char | [待确认] TAB 等控制字符行为 | inferred | needs-confirmation |
| IVP-PWD-NONASCII | `密码123` (中文) | invalid | non-ascii | [待确认] 非 ASCII 字符是否允许 | inferred | needs-confirmation |
| IVP-PWD-SHELL | `$(whoami)` | invalid | shell-injection | [待确认] shell 注入字符行为 | inferred | needs-confirmation |

---

## 五、边界值表（字符串长度边界）

| factor_id | min-off | min-on | typical | max-on | max-off |
|-----------|:-----:|:-----:|:-----:|:-----:|:-----:|
| FAC-BGP-MD5-PASSWORD (长度) | 0 | 1 | 13 | 32 | 33 |

注：
- 字符串型因子附加维度：字符集约束（无空格+拒绝<>）。
- `min-off` = 空串（长度 0）；`max-off` = 33 字符。
- 除长度边界外，还需覆盖"含空格/含<>"等价类。

---

## 六、独立性检查

| 检查项 | 结论 | 证据 | 处理 |
|--------|:--:|------|------|
| MD5 密码是否依赖 MD5 开关状态 | 否（D-Data 层面） | 密码值合法性校验（长度/字符集）独立于 authEnable 状态；authEnable=false 时密码可不传 | 保持 D-Data |
| MD5 密码是否依赖邻居 IP | 否 | 所有邻居共享相同的密码格式约束 | 保持 D-Data |
| 是否需跨因子交叉覆盖 | 否 | D-Data 维度仅关注密码值本身的合法性 | 保持 D-Data |
| 是否存在可能应回退的信号 | 否 | S-State 维度（MD5 启用/禁用 + 密码一致/不一致状态）由 state-design 覆盖 | — |

**结论**：MD5 密码作为数据因子通过独立性检查，等价类+边界值完全适用。

---

## 七、Data Row 分配与 LC 叠加

| data_row_id | data_type | lc_step_ref | 分配值 | td_refs | 预期结果 | fact_status |
|:---|:---|:---|---------|:---|---------|:---|
| DR-001 | valid-typical | P1/P2 | pwd=TestPass123 | TP-M1-016 | 接受，认证成功 | confirmed |
| DR-002 | valid-typical | P2 | pwd=!@#$%^&*() | TP-M1-018 | 接受 | confirmed |
| DR-003 | valid-boundary | P2 | pwd=A (1字符) | SR-007e | 接受 | confirmed |
| DR-004 | valid-boundary | P2 | pwd=a×32 (32字符) | SR-007e | 接受 | confirmed |
| DR-005 | valid-typical | P2 | 对端 pwd=Abc vs pwd=abc (大小写不一致) | TP-M1-018 | 视为不同密码，认证失败 | confirmed |
| DR-006 | invalid-boundary | P2 | pwd="" (空串) | TP-M1-018 | 拒绝 | confirmed |
| DR-007 | invalid-boundary | P2 | pwd=a×33 (33字符) | TP-M1-018 | 拒绝 | confirmed |
| DR-008 | invalid | P2 | pwd=abc 123 (含空格) | TP-M1-018 | 拒绝 | confirmed |
| DR-009 | invalid | P2 | pwd=pass<word (含<) | TP-Q-SEC-002 | 拒绝 | confirmed |
| DR-010 | invalid | P2 | pwd=pass>word (含>) | TP-Q-SEC-002 | 拒绝 | confirmed |
| DR-011 | invalid | P2 | pwd= pass (前导空格) | inferred | 拒绝 | confirmed |
| DR-012 | invalid | P2 | pwd=pass (尾随空格) | inferred | 拒绝 | confirmed |
| DR-013 | invalid | P2 | pwd=null/missing | inferred | 拒绝 | confirmed |
| DR-014 | invalid-gap | P2 | pwd=含 TAB 控制字符 | — | [待确认] | needs-confirmation |
| DR-015 | invalid-gap | P2 | pwd=中文密码 (非ASCII) | — | [待确认] | needs-confirmation |
| DR-016 | invalid-gap | P2 | pwd=$(whoami) (shell注入) | — | [待确认] | needs-confirmation |

---

## 八、物理用例概要

物理用例（PC）由 `LC-BPG-004 P2（密码边界） + data_row → PC` 叠加生成，完整 16 列表输出见 `ppdcs/pc/BGP-配置管理-BPG配置-LC-BPG-004-MD5.md`。

| PC-ID | data_row | 测试场景 | 优先级 | fact_status |
|:---|:---|---------|:---:|:---|
| PC-BPG-DAT-053 | DR-001 | 正常密码（字母+数字） | P1 | confirmed |
| PC-BPG-DAT-054 | DR-002 | 特殊字符密码 | P1 | confirmed |
| PC-BPG-DAT-055 | DR-003 | 最小长度 1 字符 | P2 | confirmed |
| PC-BPG-DAT-056 | DR-004 | 最大长度 32 字符 | P2 | confirmed |
| PC-BPG-DAT-057 | DR-005 | 大小写不一致 | P2 | confirmed |
| PC-BPG-DAT-058 | DR-006 | 空密码拒绝 | P2 | confirmed |
| PC-BPG-DAT-059 | DR-007 | 超长 33 字符拒绝 | P2 | confirmed |
| PC-BPG-DAT-060 | DR-008 | 含空格拒绝 | P2 | confirmed |
| PC-BPG-DAT-061 | DR-009 | 含 < 拒绝 (XSS/SQL注入防护) | P2 | confirmed |
| PC-BPG-DAT-062 | DR-010 | 含 > 拒绝 | P2 | confirmed |
| PC-BPG-DAT-063 | DR-011 | 前导空格拒绝 | P3 | confirmed |
| PC-BPG-DAT-064 | DR-012 | 尾随空格拒绝 | P3 | confirmed |
| PC-BPG-DAT-065 | DR-013 | null/missing 拒绝 | P3 | confirmed |
| PC-BPG-DAT-066 | DR-014 | 控制字符 [待确认] | P3 | needs-confirmation |
| PC-BPG-DAT-067 | DR-015 | 非 ASCII [待确认] | P3 | needs-confirmation |
| PC-BPG-DAT-068 | DR-016 | shell 注入 [待确认] | P3 | needs-confirmation |

---

## 九、待确认事项清单

| gap_ref | 事项 | 影响范围 | 来源 |
|---------|------|---------|------|
| LC-BPG-004-GAP-CTRL | TAB、换行等控制字符是否被拒绝 | 密码字符集边界 | IVP-PWD-CTRL |
| LC-BPG-004-GAP-NONASCII | 中文/Unicode 字符是否允许作为密码 | 密码编码边界 | IVP-PWD-NONASCII |
| LC-BPG-004-GAP-SHELL | shell 注入字符 (${}, backtick, ;) 是否被转义/拒绝 | 安全注入防护 | IVP-PWD-SHELL |

---

> 关联文件: `ppdcs/pc/BGP-配置管理-BPG配置-LC-BPG-004-MD5.md`（物理用例输出）
