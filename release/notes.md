GBA Catalog, storage v4 (1 MiB blocks, 104 solid LZMA2 groups of up to 256 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- First release.
- Source: 3,946 No-Intro ZIPs, 14.42 GiB (3,946 ROM files, 30.92 GiB uncompressed). Populated database: 5.51 GiB (38.2% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 3,734 ROM records, 1,901 games, 3,750 releases; DAT versions: 20260531-074517, 20260707-143610, 20260812-060017, 20260929-130236.
- RetroAchievements: 581 of 774 games with achievements have a local ROM.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole set in storage order 22.2 MiB/s (3,946 ROM files); single file with a cold cache 2.374 s (ROM) / 2.751 s (TorrentZip) on average.
- Full audit of the populated database: 3,740 objects, 104 groups, 3,940 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-GBA/blob/main/README.zh-CN.md)
