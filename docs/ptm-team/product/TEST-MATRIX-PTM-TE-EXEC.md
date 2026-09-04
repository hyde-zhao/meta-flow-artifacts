---
status: confirmed
version: "1.2"
source_scenarios: "docs/product/SCENARIOS-PTM-TE-EXEC.yaml"
---

# ptm-te 执行引擎 - 用户旅程 / 需求 / 验证覆盖矩阵

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-pm | CR-033 初始工程场景覆盖矩阵：20 场景覆盖 21 功能需求 + 8 约束 + 5 非功能 |
| 1.1 | 2026-07-28 | meta-pm | CP2 范围修改：新增 SCN-EX-21..25（按标签/关键字执行 + 用例结构化迁移 + 命名冲突 + 全量整改校验），总计 25 场景 |
| 1.2 | 2026-07-28 | meta-pm | CP3 评审范围扩展（环境文件驱动）：新增 SCN-EX-26..29（环境解析成功/占位符失败/多环境切换/DUT接口预配置），总计 29 场景 |

## 覆盖矩阵

| 用户旅程场景 | 用户可观察结果 | Requirement ID | Story ID（待拆） | 验证类型 | 自动化状态 | 手工验收状态 | 验证文件 / 命令 | 未覆盖原因 |
|---|---|---|---|---|---|---|---|---|
| SCN-EX-01（TG 建模 6 组合） | devices.yaml tg 块 + device-reference.md 6 组合全覆盖 | R-F-001,002,003 | ST-EX-TBD | static review | planned | planned | device-management SKILL.md + device-reference.md + devices.yaml fixture | 待 CP5 后实现 fixture |
| SCN-EX-02（目录 glob 批量执行） | 全部用例执行完成，输出四态 result.json + 清理记录 | R-F-004,005,007,015 | ST-EX-TBD | runtime | planned | planned | case_runner.py run --cases-dir --execute | 需 runtime_authorization |
| SCN-EX-03（单用例执行） | 单用例执行完成，输出 result.json | R-F-004 | ST-EX-TBD | runtime | planned | planned | case_runner.py run --case-file | 需 runtime_authorization |
| SCN-EX-04（dry-run 校验） | 每步输出命令和校验状态，不连接设备 | R-F-006 | ST-EX-TBD | static review | planned | planned | case_runner.py run --cases-dir（dry-run） | 待 CP5 后实现 |
| SCN-EX-05（ARP 预热自动清理） | warming_up step 主 op 后自动执行 post_op | R-F-008 | ST-EX-TBD | runtime | planned | planned | case_runner.py + 含 warming_up 用例 | 需 runtime_authorization |
| SCN-EX-06（fw_logout 登出） | session 文件清理，result.json 含 logout 状态 | R-F-016,017 | ST-EX-TBD | runtime | planned | planned | case_runner.py cleanup + op_mapper fw_logout | 需 runtime_authorization |
| SCN-EX-07（dry-run 拒绝写操作） | 只输出命令，不连接设备 | R-F-006, R-NF-001 | ST-EX-TBD | static review | planned | planned | case_runner.py dry-run 模式 | 待 CP5 后实现 |
| SCN-EX-08（未授权 --execute 被拒绝） | 返回 EXEC_FAILED error | R-F-007, R-NF-001 | ST-EX-TBD | static review | planned | planned | op_mapper execute --execute（无 --authorized） | 待 CP5 后实现 |
| SCN-EX-09（known_issue KNOWN_FAIL） | 标记 step 失败输出 KNOWN_FAIL | R-F-010,011 | ST-EX-TBD | runtime | planned | planned | case_runner.py + M4-01-09 用例 | 需 runtime_authorization |
| SCN-EX-10（md 格式错误解析失败） | 输出 ERROR 和解析失败原因，不崩溃 | R-F-004 | ST-EX-TBD | static review | planned | planned | case_runner.py + 格式错误 md | 待 CP5 后实现 |
| SCN-EX-11（空目录 glob） | 输出'未找到用例文件'提示 | R-F-004 | ST-EX-TBD | static review | planned | planned | case_runner.py --cases-dir empty_dir | 待 CP5 后实现 |
| SCN-EX-12（devices.yaml 缺 tg 块） | 输出错误提示，不崩溃 | R-F-005 | ST-EX-TBD | static review | planned | planned | case_runner.py + 缺 tg 块 devices.yaml | 待 CP5 后实现 |
| SCN-EX-13（STATE_INVALID 重连） | 自动重新 login 后重试成功 | R-NF-004 | ST-EX-TBD | runtime | planned | planned | op_mapper execute + 过期 session | 需 runtime 环境 |
| SCN-EX-14（ConnectTimeout 重试） | 按 15/20/25s 递增重试最多 3 次 | R-NF-005 | ST-EX-TBD | runtime | planned | planned | case_runner.py + trex-api 瞬断 | 需 runtime 环境 |
| SCN-EX-15（runtime_auth 审计） | result.json 含 runtime_authorization 字段 | R-NF-002 | ST-EX-TBD | static review | planned | planned | case_runner.py --execute result.json | 待 CP5 后实现 |
| SCN-EX-16（逆序清理 mutation ops） | 清理结果记入 result.json | R-F-015 | ST-EX-TBD | runtime | planned | planned | case_runner.py cleanup | 需 runtime_authorization |
| SCN-EX-17（幂等容错） | 对象已存在/流不存在/被引用阻止视为 PASS | R-F-011 | ST-EX-TBD | runtime | planned | planned | case_runner.py + 幂等场景用例 | 需 runtime_authorization |
| SCN-EX-18（op_mapper 一致性校验） | 22 个 op_id 全覆盖，三表一致 | R-F-016 | ST-EX-TBD | static review | planned | planned | op_mapper.py validate | 待 CP5 后实现 |
| SCN-EX-19（install.py 重装一致性） | 规则块 + skill 存在 + validate PASS | R-F-019, R-NF-003 | ST-EX-TBD | static review | planned | planned | install.py install + 验证脚本 | 待 CP5 后实现 |
| SCN-EX-20（24 用例 ARP 预热校验） | 24/24 用例 warming_up step 合规 | R-F-021 | ST-EX-TBD | static review | planned | planned | ARP 预热校验脚本 | 待 CP5 后实现 |
| SCN-EX-21（按标签精确过滤执行） | --tag 精确匹配 tags 列返回用例并执行 | R-F-004,025,026 | ST-EX-TBD | static review | planned | planned | case_runner.py --tag 策略路由,P0 | 待 CP5 后实现 |
| SCN-EX-22（按关键字模糊匹配执行） | --keyword 子串匹配关键词列返回用例并执行 | R-F-004,025,026 | ST-EX-TBD | static review | planned | planned | case_runner.py --keyword fw_config | 待 CP5 后实现 |
| SCN-EX-23（用例目录结构迁移与命名） | 24/24 用例迁移到三级结构，文件名符合命名约定，frontmatter 16 列完整 | R-F-022,023,024,025 | ST-EX-TBD | static review | planned | planned | 目录结构校验脚本 + frontmatter 校验 | 待 CP5 后实现 |
| SCN-EX-24（用例名称连字符冲突） | 以 frontmatter 用例编号列为唯一标识，文件名解析按编号前缀匹配 | R-F-023 | ST-EX-TBD | static review | planned | planned | 文件名解析测试 | 待 CP5 后实现 |
| SCN-EX-25（24 用例全量整改校验） | 24/24 用例：路径+命名+frontmatter+tags+ARP 预热 全合规 | R-F-021 | ST-EX-TBD | static review | planned | planned | 整改校验脚本 | 待 CP5 后实现 |
| SCN-EX-26（环境文件解析成功） | ${ENV.*} 全部替换为环境文件物理值，测试意图参数保持字面值 | R-F-027,029 | ST-EX-17 | static review | planned | planned | case_runner + --env-file + ${ENV.*} 用例 | 待 CP5 后实现 |
| SCN-EX-27（占位符解析失败） | ${ENV.*} 路径不存在返回 VALIDATION_FAILED，不执行 op | R-F-027 | ST-EX-17 | static review | planned | planned | case_runner + 缺字段环境文件 | 待 CP5 后实现 |
| SCN-EX-28（多环境切换 link3->link4） | 同一用例文件不改一行，换 --env-file 即可在不同环境执行 | R-F-027,028,029 | ST-EX-17 | runtime | planned | planned | case_runner --env-file topology-link3/link4 | 需 runtime_authorization |
| SCN-EX-29（DUT 接口自动预配置+清理） | login 后自动 fw_update_interface 配 IP，用例后逆序清理 | R-F-028 | ST-EX-17 | runtime | planned | planned | case_runner + nodes.dut1.interfaces | 需 runtime_authorization |

