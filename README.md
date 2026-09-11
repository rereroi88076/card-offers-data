# card-offers-data — 信用卡優惠資料

[CardCompass](https://github.com/rereroi88076/CardCompass) App 的優惠資料來源。App 啟動時抓取 `offers.json` 並快取，因此**更新優惠只要改這個 repo，不用重新打包 App**。所有異動都有 Git 紀錄可回溯，配合 `source_url` / `verified_at` 欄位維持資料可信度。

## offers.json 結構

### 頂層

| 欄位 | 說明 |
|---|---|
| `schema_version` | 結構版本，App 端據此判斷相容性 |
| `updated_at` | 最後更新日（YYYY-MM-DD） |
| `cards` | 卡片與其優惠規則 |
| `merchants` | 店家主檔與別名（搜尋比對用） |
| `categories` | 消費類別定義 |

### cards[]

| 欄位 | 說明 |
|---|---|
| `id` | 唯一識別碼，kebab-case，如 `esun-ubear` |
| `bank` / `name` / `network` | 銀行、卡名、卡別（visa/mastercard/jcb） |
| `base_reward` | 基礎回饋：`{ type, rate, note }`，`rate` 為小數（1% = 0.01） |
| `rules` | 加碼優惠規則陣列，見下 |

### cards[].rules[]

| 欄位 | 說明 |
|---|---|
| `id` | 規則唯一識別碼，建議帶期別，如 `esun-ubear-online-2026q3` |
| `title` | 顯示名稱 |
| `channels` | 適用範圍：`merchants`（店家 id 陣列）與/或 `categories`（類別 id 陣列） |
| `reward` | `{ type: "cashback"\|"points", rate, includes_base }`；`includes_base: true` 表示 rate 已含基礎回饋（不可再疊加） |
| `cap` | 上限：`{ amount, period: "monthly"\|"campaign", basis: "reward"\|"spending" }`，`basis: "reward"` 指回饋金額上限；無上限則省略 |
| `valid_from` / `valid_to` | 活動期間（含當日） |
| `conditions` | 附加條件文字陣列（需登錄、限行動支付等），App 顯示為提醒 |
| `source_url` | 優惠出處（銀行官網活動頁） |
| `verified_at` | 最後人工核對日期 |

### merchants[]

| 欄位 | 說明 |
|---|---|
| `id` | 唯一識別碼 |
| `name` | 正式名稱 |
| `aliases` | 別名/關鍵字陣列，搜尋時模糊比對 |
| `categories` | 所屬類別 id 陣列；規則以 `categories` 指定範圍時據此匹配 |

## 維護流程

1. 從 `develop` 開 `feature/<異動內容>` 分支修改
2. 改完確認 JSON 格式合法（`python3 -m json.tool offers.json`）
3. 合回 `develop`，確認無誤後合進 `main`（App 抓 `main` 上的檔案）
4. 更新頂層 `updated_at` 與異動規則的 `verified_at`
