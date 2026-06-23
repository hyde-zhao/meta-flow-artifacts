# BGP4+ 测试因子候选对比表

> 来源: M分析(24) + Q分析(10) + HA新增(5) = 39 候选因子

## M 分析因子候选（24 个）

| factor_id | 候选名称 | data_domain | 来源 | 匹配结果 |
|-----------|---------|-------------|:--:|:---:|
| FAC-BGP-ENABLE | BGP使能状态 | enum(enable,disable) | M | 🆕 新增 |
| FAC-BGP-ASN | AS号 | 1~4294967295 | M | 🆕 新增 |
| FAC-BGP-ROUTERID | Router ID | 合法IPv4,≠0.0.0.0 | M | 🆕 新增 |
| FAC-BGP6-EBGP-DIST | EBGP优先级 | 1~255,默认20 | M | 🆕 新增 |
| FAC-BGP6-IBGP-DIST | IBGP优先级 | 1~255,默认200 | M | 🆕 新增 |
| FAC-BGP-NEIGHBOR-TYPE | 邻居类型 | enum(IBGP,EBGP) | M | 🆕 新增 |
| FAC-BGP6-NEIGHBOR-IP | 邻居IPv6地址 | IPv6单播,拒绝::1/ff00::/8/:: | M | 🆕 新增 |
| FAC-BGP6-INTERFACE | 发送接口 | 本机网卡名(必填,≤31字符) | M | 🆕 新增 |
| FAC-BGP6-MULTIHOP | EBGP多跳数 | 1~255,默认1 | M | 🆕 新增 |
| FAC-BGP-MD5-AUTH | MD5认证开关 | enum(enable,disable) | M | 🆕 新增 |
| FAC-BGP-MD5-PASSWORD | MD5密码 | ≤32字符,无空格,大小写敏感 | M | 🆕 新增 |
| FAC-BGP-NEIGHBOR-DESC | 邻居描述 | 可选字符串 | M | 🆕 新增 |
| FAC-BGP6-REDIST-PROTO | 引入协议类型 | enum(static,ospfv3,connected) | M | 🆕 新增 |
| FAC-BGP6-REDIST-MED | 引入路由MED值 | 0~4294967295 | M | 🆕 新增 |
| FAC-BGP6-NETWORK-CIDR | 宣告IPv6 CIDR | IPv6 CIDR,拒绝::/0等 | M | 🆕 新增 |
| FAC-BGP6-NETWORK-MASK | 宣告掩码类型 | enum(标准,非标准包含,非标准不包含) | M | 🆕 新增 |
| FAC-BGP-NEIGHBOR-STATE | 邻居状态 | 六态状态机 | M | 🆕 新增 |
| FAC-BGP-KEEPALIVE | Keepalive状态 | enum(正常,部分丢失,全部丢失) | M | 🆕 新增 |
| FAC-BGP-ROUTE-SELECT | 路由选路规则 | enum(MED最低,EBGP优先,本地优先) | M | 🆕 新增 |
| FAC-BGP6-MED-COMPARE | MED值比较方向 | 取最小值 | M | 🆕 新增 |
| FAC-BGP6-ECMP | 等价路由ECMP | enum(启用,未启用),最大16条 | M | 🆕 新增 |
| FAC-BGP6-LB-MODE | 负载均衡模式 | enum(带宽,权重) | M | 🆕 新增 |
| FAC-BGP6-LOG-TYPE | 日志类型 | enum(邻居变化,路由增删,认证) | M | 🆕 新增 |
| FAC-BGP6-LOG-PAGE | 日志分页参数 | page≥1,size:1~300000,默认10 | M | 🆕 新增 |

## Q 分析因子候选（10 个）

| factor_id | 候选名称 | data_domain | 来源 | 匹配结果 |
|-----------|---------|-------------|:--:|:---:|
| FAC-CAND-Q-001 | bgpd进程恢复时间 | ≤5s | Q | 🆕 新增 |
| FAC-CAND-Q-002 | BGP4+配置持久化 | 一致/不一致/丢失 | Q | 🆕 新增 |
| FAC-CAND-Q-003 | Hold Timer超时 | 180s(默认) | Q | 🆕 新增 |
| FAC-CAND-Q-004 | 路由收敛时间 | ≤5s | Q | 🆕 新增 |
| FAC-CAND-Q-005 | 邻居建立时间 | ≤5s | Q | 🆕 新增 |
| FAC-CAND-Q-006 | IPv6路由表容量 | 5K/50K/160K | Q | 🆕 新增 |
| FAC-CAND-Q-007 | 邻居规格上限 | 16 | Q | 🆕 新增 |
| FAC-CAND-Q-008 | ECMP最大路径数 | 16 | Q | 🆕 新增 |
| FAC-CAND-Q-009 | 异构设备互通性 | 华为/华三/锐捷 | Q | 🆕 新增 |
| FAC-CAND-Q-010 | 24h长稳内存增长 | <5%/5-20%/>20% | Q | 🆕 新增 |

## HA 新增因子（5 个）

| factor_id | 候选名称 | data_domain | 来源 | 匹配结果 |
|-----------|---------|-------------|:--:|:---:|
| FAC-BGP6-HA-VRRP-MODE | VRRP配置模式 | enum(有VRRP,无VRRP) | HA | 🆕 新增 |
| FAC-BGP6-HA-ROLE | HA角色 | enum(Master,Backup) | HA | 🆕 新增 |
| FAC-BGP6-HA-ROUTE-COST | 路由开销值(HA) | normal/max(4294967295) | HA | 🆕 新增 |
| FAC-BGP6-HA-COST-TRANS | 开销值转换方向 | normal→max/max→normal | HA | 🆕 新增 |
| FAC-BGP6-HA-ROUTE-TYPE | HA场景路由类型 | 网络/直连/静态/OSPFv3 | HA | 🆕 新增 |

> 总计 39 个候选因子，39 个新增。公共因子库(ngfw-ipv4-route/ngfw-interface/ngfw-policy-routing/ngfw-load-balance)中无BGP相关因子命中。建议提交因子库维护流程进行评审入库。
