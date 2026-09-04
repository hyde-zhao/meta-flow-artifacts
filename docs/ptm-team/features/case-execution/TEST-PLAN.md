---
cr_id: "CR-033"
artifact_type: "feature-test-plan"
feature_id: "FE-EX-02"
feature_name: "case-execution 执行引擎"
version: "1.0"
created_at: "2026-07-28T16:05:00+08:00"
author: "meta-se"
status: "draft"
source_design: "docs/features/case-execution/DESIGN.md"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_scenarios: "docs/product/SCENARIOS-PTM-TE-EXEC.yaml"
source_matrix: "docs/design/FEATURE-DESIGN-MATRIX-PTM-TE-EXEC.md"
validation_mode: "static-only + dry-run-only + review-only"
covered_stories:
  - "ST-EX-04"
  - "ST-EX-05"
  - "ST-EX-06"
  - "ST-EX-07"
  - "ST-EX-08"
  - "ST-EX-09"
  - "ST-EX-10"
  - "ST-EX-11"
  - "ST-EX-12"
  - "ST-EX-14"
  - "ST-EX-15"
  - "ST-EX-16"
  - "ST-EX-17"
---

# FE-EX-02 case-execution 执行引擎 Feature 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-se | CP3 approved / CP4 PASS 后生成：Feature 级测试策略（static-only + dry-run-only + review-only）；13 Story × 验证点覆盖矩阵；CP7 不真实设备写；真实 --execute 验证需 runtime_authorization 超出 CR-033 范围 |

## 1. 测试策略

### 1.1 validation_mode 声明

`validation_mode = static-only + dry-run-only + review-only`（HLD §13.1/§13.3，ADR-02）。

| 模式 | 适用 | CP7 可验证 | 说明 |
|---|---|---|---|
| static-only | code review / import 路径 / 异常处理 / 无直接 TG REST 调用 | 是 | 检查 case_runner.py 代码结构 |
| dry-run-only | case_runner --dry-run 批量校验 / resolve_env_refs / build_command | 是 | 不连接设备，校验命令构建与解析 |
| review-only | report.md 模板 / 四态矩阵 / extract_payload 字段表 / frontmatter 16 列 | 是 | 人工审查产物 |
| runtime（--execute） | 真实设备写操作 | **否**（CP7 N/A） | 需 runtime_authorization，超出 CR-033 CP7 范围 |

**CP7 N/A 理由**：CR-033 不真实设备写（NO_PRODUCTION_WRITE，HLD 非目标）。真实 --execute 验证需独立 runtime_authorization 决策项，由后续运行验证 CR 承载。HLD §13.1 性能指标（24 用例 < 5 分钟）在 static-only 下不可直接验证，CP7 记录未覆盖风险。

### 1.2 测试目标（量化）

| 指标 ID | 指标 | 目标值 | 验证方式 |
|---|---|---|---|
| TP-FE02-01 | 三入口覆盖 | 3 种（--case-file/--cases-dir/--tag+--keyword） | dry-run 校验 |
| TP-FE02-02 | 零硬编码 | 0（case_runner.py 无硬编码 IP） | static review |
| TP-FE02-03 | extract_payload 覆盖 | 22 op_id（含 fw_logout） | review 字段表 |
| TP-FE02-04 | 四态分级 | 4 态（PASS/FAIL/KNOWN_FAIL/ERROR） | dry-run + review |
| TP-FE02-05 | ${ENV.*} 占位符 | 9 类 | dry-run resolve_env_refs |
| TP-FE02-06 | 重装一致性 | 0 丢失（case-execution skill） | install/uninstall/install 循环 |
| TP-FE02-07 | 24 用例 dry-run | 24/24 解析成功 | dry-run 批量 |
| TP-FE02-08 | fw_logout 降级 | 2 路径（op 映射 / session 清理） | dry-run + review |

### 1.3 测试依据来源（理论依据）

