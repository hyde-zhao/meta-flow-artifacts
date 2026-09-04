---
cr_id: "CR-033"
story_id: "STORY-EX-04"
artifact_type: "implementation"
lld_ref: "process/stories/STORY-EX-04-LLD.md"
status: "implemented"
implemented_at: "2026-07-29T17:20:00+08:00"
author: "meta-dev"
---

# ST-EX-04 实现执行证据：case_runner 核心引擎

## 1. 实现对象清单

| 对象 | 文件路径 | 锚点 | 动作 | 行数 |
|---|---|---|---|---|
| case_runner.py 模块骨架 | skills/case-execution/scripts/case_runner.py | 模块级（imports/sys.path/异常类/常量） | 新建 | L1-L100 |
| main(argv) | skills/case-execution/scripts/case_runner.py#main | ST-EX-04#main | 新建 | main 函数 |
| discover_cases | skills/case-execution/scripts/case_runner.py#discover_cases | ST-EX-04#discover_cases | 新建 | discover_cases 函数 |
| parse_case_file | skills/case-execution/scripts/case_runner.py#parse_case_file | ST-EX-04#parse_case_file | 新建 | parse_case_file 函数 |
| resolve_addresses | skills/case-execution/scripts/case_runner.py#resolve_addresses | ST-EX-04#resolve_addresses | 新建 | resolve_addresses 函数 |
| load_env_file | skills/case-execution/scripts/case_runner.py#load_env_file | ST-EX-04#load_env_file | 新建 | load_env_file 函数 |
| _resolve_tg_api_server | skills/case-execution/scripts/case_runner.py#_resolve_tg_api_server | ST-EX-04#_resolve_tg_api_server | 新建 | _resolve_tg_api_server 函数 |
| _validate_startup | skills/case-execution/scripts/case_runner.py#_validate_startup | ST-EX-04#_validate_startup | 新建 | _validate_startup 函数 |
| SKILL.md | skills/case-execution/SKILL.md | ST-EX-04#SKILL.md | 新建 | 全文 |
| install.py PTM_TE_SKILLS | script/ptm_team/install.py#PTM_TE_SKILLS | ST-EX-04#PTM_TE_SKILLS | 修改 | 新增 "case-execution" 条目 |

## 2. 设计契约映射

