# 旅購：功能、前端安全與隔離負載檢查

日期：2026-10-28（台灣）  
基準：main `627cb0a788cc7bf8d8143df6fa8361345396a84a`  
工作分支：`security/full-site-audit`

## 結論與可用範圍

已擴充為目前網頁功能的完整主要流程測試，並修正實際重現的前端腳本注入與資料驗證問題。這不是零漏洞、零外洩、無限承載量的認證；目前仍不具備正式會員多人服務所需的身分驗證、伺服器授權、共享資料庫隔離及後端負載驗收。

現在的登入／註冊／忘記密碼是本機流程模擬；不同裝置使用各自的 localStorage，沒有帳號同步。同一個瀏覽器上的不同「模擬帳號」不能視為安全隔離的會員。登出也不會清除本機旅行及回饋內容，重新進入 Beta 可再看到資料。請勿把護照、付款資料、正式帳號密碼或其他敏感資訊輸入此測試版。

## 功能覆蓋矩陣

`gps-reminders.cjs` 保留原有 CRUD 與 GPS 測試；新增 `full-site.cjs`，目前每種環境包含 18 組功能／邊界檢查。測試不只載入頁面，亦會實際操作表單、上傳與匯出檔案。

| 功能 | 本輪涵蓋 |
| --- | --- |
| 模擬帳號 | 登入、註冊確認密碼、密碼不符、忘記密碼模擬、登出、不保存輸入密碼 |
| 我的旅行 | 新增／編輯／刪除、重新整理保存、總覽、刪除後關聯清理 |
| 旅行封面 | PNG 上傳及 JPEG 壓縮、預覽、縮放、拖曳、位置重設、移除封面 |
| 收藏 | 新增／編輯／刪除、類別、用途、搜尋、卡片／列表、圖片上傳與放大、來源安全網址 |
| 購物清單 | 全部／單一旅行、搜尋、分類、已購切換與保存 |
| 行程 | CRUD、日期與時間排序、依旅行顯示 |
| 錢包 | CRUD、付款人、不同幣別、TWD 預算計算與其餘幣別不混加 |
| 旅行資訊 | CRUD、文字及備註顯示 |
| GPS | 同店合併、排除已購、數量、距離、精度／時效、跨頁提醒、冷卻／重新整理去重、背景暫停恢復、拒絕權限、店家及距離設定 |
| 備份 | 匯出、有效匯入、取消、畸形 JSON、錯誤欄位型別、重複 ID、超過 10MB、危險網址、數字欄位注入 |
| 錯誤復原 | 儲存額度不足時回滾、損壞資料保留並可原始匯出、阻止舊分頁覆蓋已更新資料 |
| 操作回饋 | 成功送出前端流程、離線保存、HTML 跳脫、匯出／刪除；網路請求攔截，不寄信 |
| 跨頁／顯示 | 七個主要頁面、瀏覽器上一頁／下一頁、手機無水平溢出、JS 錯誤檢查 |

沒有宣稱窮舉所有可能的操作排列、所有手機型號或全部瀏覽器版本。實體手機單指／雙指操作、真實 GPS 移動及鎖屏、原生 App、實際信箱收件仍需另外驗收。

## 已找出並修正

| 問題 | 驗證與修正 |
| --- | --- |
| 旅行名稱進入 inline onclick 的 JavaScript 字串 | 在隔離環境用無外連的 marker 重現腳本執行；改以 dataset 傳遞文字，名稱不再被解析為程式。加入回歸測試。 |
| 匯入只驗證集合及 ID，欄位型別與內容不足 | 對所有集合、數值範圍、文字長度、圖片、引用 ID、重複 ID、定位資料及距離做結構驗證；未知欄位不帶入。載入與保存也經驗證。 |
| 幣別直接進入 HTML 字串 | money() 對幣別進行 HTML 跳脫，備份同時限制為三碼大寫代號。 |
| 圖片來源可接受過多格式 | 限定 PNG／JPEG／WebP base64 或不含帳密的 HTTPS；拒絕 SVG、javascript 等來源。 |
| 損壞本機資料易導致當機或覆寫 | 顯示復原提示、禁止直接覆寫，保留原文並允許匯出，需有效匯入或明確重設才恢復保存。 |
| 多分頁舊資料覆蓋 | 保存前比對最後讀取的儲存快照；偵測另一分頁已更新則拒絕過期寫入並提示。這是衝突檢查，不是可跨程序原子交易的資料庫鎖。 |
| 外部資源來源資訊與基本瀏覽器限制 | 加入 no-referrer，禁止 base、object 及原生 form 導航。既有回饋 fetch 不受影響。 |

