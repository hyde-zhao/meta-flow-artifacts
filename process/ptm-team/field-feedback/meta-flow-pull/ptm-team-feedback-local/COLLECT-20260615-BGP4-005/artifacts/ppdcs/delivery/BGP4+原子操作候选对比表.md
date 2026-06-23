# BGP4+ 原子操作候选对比表

> 来源: mfq/m-analysis/candidate-ptm-atomic.yaml

## BGP6 API 候选（15 个）

| 候选 op_id | 操作描述 | 现有库匹配 | 匹配结果 | 来源场景 |
|-----------|---------|-----------|:---:|----------|
| fw_config_bgp_global | PUT /api/v1/bgpconfig 全局配置 | — | 🆕 新增 | S01 |
| fw_verify_bgp_global | GET /api/v1/bgpconfig 查询配置 | — | 🆕 新增 | S01 |
| fw_config_bgp6_distance | PUT /api/v1/bgp6distance 优先级配置 | — | 🆕 新增 | S01 |
| fw_verify_bgp6_distance | GET /api/v1/bgp6distance 查询优先级 | — | 🆕 新增 | S01 |
| fw_config_bgp6_redist | PUT /api/v1/bgp6redist 路由引入配置 | — | 🆕 新增 | S10-S12 |
| fw_verify_bgp6_redist | GET /api/v1/bgp6redist 查询引入配置 | — | 🆕 新增 | S10-S12 |
| fw_create_bgp6_network | POST /api/v1/bgp6network 创建网络宣告 | — | 🆕 新增 | S03,S14 |
| fw_update_bgp6_network | PUT /api/v1/bgp6network 修改网络宣告 | — | 🆕 新增 | S03 |
| fw_delete_bgp6_network | DELETE /api/v1/bgp6network 删除网络宣告 | — | 🆕 新增 | S03 |
| fw_verify_bgp6_network | GET /api/v1/bgp6network 查询网络宣告 | — | 🆕 新增 | S03,S14 |
| fw_create_bgp6_neighbor | POST /api/v1/bgp6neighbor 创建邻居 | — | 🆕 新增 | S04-S09 |
| fw_update_bgp6_neighbor | PUT /api/v1/bgp6neighbor 修改邻居 | — | 🆕 新增 | S02,S07 |
| fw_delete_bgp6_neighbor | DELETE /api/v1/bgp6neighbor 删除邻居 | — | 🆕 新增 | S02 |
| fw_verify_bgp6_neighbor | GET /api/v1/bgp6neighbor 查询邻居状态 | — | 🆕 新增 | S04-S09 |
| fw_capture_bgp6_log | GET /api/v1/bgp6log 查询日志 | — | 🆕 新增 | S21 |

## 非 BGP6 候选（16 个）

| 候选 op_id | 操作描述 | 类别 | 匹配结果 |
|-----------|---------|:--:|:---:|
| fw_verify_etcd_bgp_conf | CLI校验etcd配置一致性 | CLI校验 | 🆕 |
| fw_verify_frr_bgp6_config | CLI校验FRR配置加载 | CLI校验 | 🆕 |
| fw_verify_bgp6_route_table | CLI查询BGP4+路由表 | CLI校验 | 🆕 |
| fw_verify_bgp6_route_med | CLI验证路由MED属性 | CLI校验 | 🆕 |
| fw_verify_bgp6_route_next_hop | CLI验证路由下一跳 | CLI校验 | 🆕 |
| fw_verify_bgp6_no_blackhole | CLI验证黑洞路由过滤 | CLI校验 | 🆕 |
| fw_verify_bgp6_neighbor_count | CLI验证邻居数量 | CLI校验 | 🆕 |
| fw_verify_bgp6_route_count | CLI验证路由条目数 | CLI校验 | 🆕 |
| fw_verify_bgp6_ecmp_paths | CLI验证等价路径数 | CLI校验 | 🆕 |
| fw_verify_ha_sync_bgp6 | CLI验证HA同步状态 | CLI校验 | 🆕 |
| fw_ha_active_switchover | HA主动主备切换 | HA | 🆕 |
| fw_ha_passive_switchover | HA被动主备切换 | HA | 🆕 |
| fw_ha_verify_role | HA角色状态查询 | HA | 🆕 |
| tg_inject_bgp6_routes | TG注入IPv6 BGP路由 | TG路由 | 🆕 |
| tg_withdraw_bgp6_routes | TG撤销IPv6 BGP路由 | TG路由 | 🆕 |
| fw_monitor_bgp6_stability | BGP4+长稳监控 | 监控 | 🆕 |

> 总计 31 个候选，31 个新增。建议提交 ptm-tae 进行工具开发。
