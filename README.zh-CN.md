# RetroBoxDB NGP

[English](README.md) | 中文

SNK NeoGeo Pocket的单文件 SQLite 保存库。公开的 Catalog 只含元数据（校验值、DAT 与来源记录、头部字段、打包配方和程序），不含 ROM 数据，不能独立恢复文件；完整库保留在本地。

| 项目 | 数值 |
| --- | --- |
| 原始大小 | 源 ZIP 14 个，5.3 MiB（No-Intro 13 个，RetroAchievements 集合 1 个）；解压后 ROM 14 个，13.7 MiB |
| 入库后大小 | 完整库 6.2 MiB；公开 Catalog 1.7 MiB（不含 ROM 数据） |
| 比例 | 完整库为原 ZIP 的 117.0%，为解压后 ROM 总量的 45.0% |
| 使用的技术 | 存储 v4：128 KiB 块按 SHA256 去重，按 No-Intro 游戏族顺序装入最大 32 MiB 的 LZMA2 实体组（字典 32 MiB）；逐块 SHA256、逐对象 CRC32／MD5／SHA1／SHA256 校验；源 ZIP 由 TorrentZip 配方逐字节重建 |
| 导出性能 | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz，空闲负载，Python 3.14.4，含全部校验。按最新 DAT 整套导出（`export_set.py`，13 个文件，逐个按 DAT 哈希校验）：29.5 MiB/s，平均 30 毫秒／个；单个文件冷缓存（每次清空缓存，需解压所在组的前段）：ROM 平均 0.091 秒，TorrentZip 平均 0.186 秒 |

## 下载与说明

| 文件／文档 | 内容 |
| --- | --- |
| [RetroBoxDB.NGP.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-NGP/releases/latest/download/RetroBoxDB.NGP.Catalog.sqlite) | 公开 Catalog（Release 附件，附 `SHA256SUMS`） |
| [存储 v4 说明](RetroBoxDB.Storage-v4.zh-CN.md)／[English](RetroBoxDB.Storage-v4.en.md) | 各平台的存储评估、内容、RA、中文名与维护 |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | 存储格式、平台适配、增量更新、校验 |
| [RA 清单](reports/ra-ngp-games.csv)／[汇总](reports/ra-ngp.json)、[构建报告](reports/ngp-build-report.json)、[审计处理](reports/audit-resolution-20261004.md) | 逐项数据 |

## 本平台的存储选择与特殊情况

全部本地收藏实测 7 种块／组组合（`assessment/data/storage-experiment-ngp.json`）：最小为 512 KiB / 32 MiB 3.07 MiB；按规则（最小值 0.5% 以内选块最小、再选组最小）采用 128 KiB / 32 MiB 3.08 MiB。ZIP 5.26 MiB，逐文件 LZMA 3.72 MiB。

- 头部：开头 64 字节（`COPYRIGHT BY SNK CORPORATION` 或 ` LICENSED BY SNK CORPORATION`、启动地址、软件 ID、子版本、彩色模式、标题）存入 `ngp_hardware`；BIOS 没有卡带头，记为 `unclassified`。
- RetroAchievements 把 NeoGeo Pocket 和 NeoGeo Pocket Color 放在同一个主机（14）和同一个目录下。两个库都导入该目录：本库收 `.ngp` 文件，`.ngc` 文件跳过，由 [RetroBoxDB-NGPC](https://github.com/rshi0212/RetroBoxDB-NGPC) 收录。RA 报告只统计与本库有关的游戏。
- 平台很小（13 个文件、9.7 MiB ZIP），完整库比 ZIP 还大：每个库都内嵌约 3 MiB 的引擎、文档和报告。

## 内容

| 项目 | 数值 |
| --- | --- |
| ROM 记录／游戏组／发行版本 | 13／12／13 |
| 各版 DAT 覆盖 | 20250904-215533：13/13 |
| 不在任何 DAT 的本地 ROM | 0 |
| RetroAchievements 集合中的 ROM 文件 | DAT 中有 1，仅 RA 收录 0，哈希不在最新 RA 快照 0（[清单](reports/ra-ngp-collection-unknown.csv)）；仍缺本地 ROM 的 RA 游戏见 [缺口清单](reports/ra-ngp-missing.csv) |
| No-Intro DB Export＋Dump Log unknown | 13 个档案、22 个文件身份、11 条有文档的硬件声明；Dump Log Verified 8 |
| RetroAchievements（console 14） | 有成就的游戏 1 个：本地有 ROM 1（1 个 ROM），ROM 在兄弟库中 0，仅 DAT 有 0，仅 DB 文件 0，无 No-Intro 对应 0 |
| 中文名 | 10 条记录中 10 条有中文（9 个唯一名）；本地 ROM 10 个有中文名 |
| 完整库审计 | 16 个对象、1 个组、16 个 ZIP 配方，全部通过 |

源 ZIP 均可由 TorrentZip 配方逐字节重建（`v_file_checksums.exported_bytes_equal_source`）。

## 使用

```bash
# 用 Catalog 内嵌引擎做只读审计（stats、checksums FILE_ID、help 同理）
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.NGP.Catalog.sqlite audit
# 完整库：按 DAT 版本、1G1R、RA 成就、TorrentZip／裸 ROM 组合导出
python3 -B tools/export_set.py RetroBoxDB.NGP.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# 增量加入新 DAT、DB Export／Dump Log、ROM 与 RA 快照
python3 -B tools/update_db.py RetroBoxDB.NGP.sqlite --discover --ra --catalog RetroBoxDB.NGP.Catalog.sqlite
```

只需 Python 3.10+ 标准库。`resources` 中的 `engine.py` 等是可执行代码，只应从自己构建或 SHA256 已核对的 Release 附件中执行。发布由 `.github/workflows/publish-catalog.yml` 完成：工作流从 `release/catalog-release.json` 固定的基础 Catalog 出发，注入本仓库提交中的引擎与文档，核对全部数据表摘要、运行测试与审计后发布。
