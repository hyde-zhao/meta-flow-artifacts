---
story_id: "STORY-RA-05.1-INGEST"
canonical_story_id: "ST-RA-05.1-INGEST"
title: "ITR 问题单受控摄取与原始快照保存"
source_cr: "CR-030"
feature: "FEAT-RA-INGESTION"
lld_policy: "full-lld"
tier: "Standard"
design_evidence_type: "full-lld"
created_by: "meta-dev"
created_at: "2026-07-16"
open_items: []
shared_fragments:
  - "shared_skill: skills/itr-ticket-ingestion/SKILL.md (with ST-RA-05.2-CLEAN, ST-NRA-03, ST-RA-06.1-DETECT)"
---

# ST-RA-05.1-INGEST: ITR 问题单受控摄取与原始快照保存 — LLD

## 0. 工程依据与模板索引

| 来源 | 消费内容 |
|---|---|
| HLD / Feature DESIGN / Feature Matrix | 固定 GET、受限快照和 `full-lld` 约束 |
| Story `ST-RA-05.1-INGEST` | 输出文件、验收标准和依赖 |

本文保留原有详细章节；目标、需求、模块拆分、代码结构、数据模型、API、流程、技术细节、安全、测试、实施、风险与 DoD 分别由后续编号章节定义。

> 对应 HLD REV-03 的 `itr-ticket-ingestion` Skill HTTP 摄取段。
> 本 Story 为该 Skill 写入 §HTTP 摄取与快照保存部分。
> 共享 SKILL.md 文件：ST-RA-05.2-CLEAN 追加 §清洗与质量报告，ST-NRA-03 追加 §失败保护，
> ST-RA-06.1-DETECT 追加 §变更检测与版本历史。

## 1. 文件影响范围

| 文件 | 操作 | 所有权 | 说明 |
|---|---|---|---|
| `skills/itr-ticket-ingestion/SKILL.md` | 新建（共享写入） | F-020 独占 | Skill 定义：§触发场景、§HTTP 摄取与快照保存（本 Story 写入段）。后续 Story 追加其他段 |
| `skills/itr-ticket-ingestion/templates/batch-manifest.yaml` | 新建 | F-020 独占 | Batch 清单模板 |
| `skills/itr-ticket-ingestion/templates/allowlist-config.yaml` | 新建 | F-020 独占 | Allowlist URL pattern 配置模板（本文档定义，实现时创建） |

**不修改的文件**: `data/.gitignore`、`data/schema.sql`、`data/dao.py`（分别由 ST-RA-INGEST-DB 独占写入；本 Story 只调用其受限存储接口）。

## 2. 接口设计

### 2.1 Skill 触发接口

| 属性 | 值 |
|---|---|
| Skill 名称 | `itr-ticket-ingestion` |
| 触发场景 | 用户提供 ITR URL、产品、时间范围和分页参数，请求拉取现网问题单 |
| 触发示例 | `请从 ITR 拉取 product=TGFW 最近 30 天的问题单` |

### 2.2 ITR HTTP GET 请求接口

| 属性 | 约束 | 校验位置 |
|---|---|---|
| HTTP 方法 | **GET only** — POST/PUT/PATCH/DELETE 直接拒绝 | 请求发起前 |
| URL | 必须匹配 allowlist pattern | allowlist 校验 |
| 查询参数 | 只允许白名单参数：`product`、`time_range_start`、`time_range_end`、`page`、`page_size` | 参数白名单校验 |
| 认证头 | **禁止** — Authorization、X-Auth-Token、Cookie 等一律拒绝 | 请求发起前 |
| 超时 | 30s（可配置，`DEFAULT_TIMEOUT` 常量） | HTTP 客户端 |
| 重试 | 不自动重试 | — |

### 2.3 Allowlist 配置结构

```yaml
# skills/itr-ticket-ingestion/templates/allowlist-config.yaml
allowlist:
  - pattern: "http://10.113.53.108/itr/v1/itrs"
    description: "TGFW ITR 问题单接口"
    allowed_params:
      - product
      - time_range_start
      - time_range_end
      - page
      - page_size
    max_page_size: 100
```

### 2.4 Allowlist 校验算法

