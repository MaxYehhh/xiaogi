# XiaoGi 專案狀態

最後更新：2026-03-21

---

## 快速定位

**現在階段：** Phase 1 完成 — 基礎設施 + 第一篇 IG 貼文已成功發布

**上次執行：** 2026-03-20
- 場景：A類 — 主持董事會
- 狀態：✅ 已發布（2026-03-21 12:50 UTC+8）
- IG 貼文：https://www.instagram.com/p/DWIj0lYlCnL/

**下一步（優先序）：**
1. 繼續每日執行累積資料（`/run`）
2. 測試 REVISE CAPTION 流程
3. 設定 FAL_API_KEY → 測試 REVISE IMAGE 流程
4. 確認 STEP 8 analytics endpoint 是否存在
5. 待開發：Telegram 發布通知

---

## 已驗證可用（勿重複除錯）

| 元件 | 狀態 | 備注 |
|------|------|------|
| Ideogram v3 圖片生成 | ✅ | multipart/form-data，Api-Key header |
| Postiz 媒體上傳 | ✅ | `/public/v1/upload` |
| Postiz 排程發布 | ✅ | `/public/v1/posts`，value 陣列格式 |
| Cloudflare R2 儲存 | ✅ | 圖片 URL 公開存取，Instagram 可抓取 |
| Instagram 實際發布 | ✅ | state: PUBLISHED 確認 |
| Postiz 狀態查詢 | ✅ | `GET /public/v1/posts?startDate=...&endDate=...` |
| Docker 全容器啟動 | ✅ | 5 個 services 正常運行 |

---

## 待完成事項

| 項目 | 優先度 | 說明 |
|------|--------|------|
| REVISE CAPTION 流程 | 高 | 未測試過，每次都可能用到 |
| FAL_API_KEY 設定 | 中 | `.env` 目前為空，REVISE IMAGE 無法執行 |
| REVISE IMAGE (FLUX Kontext) 測試 | 中 | 需先完成上一項 |
| STEP 8 成效追蹤 | 中 | 等 2026-03-20 的貼文滿 48 小時後執行，需確認 Postiz analytics endpoint |
| performance-log.md 資料累積 | 低 | 需 5+ 筆後才有 skill 自動更新效果 |
| Telegram 發布通知 | 低 | 完整架構：系統 crontab + Telegram Bot，詳見 memory |

---

## 已知問題（暫時接受）

| 問題 | 現狀 | 解決方向 |
|------|------|---------|
| 圖片角色一致性 | Ideogram character_reference 效果不穩，每次生成的小吉外貌略有差異 | 待 FAL_API_KEY 設定後，用 FLUX Kontext img2img 修正 |
| publish-result.md 狀態未更新 | 檔案記錄「SCHEDULED」，實際已 PUBLISHED | 低優先，STEP 8 執行時一併更新 |

---

## 關鍵配置快查

| 項目 | 值 |
|------|-----|
| Postiz URL | http://localhost:4007 |
| Instagram Integration ID | `cmmyg00570001o7gxxr7xc0dy` |
| Cloudflare R2 Bucket | `xiaogi-uploads` |
| Cloudflare Bucket URL | `https://pub-a5d5781a73cf4678a2e8bf67c2371648.r2.dev` |
| .env 位置 | `$XIAOGI_BASE_DIR/.env` |
| Docker compose 位置 | `$XIAOGI_BASE_DIR/docker/docker-compose.yml` |

---

## 重要檔案地圖

```
CLAUDE.md                   ← Agent 執行規則（每次 /run 的完整 SOP）
docs/project-status.md      ← 本文件：進度快照
docs/setup-sop.md           ← 從零重建系統的 SOP
docs/docker-setup.md        ← Docker / Postiz 架設細節

knowledge/character.md      ← 小吉角色設定（外型、人設、禁忌）
knowledge/viral-formula.md  ← 場景庫 + 輪換規則

skills/image-generation.md  ← Ideogram prompt 公式（可動態更新）
skills/caption-writing.md   ← 文案公式 + hook 輪換（可動態更新）
skills/performance-log.md   ← 成效追蹤日誌（每 48h 填入一筆）

docker/docker-compose.yml   ← 5 個服務的完整配置
outputs/YYYY-MM-DD/         ← 每日執行產出（gitignore）
references/小吉.png         ← 角色參考主圖
```
