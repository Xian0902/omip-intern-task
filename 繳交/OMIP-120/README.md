# OMIP-120 繳交區

把四個檔案放進這個資料夾。**檔名不要手打**，從 Wiley 下載的 PDF 推導：

```bash
cd 繳交/OMIP-120
PDF=$(ls *.pdf); BASE="${PDF%.pdf}"
```

| 檔案 | 之後會被匯入到 |
|---|---|
| `<BASE>.pdf` （Wiley 原檔名，不改） | `pdfs/` |
| `<BASE>.md` | `markdown/` |
| `<BASE>_summary.md` | `translation/` |
| `<BASE>_panel.md` | `panels/` |

搬檔案是 maintainer 做的，**你只要把檔名交對、內容做好**。

## 交件前自檢

```bash
cd 繳交/OMIP-120
PDF=$(ls *.pdf); BASE="${PDF%.pdf}"
for f in "$BASE.pdf" "$BASE.md" "$BASE"_summary.md "$BASE"_panel.md; do
  [ -f "$f" ] && echo "✅ $f" || echo "❌ 缺少或檔名錯誤：$f"
done
file "$BASE.pdf" | grep -q "PDF document" && echo "✅ PDF 格式正確" || echo "❌ PDF 是假的，重下"
```

規格見 [../../格式規格.md](../../格式規格.md)，範例見 [../../範例_OMIP-119/](../../範例_OMIP-119/)。
完整驗收清單見 [../../README.md](../../README.md)。
