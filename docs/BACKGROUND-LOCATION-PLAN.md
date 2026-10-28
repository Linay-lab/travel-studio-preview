# iOS / Android 背景購物提醒規劃

本分支僅實作網頁前景 GPS。以下為後續原生 App 設計，尚未實作或真機驗收。

## 共用資料及流程

以 shoppingPlaces 的穩定 id 對應確切分店，商品使用 shoppingPlaceId。進入圍欄時先從本機資料庫讀取最新商品，只取 type=商品、bought=false；按分店合併後發出一則本機通知。已買完的分店移除監控與尚未送出的通知。通知內容與去重紀錄應存於原生 SQLite 等持久層，避免依賴 WebView 啟動。網頁與原生監控需互斥，防止雙重通知。

去重採分店 id、最後通知時間、進出狀態與位置設定版本；跨出圍欄後且間隔滿 30 分鐘才重新提醒。通知點擊導向該分店的待購清單。購買狀態、店家位置、距離、刪除、匯入資料變更時同步更新圍欄。

## iOS

使用 Core Location 的 CLMonitor / 地理圍欄機制，搭配 UserNotifications 本機通知。依最低支援版本決定 CLMonitor 或 CLLocationManager region monitoring。由使用者啟用背景提醒後，逐步說明並取得所需的定位與通知授權；設定對應用途說明與背景能力。每次背景喚醒重建監控與資料庫連線，並重新檢查待購狀態。

單一 App 監控數量有限，優先註冊目前旅行／附近的最多 20 個店家，使用者移動或回到 App 時更新。鎖定及一般背景情境可由系統監控；系統終止、使用者強制結束、背景重新整理設定、重開機尚未解鎖等情況須分開真機驗收，不能承諾「任何關閉方式都一定會通知」。不同 iOS 版本與 API 的重新喚醒行為亦須個別驗證。

## Android

使用 Google Play services GeofencingClient + PendingIntent + BroadcastReceiver，搭配 NotificationChannel 及本機通知。取得精確定位、所需背景定位與對應 Android 版本的通知權限。使用圍欄節省耗電，預設不長期啟動高耗電的 GPS 前景服務。

每 App／裝置使用者最多 100 個圍欄；依旅行與位置排序更新。背景事件可能延遲數分鐘，介面需清楚說明。開機後重建註冊、資料變更同步調整，強制停止後要求使用者重新開啟恢復監控。另驗證省電模式、廠商背景限制、Google Play services 不可用與權限撤回。

## 後續交付與驗收

1. 決定原生框架（可使用 Capacitor 保留現有介面，但定位與通知需原生橋接）。
2. 建立 SQLite 資料層、網頁資料遷移與原生提醒引擎。
3. iPhone、Android 真機逐項測試：前景、背景、鎖屏、系統回收、強制結束、重開機、無網路、低電量、權限撤回、已買完、不同分店及範圍邊緣漂移。
4. 量測耗電、延遲與漏提醒，完成商店權限說明後才進行發佈審查。

## 官方資料（查核日期 2026-10-28）

- Apple Core Location：https://developer.apple.com/documentation/corelocation
- Apple 區域監控：https://developer.apple.com/documentation/corelocation/monitoring-the-user-s-proximity-to-geographic-regions
- Apple 傳統區域監控限制：https://developer.apple.com/documentation/corelocation/cllocationmanager/startmonitoring(for:)
- Android 圍欄與權限：https://developer.android.com/develop/sensors-and-location/location/geofencing
- Android 背景定位：https://developer.android.com/develop/sensors-and-location/location/background
- GeofencingClient 生命週期：https://developers.google.com/android/reference/com/google/android/gms/location/GeofencingClient