```
function validate_request(url, method, params, headers):
    1. IF method != "GET":
       → reject("仅允许 GET 方法")
    2. IF any key in headers matches (case-insensitive):
        'authorization', 'x-auth-token', 'cookie', 'x-api-key':
       → reject("禁止携带认证凭据")
    3. matched_pattern = None
       FOR each pattern in allowlist:
         IF url matches pattern.pattern:
           matched_pattern = pattern; BREAK
    4. IF matched_pattern IS None:
       → reject("URL 不在白名单中: {url}")
    5. FOR each key in params:
       IF key NOT IN matched_pattern.allowed_params:
         → reject("参数不在白名单中: {key}")
    6. IF params.page_size > matched_pattern.max_page_size:
       → reject("分页大小超过上限: {params.page_size}")
    7. → pass(url, params, matched_pattern)
```

### 2.5 原始快照保存接口

```python
def save_raw_snapshot(response_body: str, request_url: str, params: dict,
                      http_status: int, headers: dict) -> dict:
    """
    保存原始 HTTP 响应快照到受限数据区。
    返回: {
        "snapshot_ref": "data/snapshots/batch-{batch_id}-raw.json",
        "response_hash": "sha256:...",
        "saved_at": "2026-07-16T10:30:00Z",
        "file_size_bytes": 12345
    }
    """
```

**快照文件结构**:
```json
{
  "snapshot_metadata": {
    "batch_id": "batch-20260716-001",
    "request_url": "http://10.113.53.108/itr/v1/itrs",
    "request_params": {"product": "TGFW", "time_range_start": "2026-06-01", "page": 1, "page_size": 50},
    "request_time": "2026-07-16T10:30:00Z",
    "http_status": 200,
    "response_hash": "sha256:abc123...",
    "response_size_bytes": 45678
  },
  "raw_response": "<ITR 原始 JSON 响应体>"
}
```

**受限写入约束**：调用前必须通过 ST-RA-INGEST-DB 的存储初始化校验；快照先写入 `data/snapshots/*.tmp`，显式设置并验证 `0600` 后原子替换为目标 JSON。目录必须为 `0700`。任一校验失败时删除本批次临时/未提交快照，停止摄取且不创建 ingestion batch。

### 2.6 Batch Manifest 模板

```yaml
# skills/itr-ticket-ingestion/templates/batch-manifest.yaml
batch_id: "string"
request:
  url: "string"
  params: {}
  timestamp: "ISO8601"
http_status: 200
response_hash: "sha256:..."
snapshot:
  ref: "data/snapshots/batch-{batch_id}-raw.json"
  saved_at: "ISO8601"
schema_version: "1.0"
ingestion_result:
  total_fetched: 0
  total_cleaned: 0
  total_failed: 0
quality_report_ref: "skills/itr-ticket-ingestion/templates/quality-report.yaml"
errors: []
```

### 2.7 快照存储路径策略

| 属性 | 值 |
|---|---|
| 根目录 | `data/snapshots/` |
| 命名规则 | `batch-{batch_id}-raw.json` |
| Git 排除 | `data/snapshots/` 在 `data/.gitignore` 中排除 |
| 目录权限 | `0700` 硬断言；由 ST-RA-INGEST-DB 的受限存储初始化逻辑创建并验证 |
| 文件权限 | `0600` 硬断言；先写同目录临时文件，验证后原子替换 |
| 磁盘空间检查 | 写入前检查可用空间 ≥ 100MB |

## 3. 数据模型

### 3.1 请求与响应的规范表示

```python
@dataclass
class IngestRequest:
    url: str
    params: dict[str, str]
    method: str = "GET"
    timeout: int = 30

@dataclass
class IngestResponse:
    http_status: int
    body: str                # 原始响应 JSON 字符串
    headers: dict[str, str]
    elapsed_ms: int
    content_length: int

@dataclass
class SnapshotResult:
    snapshot_ref: str         # 快照文件路径
    response_hash: str        # sha256
    saved_at: str             # ISO8601
    file_size_bytes: int
```

### 3.2 错误模型

