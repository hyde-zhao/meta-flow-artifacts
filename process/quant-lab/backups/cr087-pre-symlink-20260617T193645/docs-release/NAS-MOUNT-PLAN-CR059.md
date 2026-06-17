# CR059 NAS Mount Plan

## Scope

本文件记录 quant-lab 三层 NAS 挂载准备。该准备已执行完成，但不包含数据迁移。

## Mounts

| Tier | NAS export | Local mount | Purpose | Status |
|---|---|---|---|---|
| hot | `192.168.101.83:/volume3/quant-lab-hot` | `/mnt/quant-lab-hot` | staging、cache、preflight | mounted, rw, enabled |
| primary | `192.168.101.83:/volume2/quant-lab-primary` | `/mnt/quant-lab-primary` | data lake、research、process evidence | mounted, rw, enabled |
| cold | `192.168.101.83:/volume1/quant-lab-cold` | `/mnt/quant-lab-cold` | backup、git bundle、restore drill、cold archive | mounted, rw, enabled |

## Systemd Units

仓库内源文件：

| Source | Installed unit |
|---|---|
| `ops/systemd/quant-lab-hot.mount` | `/etc/systemd/system/mnt-quant\\x2dlab\\x2dhot.mount` |
| `ops/systemd/quant-lab-primary.mount` | `/etc/systemd/system/mnt-quant\\x2dlab\\x2dprimary.mount` |
| `ops/systemd/quant-lab-cold.mount` | `/etc/systemd/system/mnt-quant\\x2dlab\\x2dcold.mount` |

启用状态：

```text
mnt-quant\x2dlab\x2dhot.mount     enabled
mnt-quant\x2dlab\x2dprimary.mount enabled
mnt-quant\x2dlab\x2dcold.mount    enabled
```

## Verified Facts

| Check | Result |
|---|---|
| NFS protocol | NFS v3 |
| mount options | `rw,hard,timeo=600,retrans=2,sec=sys` |
| capacity hot | 454G total, about 408G available at verification time |
| capacity primary | 3.6T total, about 3.5T available at verification time |
| capacity cold | 13T total, about 3.7T available at verification time |
| write marker | `.quant-lab-mount-check` and `.cr059-write-check` created successfully |

## Boundaries

- 本文件不授权 copy / move / delete。
- 本文件不授权 `MARKET_DATA_LAKE_ROOT` 切换。
- 本文件不授权 Git remote / push / tag / branch rename。
- 本文件不保存 sudo 密码或 NAS 凭据。
