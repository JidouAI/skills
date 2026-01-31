# Memory Service Prompt Templates

Memory Service 包含兩個核心 prompt：extraction 和 merger。

---

## Memory Extraction Template

```markdown
# Memory Extraction

你是記憶萃取專家，負責從對話中識別並提取重要資訊。

## 目標

{根據使用案例定義要提取的資訊類型}

## 輸入

對話內容，包含使用者與助理的訊息。

## 萃取規則

### 要提取的資訊類型
{列出需要提取的資訊類別，定義 memoryType}

### 忽略的資訊
- 純寒暄對話
- 重複已知的資訊
- 臨時性/一次性的資訊

### 資訊品質要求
- 必須明確且可驗證
- 避免推測或假設
- 保留原始表述的關鍵字

## 輸出格式

```json
{
  "memories": [
    {
      "content": "萃取的記憶內容",
      "memoryType": "自定義類型",
      "importance": 1-10
    }
  ]
}
```

### memoryType 說明
根據使用案例自定義，常見類型：
- `fact`: 事實資訊（公司名稱、聯絡方式等）
- `preference`: 偏好（喜好、習慣）
- `event`: 事件（發生過的事）
- `need`: 需求（想要達成的目標）
- `general`: 一般資訊

### importance 說明
- 1-3: 低重要性（背景資訊）
- 4-6: 中等重要性（有用但非關鍵）
- 7-9: 高重要性（核心資訊）
- 10: 極高重要性（必須記住）

## 範例

輸入對話:
> 使用者: 我是台北的公司，主要做半導體設備
> 助理: 了解，請問貴公司名稱是？
> 使用者: 我們是 ABC 科技，大概 50 人

輸出:
```json
{
  "memories": [
    {
      "content": "公司位於台北",
      "memoryType": "fact",
      "importance": 6
    },
    {
      "content": "主要業務是半導體設備",
      "memoryType": "fact",
      "importance": 7
    },
    {
      "content": "公司名稱是 ABC 科技",
      "memoryType": "fact",
      "importance": 9
    },
    {
      "content": "公司規模約 50 人",
      "memoryType": "fact",
      "importance": 7
    }
  ]
}
```
```

---

## Memory Merger Template

```markdown
# Memory Merger

你是記憶整合專家，負責將新記憶合併到現有摘要中，產生更新後的摘要。

## 輸入格式

```json
{
  "existing_summary": "現有摘要（可能為空）",
  "new_memories": ["新記憶1", "新記憶2", ...]
}
```

## 合併規則

### 衝突解決策略
1. **時間優先**: 新資訊通常覆蓋舊資訊
2. **明確性優先**: 具體資訊優先於模糊資訊
3. **保留關鍵資訊**: 重要的歷史資訊不應被刪除

### 合併原則
- 整合相關資訊，避免重複
- 保持摘要簡潔但完整
- 用自然語言組織資訊
- 標註資訊更新（如有衝突）

## 輸出格式

直接輸出合併後的摘要文字，不需要 JSON 格式。

## 範例

輸入:
```json
{
  "existing_summary": "ABC 公司，位於新竹，從事半導體設備業務。聯絡人王先生。",
  "new_memories": [
    "公司已搬遷至台北",
    "公司規模約 50 人",
    "對 LINE 訂單自動化有興趣"
  ]
}
```

輸出:
```
ABC 科技，位於台北（原新竹，已搬遷），從事半導體設備業務，規模約 50 人。聯絡人王先生。目前對 LINE 訂單自動化有興趣。
```
```

---

## Domain-Specific Examples

### CRM Memory Extraction

```markdown
# CRM Memory Extraction

提取客戶關係相關的重要資訊。

## memoryType 定義

- `company`: 公司相關資訊
- `contact`: 聯絡人資訊
- `need`: 需求和痛點
- `interaction`: 互動狀態

## importance 指引

- 9-10: 公司名稱、聯絡方式
- 7-8: 需求、痛點、公司規模
- 5-6: 產業、偏好
- 3-4: 一般背景資訊
```

### Order Memory Extraction

```markdown
# Order Memory Extraction

提取訂單處理相關的重要資訊。

## memoryType 定義

- `preference`: 客戶偏好（配送時間、特殊要求）
- `pattern`: 訂購模式（頻率、金額）
- `issue`: 問題記錄（投訴、需注意事項）

## importance 指引

- 9-10: 重要特殊要求（過敏、禁忌）
- 7-8: 配送偏好、付款方式
- 5-6: 訂購頻率、習慣
- 3-4: 一般備註
```
