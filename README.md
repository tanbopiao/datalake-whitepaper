# ZONGYUAN-ROOT · 数据湖持久层白皮书

**LOCAL-PERSIST-DATA-LAKE V2.0 架构规范**

> Ω₀⊂⊙∞⊂Ω | DID-BR-000002 | 2026-09-30 首版 / 2026-10-21 白皮书提炼

> 本白皮书从 ZONGYUAN-ROOT 仓库 `DATA-LAKE-PERSIST/`（基准资产）提炼，遵循「元极恒一」治理规范：基准只读、衍生独立建库。

---

## 1. 要解决什么问题

自治系统的持久资产有三类失效模式：

1. **媒体黑洞** — 大体积媒体被推进 Git，仓库膨胀至数百 MB，clone 瘫痪
2. **孤儿文件** — 文件存在但没有登记：无指纹、无来源、无法确权
3. **账本脱节** — 索引、哈希账本、媒体登记三处各自演化，对不上账

数据湖持久层的回答：**媒体文件不进 Git，进湖登记；文本内核推多副本 Git；三者用 SHA256 指纹闭环对账。**

## 2. 六层架构

| 层 | 用途 | 内容 |
|---|---|---|
| 00-raw | 原始采集/上传层 | 原始镜像、原始上传、未处理素材 |
| 01-curated | 整理层 | 去重、清洗、分类后数据 |
| 02-refined | 成品层 | 确权资产、锁档文档、成品包 |
| 03-index | 索引层 | MANIFEST-LAKE.json、目录清单 |
| 04-media-registry | 媒体登记层 | MEDIA-REGISTRY.json（仅引用+指纹，不存文件） |
| 05-ledger | 账本层 | HASH-LEDGER 联动、确权指纹 |

## 3. 四条登记铁律

1. **登记先行** — 每次新资产归档，先登记 `03-index/MANIFEST-LAKE.json`（或追加 media 域）
2. **媒体引用化** — 资产 >2MB：物理文件只存本地媒体池（`LOCAL-PERSIST-MEDIA/<域>/`），Git 内仅登记 `rel_path + size + sha256`
3. **文本双副本** — 文本内核（索引/清单/账本）同步追加 HASH-LEDGER.csv，推送 Gitee+GitHub 双副本
4. **指纹全量** — 每个资产 sha256 全量计算，登记于 MEDIA-REGISTRY.json

## 4. 实测规模（2026-09-30 锁档）

| 域 | 文件数 | 体积 |
|---|---|---|
| unified-image-archive | 265 | 228.6 MB |
| zongyuan-root | 44 | 82.7 MB |
| kunlun-assets | 33 | 52.0 MB |
| zongyuan-total（含网站备份） | 7 | 16.7 MB |
| zongyuan-governance | 5 | 0.8 MB |
| frontier-tech-page | 2 | 0.2 MB |
| **合计** | **356** | **380.9 MB** |

其中媒体登记层覆盖 1196+ 文件 / 1280+ MB（kunlun-drama 域 1096 文件 1218 MB 全部为「登记引用、物理不进 Git」模式）——正是这条规则让主 Git 仓保持轻量。

## 5. 与治理内核的关系

- **只读锁档**：固化资产 RO-444，与「元极恒一」基准只读铁律同构
- **哈希账本**：`05-ledger/` 与主仓 `HASH-LEDGER.csv` 联动，构成 Merkle 式证据链
- **多副本分发**：文本内核 → Git 多平台；媒体 → 本地池；确权 → 登记层引用。三层各走最优通道

## 6. 可复用的设计模式

任何需要「大文件 + 可确权 + Git 友好」的项目都可套用此模式：

```
project/
├── repo/                  # Git 仓：代码 + 文本内核 + 登记文件（轻量）
│   ├── 03-index/MANIFEST-LAKE.json
│   ├── 04-media-registry/MEDIA-REGISTRY.json   # rel_path+size+sha256
│   └── 05-ledger/HASH-LEDGER.csv
└── media-pool/            # 本地/对象存储：物理媒体（gitignore）
    └── <domain>/...
```

要点：**登记文件进 Git，物理文件出 Git，双方靠 sha256 对账。**

## 7. 来源

- 基准：[ZONGYUAN-ROOT](https://atomgit.com/zongyuangen/ZONGYUAN-ROOT) `DATA-LAKE-PERSIST/`（README、MANIFEST-LAKE.json、MEDIA-REGISTRY-V2.json、LAKE-LEDGER）
- 关联作品：[AxiomKit](https://atomgit.com/zongyuangen/AxiomKit) · [yuanji-agent-stack](https://atomgit.com/zongyuangen/yuanji-agent-stack) · [TruthLedger](https://atomgit.com/zongyuangen/TruthLedger) · [NodeHub](https://atomgit.com/zongyuangen/NodeHub) · [SimBench](https://atomgit.com/zongyuangen/SimBench)
- 许可：CC-BY-4.0（文档）