```python
class IngestionError(Exception):
    """摄取异常基类"""
    def __init__(self, reason: str, recoverable: bool, details: dict):
        ...

class AllowlistDeniedError(IngestionError):
    """URL 或参数不在白名单中"""
    reason: str  # e.g., "URL not in allowlist", "Param not allowed"

class CredentialDeniedError(IngestionError):
    """请求携带了禁止的认证凭据"""
    detected_header: str

class HTTPMethodDeniedError(IngestionError):
    """使用了非 GET 的 HTTP 方法"""
    attempted_method: str

class HTTPFetchError(IngestionError):
    """HTTP 请求失败（非 2xx、超时、连接错误）"""
    http_status: int | None

class SnapshotSaveError(IngestionError):
    """快照保存失败（磁盘满、权限不足）"""
    path: str
```

## 4. 核心流程

### 4.1 ITR HTTP 摄取完整流程

```
── 输入 ──
用户请求: {url, product, time_range, page, page_size}
          │
          ▼
┌─ 1. Allowlist 校验 ───────────────────────────────────┐
│  validate_request(url, "GET", params, headers)          │
│  ├─ 方法白名单: GET only                                │
│  ├─ URL 白名单: pattern match                           │
│  ├─ 参数白名单: product, time_range_*, page, page_size │
│  └─ 拒绝认证头: Authorization, Cookie, X-API-Key       │
│  失败 → AllowlistDeniedError / CredentialDeniedError   │
└────────────────────────────────────────────────────────┘
          │ 通过
          ▼
┌─ 2. HTTP GET 请求 ─────────────────────────────────────┐
│  send_request(url, params, timeout=30s)                 │
│  ├─ 超时: 30s                                           │
│  ├─ 状态码: 2xx → 成功 / 非 2xx → HTTPFetchError       │
│  └─ 记录请求时间和响应元数据                             │
│  失败 → HTTPFetchError                                  │
└────────────────────────────────────────────────────────┘
          │ 成功
          ▼
┌─ 3. 保存原始快照 ──────────────────────────────────────┐
│  save_raw_snapshot(body, url, params, status, headers)  │
│  ├─ 计算 sha256(response_body)                          │
│  ├─ 写入 data/snapshots/batch-{batch_id}-raw.json       │
│  ├─ 包含 metadata: url, params, timestamp, hash        │
│  └─ 磁盘空间预检                                        │
│  失败 → SnapshotSaveError                               │
└────────────────────────────────────────────────────────┘
          │ 成功
          ▼
┌─ 4. 生成 Batch Manifest ───────────────────────────────┐
│  输出 batch-manifest.yaml 结构                          │
│  ├─ batch_id, request 元数据                             │
│  ├─ http_status, response_hash                          │
│  ├─ snapshot_ref, saved_at                              │
│  └─ schema_version                                      │
└────────────────────────────────────────────────────────┘
          │
          ▼
── 输出 ──
{snapshot_ref, batch_id, response_hash, total_fetched, errors}
→ 传递给 ST-RA-05.2-CLEAN 的清洗流程
```

### 4.2 分页摄取流程

```
function fetch_all_pages(url, base_params):
    all_tickets = []
    page = 1
    WHILE True:
        params = base_params | {page: page, page_size: max_page_size}
        response = fetch_page(url, params)
        tickets = parse_tickets(response.body)
        IF tickets IS EMPTY:
            BREAK
        all_tickets.extend(tickets)
        save_page_snapshot(page, response)
        page += 1
        IF len(tickets) < max_page_size:
            BREAK  # 最后一页
    RETURN all_tickets
```

## 5. 状态机

本 Story 覆盖的是无状态请求-响应流程。`IngestionBatch` 的状态由 `quality_report_ref` 关联的 `IngestionQualityReport.overall_status` 描述（由 ST-RA-05.2-CLEAN 管理）：

```
HTTP 摄取完成 → (未完成清洗) → pending
清洗完成 → overall_status = clean|flagged|blocked
```

HTTP 摄取本身不维护独立状态机。

## 6. 错误处理与降级

