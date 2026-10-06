# RetroBoxDB NGP

English | [中文说明](README.zh-CN.md)

Single-file SQLite preservation database for SNK NeoGeo Pocket. The public Catalog holds metadata only (checksums, DAT and provenance records, header fields, archive recipes and the processing code); it contains no ROM data and cannot restore files. The populated database stays local.

| Item | Value |
| --- | --- |
| Original size | 14 source ZIPs, 5.3 MiB (No-Intro 13, RetroAchievements sets 1); 14 ROM files, 13.7 MiB uncompressed |
| Stored size | populated database 6.0 MiB; public Catalog 1.6 MiB (no ROM data) |
| Ratio | 114.6% of the source ZIPs, 44.1% of the uncompressed ROM files |
| Technology | storage v4: SHA256-deduplicated 128 KiB blocks packed in No-Intro family order into solid LZMA2 groups of up to 32 MiB (32 MiB dictionary); per-block SHA256 and per-object CRC32/MD5/SHA1/SHA256 verification; source ZIPs reproduced byte-for-byte from TorrentZip plans |
| Export performance | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, Python 3.14.4, all checks included. whole newest-DAT set with `export_set.py` (13 files, each checked against the DAT hashes): 29.5 MiB/s, 30 ms per file on average; single file with a cold cache (the group is decoded up to the file): ROM 0.091 s, TorrentZip 0.186 s on average |

## Downloads and documents

| File / document | Content |
| --- | --- |
| [RetroBoxDB.NGP.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-NGP/releases/latest/download/RetroBoxDB.NGP.Catalog.sqlite) | Public Catalog (Release asset with `SHA256SUMS`) |
| [Storage v4 guide](RetroBoxDB.Storage-v4.en.md) / [中文](RetroBoxDB.Storage-v4.zh-CN.md) | Storage evaluation, contents, RA, names and maintenance for every platform |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | Storage format, platform adapters, incremental updates, verification |
| [RA list](reports/ra-ngp-games.csv) / [summary](reports/ra-ngp.json), [build report](reports/ngp-build-report.json), [audit resolution](reports/audit-resolution-20261004.md) | Detailed data |

## Storage choice and platform specifics

7 block/group combinations measured on the whole local collection (`assessment/data/storage-experiment-ngp.json`): smallest 512 KiB / 32 MiB at 3.07 MiB; by the rule (within 0.5% of the smallest, the smallest block, then the smallest group) 128 KiB / 32 MiB at 3.08 MiB. ZIPs 5.26 MiB, per-file LZMA 3.72 MiB.

- Header: 64 bytes at 0 (`COPYRIGHT BY SNK CORPORATION` or ` LICENSED BY SNK CORPORATION`, start address, software id, sub code, colour mode, title) stored in `ngp_hardware`; BIOS images have no header and stay `unclassified`.
- RetroAchievements lists NeoGeo Pocket and NeoGeo Pocket Color under one console (14) and one folder. Both databases import that folder; this one keeps `.ngp` files and skips `.ngc` files, which [RetroBoxDB-NGPC](https://github.com/rshi0212/RetroBoxDB-NGPC) holds. The RA report covers only games tied to this database.
- The platform is small (13 files, 9.7 MiB of ZIPs), so the populated database is larger than its ZIPs: the embedded engine, documents and reports take about 3 MiB in every database.

## Contents

| Item | Value |
| --- | --- |
| ROM records / games / releases | 13 / 12 / 13 |
| DAT coverage per version | 20250904-215533: 13/13 |
| Local ROMs in no DAT | 0 |
| ROM files of the RetroAchievements set | in a No-Intro DAT 1, RA only 0, hash not in the latest RA snapshot 0 ([list](reports/ra-ngp-collection-unknown.csv)); RA games still without a local ROM: [gap list](reports/ra-ngp-missing.csv) |
| No-Intro DB Export + Dump Log unknown | 13 archives, 22 file identities, 11 documented hardware assertions; Dump Log Verified 8 |
| RetroAchievements (console 14) | 1 games with achievements: 1 with a local ROM (1 ROMs), 0 with the ROM in a sibling database, 0 DAT only, 0 DB file only, 0 without a No-Intro counterpart |
| Chinese names | 10 of 10 rows translated (9 unique); 10 local ROMs have a Chinese name |
| Populated-database audit | 16 objects, 1 groups, 16 archive plans, all passed |

Every source ZIP is reproduced byte-for-byte from its TorrentZip plan (`v_file_checksums.exported_bytes_equal_source`).

## Usage

```bash
# Query-only audit with the Catalog's embedded engine (also: stats, checksums FILE_ID, help)
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.NGP.Catalog.sqlite audit
# Populated database: export by DAT version, 1G1R, RA achievements, TorrentZip or plain ROMs
python3 -B tools/export_set.py RetroBoxDB.NGP.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# Add new DATs, DB Export / Dump Log snapshots, ROMs and an RA snapshot incrementally
python3 -B tools/update_db.py RetroBoxDB.NGP.sqlite --discover --ra --catalog RetroBoxDB.NGP.Catalog.sqlite
```

Python 3.10+ standard library only. `engine.py` and the other `resources` entries are executable code; run them only from a database you built or a Release asset whose SHA256 you verified. Releases are produced by `.github/workflows/publish-catalog.yml`: it starts from the base Catalog pinned in `release/catalog-release.json`, injects the engine and documents of this commit, checks every data-table digest, runs the tests and the audit, then publishes.
