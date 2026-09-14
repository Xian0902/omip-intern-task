# 範例：OMIP-119

Harris RJ. et al. *OMIP-119: A 36-Color Full-Spectrum Flow Cytometry Panel for Deep Immunophenotyping of Peripheral Blood and Ex Vivo Expanded Human T Cells.* Cytometry Part A. 2025;107(12):787-792. DOI: [10.1002/cytoa.70001](https://doi.org/10.1002/cytoa.70001) · Open Access

四件套，就是你要做出來的東西。**檔名也是範例的一部分**，四個檔共用同一個 BASE：

```
BASE = Cytometry Pt A - 2025 - Harris - OMIP‐119  A 36‐Color Full‐Spectrum Flow Cytometry Panel for Deep Immunophenotyping of
```

| 檔案 | 上版後會在 | 看什麼 |
|---|---|---|
| `<BASE>.pdf` | `pdfs/` | 原始論文，以及 Wiley 下載時的原始檔名 |
| `<BASE>.md` | `markdown/` | pymupdf4llm 轉出來的樣子。表格被攤平成純文字是正常的 |
| `<BASE>_summary.md` | `translation/` | 翻譯檔的章節結構 |
| `<BASE>_panel.md` | `panels/` | **最重要**。欄位、frontmatter、章節順序全部照這份 |

這份面板檔在主專案裡已經上版，位置是 `panels/<BASE>_panel.md`。

## 檔名的兩個坑

用 `ls` 看一下就知道為什麼叫你不要手打檔名：

- `OMIP‐119` 的連字號是 **U+2010（‐）**，不是鍵盤上的 `-`（U+002D）。長得一樣，但 glob 比對不到。
- `OMIP‐119` 後面是**兩個空格**，不是一個。

所以：PDF 下載下來不要改名，其他三個檔用 `BASE="${PDF%.pdf}"` 推導。

## 讀這份範例時，注意這三件事

1. **PD-1 有兩列**（第 11 列 EH12.1 / 第 31 列 MIH4）。同一個標記兩個克隆是刻意的面板設計，不是重複，不要合併。
2. **N/A 就是 N/A**（PE/TCR、Live/Dead Blue 的 Clone 與 RRID）。論文沒寫就不要去補。
3. **用途分類統計的加總是 45，大於 36**。因為一個標記可跨多類。這不是算錯。

欄位規格見上一層的 [`格式規格.md`](../格式規格.md)。
