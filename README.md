# 實習任務：OMIP 論文轉換（OMIP-120 / OMIP-121）

## 一句話任務

把兩篇 Wiley 開放存取的 OMIP 論文，各自產出 **繁體中文翻譯 markdown** 與 **抗體面板 markdown**，格式與檔名比照 `範例_OMIP-119/`。

## 你的範圍到哪裡

| | 誰做 |
|---|---|
| 下載 PDF、轉檔、翻譯、整理抗體面板 | **你** |
| 檔案放進 `繳交/OMIP-XXX/`、開 PR | **你** |
| 把檔案搬進 `pdfs/` `markdown/` `translation/` `panels/` | maintainer |
| 更新 `omip_tables_v2.jsonl`、跑 `migrate.py`、重啟服務 | maintainer |

**你不用碰上版流程，也不要自己去動 `pdfs/` 等專案資料夾。**
但你交出來的檔名必須是「可以直接複製過去、不用改名」的最終檔名 —— 下面的檔名規則就是為了這件事。

---

## 專案脈絡（先看這段，不然會做偏）

**這個 repo 只是作業包。** 你交的檔案最後會被匯入到另一個內部專案 **OMIP Playground**（一個 OMIP 論文閱讀器），你不會拿到那個 repo 的存取權，也不需要。

主專案的資料流長這樣：

```
pdfs/         原始 PDF                      → API /articles/{omip}/pdf
markdown/     原文 markdown（pymupdf4llm）   （目前僅備查，前端未使用）
translation/  繁中翻譯 + 詳細摘要             → API /articles/{omip}/summary
panels/       抗體面板 markdown（canonical）
  ↓ 抽取
frontend/omip_tables_v2.jsonl → backend/scripts/migrate.py → DuckDB → API
```

後端是用 **glob 比對檔名** 去找檔案的（`backend/api/v1/files.py`），
例如翻譯檔的比對樣式是 `*OMIP*120*_summary.md`。
**檔名錯了，網站就是 404，沒有其他補救方式。** 所以檔名規則是硬性規定。

---

## 你要處理的兩篇

| # | 編號 | DOI | 連結 | 標題 | 物種 / 平台 |
|---|---|---|---|---|---|
| 1 | **OMIP-120** | `10.1002/cyto.a.70034` | https://onlinelibrary.wiley.com/doi/full/10.1002/cyto.a.70034 | A 22-Marker Spectral Flow Cytometry Panel for the Characterization of Major Immune Populations in Murine Bone Marrow and Osteosarcoma Tissue | 小鼠 / 全光譜流式 |
| 2 | **OMIP-121** | `10.1002/cyto.a.70040` | https://onlinelibrary.wiley.com/doi/10.1002/cyto.a.70040 | Immune Phenotyping of Canine Peripheral Leukocytes by Mass Cytometry | 犬 / 質譜流式（CyTOF） |

兩篇都是 **CC BY-NC 開放存取**，可以合法下載與再利用（記得標示出處）。

書目資訊（引用時用）：
- OMIP-120：Foderaro S. et al., *Cytometry Part A*. 2026;109(5):317-349.
- OMIP-121：Luong HTT. et al., *Cytometry Part A*. 2026;109(6):393-412.

---

## 檔名規則（最重要，先讀這段）

### BASE 怎麼來

**從 Wiley 下載的 PDF，原始檔名不要改。** 那個檔名就是 `BASE`。

Wiley 下載下來會長這樣（OMIP-119 的實例）：

```
Cytometry Pt A - 2025 - Harris - OMIP‐119  A 36‐Color Full‐Spectrum Flow Cytometry Panel for Deep Immunophenotyping of.pdf
```

注意裡面有兩個坑，**這就是為什麼叫你不要手打檔名**：

- `OMIP‐119` 的連字號是 **U+2010（‐）不是 ASCII 的 `-`**，看起來一模一樣但不是同一個字元
- `OMIP‐119` 後面是**兩個空格**

手打幾乎一定打錯。用指令從 PDF 檔名自動推導：

```bash
cd 繳交/OMIP-120
PDF=$(ls *.pdf)
BASE="${PDF%.pdf}"
echo "BASE = $BASE"
```

### 四個檔案的檔名

| 檔案 | 檔名 |
|---|---|
| 原始 PDF | `<BASE>.pdf` （= Wiley 原檔名，不改） |
| 原文 markdown | `<BASE>.md` |
| 翻譯 markdown | `<BASE>_summary.md` |
| 抗體面板 markdown | `<BASE>_panel.md` |

看 `範例_OMIP-119/` 裡的四個檔案，就是長這樣。

### 檔案會被匯入到哪裡（maintainer 執行，你不用做）

| 你交的檔案 | 匯入目的地 | 用途 |
|---|---|---|
| `<BASE>.pdf` | **`pdfs/`** | 網站 PDF 檢視器 |
| `<BASE>.md` | **`markdown/`** | 原文備查 |
| `<BASE>_summary.md` | **`translation/`** | 網站右側翻譯面板 |
| `<BASE>_panel.md` | **`panels/`** | 抗體面板 canonical source，之後抽成 `omip_tables_v2.jsonl` |