| 错误场景 | 错误类型 | 处理 | 降级行为 | 对下游影响 |
|---|---|---|---|---|
| 非 GET 方法 | `HTTPMethodDeniedError` | 立即拒绝，记录 attempted_method | 不发起请求 | 批次不创建 |
| URL 不在 allowlist | `AllowlistDeniedError` | 立即拒绝，记录 url | 不发起请求 | 批次不创建 |
| 参数不在白名单 | `AllowlistDeniedError` | 立即拒绝，记录 key | 不发起请求 | 批次不创建 |
| 含认证头 | `CredentialDeniedError` | 立即拒绝，记录 detected_header | 不发起请求 | 批次不创建 |
| 连接超时（30s） | `HTTPFetchError` | 记录错误，返回 recoverable=True | 不创建 batch，不写快照 | 批次失败，历史数据安全 |
| HTTP 4xx/5xx | `HTTPFetchError` | 记录 http_status，返回 recoverable=False | 不创建 batch，不写快照 | 批次失败，历史数据安全 |
| 响应体为空 | `HTTPFetchError` | 记录，返回 recoverable=False | 不保存快照 | 批次失败 |
| 快照保存失败（磁盘满） | `SnapshotSaveError` | 记录路径和可用空间 | 不保存快照 → 不创建 batch | 批次失败 |
| 分页中断（某页失败） | `HTTPFetchError` | 记录已成功页和失败页 | 已成功页的快照保留 → batch 创建 | 批次标记 partial，total_fetched = 成功页之和 |

**降级原则**：
- 不自动重试 HTTP 请求（外部系统，重试策略需用户明确）
- 失败不覆盖任何历史快照或 batch 记录
- 分页成功部分保留（已保存的快照不变，batch 标记 partial），失败页可单独重试
- 每种错误输出人类可读的 `error_reason` + `recoverable` 标记 + `details`

## 7. 测试设计

### 7.1 Fixture 计划

| Fixture | 内容 | 覆盖场景 |
|---|---|---|
| `fixtures/itr_valid_response.json` | 有效 ITR JSON（含 5 条 ticket） | T-ING-01 |
| `fixtures/itr_unauthorized_response.json` | HTTP 401 响应（模拟） | T-ING-NEG |
| `fixtures/allowlist_config.yaml` | 含 1 条 allowlist entry | T-ING-02 |
| `fixtures/mock_http_client.py` | Mock HTTP 客户端（返回预定义响应） | 全部 HTTP 测试 |

### 7.2 测试用例与覆盖矩阵

| ID | 场景 | 类型 | 输入 | 预期结果 | 对应 AC |
|---|---|---|---|---|---|
| T-ING-01 | Allowlist URL GET 成功 | positive | 白名单 URL + 有效参数 | 200 → 快照保存 → 返回 batch 摘要 | AC-2, AC-4 |
| T-ING-02 | 非 allowlist URL 拒绝 | security | URL 不在白名单 | AllowlistDeniedError，记录拒绝原因 | AC-2 |
| T-ING-03 | POST 方法拒绝 | security | POST 请求 | HTTPMethodDeniedError | AC-3 |
| T-ING-04 | PUT 方法拒绝 | security | PUT 请求 | HTTPMethodDeniedError | AC-3 |
| T-ING-05 | 含 Authorization header 拒绝 | security | headers={'Authorization': 'Bearer x'} | CredentialDeniedError | AC-6 |
| T-ING-06 | 含 X-Auth-Token header 拒绝 | security | headers={'X-Auth-Token': 'x'} | CredentialDeniedError | AC-6 |
| T-ING-07 | 含 Cookie header 拒绝 | security | headers={'Cookie': 'session=x'} | CredentialDeniedError | AC-6 |
| T-ING-08 | 非白名单参数拒绝 | security | params={'extra': 'value'} | AllowlistDeniedError | AC-2 |
| T-ING-09 | 分页大小超限拒绝 | boundary | params={'page_size': 200}（超过 max） | AllowlistDeniedError | AC-2 |
| T-ING-10 | HTTP 500 响应处理 | negative | mock 返回 500 | HTTPFetchError，不创建 batch | AC-1（隐式） |
| T-ING-11 | 响应超时处理 | negative | mock 延迟 35s | HTTPFetchError(recoverable=True) | AC-1 |
| T-ING-12 | 快照保存后路径与权限检查 | boundary | 保存后检查文件路径和 mode | 在 `data/snapshots/` 下、不在 Git 跟踪、目录 `0700`、文件 `0600` | AC-4, AC-5 |
| T-ING-15 | 快照权限断言失败 | security | mock chmod/stat 失败 | 删除本批次临时/未提交快照、停止摄取、不创建 batch | AC-4 |
| T-ING-13 | 快照文件包含完整元数据 | positive | 保存后读取快照 JSON | metadata 含 url、params、hash、timestamp | AC-5 |
| T-ING-14 | Batch manifest 模板结构校验 | positive | 生成 manifest YAML | 含 request_url, params, timestamp, hash, snapshot_ref | AC-5 |

