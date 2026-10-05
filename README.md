# RetroBoxDB GBA

English | [中文说明](README.zh-CN.md)

Single-file SQLite preservation database for Nintendo Game Boy Advance. The public Catalog holds metadata only (checksums, DAT and provenance records, header fields, archive recipes and the processing code); it contains no ROM data and cannot restore files. The populated database stays local.

| Item | Value |
| --- | --- |
| Original size | 3,946 No-Intro ZIPs, 14.42 GiB; 3,946 ROM files, 30.92 GiB uncompressed |
| Stored size | populated database 5.51 GiB; public Catalog 54.7 MiB (no ROM data) |
| Ratio | 38.2% of the source ZIPs, 17.8% of the uncompressed ROM files |
| Technology | storage v4: SHA256-deduplicated 1 MiB blocks packed in No-Intro family order into solid LZMA2 groups of up to 256 MiB (256 MiB dictionary); per-block SHA256 and per-object CRC32/MD5/SHA1/SHA256 verification; source ZIPs reproduced byte-for-byte from TorrentZip plans |
| Export performance | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, Python 3.14.4, all checks included. Whole-set export (3,946 ROM files in storage order, each group decoded once): 22.2 MiB/s, 362 ms per file on average; single file with a cold cache (the group is decoded up to the file): ROM 2.374 s, TorrentZip 2.751 s on average |

## Downloads and documents

| File / document | Content |
| --- | --- |
| [RetroBoxDB.GBA.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-GBA/releases/latest/download/RetroBoxDB.GBA.Catalog.sqlite) | Public Catalog (Release asset with `SHA256SUMS`) |
| [Storage v4 guide](RetroBoxDB.Storage-v4.en.md) / [中文](RetroBoxDB.Storage-v4.zh-CN.md) | Storage evaluation, contents, RA, names and maintenance for all six platforms |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | Storage format, platform adapters, incremental updates, verification |
| [RA list](reports/ra-gba-games.csv) / [summary](reports/ra-gba.json), [build report](reports/gba-build-report.json), [audit resolution](reports/audit-resolution-20261004.md) | Detailed data |

## Storage choice and platform specifics

Change against 128 MiB groups on real data (first 8 family-ordered groups, 956 MiB; 512 MiB exceeds the 256 MiB engineering ceiling): 256 MiB −2.08%, 512 MiB −3.29%; 256 MiB chosen by the rule.

- ROMs are mostly 4–32 MiB, so regional versions of one game only share a dictionary when groups are large; 1 MiB blocks compress like 64 KiB blocks with about 15 times fewer block rows.
- xz's ARM and ARM-Thumb BCJ filters were measured on six real 128 MiB groups: ARM-Thumb made the output 2.35% larger and ARM 0.73% larger (graphics/audio data dominates and the conversion disturbs cross-version repeats), so plain LZMA2 is used. 512 MiB groups would save a further 1.24% but need about 6 GB of encoder memory per worker and decode about 256 MiB per read on average; 256 MiB is the engineering ceiling.
- Cartridge header at 0x00–0xBF: title, game code, maker code, fixed byte 0x96, device type, version and complement check; save-library IDs (EEPROM_Vnnn, SRAM_Vnnn, FLASH_Vnnn, …) found at word-aligned offsets; trailing 0xFF/0x00 padding length (padding blocks deduplicate).

## Contents

| Item | Value |
| --- | --- |
| ROM records / games / releases | 3,734 / 1,901 / 3,750 |
| DAT coverage per version | 20260531-074517: 3,676/3,745; 20260707-143610: 3,676/3,748; 20260812-060017: 3,676/3,749; 20260929-130236: 3,676/3,750 |
| Local ROMs in no DAT | 58 |
| No-Intro DB Export + Dump Log 20260929-130236 | 3,793 archives, 4,563 file identities, 3,188 documented hardware assertions; Dump Log Verified 770 |
| RetroAchievements (console 5) | 774 games with achievements: 581 with a local ROM (823 ROMs), 2 DAT only, 3 DB file only, 188 without a No-Intro counterpart |
| Chinese names | 3,410 of 3,522 rows translated (1,882 unique); 3,350 local ROMs have a Chinese name |
| Populated-database audit | 3,740 objects, 104 groups, 3,940 archive plans, all passed |

Every source ZIP is reproduced byte-for-byte from its TorrentZip plan (`v_file_checksums.exported_bytes_equal_source`).

## Usage

```bash
# Query-only audit with the Catalog's embedded engine (also: stats, checksums FILE_ID, help)
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.GBA.Catalog.sqlite audit
# Populated database: export by DAT version, 1G1R, RA achievements, TorrentZip or plain ROMs
python3 -B tools/export_set.py RetroBoxDB.GBA.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# Add new DATs, DB Export / Dump Log snapshots, ROMs and an RA snapshot incrementally
python3 -B tools/update_db.py RetroBoxDB.GBA.sqlite --discover --ra --catalog RetroBoxDB.GBA.Catalog.sqlite
```

Python 3.10+ standard library only. `engine.py` and the other `resources` entries are executable code; run them only from a database you built or a Release asset whose SHA256 you verified. Releases are produced by `.github/workflows/publish-catalog.yml`: it starts from the base Catalog pinned in `release/catalog-release.json`, injects the engine and documents of this commit, checks every data-table digest, runs the tests and the audit, then publishes.
