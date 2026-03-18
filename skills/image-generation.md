# 圖片生成技能（Image Generation Skill）

## API：Ideogram v3（主力）

```
Endpoint: POST https://api.ideogram.ai/v1/ideogram-v3/generate
Auth:     Authorization: Bearer $IDEOGRAM_API_KEY
```

## 請求模板

```bash
curl -X POST https://api.ideogram.ai/v1/ideogram-v3/generate \
  -H "Authorization: Bearer $IDEOGRAM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "image_request": {
      "prompt": "[GENERATED_PROMPT]",
      "model": "V_3",
      "style": "REALISTIC",
      "rendering_speed": "QUALITY",
      "aspect_ratio": "ASPECT_1_1",
      "expand_prompt": false,
      "reference_image_urls": ["'"$CHARACTER_REFERENCE_URL"'"],
      "style_codes": ["[LOCKED_STYLE_CODE]"]
    }
  }'
```

---

## Prompt 建構公式

每個 prompt 的結構：
```
[主體描述]. [環境描述]. [光線描述]. [氛圍描述].
```

### SUBJECT_BLOCK（主體描述）
```
A small fawn Chihuahua with large erect ears, amber eyes, and a white chest blaze, [具體的動作描述]
```

範例：
```
sitting at the head of a long mahogany boardroom table, front paws resting
on the table surface, gazing authoritatively at documents spread before him
```

### ENVIRONMENT_BLOCK（環境描述）
明確說明環境，至少帶入 **2 個真實道具**。
```
in a [具體場所] with [道具1], [道具2], and [道具3]
```

**必帶體型對比錨點**（至少一個）：
- "standard-height office chairs surrounding him"
- "a full-size coffee mug on the table beside him"
- "human-scale instruments and equipment nearby"

### LIGHTING_BLOCK（光線描述）
| 場所 | 光線描述 |
|------|---------|
| 會議室/辦公室 | "Soft overhead fluorescent with window fill light. Professional." |
| 高級餐廳 | "Warm candlelight and ceiling pendants. Intimate and upscale." |
| 戶外 | "Golden hour side light. Crisp shadows." |
| 醫院/手術室 | "Cool clinical overhead lighting. Sharp and sterile." |
| 工廠/工地 | "Industrial overhead lighting with ambient dust haze." |

### MOOD_BLOCK（氛圍描述）
永遠以這三選一收尾：
- `"Photorealistic. Deadpan. No eye contact with camera."`
- `"Photorealistic. Focused. Completely absorbed in task."`
- `"Photorealistic. Authoritative. Scene feels real and unposed."`

---

## 完整 Prompt 範例

### 場景：董事會主持
```
A small fawn Chihuahua with large erect ears, amber eyes, and a white chest
blaze, sitting at the head of a long mahogany boardroom table, front paws
resting on the table surface, gazing authoritatively at documents spread
before him. In a modern corporate boardroom with floor-to-ceiling windows,
leather executive chairs on both sides, and a large presentation screen
behind him, standard-height chairs clearly dwarfing him. Soft overhead
fluorescent with window fill light. Professional. Photorealistic. Deadpan.
No eye contact with camera.
```

### 場景：高級餐廳服務生
```
A small fawn Chihuahua with large erect ears, amber eyes, and a white chest
blaze, standing upright holding a small notepad, wearing a formal black
waiter's vest, attentively taking an order. In an upscale fine dining
restaurant with white tablecloths, crystal glassware, and warm chandelier
lighting, a full-size dining chair visible beside him for scale. Warm
candlelight and ceiling pendants. Intimate and upscale. Photorealistic.
Focused. Completely absorbed in task.
```

### 場景：工地監工
```
A small fawn Chihuahua with large erect ears, amber eyes, and a white chest
blaze, wearing a yellow hard hat, standing on a construction site holding a
rolled-up blueprint under one paw, surveying the work with an authoritative
expression. Industrial construction site with scaffolding, heavy machinery,
and workers in background, standard construction equipment towers over him.
Industrial overhead lighting with ambient dust haze. Photorealistic.
Authoritative. Scene feels real and unposed.
```

---

## 生成 → 審核 → 鎖定循環

### 首次生成（尚無 reference）
1. 省略 `reference_image_urls` 和 `style_codes`
2. 生成 3 張變體（3 次獨立呼叫）
3. 挑最佳一張 → 成為 canonical reference
4. 上傳到永久 URL（GitHub raw 或 Cloudinary 免費方案）
5. 從回應複製 `style_code`
6. 將兩者填入 `/knowledge/character.md` 的對應欄位
7. **後續所有生成都帶入這兩個值**

### 每日生成流程
1. 從 character.md 讀取 `reference_image_urls` 和 `style_codes`
2. 依照上方公式建構 prompt
3. 寫入 `/outputs/YYYY-MM-DD/prompt.md`
4. 呼叫 Ideogram v3 API
5. 品質檢查（見下方）
6. 下載圖片到 `/outputs/YYYY-MM-DD/image.png`：
   ```bash
   curl -L "[IMAGE_URL_FROM_RESPONSE]" -o /outputs/YYYY-MM-DD/image.png
   ```

---

## 品質檢查清單（自動執行）

生成後，看圖並確認：
- [ ] XiaoGi 清楚是一隻吉娃娃（不是普通狗）
- [ ] 場景不需要文案就能理解
- [ ] XiaoGi 是畫面視覺中心
- [ ] 體型反差清楚可見
- [ ] 沒有明顯的 AI 失真（多餘肢體、臉部熔化）

**任何一項失敗 → 修改 prompt 重新生成一次**
**第二次仍失敗 → 接受圖片，在 prompt.md 記錄失敗項目**

---

## FLUX Kontext 備援（fal.ai）

用於 Ideogram 結構正確但細節有問題的情況（外科手術式修正）。

```
Endpoint: POST https://fal.run/fal-ai/flux-pro/kontext
Auth:     Authorization: Key $FAL_API_KEY
```

```bash
curl -X POST https://fal.run/fal-ai/flux-pro/kontext \
  -H "Authorization: Key $FAL_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "image_url": "[URL_OF_BEST_IDEOGRAM_OUTPUT]",
    "prompt": "Keep the Chihuahua identical. [具體修正指令]",
    "guidance_scale": 3.5,
    "num_inference_steps": 28
  }'
```

範例修正指令：
- `"Fix the right front paw, it looks deformed."`
- `"Add a full-size coffee mug on the table next to him for scale contrast."`
- `"Make the boardroom background sharper and more detailed."`

---

## 畫面比例規則

| 用途 | aspect_ratio | 解析度 |
|------|-------------|--------|
| Instagram 貼文（預設） | `ASPECT_1_1` | 1080×1080 |
| Instagram Story / TikTok | `ASPECT_9_16` | 1080×1920 |
| 每週一額外輸出 | 同時生成 1:1 和 9:16 |

---

## 成本追蹤

| API | 單價 |
|-----|------|
| Ideogram v3 QUALITY | ~$0.08/張 |
| fal.ai FLUX Kontext | ~$0.04/次 |
| 每日目標上限 | 3 次 Ideogram + 1 次 Kontext = ~$0.28/天 |

---

## 已學到的規則（初始為空，隨執行累積）

_（agent 發現有效/無效的 prompt 元素後在此記錄）_
