# CR-173 CP2 人工门禁发起消息

CR-173 的 CP0、CP1 和 CP2 自动预检均为 `PASS`，blocker=`0`。本门禁确认的是离线 effective-trial 方法的用户可观察合同，不是具体算法或实现授权。

## 审批摘要

- 用户价值：避免把相关试验的 `raw_trial_count` 冒充独立有效试验数；在不接触真实数据的前提下，为未来 C1 multiple-testing/overfit 评估建立可信方法基础。
- 范围：strategy-agnostic、synthetic/fixture-only estimator、7 字段 typed evidence、6 类 golden vectors（每类 3/3 重复）、C1 offline consumer projection。
- 当前计数：`1 UC / 8 REQ / 8 SC / 8 matrix / 8 DQ`；六类=`6/6`；schema=`7/7`；Story/实现/真实操作=`0`。
- `approve` 效果：接受 8 项推荐，只进入 CP3 solution-design；具体算法、输入有效域、假设、method/version/hash 仍须 CP3 评审，必要时转方法 Spike。
- 不授权：真实 lake/NAS/provider/credential、策略身份、真实 producer/computation、runtime/write/trading、aggregate/admission、publish/deploy、Git remote write；也不自动恢复 CR-172。

## 待决策

| DQ | 推荐 |
|---|---|
| DQ-CR173-001 | effective count=依赖调整后的独立试验等价量；raw alias/default/fallback=0。 |
| DQ-CR173-002 | CP3 冻结输入 inventory/有效域/依赖表示/method/version/hash/假设；不能收敛则转 Spike。 |
| DQ-CR173-003 | 缺失/不足=`typed_unavailable`；矛盾/篡改=`blocked`；永不退 raw。 |
| DQ-CR173-004 | 七字段基线 schema=`7/7`；CP3 可增不可删，且需 ADR。 |
| DQ-CR173-005 | estimator strategy-agnostic；真实策略 identity required/inferred=`0`。 |
| DQ-CR173-006 | golden vectors=`6/6 × 每类 3/3`，合法组 canonical result=1，无效组 available=0。 |
| DQ-CR173-007 | C1 projection 只产生 offline typed evidence；real/Stage3/admission/aggregate claims=0。 |
| DQ-CR173-008 | CR-173 完成不自动恢复/关闭 CR-172；恢复仍需 five fields + fresh precheck + 重开 CR-172 CP2。 |

## 可回复

- `approve`：接受上述 8 项推荐，只推进到 CR-173 CP3 人工设计门禁。
- `修改: DQ-CR173-00N=<具体值>`：修改指定决策；若影响产品基线，将回到 meta-pm 增量整改。
- `reject`：CR-173 保持 CP2 pending；effective count 继续 typed_unavailable。

人工 checklist：`process/checkpoints/CP2-CR173-REQUIREMENTS-BASELINE.md`

Context capsule：`process/context/CP2-CR173.context.json`
