# gov.govinfo.bulkdata

`govinfo.gov` の bulkdata から取得した、**米国連邦議会 第119議会の法案・決議・成立法の全文と関係グラフ** を保全する DataLad dataset です。統合用 datom は `etzhayyim/global-legislation-datoms` が生成します。

**この dataset だけが「法案 (bill)」を持ちます。** 他の 3 つは成立した法令を扱いますが、ここには審議中のものを含む全 measure が入ります。

## 現在地（2026-08-04 実測）

| | |
|---|---|
| 対象 | **第119議会**（`hr` `s` `hjres` `sjres` `hconres` `sconres` `hres` `sres` 全種別） |
| measure | **18,052** |
| うち全文取得済み | **17,964** |
| 成立公法 (Public Law) | **102** |
| 依存辺 | **11,265** |

## レイヤ

| パス | 中身 | git での扱い |
|---|---|---|
| `raw/billstatus/BILLSTATUS-119-<type>.zip` | 各 measure の status レコード。**関係グラフはここ** | git-annex → B2 |
| `raw/bills/BILLS-119-<session>-<type>.zip` | 各 bill version の USLM 全文 | git-annex → B2 |
| `raw/plaw/PLAW-119-public.zip` | 成立公法の USLM 全文 | git-annex → B2 |
| `raw/source-catalog.edn` | 各 zip の sha256 | **git 本体** |
| `index/laws.edn` / `index/relations.edn` | 消費者が読む索引 | **git 本体** |

**zip のまま保全しているのは意図的です。** govinfo が bulk zip を公開しているのは、まさに消費者に 1 件ずつ叩かせないためで、第119議会だけで 3 万件超の文書があります。個別 GET なら 3 万リクエスト、zip なら **25 リクエスト**です。索引生成時に一度だけ展開して各エントリの sha256 を取り、`index/laws.edn` は「どの zip の・どのエントリか・その sha256」で本文を指します。

## 依存辺

`BILLSTATUS` XML が持つ構造化データから:

| 出どころ | 辺 |
|---|---|
| `<laws>` | `:law.rel/became-law` — この法案が成立してこの公法になった |
| `<relatedBills>` の `Identical bill` | `:law.rel/identical-measure` |
| `<relatedBills>` の `Companion measure` | `:law.rel/companion-measure` |
| `<relatedBills>` の `Procedurally-related` | `:law.rel/procedurally-related` |
| `<relatedBills>` のその他 | `:law.rel/related-measure` |

実装上の注意: `<relatedBills><item>` の中に `<relationshipDetails><item>` が**入れ子で**入っているため、`</item>` での素朴な分割は関係種別を隣の法案に付け替えてしまいます。深さを追う分割で直接の子だけを取っています。

## wave-1 で取っていないもの

- **第118議会以前**（URL 形状は同じで議会番号が違うだけ。113th 以降が公開されている）。
- **US Code / CFR**。`uscode.house.gov` はこのネットワークから到達不能でした（実測 2026-08-04、75 秒でタイムアウト）ので、成典化された法典層は別途。

## ライセンス

合衆国連邦政府の法令は **著作権の対象外**（17 U.S.C. §105、および edicts of government の原則）。govinfo は bulk 再利用のために公開しています。**Tier-A**。

## 再取得 / 再生成

```bash
kbb --backend sci --classpath bin bin/fetch.cljk --congress 119   # 25 個の zip
kbb --backend sci --classpath bin bin/index.cljk                  # 展開して索引化（system unzip を使用）
```

`bin/index.cljk` は system の `unzip` を使います。Node は zlib を持ちますがアーカイブリーダを持たず、保全 dataset の索引再生成が数年後に npm install を要求する状態にはしたくないためです（`unzip` は POSIX）。
