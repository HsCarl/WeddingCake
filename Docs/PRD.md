# 📋 WeddingCake — Product Requirements Document (PRD)

> **文件編號**: PRD-WC-001
> **版本**: v1.0
> **發起人**: Sam (PM-Orchestrator)
> **日期**: 2026-05-10
> **狀態**: 🟡 待 Carl 審閱確認

---

## 1. 專案概述

| 項目 | 內容 |
|------|------|
| **專案名稱** | WeddingCake — Carl & Minnie 電子喜帖 |
| **新郎** | 柯欣旻 (Carl) |
| **新娘** | 蔡煜甄 (Minnie) |
| **婚禮日期** | 2027/01/23 |
| **網站上線目標** | 2026/10/01 |
| **預估賓客數** | 200 人 |
| **語言支援** | 繁體中文 / English / 한국어 |
| **存取控制** | 公開（不需密碼保護） |

### 1.1 賓客輪廓

- 大學朋友（中文為主）
- 韓國朋友（韓文需求）
- 女方親友（中文為主）
- 部分賓客可能需要英文介面

### 1.2 核心目標

1. 提供婚禮資訊（日期、地點、交通）給所有賓客
2. 線上收集 RSVP 回覆並管理桌次
3. 提供禮金轉帳資訊，並在婚禮前一天自動/手動關閉
4. 婚禮當天提供共同相簿，讓賓客即時上傳照片
5. 提供後台管理介面，追蹤邀請發送狀態與回覆統計

---

## 2. 功能模組規格

### ✅ M1 — Hero 主視覺

| 項目 | 規格 |
|------|------|
| **倒數計時器** | 距離婚禮日期的即時倒數（天/時/分/秒） |
| **主視覺** | 支援大圖 + 影片背景 |
| **內容** | 新人姓名、婚禮日期、簡短標語 |
| **響應式** | Mobile-first，圖片需做 responsive sizing |

---

### ✅ M2 — 婚禮資訊 (Wedding Info)

| 項目 | 規格 |
|------|------|
| **儀式場次** | 單場宴客儀式 |
| **顯示資訊** | 日期、時間、地點名稱、地址 |
| **地圖** | 嵌入 Google Maps |
| **交通指引** | 停車資訊、大眾運輸路線 |

> 地點等具體資訊待 Carl 後續提供。

---

### ✅ M3 — RSVP 線上回覆

| 項目 | 規格 |
|------|------|
| **表單欄位** | 姓名、出席人數、餐飲偏好（葷/素）、聯絡方式、備註 |
| **桌次連動** | RSVP 確認後，管理員在後台分配桌次 |
| **查詢功能** | 賓客可依姓名查詢已分配的桌次 |
| **資料儲存** | DynamoDB |
| **通知觸發** | 每筆 RSVP 提交後觸發 LINE + Email 通知給管理員 |

#### 資料模型（初步）

```
RSVP_Table
├── PK: GUEST#<guest_id>
├── name: string
├── attendance: "attending" | "not_attending"
├── party_size: number
├── dietary: "meat" | "vegetarian"
├── contact: string
├── note: string
├── table_number: number | null   // admin assigns later
├── invited_at: ISO8601 | null    // when invitation was sent
├── invited_via: "line" | "email" | "in_person" | null
├── responded_at: ISO8601 | null
├── created_at: ISO8601
└── updated_at: ISO8601
```

---

### ✅ M4 — 共同相簿 (Shared Gallery)

| 項目 | 規格 |
|------|------|
| **上傳方式** | 賓客透過前台直接上傳（多張） |
| **存儲** | S3 Bucket（Pre-signed URL 上傳） |
| **Metadata** | DynamoDB 記錄上傳者、時間戳、檔案 key |
| **容量限制** | 不限制總量 |
| **顯示** | 瀑布流 / 網格展示，支援 Lightbox 預覽 |
| **併發設計** | 婚禮當天預估 2000 人同時使用，需考量 S3 上傳吞吐 |

> S3 Pre-signed URL 的產生由 Lambda 處理，需設定合理的 URL 過期時間（建議 15 分鐘）與檔案大小限制（單檔 10MB）。圖片上傳後可透過 Lambda 觸發縮圖產生（用於前台瀏覽列表）。

---

### ✅ M5 — 禮金資訊 (Gift / Registry)