## 覆盖统计

| 维度 | 总数 | covered | gap | waived | 说明 |
|------|------|---------|-----|--------|------|
| 正向场景 | 11 | 0 | 11 | 0 | 全部 planned，待 CP5 后实现 |
| 负向场景 | 5 | 0 | 5 | 0 | 全部 planned |
| 边界场景 | 5 | 0 | 5 | 0 | 全部 planned |
| 权限场景 | 1 | 0 | 1 | 0 | 全部 planned |
| 失败恢复 | 2 | 0 | 2 | 0 | 全部 planned |
| 预检场景 | 5 | 0 | 5 | 0 | 全部 planned |
| **合计** | **29** | **0** | **29** | **0** | 全部 planned，待 CP5 后实现 |

## 验证类型分布

| 验证类型 | 场景数 | 说明 |
|---------|--------|------|
| static review | 16 | dry-run / 格式校验 / 映射表一致性 / 安装验证 / 标签过滤 / 目录迁移 / 环境解析 |
| runtime | 13 | 实际执行 / 重连 / 重试 / 幂等容错 / 多环境切换 / DUT 接口预配置（需 runtime_authorization） |

## 未覆盖风险

| 风险 ID | 场景 | 风险说明 | 缓解 |
|---------|------|---------|------|
| RISK-TM-01 | SCN-EX-02,05,06,09,13,14,16,17 | runtime 场景需真实设备和 trex-api，CP7 可能无法全量执行 | static review 替代 + dry-run 验证命令构建 + 人工审查 |
| RISK-TM-02 | SCN-EX-09 | known_issue 判定依赖用例 md 标注完整性 | ARP 预热整改时同步检查 known_issue 标注 |
| RISK-TM-03 | SCN-EX-13,14 | STATE_INVALID/ConnectTimeout 需要真实 session 过期/trex-api 瞬断环境 | mock 测试或人工注入故障 |
