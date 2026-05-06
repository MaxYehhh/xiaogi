# 從零啟動 SOP

本文記錄在全新環境重建 XiaoGi 自動化內容系統的完整步驟，以及所有踩過的坑。
按順序執行，不要跳步驟。

---

## 前置條件

### 必要帳號
- [ ] **Cloudflare 帳號**（免費）— R2 Object Storage，圖片公開存取用
- [ ] **Facebook Developer 帳號**（免費）— 連接 Instagram
- [ ] **Ideogram 帳號**（付費）— 圖片生成，約 $0.08/張

### 必要工具
- [ ] **Docker Desktop for macOS**（見 `docs/docker-setup.md`）
- [ ] **Claude Code CLI**（執行 agent 主流程）

### 選備帳號（非必要，但影響功能完整性）
- [ ] **fal.ai 帳號** — REVISE IMAGE / FLUX Kontext 圖片修正備援

---

## Phase 1：Docker + Postiz 架設

**完整步驟見 `docs/docker-setup.md`**，以下僅列關鍵指令：

```bash
# 啟動所有容器
cd /path/to/xiaogi/docker
docker compose up -d

# 確認全部正常
docker compose ps

# 如果 Postiz 頁面有畫面但 API 502
docker exec postiz pm2 restart backend
```

確認可存取：http://localhost:4007

---

## Phase 2：Cloudflare R2 設定

Postiz 上傳的圖片需要公開 URL（Instagram API 從外部抓取）。localhost URL 不可行，必須用 R2。

### 步驟

1. **登入 Cloudflare → R2 Object Storage → 建立 Bucket**
   - Bucket 名稱：`xiaogi-uploads`
   - 建立後 → Settings → Public Access → **Allow Access**（設為公開）

2. **建立 R2 API Token**
   - Cloudflare Dashboard → My Profile → API Tokens → Create Token
   - 使用 "Create Custom Token"
   - 權限：**Object Read & Write**（重要：不是 Admin Read Only）
   - Bucket：限定 `xiaogi-uploads`
   - 取得：Access Key ID、Secret Access Key

3. **取得 Account ID**
   - Cloudflare Dashboard 右側欄（Account Home 頁面）

4. **填入 `docker/docker-compose.yml`**（`postiz` service 的環境變數）：
   ```yaml
   STORAGE_PROVIDER: "cloudflare"
   CLOUDFLARE_ACCOUNT_ID: "你的 Account ID"
   CLOUDFLARE_ACCESS_KEY: "你的 Access Key ID"
   CLOUDFLARE_SECRET_ACCESS_KEY: "你的 Secret Access Key"
   CLOUDFLARE_BUCKETNAME: "xiaogi-uploads"
   CLOUDFLARE_REGION: "auto"
   CLOUDFLARE_BUCKET_URL: "https://[public-bucket-url].r2.dev"
   ```

5. **重啟 Postiz**：
   ```bash
   docker compose down && docker compose up -d
   ```

6. **驗證**：上傳一張圖，確認回應中的 URL 是 `https://...r2.dev/...` 而非 `http://localhost:4007/...`

---

## Phase 3：API 帳號設定（`.env`）

複製 `.env.example` 為 `.env`，填入以下值：

```bash
XIAOGI_BASE_DIR=/Users/你的名字/projects/xiaogi   # 各機器自行調整

IDEOGRAM_API_KEY=           # ideogram.ai → Settings → API Keys
FAL_API_KEY=                # fal.ai → Dashboard → API Keys（選備）

POSTIZ_BASE_URL=http://localhost:4007
POSTIZ_API_KEY=             # 見下方 Phase 4
POSTIZ_INSTAGRAM_INTEGRATION_ID=   # 見下方 Phase 4

CHARACTER_REFERENCE_URL=    # 首次執行後填入（小吉圖片的永久 URL）
```

---

## Phase 4：Postiz 設定 + Instagram 連接

**完整步驟見 `docs/docker-setup.md`**，重點：

1. 登入 http://localhost:4007 → Settings > API Keys → 建立 Key → 填入 `.env`
2. Settings > Integrations → Connect Instagram → 取 Integration ID → 填入 `.env`

> Facebook App 需在 developers.facebook.com 加入 redirect URI：
> `http://localhost:4007/integrations/social/instagram`

---

## Phase 5：第一次執行

