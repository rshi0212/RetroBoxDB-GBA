# RetroBoxDB GBA

[English](README.md) | 中文

任天堂 Game Boy Advance的单文件 SQLite 保存库。公开的 Catalog 只含元数据（校验值、DAT 与来源记录、头部字段、打包配方和程序），不含 ROM 数据，不能独立恢复文件；完整库保留在本地。

| 项目 | 数值 |
| --- | --- |
| 原始大小 | 源 ZIP 5,152 个，21.20 GiB（No-Intro 3,946 个，RetroAchievements 集合 1,206 个）；解压后 ROM 5,152 个，44.24 GiB |
| 入库后大小 | 完整库 7.01 GiB；公开 Catalog 57.8 MiB（不含 ROM 数据） |
| 比例 | 完整库为原 ZIP 的 33.1%，为解压后 ROM 总量的 15.9% |
| 使用的技术 | 存储 v4：1 MiB 块按 SHA256 去重，按 No-Intro 游戏族顺序装入最大 256 MiB 的 LZMA2 实体组（字典 256 MiB）；逐块 SHA256、逐对象 CRC32／MD5／SHA1／SHA256 校验；源 ZIP 由 TorrentZip 配方逐字节重建 |
| 导出性能 | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz，空闲负载，Python 3.14.4，含全部校验。按最新 DAT 整套导出（`export_set.py`，3,676 个文件，逐个按 DAT 哈希校验）：25.1 MiB/s，平均 323 毫秒／个；单个文件冷缓存（每次清空缓存，需解压所在组的前段）：ROM 平均 2.413 秒，TorrentZip 平均 2.682 秒 |

## 下载与说明

| 文件／文档 | 内容 |
| --- | --- |
| [RetroBoxDB.GBA.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-GBA/releases/latest/download/RetroBoxDB.GBA.Catalog.sqlite) | 公开 Catalog（Release 附件，附 `SHA256SUMS`） |
| [存储 v4 说明](RetroBoxDB.Storage-v4.zh-CN.md)／[English](RetroBoxDB.Storage-v4.en.md) | 七个平台的存储评估、内容、RA、中文名与维护 |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | 存储格式、平台适配、增量更新、校验 |
| [RA 清单](reports/ra-gba-games.csv)／[汇总](reports/ra-gba.json)、[构建报告](reports/gba-build-report.json)、[审计处理](reports/audit-resolution-20261004.md) | 逐项数据 |

## 本平台的存储选择与特殊情况

真实全量数据（按族排序的前 8 个组，956 MiB；512 MiB 超过 256 MiB 工程上限）上相对 128 MiB 组的变化：256 MiB −2.08%，512 MiB −3.29%；按规则采用 256 MiB。

- ROM 多为 4–32 MiB，同一游戏的地区版只有在大组里才能共用字典；1 MiB 块与 64 KiB 块压缩效果相同，块记录约少 15 倍。
- 在 6 个真实的 128 MiB 组上实测 xz 的 ARM、ARM-Thumb BCJ 过滤器：ARM-Thumb 使结果增大 2.35%，ARM 增大 0.73%（图形、音频数据占多数，地址转换破坏跨版本重复），因此保持纯 LZMA2。512 MiB 组还能再省 1.24%，但每个编码进程约需 6 GB 内存，平均每次读取要解压约 256 MiB；256 MiB 为工程上限。
- 0x00–0xBF 卡带头：标题、游戏代码、厂商代码、固定字节 0x96、设备类型、版本与补码校验；按 4 字节对齐扫描存档库标识（EEPROM_Vnnn、SRAM_Vnnn、FLASH_Vnnn 等）；记录末尾 0xFF／0x00 填充长度（填充块去重）。

## 内容

| 项目 | 数值 |
| --- | --- |
| ROM 记录／游戏组／发行版本 | 4,143／1,901／3,750 |
| 各版 DAT 覆盖 | 20260531-074517：3,676/3,745；20260707-143610：3,676/3,748；20260812-060017：3,676/3,749；20260929-130236：3,676/3,750 |
| 不在任何 DAT 的本地 ROM | 467 |
| RetroAchievements 集合中的 ROM 文件 | DAT 中有 775，仅 RA 收录 426，哈希不在最新 RA 快照 5（[清单](reports/ra-gba-collection-unknown.csv)）；仍缺本地 ROM 的 RA 游戏见 [缺口清单](reports/ra-gba-missing.csv) |
| No-Intro DB Export＋Dump Log 20260929-130236 | 3,793 个档案、4,563 个文件身份、3,188 条有文档的硬件声明；Dump Log Verified 770 |
| RetroAchievements（console 5） | 有成就的游戏 775 个：本地有 ROM 750（1,230 个 ROM），仅 DAT 有 1，仅 DB 文件 1，无 No-Intro 对应 23 |
| 中文名 | 3,522 条记录中 3,410 条有中文（1,882 个唯一名）；本地 ROM 3,350 个有中文名 |
| 完整库审计 | 4,149 个对象、123 个组、4,396 个 ZIP 配方，全部通过 |

源 ZIP 均可由 TorrentZip 配方逐字节重建（`v_file_checksums.exported_bytes_equal_source`）。

## 使用

```bash
# 用 Catalog 内嵌引擎做只读审计（stats、checksums FILE_ID、help 同理）
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.GBA.Catalog.sqlite audit
# 完整库：按 DAT 版本、1G1R、RA 成就、TorrentZip／裸 ROM 组合导出
python3 -B tools/export_set.py RetroBoxDB.GBA.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# 增量加入新 DAT、DB Export／Dump Log、ROM 与 RA 快照
python3 -B tools/update_db.py RetroBoxDB.GBA.sqlite --discover --ra --catalog RetroBoxDB.GBA.Catalog.sqlite
```

只需 Python 3.10+ 标准库。`resources` 中的 `engine.py` 等是可执行代码，只应从自己构建或 SHA256 已核对的 Release 附件中执行。发布由 `.github/workflows/publish-catalog.yml` 完成：工作流从 `release/catalog-release.json` 固定的基础 Catalog 出发，注入本仓库提交中的引擎与文档，核对全部数据表摘要、运行测试与审计后发布。
