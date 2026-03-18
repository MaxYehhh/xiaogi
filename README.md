# XiaoGi — 每日內容 Agent

> 一隻完全不知道自己有多小的吉娃娃。全自動 Instagram + TikTok 內容生產系統。

---

## 核心概念

**病毒公式：**
```
吉娃娃 + 與其體型身份不符的場景 = 視覺衝擊
```

XiaoGi（小吉）是一隻蘋果頭吉娃娃，在任何場合都以十足的主管自信行事。幽默感來自他的**認真**，不是搞笑。他永遠是主控方，永遠不扮演弱者。

---

## 使用方式

在 Claude Code 環境中執行：

```
/run
```

系統全自動執行 Step 1–6，最終在終端機印出當日貼文預覽。

**人類只需要 30 秒：**

```
APPROVE              → 確認排程發布（明日 09:00）
REVISE CAPTION [說明] → 修改文案後重新確認
REVISE IMAGE [說明]  → 用 FLUX Kontext 修正圖片
SKIP                 → 跳過今天
```

---

## 架構

```
xiaogi/
├── CLAUDE.md                  # Agent 執行規則（每日流程、錯誤處理、Cold Start）
├── .env                       # API 金鑰（不進 git）
│
├── knowledge/                 # 靜態知識庫（人工維護）
│   ├── character.md           # 角色設定書：XiaoGi 外型、視覺規格、Prompt 禁忌
│   └── viral-formula.md       # 病毒公式：四類場景庫、輪換規則、有效/無效要素
│
├── skills/                    # 動態技能檔（agent 隨執行自動更新）
│   ├── image-generation.md    # Ideogram v3 Prompt 公式、品質清單、FLUX 備援
│   ├── caption-writing.md     # 文案公式、Hook 輪換表、Hashtag 策略
│   └── performance-log.md     # 每篇貼文 48h 後的成效記錄與聚合洞察
│
└── outputs/                   # 每日執行產出（自動建立）
    └── YYYY-MM-DD/
        ├── scene.md           # 今日場景選擇與輪換確認
        ├── prompt.md          # Ideogram prompt（含失敗記錄）
        ├── image.png          # 生成圖片
        ├── caption.md         # Instagram + TikTok 文案
        └── publish-result.md  # Postiz API 回應 / SKIPPED 記錄
```

---

## 每日執行流程

| Step | 內容 | 工具 |
|------|------|------|
| 1 | 讀取 character.md、viral-formula.md、performance-log.md | — |
| 2 | 依輪換規則選擇今日場景，建立 `outputs/YYYY-MM-DD/` | — |
| 3 | 建構 prompt → 呼叫 Ideogram v3 API → 品質檢查 → 下載圖片 | Ideogram v3 |
| 4 | 依星期幾 / 歷史成效選 hook 類型，撰寫 IG + TikTok 文案 | — |
| 5 | 上傳圖片至 Postiz，取得 `MEDIA_ID` | Postiz Upload API |
| 6 | 印出預覽，等待人類輸入 | — |
| 7 | APPROVE → 建立排程；REVISE → 修正後回到 Step 6；SKIP → 記錄 | Postiz Posts API |
| 8 | *(48h 後獨立觸發)* 讀取成效 → 寫入 performance-log.md → 觸發 skill 更新 | Postiz Analytics |

---

## 場景分類

| 類別 | 定義 | 預期表現 |
|------|------|---------|
| **A類 — 權威人物** | XiaoGi 是負責人（董事會、工地監工、法官席） | 最高 |
| **B類 — 專業服務** | XiaoGi 執行技術工作（餐廳服務、手術室、飛機駕駛艙） | 中高 |
| **C類 — 高端休閒** | XiaoGi 是資深行家（賭場、高爾夫、雪茄室） | 中等 |
| **D類 — 日常官僚** | XiaoGi 面對成年人生活（帳單、候診室、ATM） | 高共鳴黑馬 |

輪換規則：不能連續兩天同分類；14 天內不重複同場景。

---

## 自我強化迴圈

系統隨時間學習，每 5 筆成效資料後自動更新 skill files：

| 觸發條件 | 行動 |
|---------|------|
| 某場景分類持續高於平均 | viral-formula.md 標記「proven high-performer」，提高優先級 |
| 某 hook 類型 3/5 勝出 | caption-writing.md 設為 default |
| 某場景分類持續低於平均 5 次 | viral-formula.md 加「AVOID」標記 |
| 某視覺元素與高收藏率相關 | image-generation.md 加入固定規則 |

---

## 外部服務

| 服務 | 用途 | 環境變數 |
|------|------|---------|
| [Ideogram v3](https://ideogram.ai) | 圖片生成（主力） | `IDEOGRAM_API_KEY` |
| [fal.ai FLUX Kontext](https://fal.ai) | 圖片細節修正（備援） | `FAL_API_KEY` |
| [Postiz](https://postiz.com) | 社群媒體排程發布 | `POSTIZ_API_KEY` |

每日 API 成本上限：約 **$0.28 USD**（3 次 Ideogram + 1 次 Kontext）。

---

## 首次執行（Cold Start）

若 `.env` 中 `CHARACTER_REFERENCE_URL` 為空：

1. 系統自動生成 3 張基礎圖片（`cold-start-1.png` ~ `cold-start-3.png`）
2. 你選擇最符合 XiaoGi 人設的一張
3. 上傳到永久 URL（GitHub raw、Cloudinary 等）
4. 填入 `.env` 的 `CHARACTER_REFERENCE_URL`
5. 系統鎖定 style_code，後續所有生成保持角色一致性
6. 繼續正常每日流程

---

## 環境設定

複製 `.env.example`（若有）或手動建立 `.env`：

```bash
IDEOGRAM_API_KEY=...
FAL_API_KEY=...
POSTIZ_API_KEY=...
POSTIZ_INSTAGRAM_INTEGRATION_ID=...
POSTIZ_TIKTOK_INTEGRATION_ID=...
CHARACTER_REFERENCE_URL=...
```