```bash
cd /Users/你的名字/projects/xiaogi
claude  # 啟動 Claude Code
```

然後輸入 `/run`，全程跟著流程走。

**如果 `CHARACTER_REFERENCE_URL` 為空**，agent 會自動執行 Cold Start 協議，生成 3 張初始圖讓你選。

---

## 踩坑紀錄

這是本文件最重要的部分。以下所有問題都在 2026-03-20 首次執行時實際踩到並修正。

---

### Ideogram API 格式

**坑：** 舊文件用錯了 header 和 endpoint。

| 項目 | 錯誤 | 正確 |
|------|------|------|
| Header | `Authorization: Bearer KEY` | `Api-Key: KEY` |
| Endpoint | `https://api.ideogram.ai/generate` | `https://api.ideogram.ai/v1/ideogram-v3/generate` |
| Request format | JSON body | `multipart/form-data` |
| Aspect ratio | `ASPECT_1_1` | `1x1` |
| 角色參考 | URL（`reference_image_urls`） | File upload（`-F "character_reference_images=@path"`） |

正確的呼叫方式：
```bash
curl -X POST https://api.ideogram.ai/v1/ideogram-v3/generate \
  -H "Api-Key: $IDEOGRAM_API_KEY" \
  -F "prompt=..." \
  -F "style_type=REALISTIC" \
  -F "rendering_speed=QUALITY" \
  -F "aspect_ratio=1x1" \
  -F "character_reference_images=@$XIAOGI_BASE_DIR/references/小吉.png"
```

---

### Postiz POST /posts 格式

**坑：** `posts[0].value must be an array` 錯誤。

`content` 和 `image` 必須包在 `value` 陣列內：

```json
{
  "posts": [{
    "integration": { "id": "..." },
    "value": [{
      "content": "文案內容",
      "image": [{ "id": "MEDIA_ID", "path": "https://...r2.dev/..." }]
    }],
    "settings": {
      "__type": "instagram-standalone",
      "post_type": "post"
    }
  }]
}
```

---

### Postiz Authorization Header

**坑：** API Key 不需要加 `Bearer` 前綴。

```bash
# 錯誤
-H "Authorization: Bearer $POSTIZ_API_KEY"

# 正確
-H "Authorization: $POSTIZ_API_KEY"
```

---

### Instagram 圖片需公開 URL

**坑：** Postiz 本機架設時，圖片存在 Docker volume，URL 為 `http://localhost:4007/uploads/...`。Instagram API 無法從外部存取 localhost，導致 `state: "ERROR"`。

**解法：** 改用 Cloudflare R2（見 Phase 2）。設定正確後圖片 URL 會是 `https://...r2.dev/...`，Instagram 可正常存取。

---

### Cloudflare R2：Region is missing

**坑：** docker-compose.yml 沒有設 `CLOUDFLARE_REGION`，Postiz 報 `Region is missing`。

**解法：** 在 `postiz` service 環境變數加：
```yaml
CLOUDFLARE_REGION: "auto"
```

---

### Cloudflare API Token 權限

**坑：** 建立 API Token 時選錯權限。

- ❌ `Admin Read Only`
- ✅ `Object Read & Write`（Custom Token，限定 Bucket）

---

### Docker + Temporal 相關

見 `docs/docker-setup.md` 的「關鍵試錯教訓」表格，包含：
- `DB=postgres12`（非 `postgresql`）
- 需要獨立的 `temporal-postgresql` + `temporal-elasticsearch`
- 必須掛載 `dynamicconfig/` volume

---

## 驗證整個系統可用

```bash
# 1. Postiz 可存取
curl http://localhost:4007/public/v1/integrations \
  -H "Authorization: $POSTIZ_API_KEY"
# 應回傳 integrations 陣列

# 2. 上傳圖片（確認 R2 URL）
curl -X POST $POSTIZ_BASE_URL/public/v1/upload \
  -H "Authorization: $POSTIZ_API_KEY" \
  -F "file=@$XIAOGI_BASE_DIR/references/小吉.png"
# path 欄位應為 https://...r2.dev/...

# 3. Ideogram API 可用
curl -X POST https://api.ideogram.ai/v1/ideogram-v3/generate \
  -H "Api-Key: $IDEOGRAM_API_KEY" \
  -F "prompt=test" \
  -F "aspect_ratio=1x1"
# 應回傳 data[0].url
```
