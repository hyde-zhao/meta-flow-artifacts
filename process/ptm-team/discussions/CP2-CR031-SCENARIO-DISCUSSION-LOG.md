# CP2 CR-031 场景讨论记录

## 元数据

| 字段 | 值 |
|---|---|
| CR | CR-031 |
| 讨论主题 | ptm-tse 项目级安装事实与既有运行数据治理 |
| 状态 | scope reframed |
| 日期 | 2026-07-17 |
| 讨论方式 | 用户直接确认 |

## SGQ-031-01

| 字段 | 内容 |
|---|---|
| 面向用户的问题 | `ptm-tse` 的 Agent/Skill 安装位置与默认运行根应如何解析，才能让不同项目的数据隔离？ |
| 推荐方案 | Agent/Skill 默认安装到执行安装命令的项目目录；`--runtime-root` 默认该项目级安装根；用户可显式指定其他目标。 |
| 用户回答 | `--runtime-root默认是ptm-tse安装的项目根目录。安装的agnet/skill默认是项目级的安装，就是ptm-team命令运行项目目录，也可以显示由用户指定。` |
| 已确认理解 | 默认安装根为执行 `ptm-team` 安装命令的项目目录；显式目标覆盖默认值；默认 runtime root 等于安装根；真实运行数据仅在 `<runtime-root>/data/` 创建；不得回退到 `ptm-team/data`、全局用户目录或任意运行时 CWD。 |
| 影响面 | UC-RA-09、REQ-RA-023..026、SCN-RA-14/15、ST-RA-07、ST-NRA-04、后续 HLD 的 runtime-support 布局。 |
| 状态 | resolved-by-user |

## 延后项

| ID | 内容 | 状态 | 原因 / 后续 |
|---|---|---|---|
| DEF-CR031-01 | 全局用户级默认安装 / 运行根 | out-of-scope | 本 CR 只确认项目级默认；若需要全局模式，必须另建 CR 并重新审计数据隔离。 |

## SGQ-031-02

| 字段 | 内容 |
|---|---|
| 面向用户的问题 | CR-031 应重新实现项目级安装，还是治理已安装实例中的运行数据？ |
| 用户纠正 | `安装已经实现了。我已经在/home/hyde/projects/ptm-tse已进验证过了，现在需要治理的是ptm-tse的运行数据。` |
| 已确认理解 | `/home/hyde/projects/ptm-tse` 的项目级安装是事实基线；CR-031 不重做安装。当前范围是既有 `data/` 中 SQLite、WAL/SHM、原始/派生快照和元数据的分类、权限、support/data 分界与生命周期治理。 |
| 只读事实 | 数据目录和 snapshots 为 `0700`；数据库及部分非原始快照为 `0644`；`dao.py`、`schema.sql` 与运行数据混放且未出现在安装 manifest。未读取问题单内容，未修改权限或数据。 |
| 影响面 | UC-RA-09、REQ-RA-023..026、SCN-RA-14/15、ST-RA-07、ST-NRA-04，以及 replacement CP1/CP2。 |
| 状态 | resolved-by-user |

## 结论

SGQ-031-01 的安装根语义继续作为已验证事实保留，但原 CP2 的“安装改造”范围已撤回。SGQ-031-02 将 CR-031 收敛为运行数据治理。replacement CP2 需要用户确认的仅是：受管数据的范围和最小权限、support/data 分界与来源、以及默认不自动删除的生命周期策略；HLD 再决定具体实现与可选物理布局。
