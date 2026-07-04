# CR067 Clean Publication Scan Gate

## 1. Scope

本 scan gate 只适用于 CR067 post-approval temporary clean snapshot。当前阶段不执行扫描、不读取 `.env`、不访问 NAS / data lake / provider / runtime。

## 2. Required Scan Classes

| Scan ID | 类别 | 范围 | Blocking 条件 | 处理 |
|---|---|---|---|---|
| SCAN-CR067-01 | no-secret scan | snapshot 全量文本文件 | token/password/cookie/session/private key / API key 疑似真实值 | STOP；不执行 Git 写动作。 |
| SCAN-CR067-02 | `.env.example` placeholder scan | `.env.example` only | 非占位符真实值或隐式凭据 | 排除 `.env.example` 或重开 DQ。 |
| SCAN-CR067-03 | data/report/run path scan | snapshot 路径列表 | `data/**`、`reports/**`、`runs/**`、`.env`、`.env.local`、`.envrc` | STOP；修正 manifest。 |
| SCAN-CR067-04 | large-file scan | snapshot 文件元数据 | 单文件超过 CP5 批准阈值或明显二进制数据体 | STOP 或 DQ 风险接受。 |
| SCAN-CR067-05 | internal topology scan | docs/process/release evidence | 内网 IP、NAS export、真实 lake root、系统用户名、机器路径 | STOP；redaction 或 exclude。 |
| SCAN-CR067-06 | cache artifact scan | snapshot 路径列表 | `__pycache__/**`、`*.pyc`、build cache | STOP；清理 snapshot。 |

## 3. Severity

| 级别 | 定义 | Gate 行为 |
|---|---|---|
| BLOCKING | 凭据、数据体、forbidden path、remote mismatch | fail-closed |
| HIGH | 内网拓扑、真实路径、大文件、疑似敏感配置 | fail-closed，除非新 DQ |
| MEDIUM | 可公开性不确定的过程证据 | 默认排除或人工复核 |
| LOW | 文档措辞 / 体积优化 | 可记录为 residual risk |

## 4. Evidence Requirements

每个 scan 必须在 post-approval execution evidence 中记录：

- 命令或等价检查方式
- 扫描范围
- finding count
- finding severity
- allow / exclude / redact 处理结果
- stop condition 是否触发

## 5. Non-Authorization

CP5 approval 前不得执行本文件中的任何扫描命令。即使 CP5 approval 后，也不得读取 `.env`、`.env.local`、`.envrc` 或真实凭据；唯一可审查的 env 类文件是经 DQ 接受的 `.env.example`。