### 7.3 测试执行方式

- **Allowlist 测试**: 纯函数测试，无需 mock
- **HTTP 测试**: 使用 mock HTTP client 返回预定义响应（不发起真实网络请求）
- **快照测试**: 使用 `tmp_path` 写入临时目录，验证文件内容和元数据

## 8. 安全与权限

| 安全规则 | HLD 来源 | 实现方式 | 验证方式 | 对应 AC |
|---|---|---|---|---|
| deny-by-default | HLD §可信分析治理约束 | 拒绝任何不在 allowlist 的 URL、方法、参数、header | T-ING-02..09 | AC-2, AC-3 |
| 无凭据读取 | REQ-RA-011 | 检测并拒绝认证相关 header（Authorization, X-Auth-Token, Cookie, X-API-Key） | T-ING-05..07 | AC-6 |
| 不推断认证头 | REQ-RA-011 | 不自动添加任何认证信息 | 代码审查 | AC-6 |
| 无 ITR 写入 | HLD §推荐边界与数据模型 | 只发 GET 请求，方法白名单拒绝 POST/PUT/DELETE/PATCH | T-ING-03, T-ING-04 | AC-3 |
| 原始快照不进 Git | HLD §敏感字段策略 | `data/snapshots/` 在 `data/.gitignore` 排除 | T-ING-12 | AC-4 |
| 参数白名单 | FEAT-RA-INGESTION DESIGN.md §3.1 | 只允许 product, time_range_start, time_range_end, page, page_size | T-ING-08, T-ING-09 | AC-2 |

**不做的事**：
- 不发起 HTTPS 请求（首版只支持 HTTP，由 allowlist 限定为内网地址）
- 不处理重定向（HTTP 3xx 视为错误，返回 HTTPFetchError）
- 不读取系统凭据存储（Keychain、.netrc、环境变量等）
- 不自动生成 URL（URL 必须由用户显式提供并通过 allowlist 校验）

## 9. 实施步骤

### 切片 1：Skill 骨架 + Allowlist 校验（TASK-ING-01, TASK-ING-02）
1. 创建 `skills/itr-ticket-ingestion/` 目录
2. 写入 SKILL.md frontmatter（name, description, trigger, tools）
3. 写入 allowlist 校验逻辑（`validate_request` 函数）
4. 写入 allowlist-config 模板
5. 测试：T-ING-02..09（安全拒绝场景优先）

### 切片 2：HTTP GET 请求 + 响应处理（TASK-ING-03）
1. 实现 `fetch_itr_tickets()`：超时控制、状态码检查、响应体读取
2. 实现 `fetch_all_pages()`：分页遍历
3. 测试：T-ING-01, T-ING-10, T-ING-11

### 切片 3：原始快照保存（TASK-ING-04）
1. 实现 `save_raw_snapshot()`：受限目录预检 + 临时文件写入 + `0600` 校验 + 原子替换 + sha256 + 元数据
2. 实现磁盘空间预检（`shutil.disk_usage` 检查 >= 100MB）
3. 测试：T-ING-12, T-ING-13

### 切片 4：Batch Manifest 生成（TASK-ING-05, TASK-ING-06）
1. 实现 `generate_batch_manifest()`：填充 batch-manifest 模板
2. 调用 ST-RA-INGEST-DB 提供的受限存储初始化/校验接口；不得修改 `data/.gitignore`
3. 测试：T-ING-14, T-ING-15

### 验证汇总
- 所有 15 个测试用例通过
- allowlist 配置可正常解析
- SKILL.md frontmatter 符合 Skill 契约规范

## 10. 回滚策略

| 变更类型 | 回滚方式 |
|---|---|
| Allowlist 错误（URL 过于宽松） | 收紧 allowlist pattern，重载配置 |
| 快照路径或权限错误 | 修正受限存储初始化/校验逻辑；不由本 Story 修改 `.gitignore` |
| HTTP 超时/重试策略调整 | 修改 `DEFAULT_TIMEOUT` 常量或增加重试逻辑 |
| Skill 回退 | 删除或归档 `skills/itr-ticket-ingestion/SKILL.md`；已有快照和 batch 记录保留不动 |

