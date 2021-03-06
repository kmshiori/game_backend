# API 服務

> 應用程式介面除了提供基礎的資料存取介面之外，通常也用來實現伺服器端的邏輯。在目前新興的服務中，更會包含了資料情境連動的能力（如: Database Trigger, Schedule/Job, Web Hook）

在 Parse 與 Firebase 的平台中，分別有 Cloud Code 與 Cloud Functions 的工具，用來擴充後端伺服器能力。我們接著把他們從「部署與偵錯」、「支援功能」...等等各方面進行簡單的比較。並會在後來的範例中，展示一個簡單的 Parse Cloud Code 部署方法。

## 部署與偵錯 <a id="deploy"></a>

| Cloud Code | Cloud Functions |
| :--- | :--- |
| 需手動進行驗證、部署，並重啟 Parse Service 來作動 Cloud Code | 以 firebase-tools 完成驗證、部署、生效 |
| 可在本機端建立臨時的 Parse Service 環境，可透過 Dashboard 的 API Console 或是透過外部工具模擬 Trigger 進行測試 | 可透過 Cloud Functions shell 模擬各種 Trigger 與資料，或是透過 firebase serve 指令來模擬 HTTPS functions |

## 支援功能 <a id="function"></a>

|  | Cloud Code | Cloud Functions |
| :--- | :--- | :--- |
| Data Trigger | BeforeSave AfterSave BeforeDelete AfterDelete | onCreate onUpdate onDelete onWrite |
| Find Trigger | BeforeFind | X |
| Authentication Trigger | 可透過監聽 User Class 來達成 | onCreate onDelete |
| Analytics Trigger | X | Google Analytics event |
| Crashlytics Trigger | X | onNewDetected onRegressed onVelocityAlert |
| Storage Trigger | X | upload, update, delete 檔案或是資料夾 |
| Pub/Sub Trigger | X | onPublish |
| HTTP Trigger | O | O |
| External | Webhook | X |

