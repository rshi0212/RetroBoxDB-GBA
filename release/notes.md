GBA Catalog, storage v4 (1 MiB blocks, 123 solid LZMA2 groups of up to 256 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- RetroAchievements ROM set imported: 1,206 ZIPs; 775 ROM files are also in a No-Intro DAT, 426 are only in the RA set, 5 have a hash absent from the latest RA snapshot. RA games with achievements that have a local ROM: 581 → 750.
- Source collections (`source_collections`, `v_collection_files`) and RetroAchievements links per file (`v_ra_collection`).
- ROMs outside every DAT join the family of the stored ROMs they share the most blocks with (hacks next to their original).
- Imports and DAT packaging no longer decode solid groups for already stored blocks; DAT formats (e.g. FDS/QD, NES headered/headerless) are handled separately.
- Source: 5,152 ZIPs (nointro 3,946, retroachievements 1,206), 21.20 GiB (5,152 ROM files, 44.24 GiB uncompressed). Populated database: 7.01 GiB (33.1% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 4,143 ROM records, 1,901 games, 3,750 releases; DAT versions: 20260531-074517, 20260707-143610, 20260812-060017, 20260929-130236.
- RetroAchievements: 750 of 775 games with achievements have a local ROM.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole set in storage order 22.3 MiB/s (5,152 ROM files); single file with a cold cache 2.413 s (ROM) / 2.682 s (TorrentZip) on average.
- Full audit of the populated database: 4,149 objects, 123 groups, 4,396 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-GBA/blob/main/README.zh-CN.md)
