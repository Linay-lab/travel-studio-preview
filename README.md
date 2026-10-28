# 旅購 Travel Studio：GPS 購物提醒開發預覽

分支：`feature/gps-shopping-reminders`。基於 main `60b04d9`；未部署、未修改 main。

直接開啟 index.html 可操作介面。GPS 請透過 HTTPS 或 localhost 開啟，例如 `python -m http.server 8000` 後瀏覽 http://localhost:8000。

購物清單 → 提醒設定與店家 → 新增確切分店座標 → 編輯商品選擇提醒店家 → 啟用定位提醒。同店待購商品合併提醒，已購商品排除。距離可設定 100、200、500、1000 公尺；離開範圍加 100 公尺，且間隔滿 30 分鐘後才再次提醒。

- [測試報告與使用步驟](docs/TEST-REPORT.md)
- [iOS／Android 背景定位規劃](docs/BACKGROUND-LOCATION-PLAN.md)
- [自動測試](tests/gps-reminders.cjs)
- [本次測試結果](tests/results.json)

本版僅在網頁前景定位；手機鎖屏或關閉後的背景提醒尚未實作。手機尺寸 Chromium 已測試，Safari／WebKit 與實體手機 GPS 尚未驗證。Google Apps Script 回饋程式保留，測試請求已攔截，未寄出測試信。

## 自動測試機器人

[操作說明](docs/TEST-BOT.md)：更新程式、提出 PR 或 Pages 發布成功後自動測試；也可在 GitHub Actions 手動執行，下載 HTML 報告及截圖。

## 完整功能及安全檢查

[完整檢查報告](docs/FULL-AUDIT.md) · [原始測試證據](docs/AUDIT-EVIDENCE.json)。已補強文字注入與備份驗證；仍為本機 Beta，沒有真實會員後端，不能宣稱零外洩或任意人數穩定。