- ISTQB 测试结果分级（PASS/FAIL/KNOWN_FAIL/ERROR 四态，HLD §21.7）
- FMEA 故障模式分析（幂等容错/重连/重试/降级，HLD §21.7）
- JTBD（测试执行工程师"工作"是用例执行，不是写代码，HLD §21.7）

## 2. 测试覆盖矩阵（Story × 验证点）

| Story | 验证点 | 验证模式 | 预期 | 关联场景 |
|---|---|---|---|---|
| ST-EX-04 | 三入口用例发现（--case-file/--cases-dir） | dry-run | 24 用例全部发现 | SCN-EX-02,03 |
| ST-EX-04 | devices.yaml 取址（firewall.host + tg.api_server） | dry-run | DUT_URL/TREX_API_URL 正确解析 | SCN-EX-02 |
| ST-EX-04 | case_steps YAML 解析 | dry-run | 24 用例 step 全部解析 | SCN-EX-02 |
| ST-EX-04 | load_env_file 加载 --env-file | dry-run | env_topology 正确构建 | SCN-EX-26 |
| ST-EX-04 | case-execution 安装验证（PTM_TE_SKILLS 含 case-execution） | static | install.py PTM_TE_SKILLS 含 case-execution | SCN-EX-18 |
| ST-EX-04 | import op_mapper 路径 | static | sys.path 含 op_mapper 目录 | SCN-EX-16 |
| ST-EX-05 | dry-run 默认门（不执行写操作） | dry-run | mutation op 只构建命令不执行 | SCN-EX-04 |
| ST-EX-05 | --execute --authorized 授权门 | review | runtime_authorization 审计字段 | SCN-EX-07 |
| ST-EX-05 | 未授权拒绝（EXEC_FAILED） | dry-run | authorized=False 时返回 EXEC_FAILED | SCN-EX-08 |
| ST-EX-05 | dry-run fw_login 跳过实际登录 | dry-run | fw_login step 只构建命令 | SCN-EX-04 |
| ST-EX-06 | 逆序清理 mutation ops | dry-run | config 顺序逆序执行 inverse_op | SCN-EX-07 |
| ST-EX-06 | step-refs 读取（${STEP-N.id}） | dry-run | 前序 step id 正确引用 | SCN-EX-08 |
| ST-EX-06 | irreversible 类不回滚 | review | reset-hitcount 不清理 | SCN-EX-07 |
| ST-EX-07 | op_mapper fw_logout 映射 | static | OP_ID_TO_SUBCOMMAND 含 fw_logout | SCN-EX-19 |
| ST-EX-07 | EXPECTED_OP_COUNT=22 | static | validate_mapping_consistency 通过 | SCN-EX-19 |
| ST-EX-07 | fw_logout 降级 session 清理 | review | 未暴露时 os.remove(session_file) | SCN-EX-19 |
| ST-EX-07 | result.json logout 状态 | review | success/fallback_session_cleanup | SCN-EX-19 |
| ST-EX-08 | warming_up:true 强制 post_op | dry-run | tg_stop_traffic_stream 自动补充 | SCN-EX-05 |
| ST-EX-08 | post_op 参数继承（ports/txport/rxport/name） | dry-run | 从主 op 继承 | SCN-EX-05 |
| ST-EX-08 | auto_post_op=true 标记 | review | result.json 标记 | SCN-EX-05 |
| ST-EX-09 | retry 字段解析（interval/count/until） | dry-run | retry 配置正确解析 | SCN-EX-10 |
| ST-EX-09 | 轮询执行 | review | 按 interval/count 轮询 | SCN-EX-10 |
| ST-EX-10 | known_issue=true -> KNOWN_FAIL | dry-run | step fail 且 known_issue 时 KNOWN_FAIL | SCN-EX-09 |
| ST-EX-10 | 四态判定优先级（ERROR>KNOWN_FAIL>FAIL>PASS） | review | 矩阵正确 | SCN-EX-09 |
| ST-EX-11 | 失败诊断字段提取（error_type/code/reason/details/command） | review | diagnose_failure 输出完整 | SCN-EX-17 |
| ST-EX-11 | report.md 四态统计表 | review | 模板完整 | SCN-EX-17 |
| ST-EX-11 | 幂等容错记录（对象已存在/流不存在） | review | 记录正确 | SCN-EX-17 |
| ST-EX-12 | extract_payload(op_id, envelope) | review | 22 op_id 字段提取表完整 | SCN-EX-10 |
| ST-EX-12 | policy_route_id 提取 | dry-run | fw_config_policy_route -> data.policy_route_id | SCN-EX-08 |
| ST-EX-14 | tg_verify_traffic_loss 提取 tx/rx/loss_ratio | review | 字段提取表覆盖 | SCN-EX-25 |
| ST-EX-15 | 目录三级结构 | review | cases/特性/配置管理/策略配置/*.md | SCN-EX-21 |
| ST-EX-15 | 命名正则（^PC-[A-Z0-9]+-\d+-\d+-\d+） | dry-run | 24 用例编号匹配 | SCN-EX-21 |
| ST-EX-15 | frontmatter 16 列（8 必填 + 8 可选） | review | 缺失可选列填 N/A | SCN-EX-21 |
| ST-EX-15 | 忽略测试步骤/预期结果列 | review | case_runner 只读 case_steps | SCN-EX-21 |
| ST-EX-16 | --tag 精确过滤 | dry-run | tags 列精确匹配 | SCN-EX-22 |
| ST-EX-16 | --keyword 模糊匹配 | dry-run | 标题/描述模糊匹配 | SCN-EX-22 |
| ST-EX-17 | env_topology 契约（port_mapping/nodes/links） | dry-run | env-file 解析正确 | SCN-EX-26 |
| ST-EX-17 | ${ENV.*} 9 类占位符解析 | dry-run | 9 类全部解析成功 | SCN-EX-27 |
| ST-EX-17 | 向后兼容（无 ${ENV.*} 原样透传） | dry-run | 字面值透传 | SCN-EX-28 |
| ST-EX-17 | DUT 接口预配置（fw_update_interface） | review | 按 nodes.dut1.interfaces | SCN-EX-29 |
| ST-EX-17 | 清理顺序（先 mutation ops 逆序，再预配置接口逆序） | review | 顺序正确 | SCN-EX-29 |
| ST-EX-17 | TREX_API_URL 环境文件优先（devices.yaml fallback） | dry-run | ${ENV.tg.url} 优先 | SCN-EX-27 |
| ST-EX-17 | YAML 1.1 整数陷阱规避（端口名加引号） | dry-run | 端口名字符串类型校验 | SCN-EX-28 |
| ST-EX-17 | resolve_env_refs 解析失败 -> VALIDATION_FAILED | dry-run | 占位符无对应键时报错 | SCN-EX-28 |
| ST-EX-17 | 无直接 TG REST 调用 | static | code review 无 requests/urllib 调 TG | SCN-EX-28 |

## 3. 验证场景映射（SCN-EX -> Story）

| 场景 ID | 场景 | 关联 Story | 验证模式 |
|---|---|---|---|
| SCN-EX-02 | 目录 glob 批量执行 | ST-EX-04 | dry-run |
| SCN-EX-03 | 单用例执行调试 | ST-EX-04 | dry-run |
| SCN-EX-04 | dry-run 校验 | ST-EX-05 | dry-run |
| SCN-EX-05 | ARP 预热自动清理 | ST-EX-08 | dry-run |
| SCN-EX-07 | 逆序清理 + fw_logout | ST-EX-06,07 | dry-run |
| SCN-EX-08 | step-refs + 授权 | ST-EX-05,06,12 | dry-run |
| SCN-EX-09 | known_issue 四态分级 | ST-EX-10 | dry-run |
| SCN-EX-10 | retry + extract_payload | ST-EX-09,12 | dry-run |
| SCN-EX-16 | import op_mapper | ST-EX-04 | static |
| SCN-EX-17 | 失败诊断 + report | ST-EX-11 | review |
| SCN-EX-18 | case-execution 安装验证 | ST-EX-04 | static |
| SCN-EX-19 | fw_logout op 映射 | ST-EX-07 | static |
| SCN-EX-21 | 用例结构化 | ST-EX-15 | review |
| SCN-EX-22 | 标签/关键字执行 | ST-EX-16 | dry-run |
| SCN-EX-25 | verify_loss 消费侧 | ST-EX-14 | review |
| SCN-EX-26 | env-file 加载 | ST-EX-04,17 | dry-run |
| SCN-EX-27 | ${ENV.*} 解析 | ST-EX-17 | dry-run |
| SCN-EX-28 | 向后兼容 + 整数陷阱 | ST-EX-17 | dry-run |
| SCN-EX-29 | DUT 接口预配置 + 清理顺序 | ST-EX-17 | review |

## 4. 测试分层计划

| 层级 | 范围 | 方式 | 责任 |
|---|---|---|---|
| L1 静态检查 | import 路径 / 异常处理 / 无直接 TG REST / EXPECTED_OP_COUNT / OP_ID_TO_SUBCOMMAND | code review + grep | meta-qa |
| L2 dry-run | 24 用例批量 dry-run / resolve_env_refs / build_command / 三入口 / 标签过滤 | case_runner --dry-run | meta-qa |
| L3 产物审查 | report.md 模板 / 四态矩阵 / extract_payload 字段表 / frontmatter 16 列 / 清理顺序 | 人工审查 | meta-qa |
| L4 重装一致性 | install/uninstall/install 循环 / PTM_TE_SKILLS 含 case-execution / op_mapper validate | install.py 循环 | meta-qa |
| L5 runtime（N/A） | 真实设备 --execute | 需 runtime_authorization | 后续 CR |

## 5. fixture / dry-run 计划

| fixture | 用途 | 来源 |
|---|---|---|
| fixtures/devices.test.yaml | 含 firewall.host + tg.api_server 的测试 devices.yaml | meta-qa 构造 |
| fixtures/env-link3.yaml | link3 环境 env-file（port1=2_3/port2=2_4） | meta-qa 构造 |
| fixtures/env-link4.yaml | link4 环境 env-file（port1=2_1/port2=2_2） | meta-qa 构造 |
| fixtures/case-sample.md | 含 ${ENV.*} 的示例用例 | meta-qa 构造 |
| fixtures/case-legacy.md | 不含 ${ENV.*} 的旧用例（向后兼容） | meta-qa 构造 |

dry-run 命令模板：
```bash
python skills/case-execution/scripts/case_runner.py run \
  --cases-dir ptm-te/cases/IPv4策略路由/ \
  --devices-yaml fixtures/devices.test.yaml \
  --env-file fixtures/env-link3.yaml \
  --dry-run --output-dir runs/test-run-id/
```

## 6. 风险与未覆盖项

| 风险 ID | 风险 | CP7 处理 |
|---|---|---|
| R-FE02-01 | 真实设备 --execute 写操作未验证 | CP7 N/A，需 runtime_authorization 后续 CR |
| R-FE02-02 | 性能指标（24 用例 < 5 分钟）未验证 | static-only 下不可验证，CP7 记录 |
| R-FE02-03 | fw_logout 在 ptm-atomic 安装版是否暴露 | 安装前 `ptm-atomic show fw_logout` 验证，未暴露走降级 |
| R-FE02-04 | STATE_INVALID 重连 / ConnectTimeout 重试未真实触发 | dry-run 不可触发，review 逻辑分支 |
| R-FE02-05 | 24 用例 known_issue 标注完整性 | 与 ST-EX-13 协作，整改时同步检查（DQ-CP3-03） |

## 7. 阶段决策

| 决策 | 内容 |
|---|---|
| CP7 结论预期 | PASS_WITH_RISK（runtime 未覆盖为已知风险，static+dry-run+review 全过） |
| 风险接受项 | R-FE02-01/02/04 进 CP8 risk acceptance |
| 后续 CR 候选 | 真实设备 runtime 验证 CR（含 runtime_authorization） |
