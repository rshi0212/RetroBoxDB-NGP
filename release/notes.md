NeoGeo Pocket Catalog, storage v4 (128 KiB blocks, 1 solid LZMA2 group of up to 32 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- First release: NeoGeo Pocket cartridges and BIOS images from the No-Intro folder and the `.ngp` files of the shared RetroAchievements NeoGeo Pocket set; 1 Parent-Clone DAT version, DB Export and Dump Log unknown, RetroAchievements snapshot and Chinese names.
- Storage measured on the whole collection: 128 KiB blocks, 32 MiB groups (`assessment/data/storage-experiment-ngp.json`).
- RetroAchievements lists both platforms of the pair under one console and one folder; each database keeps its own files and looks up its sibling (NeoGeo Pocket<->NeoGeo Pocket Color).
- Source: 14 ZIPs (nointro 13, retroachievements 1), 5.3 MiB (14 ROM files, 13.7 MiB uncompressed). Populated database: 6.0 MiB (114.6% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 13 ROM records, 12 games, 13 releases; DAT versions: 20250904-215533.
- RetroAchievements: 1 of 1 games with achievements have a local ROM.
- Chinese names: 10 of 10 CSV rows translated; 10 local ROMs with Chinese names.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole newest-DAT set with export_set.py 29.5 MiB/s (13 files); single file with a cold cache 0.091 s (ROM) / 0.186 s (TorrentZip) on average.
- Full audit of the populated database: 16 objects, 1 group, 16 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-NGP/blob/main/README.zh-CN.md)
