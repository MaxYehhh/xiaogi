# 成效追蹤日誌（Performance Log）

## 使用說明
每篇貼文上線 **48 小時後**，agent 讀取 Postiz analytics endpoint 並在此追加一筆記錄。
累積 5+ 筆同類數據後，agent 自動更新對應的 skill file。

## 記錄格式
```
### [DATE] — [場景分類] — [場景描述]
- 平台：Instagram / TikTok
- 讚數：X
- 留言數：X
- 分享/收藏數：X
- 觸及人數：X
- 互動率：X%
- Hook 類型：[authority/relatable/contrast/stakes/procedural]
- 文案長度：[short/medium/long]
- 畫面比例：1:1 / 9:16
- 備註：[任何異常——病毒傳播、負評、平台推播]
- 結論：STRONG / AVERAGE / WEAK
```

---

## 日誌記錄

_（系統啟動後由 agent 自動填入）_

---

## 聚合洞察
（每 7 篇貼文後，agent 在此更新摘要）

### 場景分類排名
| 分類 | 平均互動率 | 筆數 |
|------|-----------|------|
| 尚無資料 | — | 0 |

### Hook 類型排名
| Hook 類型 | 平均互動率 | 筆數 |
|----------|-----------|------|
| 尚無資料 | — | 0 |

### 發布時間排名
| 發布時間 | 平均觸及 | 筆數 |
|---------|---------|------|
| 尚無資料 | — | 0 |

---

## Skill 更新規則

當某個模式累積 **5+ 筆**數據時：

| 條件 | 行動 |
|------|------|
| 某場景分類持續高於平均 | 在 viral-formula.md 標記為「proven high-performer」，提高選擇優先級 |
| 某 hook 類型 3/5 勝出 | 在 caption-writing.md 設為「default」 |
| 某場景分類持續低於平均 5 次 | 在 viral-formula.md 加「AVOID」標記 |
| 某視覺元素與高收藏率相關 | 在 image-generation.md 加入固定規則 |