因為檔名已經是最終檔名，上版就是四個 `cp`，不需要改名。
**所以檔名對不對，比內容漂不漂亮還重要。**

---

## 範例：OMIP-119（先讀完再動手）

`範例_OMIP-119/` 是完整四件套，檔名就是上面規則的實例。

| 檔案 | 看什麼 |
|---|---|
| `<BASE>.pdf` | 原始論文長相，以及 Wiley 原始檔名 |
| `<BASE>.md` | pymupdf4llm 轉出來會是什麼樣子（表格被攤平成純文字是正常的） |
| `<BASE>_summary.md` | 翻譯檔的章節結構 |
| `<BASE>_panel.md` | **最重要**，面板檔的欄位規格 |

同一份面板檔也已經放進 `panels/`，可以對照看它上版後的位置。

> ⚠️ 範例的翻譯檔在正文中途有一行「（原文在此截斷）」——那是舊 pipeline 的缺陷，**你的版本不准截斷**，正文要翻完。

詳細欄位規格見 [`格式規格.md`](格式規格.md)。

---

## 步驟

### Step 0｜環境

```bash
git clone <本 repo 的 URL>
cd <repo 目錄>
uv sync
```

用 `uv`，不要用 `pip`。轉檔要用的 `pymupdf4llm` 已經寫在 `pyproject.toml` 裡了。

沒裝過 uv 的話：

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Step 1｜下載 PDF

**用瀏覽器下載，不要用 curl/wget。** Wiley 有 Cloudflare 防護，指令列抓會拿到 403 加一頁 HTML 錯誤頁（而且副檔名還是 .pdf，很容易沒發現）。

1. 開上表的連結
2. 點頁面上的 **PDF** → **Download PDF**
3. 把下載到的檔案 **原封不動**（不要改名）移到 `繳交/OMIP-120/`

驗證它真的是 PDF：

```bash
file 繳交/OMIP-120/*.pdf
# 要看到：PDF document, version 1.x
# 看到 "HTML document text" 就是被擋了，重下
```

### Step 2｜PDF → 原文 markdown

```bash
cd 繳交/OMIP-120
PDF=$(ls *.pdf); BASE="${PDF%.pdf}"

uv run python -c "
import pymupdf4llm, pathlib, sys
base = sys.argv[1]
pathlib.Path(base + '.md').write_text(
    pymupdf4llm.to_markdown(base + '.pdf'), encoding='utf-8')
print('ok ->', base + '.md')
" "$BASE"
```

轉完打開看一眼：Table 1 和 Table 2 有沒有整段掉字？欄位有沒有被吃掉？
**如果表格轉壞了，Step 4 請直接對著 PDF 手抄，不要拿壞資料往下做。**

### Step 3｜翻譯 markdown → `<BASE>_summary.md`

可以用 LLM 輔助翻，但 **一定要自己逐段對照原文校過一遍**。

必備章節（照範例的 `_summary.md`）：

```markdown
- **處理日期**: YYYY-MM-DD HH:MM:SS

---

### 繁體中文翻譯

（標題、作者、單位、通訊作者、收稿/修訂/接受日期、摘要、正文全文）

### 詳細摘要

#### 1. 研究背景與目的
#### 2. 方法
#### 3. 主要發現
#### 4. 應用價值

## 參考資料
```

翻譯規則：
- 專有名詞 **中英並陳**，第一次出現時寫成「調節性 T 細胞（Treg）」，之後可只寫 Treg
- 標記名（CD3、FoxP3、CXCR5）、螢光染料名（BUV395、Spark NIR 685）、克隆編號（OKT3）、金屬同位素（<sup>141</sup>Pr）**一律保留原文**，不要翻譯也不要改大小寫
- 圖表編號保留原樣：`圖1A`、`圖S17`、`表2`
- 參考文獻上標編號保留：`[5]`、`[8–10]`
- 用繁體中文與台灣用語（「面板」不是「组合」；「流式細胞術」不是「流式细胞术」）

### Step 4｜抗體面板 markdown → `<BASE>_panel.md`

把 Table 1 與 Table 2 抄成結構化 markdown。這步最容易出錯，**逐行對照 PDF 檢查**。

規格完整寫在 [`格式規格.md`](格式規格.md)，重點：

- YAML frontmatter 的 key 不可改名、不可漏
- Table 2 欄位順序固定：`# | 螢光染料/金屬 | Specificity | Clone | RRID | Purpose（原文） | 用途（繁中）`
- 原文寫 `N/A` 的欄位就填 `N/A`，**不要自己去查、不要臆測**
- 列數要對得上論文宣稱的 marker 數；對不上就在「資料缺漏與註記」寫清楚為什麼

**OMIP-121 的特別處理**：它是質譜流式（mass cytometry），用的是金屬同位素不是螢光染料。
第二欄表頭改成 `Metal 金屬標籤`，frontmatter 加一行 `label_type: metal`（OMIP-120 是 `label_type: fluorochrome`）。其他欄位不變。

---

## 驗收標準

每個 OMIP 都要全部打勾才算完成：

