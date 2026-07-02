# geo-data

Geo data releases for [Qingzhou](https://github.com/qingzhou-app/qingzhou), synced weekly from v2fly upstream.

轻舟 App 的 geo 数据发布仓库。App 内「下载完整版 geo 数据」的**主源**就是本仓库的
releases（备源为 v2fly 官方 releases 直链，主源失败自动切换）。

## 数据来源

| 文件 | 上游 | 说明 |
|---|---|---|
| `geoip.dat` | [v2fly/geoip](https://github.com/v2fly/geoip) | 完整 IP 地理库（全部国家/地区码） |
| `geoip-only-cn-private.dat` | [v2fly/geoip](https://github.com/v2fly/geoip) | 精简版（仅 CN + 内网段），App 内置的就是这份 |
| `geosite.dat` | [v2fly/domain-list-community](https://github.com/v2fly/domain-list-community) | 域名分类库（上游资产名为 `dlc.dat`，发布时改名） |

## 更新频率

GitHub Action（[`.github/workflows/sync.yml`](.github/workflows/sync.yml)）**每周日 02:00 UTC**
自动拉取上述两个上游仓库的最新 release，也可随时 `workflow_dispatch` 手动触发。
每次发布的 release **tag 为当天日期**（`YYYYMMDD`），release notes 里记录对应的上游 tag。

稳定下载地址（始终指向最新一期）：

```
https://github.com/qingzhou-app/geo-data/releases/latest/download/geoip.dat
https://github.com/qingzhou-app/geo-data/releases/latest/download/geoip.dat.sha256sum
https://github.com/qingzhou-app/geo-data/releases/latest/download/geosite.dat
https://github.com/qingzhou-app/geo-data/releases/latest/download/geosite.dat.sha256sum
https://github.com/qingzhou-app/geo-data/releases/latest/download/geoip-only-cn-private.dat
https://github.com/qingzhou-app/geo-data/releases/latest/download/geoip-only-cn-private.dat.sha256sum
```

## 校验方式

- 同步时：workflow 下载上游文件后，用**上游官方的 `.sha256sum` 资产**逐个 `sha256sum -c`
  校验，任何一个不匹配则整次同步失败、**不发布** release。
- 发布时：每个 `.dat` 都附带同名 `.sha256sum`（`geosite.dat` 的校验和在改名后重新生成）。
- 客户端：App 下载 `.dat` 后按对应 `.sha256sum` 做 SHA-256 校验，不匹配即丢弃、保留现有数据。

## 失败通知

同步 workflow 任一步骤失败时，会自动在本仓库开一个 **issue**（标题 `geo sync failed: <日期>`，
正文附当次 Action run 链接）。仓库 watcher 会收到通知；修复后手动 `workflow_dispatch` 重跑即可。

另外每次运行都会提交一个 `.heartbeat` 文件，保持仓库有活动，防止 GitHub 在 60 天无活动后
自动停用定时任务。
