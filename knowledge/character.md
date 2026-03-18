# XiaoGi — 角色設定書（Character Bible）

## 身份
- 名字：XiaoGi（念法："shao-jee"，小吉）
- 品種：吉娃娃（Chihuahua）
- 性別：男
- 核心人設：完全不知道自己有多小。在任何場合都以十足的主管自信行事。
  永遠不扮演弱者。永遠不向鏡頭眨眼搞怪。幽默感來自他的認真，而不是諷刺。

## 視覺規格（每次生成都要鎖定這些）

### 外型
- 品種：短毛吉娃娃，蘋果頭（apple-head skull）
- 毛色：淺棕/黃褐色底，胸口有白色斑塊
- 體型提示：在場景中，相對於人類尺度的物件，他的體型要明顯偏小
- 眼睛：大、深琥珀色、略為突出——品種特徵
- 耳朵：大、直立、三角形——每張圖都要清楚可見
- 表情：專注、有權威感、認真——除非場景需要，否則絕對不要「可愛/微笑」

### Ideogram API Reference Anchor
```
reference_image_urls: ["[YOUR_CANONICAL_REFERENCE_URL]"]
```
**第一次執行後填入 Cold Start Protocol 選定的圖片 URL**

### Style Lock
```
style_codes: ["[FILL_AFTER_FIRST_APPROVED_GENERATION]"]
```
**第一次執行後，從 Ideogram 回應中複製 style_code 填入此處**

- style: "REALISTIC"（寫實風格，非卡通）
- rendering_speed: "QUALITY"
- aspect_ratio: "ASPECT_1_1"（IG feed）或 "ASPECT_9_16"（Stories/TikTok）

### Prompt 必帶元素
- "a small fawn Chihuahua with large erect ears and amber eyes"
- 明確說明場景情境，不要抽象描述
- 至少一個人類尺度的對比物件（椅子、桌子、工具）

### Prompt 絕對禁止
- "cute"、"adorable"、"tiny"——會引導出寵物風格而非主管風格
- "smiling dog"——破壞他的不苟言笑權威感
- "cartoon"、"illustration"——我們要寫實感
- 場景中有其他動物