**檔名（最先檢查，錯了後面都白做）**
- [ ] PDF 是 Wiley 原始檔名，沒有被改過
- [ ] 四個檔案共用同一個 BASE，只差後綴 `.pdf` / `.md` / `_summary.md` / `_panel.md`
- [ ] 跑過下面的自檢指令，四個檔都對得上

```bash
cd 繳交/OMIP-120
PDF=$(ls *.pdf); BASE="${PDF%.pdf}"
for f in "$BASE.pdf" "$BASE.md" "$BASE"_summary.md "$BASE"_panel.md; do
  [ -f "$f" ] && echo "✅ $f" || echo "❌ 缺少或檔名錯誤：$f"
done
file "$BASE.pdf" | grep -q "PDF document" && echo "✅ PDF 格式正確" || echo "❌ PDF 是假的，重下"
```

**翻譯檔**
- [ ] 正文完整翻完，**沒有任何截斷或「以下省略」**
- [ ] 六個必備章節都在，標題層級（`###` / `####`）正確
- [ ] 標記名、染料名、克隆編號、圖表編號、參考文獻編號全部保留原文
- [ ] 全篇繁體中文，沒有混入簡體字
- [ ] 自己讀過一遍，句子是通的（不是機翻直出）

**面板檔**
- [ ] frontmatter 的 key 齊全且拼字正確
- [ ] Table 1 四個欄位（Purpose / Species / Cell type / Cross-references）都有，中英並陳
- [ ] Table 2 每一列都跟 PDF 逐字核對過
- [ ] 列數 == 論文宣稱的試劑數（含活/死染劑）
- [ ] RRID 格式為 `RRID:AB_xxxxxxx` 或 `N/A`
- [ ] 用途分類統計的加總數字算得出來、跟表格一致
- [ ] markdown 表格在 GitHub 或 VS Code preview 下能正常渲染（沒有跑版）

---

## 常見陷阱

| 陷阱 | 症狀 | 怎麼避開 |
|---|---|---|
| 手打檔名 | `OMIP-120` 用了 ASCII `-`，上版後網站找不到檔案 | 用 `BASE="${PDF%.pdf}"` 推導，不要手打 |
| 改了 PDF 檔名 | BASE 跟其他 OMIP 不一致 | Wiley 下載下來就不要動 |
| curl 抓 Wiley | 5,914 bytes 的「PDF」 | 用瀏覽器下載，`file` 驗證 |
| PDF 的連字號 | `CD45RA` 變成 `CD45­RA`（中間有隱形的 soft hyphen U+00AD） | 轉檔後全域清掉 `­`（U+00AD） |
| 上標被攤平 | `CD4⁺` 變成 `CD4 [+]` | 面板檔手動修回 `CD4+`；翻譯檔用 `⁺` |
| 同一標記兩個克隆 | 只抄了一列 | OMIP-119 的 PD-1 就有 EH12.1 和 MIH4 兩列，這是刻意設計，**兩列都要留** |
| 自行補 N/A 欄位 | RRID 欄填了論文沒寫的值 | 原文 N/A 就 N/A |
| 簡繁混用 | 「面板」寫成「组合」 | 交件前搜一次常見簡體字 |
| 表格跑版 | 儲存格內文有 `\|` | 改成 `\\|` 跳脫 |

---

## 繳交方式

```bash
git checkout -b omip-120-121
git add 繳交/
git commit -m "feat: add OMIP-120 and OMIP-121 translation and panel markdown"
git push -u origin omip-120-121
```

然後在 GitHub 上開 Pull Request，內文貼上你的自檢清單（上面的驗收標準勾完）。

- **PDF 也要一起 commit**（兩篇都是 CC BY-NC，可以放進 repo）
- **只 commit `繳交/` 底下的東西**。`範例_OMIP-119/`、`README.md`、`格式規格.md` 不要動
- 搬檔案到主專案、更新資料庫，是 maintainer 收到 PR 後做的，**你不用管**

---

## 加分題（做完主線再說）

1. 寫一支 `panel_md_lint.py`：讀 `_panel.md`，檢查 frontmatter key 齊全、Table 2 欄數正確、RRID 格式合法，不合格就 exit 1。
2. 寫一支 `panel_md_to_jsonl.py`：把 `_panel.md` 的 Table 1/Table 2 轉成主專案 `frontend/omip_tables_v2.jsonl` 的 schema（`table1` / `table2.entries`）。這支做出來，上版就能自動化。
3. 比較 OMIP-119（人類）、OMIP-120（小鼠）、OMIP-121（犬）三者的標記重疊度，寫成一頁分析。

---

## 卡住的話

先自己查 30 分鐘，還是不行就來問，附上：你做到哪一步、指令是什麼、完整錯誤訊息。

---

## 授權與出處

- 本 repo 收錄的 OMIP 論文 PDF 與其衍生內容，皆為 **Cytometry Part A** 的開放存取文章，授權為 **CC BY-NC 4.0**，可在標示出處的前提下非商業使用與再散布。
- 每個檔案的 frontmatter / README 都帶有完整書目與 DOI，請不要移除。
- 你產出的翻譯與面板整理，同樣以 CC BY-NC 4.0 釋出。