| 項目 | 規格 |
|------|------|
| **顯示內容** | 銀行帳號、戶名 |
| **轉帳回報** | 賓客可在頁面上留言已轉帳資訊（金額、帳號末五碼、時間）以便核對 |
| **自動關閉** | 指定日期時間後自動隱藏轉帳頁面（婚禮前一天） |
| **手動關閉** | 後台亦可手動切換開關 |
| **關閉邏輯** | `auto_close_at` 欄位 + `manual_override` flag，取兩者之先 |

---

### ✅ M6 — 桌次查詢 (Seating Chart)

| 項目 | 規格 |
|------|------|
| **資料來源** | 與 RSVP 連動：賓客 RSVP 確認後，管理員在後台分配桌次 |
| **查詢方式** | 前台輸入姓名 → 顯示桌次號碼 |
| **管理方式** | 後台管理介面批次編輯桌次 |

---

### ✅ M7 — 菜單展示 (Menu Display)

| 項目 | 規格 |
|------|------|
| **內容** | 完整菜單（區分葷食/素食版本） |
| **格式** | 圖文並茂，支援中英韓三語 |

---

### ✅ M8 — 婚紗輪播 (Photo Carousel)

| 項目 | 規格 |
|------|------|
| **形式** | 自動 + 手動輪播 |
| **圖片來源** | 管理員上傳至 S3 |
| **優化** | Lazy loading + WebP 格式 + responsive srcset |

---

### ✅ M9 — 管理後台 (Admin Dashboard)

| 功能區塊 | 說明 |
|----------|------|
| **RSVP 統計** | 總邀請數、已回覆數、出席/不出席比例、葷素統計 |
| **邀請追蹤** | 記錄每位賓客的邀請發送狀態（已發送/未發送）、發送管道、回覆狀態 |
| **桌次管理** | 批次分配/調整賓客桌次 |
| **禮金管理** | 查看轉帳回報、手動切換轉帳頁面開關 |
| **相簿管理** | 查看/刪除上傳照片 |
| **通知設定** | 配置 LINE Webhook URL 與 Email 收件地址 |
| **婚紗管理** | 上傳/排序/刪除輪播用婚紗照 |

#### 邀請追蹤資料模型

```
Invitation_Tracking (merged into Single Table)
├── PK: GUEST#<guest_id>
├── SK: INVITE#<guest_id>
├── guest_name: string
├── invitation_sent: boolean
├── sent_at: ISO8601 | null
├── sent_via: "line" | "email" | "in_person" | null
├── response_status: "pending" | "responded"
├── response_content: string | null   // RSVP response summary
└── notes: string | null              // admin notes
```

> 此表與 RSVP_Table 合併為同一張 DynamoDB 表，使用不同的 SK pattern 區分。最終 schema 由 Holly 在實作階段設計 Single Table Design。

---

### ❌ 排除的功能模組

| 模組 | 原因 |
|------|------|
| 我們的故事 (Our Story) | Carl 明確排除 |
| 賓客留言板 (Guestbook) | Carl 明確排除 |
| 婚禮派對 (Wedding Party) | Carl 明確排除 |
| 常見問答 (FAQ) | Carl 明確排除 |
| 密碼保護 | Carl 明確排除 |

---

## 3. 技術架構

### 3.1 架構總覽

```
┌─────────────────────────────────────────────────────┐
│                   CloudFront (CDN)                   │
│              Custom Domain + HTTPS                   │
├─────────────────────┬───────────────────────────────┤
│                     │                               │
│   S3 (Static)       │   API Gateway (REST)          │
│   Next.js Export    │       │                       │
│   + Photos          │   Lambda Functions            │
│   + Wedding Photos  │   ├── RSVP Handler            │
│                     │   ├── Gallery Handler          │
│                     │   ├── Gift Handler             │
│                     │   ├── Seating Handler          │
│                     │   ├── Notification Handler     │
│                     │   └── Admin Handler            │
│                     │       │                       │
│                     │   DynamoDB                     │
│                     │   (Single Table Design)        │
├─────────────────────┴───────────────────────────────┤
│              SES (Email)  +  LINE Messaging API      │
└─────────────────────────────────────────────────────┘
```

### 3.2 技術堆疊

| 層級 | 技術 |
|------|------|
| **前端框架** | Next.js (App Router) + TypeScript |
| **樣式** | Tailwind CSS |
| **國際化 (i18n)** | next-intl（中/英/韓） |
| **靜態輸出** | `next export` → S3 |
| **IaC** | AWS CDK (TypeScript) |
| **API** | API Gateway + Lambda (TypeScript) |
| **資料庫** | DynamoDB (Single Table Design) |
| **檔案存儲** | S3 (照片上傳) |
| **CDN** | CloudFront |
| **通知** | AWS SES (Email) + LINE Messaging API |
| **DNS** | Route 53 |