## 11. 平台差异检查

| 检查项 | Codex | Claude Code | Qoder | 说明 |
|---|---|---|---|---|
| HTTP 请求库 | 取决于 Skill 所在运行时 | 同左 | 同左 | Skill 使用所在 Agent 平台的 HTTP 能力（`requests` / `httpx` / 内置）；LLD 不绑定特定库 |
| 文件系统写入 | `open()` + `json.dump()` | 同左 | 同左 | Python 标准库，无平台差异 |
| `shutil.disk_usage` | 支持（Python 3.3+） | 同左 | 同左 | 无差异 |
| YAML 模板 | `yaml` 库或 Jinja2 | 同左 | 同左 | 模板为纯 YAML 文件，不依赖渲染引擎 |
| 进程/线程安全 | GIL 保护 | GIL 保护 | GIL 保护 | 单线程执行，无并发问题 |

**结论**: 无平台差异处理项。

## 12. 与相邻模块的集成契约

### 12.1 对 ST-RA-05.2-CLEAN 的交付契约

| 数据 | 格式 | 传递方式 |
|---|---|---|
| 原始响应 JSON | `list[dict]` — ITR 返回的 ticket 数组 | 函数调用链（Skill 内部） |
| 快照引用 | `snapshot_ref: str` | batch-manifest 中 |
| 请求元数据 | `{url, params, http_status, response_hash, timestamp}` | batch-manifest 中 |
| batch_id | `str` — `batch-{YYYYMMDD}-{seq}` | 函数调用链 |

### 12.2 对 ST-RA-INGEST-DB 的交付契约

| 数据 | 使用方 | 说明 |
|---|---|---|
| `snapshot_ref` | `ingestion_batch` 表 | 用于追溯 |
| `response_hash` | `ingestion_batch` 表 | 用于完整性校验 |
| `batch_id` | `ingestion_batch` 表 | FK 关联 |

### 12.3 对 ST-NRA-03 的异常契约

| 异常出口 | 含义 | ST-NRA-03 消费方式 |
|---|---|---|
| `HTTPFetchError` | 网络/HTTP 层面失败 | §HTTP 失败保护 |
| `AllowlistDeniedError` | 安全拒绝 | 记录拒绝日志 |
| `CredentialDeniedError` | 凭据拒绝 | 记录拒绝日志 |
| `SnapshotSaveError` | 存储失败 | §快照保存失败保护 |

## 13. 开放项与假设

### 开放项

| ID | 描述 | 状态 | 影响 |
|---|---|---|---|
| O-ING-01 | ITR 响应 JSON 的真实 schema 结构 | OPEN（需首次受控探测） | 字段映射的 accuracy |
| O-ING-02 | 分页机制：ITR 是否使用 page/page_size 还是有 cursor-based 分页 | OPEN（需首次受控探测） | 分页逻辑 |

### 假设

| ID | 假设 | 依据 | 风险 |
|---|---|---|---|
| A-ING-01 | ITR 使用标准 HTTP GET + query params，无特殊认证 | HLD §推荐边界：无凭据推断 | 低（allowlist + parameter whitelist 防御） |
| A-ING-02 | ITR 响应为 JSON array，每项含 `id` 字段作为 source_ticket_id | 典型工单 API 模式 | 中（需首次探测确认） |
| A-ING-03 | 分页使用 offset/limit 模式 | 常见 REST 分页 | 中 |
| A-ING-04 | 单次摄取批次不超过 1000 条 ticket | MVP 规模假设 | 低（可通过分页处理） |

## 14. LLD 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | meta-dev | 初始 LLD：HTTP 摄取 + allowlist + 快照 + 14 章节 |
| 1.1 | 2026-07-16 | host-orchestrator | CP5 Round 4：删除对 `data/.gitignore` / `.gitkeep` 的跨 Story 写入；快照目录与文件统一为 `0700/0600` 硬断言，补充临时写入、原子替换和权限失败清理测试。 |
| 1.2 | 2026-07-16 | host-orchestrator | CP5 B12：迁移为当前 full-lld 证据兼容格式，增加 canonical Story ID 与 §0 工程依据/章节索引；不改变原设计契约。 |
