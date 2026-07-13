# CP2 自动预检 — CR168 需求 / 场景 / 范围基线

## Entry Criteria

- CP0=`PASS`，CP1=`PASS`。
- 需求、场景、矩阵、产品规划、discussion log/checkpoint 与 CP2 capsule 均存在。
- 用户可见 SGQ confirmed=`1`。

## Checklist

- 9 项 requirements、4 项量化 NFR、15 项 QAC：`PASS`。
- 17 个 scenarios（P0=16、P1=1）与 matrix 17/17：`PASS`；10/10 C3 输入 fail-closed 类别不变，新增 1 个 Gate 4 projection reason-escape guard 场景。
- In Scope / Out of Scope / Deferred / claim ceiling：`PASS`。
- Gate 4 C3+C4 联合事实、C4 absent-no-na-reason 投影、reason-escape rejection、C3/C4/FU-007/CR155 边界：`PASS`。
- `DQ-CP2-CR168-GATE4` 修改已应用，5 项开放问题重新进入 Decision Brief：`PASS`。
- Story/LLD/HLD/implementation/verification 提前产物：`0`。

## Exit Criteria

- 自动预检阻塞项=`0`。
- 人工 CP2 尚未批准；`ready_for_design=false`。

## Deliverables

- 机器真相源：`process/checks/CP2-CR168-REQUIREMENTS-BASELINE.result.json`
- 人工审查稿：`process/checkpoints/CP2-CR168-REQUIREMENTS-BASELINE.md`
- Context Capsule：`process/context/CP2-CR168-REQUIREMENT-CONTEXT.yaml`

## 结论

修订后自动预检 `PASS`，重新打开 CP2 人工门禁；该结论不等于人工批准。
