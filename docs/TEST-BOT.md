# 旅購自動測試機器人

## 會自動做什麼

GitHub Actions 工作流程名稱：Travel Studio Test Bot。

- main、feature/**、test/** 分支有程式更新時，測試該分支。
- 對 main 提出 Pull Request 時，測試變更版本。
- GitHub Pages 的 pages build and deployment 成功後，測試已上線的 https://linay-lab.github.io/travel-studio-preview/。
- 也可從 Actions → Travel Studio Test Bot → Run workflow，選 checkout（分支）或 published（上線網站）手動執行。

每次使用獨立瀏覽器與本機儲存，不會碰使用者既有的旅行資料。Google Apps Script 回饋請求會在瀏覽器攔截，不寄出測試信。流程沒有發佈、部署、修改網站程式或自動修復權限。

## 測試內容

電腦 Chromium、手機尺寸 Chromium、手機尺寸 WebKit 三組分開執行。測試包含：

- 旅行、商品、行程、支出、旅行資訊：表單新增、重新整理保存、編輯、刪除。
- 同一店家未購商品整合、排除已購與非商品收藏、顯示數量。
- GPS 距離、定位精度、過期位置、跨頁提醒、去重與重新整理、冷卻與離開重入。
- 商品購買狀態、店家設定、提醒距離、拒絕權限、單一監聽、背景暫停與前景恢復。
- 回饋表單送出流程（請求已攔截）、原有七個頁面載入、頁面 JavaScript 錯誤、手機水平溢出。
- Chromium 額外使用瀏覽器 Geolocation 模擬座標測試整合。

WebKit 是 Safari 引擎的測試環境，不等於實體 iPhone 驗證。模擬定位與背景狀態也不能替代真機街道移動、鎖屏與原生 App 的驗收。

## 查看結果

到儲存庫 Actions，開啟 Travel Studio Test Bot 的執行紀錄：

- 綠色勾勾：該組測試通過；紅色叉叉：有測試或執行環境問題。
- 每組工作有中文摘要；失敗會記錄錯誤與可取得的畫面截圖。
- 在 Artifacts 下載 browser-report 附件，解壓後開啟 index.html，查看報告和截圖。附件保留 14 天。
- 不會自動私訊、寄信或發送外部通知；GitHub 自己的通知取決於帳號設定。

## 本機執行

```sh
npm ci
npx playwright install --with-deps chromium webkit
npm test
```

報告預設在 test-results/，不會改動舊的 tests/results.json。可使用 TEST_BROWSER 選 desktop-chromium、mobile-chromium 或 mobile-webkit；TEST_TARGET_URL 只允許上面的測試站；TEST_OUTPUT_DIR 可指定報告位置。

本開發環境已通過電腦與手機尺寸 Chromium 的分支測試；本機 WebKit 安裝仍受下載限制，需以 GitHub Actions 的實際執行結果為準。當網站無法連線時會回報失敗，不能視為產品功能已失效，需依錯誤分辨網路、依賴與程式問題。

## 參考官方文件

- https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows
- https://playwright.dev/docs/ci