### 3.3 併發與擴展考量

| 場景 | 策略 |
|------|------|
| **平時** | 接近 0 流量，Lambda 冷啟動可接受 |
| **婚禮當天** | 預估 2000 並發，S3 原生支援高吞吐；Lambda 需設定合理的 Provisioned Concurrency |
| **照片上傳尖峰** | Pre-signed URL 直傳 S3，不經 Lambda 傳輸，減少 Lambda 併發壓力 |

---

## 4. 設計規範

### 4.1 風格定義

| 項目 | 規格 |
|------|------|
| **整體風格** | 手繪插畫風 / 溫馨生活感 |
| **設計核心** | Q 版自繪人物像 + 手稿線條 |
| **視覺調性** | 親切、具個人特色、非正式 |

### 4.2 色彩系統

| Token | 色碼 | 用途 |
|-------|------|------|
| `--color-primary` | `#E8C4C4` | 莫蘭迪粉（封面主色） |
| `--color-secondary` | `#F5F0EB` | 米白色（內頁底色） |
| `--color-accent-blue` | `#A8B5C8` | 灰藍色（插畫點綴） |
| `--color-accent-gray` | `#D4D0CC` | 淺灰色（輔助點綴） |
| `--color-text` | `#4A4A4A` | 主文字色 |
| `--color-text-light` | `#8A8A8A` | 次要文字 |

> 以上色碼為初步建議，實際開發階段由 Megan 微調以確保可讀性與對比度符合 WCAG AA 標準。

### 4.3 字體

| 用途 | 字體建議 |
|------|----------|
| **英文標題/人名** | Caveat / Dancing Script（手寫風） |
| **中文標題** | 思源柔黑體 / 瀬戸フォント |
| **韓文** | Nanum Pen Script（手寫風） |
| **內文** | Noto Sans TC / Noto Sans KR / Inter |

### 4.4 動畫

- 極少動畫，保持乾淨
- 僅在頁面切換與區塊首次出現時使用 fade-in
- 禁止 parallax、粒子效果等重型動畫

---

## 5. QR Code 需求

| 項目 | 規格 |
|------|------|
| **內容** | 網站自訂 domain URL |
| **輸出** | 高解析度 PNG/SVG，適合印刷 |
| **用途** | 現場印出供賓客掃描 |
| **實作** | 前端或腳本產生，嵌入網站 logo（選配） |

---

## 6. 時程規劃

| 階段 | 時間 | 內容 | 負責 Agent |
|------|------|------|------------|
| **Phase 1** | 2026/05 W3-W4 | 專案初始化 + 設計系統 + i18n 架構 | Megan + CM |
| **Phase 2** | 2026/06 W1-W2 | Hero + 婚禮資訊 + 婚紗輪播 + 菜單頁面 | Megan |
| **Phase 3** | 2026/06 W3-W4 | RSVP + 桌次查詢 + 禮金模組 | Holly + Megan |
| **Phase 4** | 2026/07 W1-W3 | 共同相簿 + 管理後台 | Holly + Megan |
| **Phase 5** | 2026/07 W4 - 08 W2 | 通知機制 (LINE + Email) + CDK 部署 | Holly + CM |
| **Phase 6** | 2026/08 W3 - 09 W4 | E2E 測試 + 壓力測試 + QR Code 產出 + UAT | Jenny + Sam |
| **上線** | 2026/10/01 | Production 部署 | CM |

---

## 7. 開放問題（待後續確認）

| # | 問題 | 狀態 |
|---|------|------|
| 1 | 婚禮地點詳細資訊（名稱、地址） | ⬜ 待提供 |
| 2 | 禮金銀行帳號/戶名 | ⬜ 待提供 |
| 3 | Q 版人物插畫素材（Carl 自行提供或需產生？） | ⬜ 待確認 |
| 4 | 婚紗照片素材 | ⬜ 待提供 |
| 5 | 菜單內容（葷/素） | ⬜ 待提供 |
| 6 | 自訂 domain 名稱（例如 carlandminnie.com） | ⬜ 待確認 |
| 7 | LINE Messaging API Channel 是否已有？ | ⬜ 待確認 |
| 8 | 管理後台是否需要登入驗證（Cognito / 簡易密碼）？ | ⬜ 待確認 |

---

## 📋 任務追蹤

請參閱 [SAM_TODO.md](./SAM_TODO.md) 以獲取最新進度與待辦清單。

