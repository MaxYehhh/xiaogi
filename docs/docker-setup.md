# Docker / Postiz 架設指南

最後更新：2026-03-20

本文記錄本機跑 Postiz + Temporal 的完整流程，包含試錯教訓。
下次在新機器或重建時，直接照此文件操作。

---

## 前置條件

### 1. Docker Desktop for macOS

- 下載安裝：https://www.docker.com/products/docker-desktop/
- **安裝後必須手動開啟 Docker Desktop App 完成初始化**，daemon 才會啟動
- PATH 設定（加入 `~/.zshrc`）：
  ```bash
  export PATH="/Applications/Docker.app/Contents/Resources/bin:$PATH"
  ```
  然後 `source ~/.zshrc`

### 2. Facebook Developer App 憑證

Postiz 連接 Instagram 需要：
- `FACEBOOK_APP_ID`
- `FACEBOOK_APP_SECRET`

取得步驟：
1. 前往 developers.facebook.com → 建立新 App（類型選「商業」）
2. 在 App 中新增產品：Instagram Graph API
3. 取得 App ID 和 App Secret
4. 加入 redirect URI：`http://localhost:4007/integrations/social/instagram`

---

## 專案 Docker 目錄結構

```
docker/
  docker-compose.yml              ← 唯一 compose 檔，自維護
  dynamicconfig/
    development-sql.yaml          ← Temporal 必要設定檔（缺少會 crash loop）
```

> **重要**：不要用官方 `postiz-docker-compose` repo。本專案自己維護 `docker/` 目錄，
> 已包含正確的 Temporal 設定。

---

## 啟動指令

```bash
cd /Users/max/projects/xiaogi/docker
docker compose up -d
```

啟動順序（由 `depends_on` healthcheck 控制）：
```
postiz-postgres → postiz-redis → temporal-postgresql + temporal-elasticsearch
  → temporal (等待 healthcheck 通過) → postiz
```

確認全部跑起來：
```bash
docker compose ps
```

確認 Postiz 可存取：http://localhost:4007

---

## 關鍵試錯教訓

| 坑 | 錯誤做法 | 正確做法 |
|----|---------|---------|
| Temporal DB driver | `DB=postgresql` | `DB=postgres12` |
| Temporal 環境變數名稱 | `TEMPORAL_URL` | `TEMPORAL_ADDRESS` |
| Temporal 啟動設定 | 不掛 dynamicconfig volume | 必須掛 `./dynamicconfig:/etc/temporal/config/dynamicconfig` |
| Temporal 依賴 | 共用 Postiz 的 PostgreSQL | 需獨立的 `temporal-postgresql` + `temporal-elasticsearch` |
| Backend 啟動時序 | Postiz 啟動時 Temporal 未就緒 → backend 靜默失敗 | `depends_on: temporal: condition: service_healthy` |

---

## Backend 靜默失敗：症狀與急救

**症狀**：網頁有畫面，但登入 / API 全部 502。

**確認方式**：
```bash
docker exec postiz ss -tlnp | grep 3000
```
有輸出 = backend 正常；無輸出 = backend 沒跑起來。

**急救**：
```bash
docker exec postiz pm2 restart backend
```

---

## Facebook / Instagram 設定

`FACEBOOK_APP_ID` 和 `FACEBOOK_APP_SECRET` 直接寫在 `docker/docker-compose.yml` 的 `postiz` service 環境變數中。

Instagram OAuth redirect URI 要在 Facebook Developers 後台加入：
```
http://localhost:4007/integrations/social/instagram
```

---

## 取得 Postiz API Key 和 Integration ID

1. 登入 http://localhost:4007
2. **Settings > API Keys** → 建立 Key → 填入 `.env` 的 `POSTIZ_API_KEY`
3. **Settings > Integrations** → 連接 Instagram → 取 ID → 填入 `.env` 的 `POSTIZ_INSTAGRAM_INTEGRATION_ID`

---

## 日常啟動（Docker Desktop 重啟後）

所有容器設定了 `restart: always`，Docker Desktop 重啟後會自動回復，**不需要手動操作**。

如果 backend 又出現 502：
```bash
docker exec postiz pm2 restart backend
```