既有 Google Apps Script 回饋 script 逐字未改，SHA256：`775bfbb84c24791fb347930c0d0d11ee5198ffa99de0b929b9dbea8e86be8d74`。

## 多人及大量資料測試

本地隔離伺服器、真實 Chromium、獨立 browser context，同時開啟、各自新增商品、重新整理並確認只看見自己的測試資料。只向 localhost 送流量，外部資源攔截，不對公開網站或 Apps Script 進行壓測。

| 同時操作瀏覽器 | 成功／錯誤 | 整段操作 p95 |
| --- | --- | --- |
| 5 | 5／0 | 697 ms |
| 10 | 10／0 | 1,236 ms |
| 25 | 25／0 | 2,769 ms |

另外，1,000 件商品的購物頁渲染加搜尋在本測試環境為 72 ms（本機門檻 3,000 ms）。這些是單次環境測值，不是正式站 SLA、每秒請求量或承載上限。原始測量見 AUDIT-EVIDENCE.json；GitHub CI 另保存當次數值，會因執行環境改變。

**未測／無法據此推論：** GitHub Pages 的流量上限與 DDoS 防護、Google Apps Script 同時寄信與每日額度、共享 API／資料庫負載、真實帳號資料列權限、100／1,000 人正式同時使用。這些不能從瀏覽器本機資料隔離推論。

## 仍未解決的正式上線條件

| 項目 | 現況／後續需要 |
| --- | --- |
| 真實帳號與權限 | 尚無身分驗證後端。需正式 Auth、伺服器驗證、每使用者資料隔離／RLS、其他帳號拒絕存取測試。不能依賴前端隱藏按鈕或模擬登入。 |
| 本機資料保密 | localStorage 不是加密保險箱；同裝置／同瀏覽器使用者、成功執行的 XSS、惡意擴充套件等仍可能讀取。需依資料敏感度重新設計儲存與清除策略。 |
| Apps Script | 儲存庫未提供 doPost／寄信腳本及部署設定，無法檢查認證、來源限制、濫用／速率限制、收件權限與保留期限。公開端點本來就能被外部看到；不應把網址視為祕密。 |
| 完整 CSP 與安全標頭 | 本次是基本 meta 限制；現有大量 inline script／事件仍在，不等於完整 script-src 防護。正式版本需移除 inline 事件，設定嚴格 CSP、frame-ancestors 等伺服器安全標頭。 |
| 外部資源 | 仍有 Unsplash 圖片及 jsDelivr 字體，第三方可能收到 IP 與資源請求；no-referrer 不會隱藏 IP。可後續改為自託管素材。 |
| 後端容量與防濫用 | 需可控測試環境、實際目標人數、監控／錯誤率、速率限制與正式後端壓測。 |
| 獨立滲透測試 | 本輪是程式檢查與針對性前端回歸測試，不是獨立第三方滲透測試或隱私合規認證。 |

## 依賴與祕密檢查

本次 npm audit 回報 0 個已知套件弱點。常見私鑰、GitHub token、AWS access key、service_role 格式檢查均未命中；沒有第三方 script src。這些檢查有限，不能證明沒有未知弱點或所有祕密均不存在。Playwright 為開發測試依賴，網頁本身沒有執行 npm 後端。

## 自動化與報告

`npm test` 會跑來源檢查、原有 GPS／CRUD 測試、完整功能與前端安全測試。`npm run test:load` 單獨跑隔離負載測試。GitHub Actions 保留三種瀏覽器矩陣；僅電腦組在分支測試時執行 localhost 負載測試，不壓測已上線網站。HTML 報告整合 JSON 與截圖，保留 14 天。

## 官方參考

- OWASP 本機儲存風險：https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html
- OWASP XSS 編碼：https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- OWASP Session／Web Storage 競爭條件：https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- CSP 限制：https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP
- Referrer Policy：https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Referrer-Policy
