# Requirement Clarification Log

## CR-046 调研发现（2026-07-11）

### 现有可复用资源

- 复用现有 state、checkpoint/result、gate/dispatch/run ledger、context/read policy 与 CR lifecycle；不建立第二套 evidence governance。
- 产品文档单一真相源位于 `/home/hyde/workspace/meta-flow-artifacts/process/meta-flow/docs/product`；源码 checkout 的 `docs/product` 缺失不是新建第二份 baseline 的理由。

### 平台能力约束

- platform receipt 和 token telemetry 可能不提供；缺失事实必须为 `unavailable`，不得合成。
- CP2 不授权 credentials、runtime、production write、publish、交易、commit/push 或 quant-lab business-code 修改。

### 场景发现摘要

- 增量新增 `UC-EI-001..005`、`REQ-EI-*`、`TC-EI-001..013` 和 `ST-EI-001..007`，保留全部 CR-037 ID 与修订历史。
- `SGA-05..08` 已由 originating CR request 收敛；BLOCKING 澄清项为 0。
- `DEF-EI-001..002` 已进入 backlog，不扩大本轮 MVP。
- `ready_for_design=true` 仅表示自动产品基线可交给 Host Orchestrator 准备 CP2；不表示 CP2 已发起或批准。

## CR-050 场景发现摘要（2026-07-15）

### 已确认真实意图

- 用户明确要求完整旅程：刷新远端主分支、创建 CR branch、提交后推送、远端合并后删除 branch。
- 当前主体是 Meta Flow 自身的 Host Orchestrator / workspace workflow；交付类型为 `workflow`，治理方式为 `review-gated / strict`。
- 当前源码仓与 artifact 仓共同构成可恢复工作区，因此“单仓 branch”与“两仓 branch”是需要 CP2 明确的产品边界。

### 关键认知盲区

- `gb` 没有唯一可移植含义；常见 shell 中只是 `git branch` 别名，也存在同名 Go 构建工具。完整分支工作流的成熟候选是 Git Town，但它引入额外安装、配置、同步/ship 策略。
- 仅 Git DAG 无法证明 squash/rebase merge 已包含原 branch tip；MVP 若没有 forge receipt 必须 fail closed。
- 自动 stage/commit 会扩大提交选择与 secret 泄漏面；推荐只推送已显式提交 refs。
- 双仓没有跨仓原子 transaction；必须逐仓记录 terminal state，partial 不得冒充成功。

### 产品基线增量

- 新增 `UC-GB-001..003`、`REQ-GB-*`、`TC-GB-001..011`、`ST-GB-001..003`、`SL-GB-01..03`。
- `SGQ-GB-001` 以用户原始自由表达确认核心旅程；`SGA-GB-01..05` 进入 CP2 Decision Brief。
- forge receipt adapter、Git Town/stacked branch adapter 和自动 commit planner 进入 Deferred/Backlog，不扩大 MVP。
- BLOCKING 信息缺口为 0；五项范围/风险选择等待 CP2 人工确认，`ready_for_design=false` 直到 CP2 approved。
