---
status: confirmed
version: "1.0"
---

# ptm-te 执行引擎 - 面向用户价值的发布切片

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-pm | CR-033 初始发布切片（P0->P3 四期） |

## 发布切片

| Slice ID | 名称 | 用户价值 | 包含 Story | 前置依赖 | 验证入口 | 发布风险 |
|---|---|---|---|---|---|---|
| RS-EX-01 | TG 建模 + 规则固化（P0） | 测试平台开发者能建模 TG 设备并固化规则；重装不丢失 | ST-EX-01,02,03 | CP2/CP3/CP5 | SCN-EX-01,18,19 | devices.yaml 格式变更；fw_logout 未暴露 |
| RS-EX-02 | case_runner 核心 + dry-run（P0） | 测试执行工程师能用一个命令跑用例；dry-run 默认安全 | ST-EX-04,05,06,07 | RS-EX-01 | SCN-EX-02,03,04,07,08,16 | 硬编码未完全消除；session 管理异常 |
| RS-EX-03 | 引擎增强 + 四态分级（P1） | 测试执行工程师得到四态结果和失败诊断；ARP 预热自动清理 | ST-EX-08,09,10,11,12 | RS-EX-02 | SCN-EX-05,09,10,17 | known_issue 标注不完整；retry 逻辑边界 |
| RS-EX-04 | 用例整改 + 消费侧（P2-P3） | 24 用例 ARP 预热合规；verify_loss 消费侧提取 | ST-EX-13,14 | RS-EX-03 | SCN-EX-20 | 24 用例整改量大；verify_loss 字段不一致 |

## 切片顺序理由

| Slice ID | 为什么先 / 后做 | 不这样切的代价 |
|---|---|---|
| RS-EX-01 | 先建模 TG 设备和固化规则，为 case_runner 提供取址源和规则保障 | 没有 devices.yaml tg 块，case_runner 无法取 TG 地址；没有规则块，重装后回退 |
| RS-EX-02 | case_runner 核心能力（三入口 + dry-run + --execute + 清理 + 登出）是后续增强的基础 | 没有核心引擎，warming_up/retry/known_issue 无处挂载 |
| RS-EX-03 | 引擎增强（warming_up/post_op/retry/known_issue/四态/诊断/报告）依赖核心引擎 | 增强功能没有核心引擎无法运行 |
| RS-EX-04 | 24 用例整改和 verify_loss 消费侧是最后一公里 | 用例不整改则 ARP 预热不合规；verify_loss 不提取则 #9 未落地 |

## 发布门禁提示

- 首版（RS-EX-01/02）允许 static review + dry-run 验证；runtime 场景需 runtime_authorization
- RS-EX-03/04 的 runtime 场景在 CP7 验证时可能使用 static review 替代（无真实设备环境）
- 不得声称 runtime-ready 除非通过 VALIDATION-ENV.yaml 授权的真实运行验证