| LLD 契约 | 实现落点 | 一致性 |
|---|---|---|
| §3.1 main 三入口（--case-file / --cases-dir / validate） | main() argparse run 子命令 + validate 子命令；--case-file 优先于 --cases-dir | ✓ 一致 |
| §3.2 discover_cases glob **/*.md + 编号正则排序（ADR-07） | rglob("*.md") + 排除 README.md + CASE_ID_RE 排序 | ✓ 一致 |
| §3.3 parse_case_file frontmatter 16 列只消费 4 列（AGA-03=C） | 只取 用例编号/用例名称/标签/关键词；完整 frontmatter 保留在 frontmatter 字段 | ✓ 一致 |
| §3.3 case_steps YAML 代码块解析 | 正则提取 ## case_steps 后 ```yaml ... ``` 代码块，yaml.safe_load | ✓ 一致 |
| §3.4 resolve_addresses devices.yaml firewall.host + tg.api_server | 读取 firewall.host（必需）+ tg.api_server（fallback）；不读密码值（NO_CREDENTIAL_READ） | ✓ 一致 |
| §3.5 load_env_file env_topology 契约 | port_mapping/nodes/links 三字段校验（DA-006）+ 端口名类型校验（Gotcha #11）；None 返回 None（Gotcha #12） | ✓ 一致 |
| §3.5 LCQ-ST-EX-04-01 --env-file 缺省复用 --topology-yaml | main: env_file_path = args.env_file or args.topology_yaml | ✓ 一致（方案 A） |
| §3.6 _resolve_tg_api_server ADR-05 环境文件优先 | env_topology.nodes.tg1.trex_api_url 优先，devices.yaml fallback | ✓ 一致 |
| §3.6 execute_op 消费契约（env_topology + tg_api_server 传参） | execute_steps 调 execute_op 传 env_topology + tg_api_server | ✓ 一致 |
| §3.7 下游 Story 调用点预留 | main 中 TODO(ST-EX-08/10/11/17) 标注；execute_steps 中 TODO(ST-EX-08/09/10) | ✓ 一致 |
| §4.1 七步执行流程 | main [1]启动校验 [2]设备准备 [3]用例发现 [4]预登录 [5]逐用例执行 [6]fw_logout [8]report | ✓ 一致 |
| §4.2 清理顺序编排 | main [5] run_cleanup -> [6] cleanup_session -> TODO(ST-EX-17) 逆序还原 | ✓ 一致 |
| §4.3 dry-run 默认门（ADR-02） | main 默认 dry_run=runtime_auth is None；execute_steps dry_run 门绕过 subprocess | ✓ 一致 |
| §5.1 异常分类（StartupError/DeviceConfigError/EnvFileError/CaseParseError） | 四异常类定义 + main 返回码 2/3/4 | ✓ 一致 |
| §5.3 import op_mapper 路径（Gotcha #2） | sys.path 注入 ../../policy-route-execution/scripts | ✓ 一致 |
| §10.3 result.json 结构 | run_id/mode/devices/env_file/cases/summary/logout/runtime_authorization | ✓ 一致 |

## 3. 测试 Fixture

| Fixture | 路径 | 用途 |
|---|---|---|
| devices.yaml | /tmp/cr033-test/configs/devices.yaml | firewall.host + tg.api_server |
| topology.yaml | /tmp/cr033-test/configs/topology.yaml | port_mapping/nodes/links（env_topology） |
| 测试用例 | /tmp/cr033-test/cases/PC-M4-01-01-01-测试用例.md | frontmatter 4 列 + 3 step（fw_login/fw_config_policy_route/tg_config_interface） |

## 4. 最小实现切片

ST-EX-04 实现编排骨架，下游 Story 填充被调函数：
- ST-EX-05 填充 execute_steps/authorize（Wave 2，已实现）
- ST-EX-06 填充 run_cleanup（Wave 2，已实现）
- ST-EX-07 填充 cleanup_session/check_fw_logout_available（Wave 2，已实现）
- ST-EX-08/09/10/11/17 预留 TODO 调用点（Wave 3）

## 5. 本地验证

| 验证项 | 命令 | 结果 |
|---|---|---|
| py_compile | `python -m py_compile case_runner.py` | OK |
| op_mapper validate | `case_runner.py validate` | PASS（22 op_id 全覆盖） |
| 单用例 dry-run | `case_runner.py run --case-file ... --env-file ...` | summary: pass=1 |
| 批量 dry-run | `case_runner.py run --cases-dir ... --env-file ...` | summary: pass=1 |
| dry-run 门不连设备 | result.json step status=dry_run，command 无 --execute | ✓ |
| ${ENV.dut.next_hop} 解析 | next_hop_ip resolved to 192.168.102.1 | ✓ |
| ${ENV.tg.ports[port1,port2]} 解析 | interfaces resolved to ["1/1/1", "1/1/2"] | ✓ |
| TREX_API_URL 来源 | tg_api_server_source=env_file（ADR-05 环境文件优先） | ✓ |
| 清理顺序 dry-run 短路 | cleanup=0 envelopes（authorized=False） | ✓ |
| logout dry-run 短路 | logout=skipped | ✓ |

## 6. 平台差异

- canonical 源（ptm-team）：skills/case-execution/scripts/ -> ../../policy-route-execution/scripts/
- workspace 安装（ptm-te）：.claude/skills/case-execution/scripts/ -> ../../policy-route-execution/scripts/
- 两场景下 sys.path 相对路径均有效（§9.2）
- install.py PTM_TE_SKILLS 新增 case-execution，get_agent_skills("ptm-te") 返回 5 skill

## 7. 交接摘要

- ST-EX-04 实现完成，main 编排骨架就绪，下游 Story 调用点预留
- Wave 2 的 ST-EX-05/06/07 已在同一 case_runner.py 中填充实现
- Wave 3 的 ST-EX-08/09/10/11/17 调用点以 TODO 标注，待后续填充
- dry-run 默认门 + resolve_env_refs + ${ENV.*} 解析 + 清理顺序 + result.json 全部验证通过
