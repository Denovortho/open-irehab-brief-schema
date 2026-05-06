# iRehab Brief Schema — Authoring Format Specification

**Public protocol name:** iRehab Brief Schema
**Schema URI base:** `https://denovortho.com/schemas/irehab-brief-schema/v1/`
**Spec version:** v0.1.0
**Release date:** 2026-05-06

### Three-layer versioning

| 層 | 識別 | 當前值 |
|---|---|---|
| **Protocol** | `irehab-brief/<MAJOR.MINOR>` | `irehab-brief/0.1` |
| **Template authoring schema** | `brief-template/<MAJOR.MINOR.PATCH>` | `brief-template/0.1.0`（schema 內 `specVersion` 對應此）|
| **Red-flag module schema** | `red-flag-module/<MAJOR.MINOR.PATCH>` | `red-flag-module/0.1.0` |
| **Repository release** | semver tag | `v0.1.0` |

**Maintainer:** De Novo Orthopedics 谷盺生物科技股份有限公司
**Lead clinical reviewer:** 林佳緯（Tom）— Orthopedic surgeon, MD
**License:** Spec / docs / examples = CC BY 4.0；Schema / validator code = Apache-2.0（詳見 [`LICENSE_STRATEGY.md`](../../LICENSE_STRATEGY.md)）

---

## 0. TL;DR

本 spec 定義 **iRehab Brief Schema 的 authoring format**——醫師設計 Brief 問卷時所遵循的格式。重點：

- **核心結構**：`questions[]`
- **type 系統**：`text | select | multiselect | boolean | scale | date`（通用公開語彙）
- **i18n**：`label: { "zh-TW": "...", "en": "..." }` localized object
- **Red flag**：`redFlagModuleRefs`（公開 module 接口，不公開判斷邏輯）
- **不在本 spec 範圍**：病患送出 payload（另份 spec）、red flag 內部邏輯、Composer UX、AI SOAP 處理

---

## 1. Purpose & Scope

### 1.1 What this spec defines

- 醫師設計 Brief template 的標準格式（authoring schema）
- Question type 系統與其子欄位
- i18n localized object 格式與 fallback 規則
- Red flag module reference 接口
- Validator hard rules 與 soft rules
### 1.2 What this spec does NOT define

- **Patient submission payload**（病患填完送出的資料格式）→ 規劃中，將以另一份公開 spec 釋出
- **Red flag module 內部邏輯**（threshold、權重、組合判斷）→ De Novo 商業資產，永不開源
- **Composer UX / 編輯流程**（PWA 互動、authoring shortcuts）→ De Novo 商業資產
- **AI SOAP / 後續處理 pipeline**（template fill 後的 LLM 處理）→ 商業資產

### 1.3 Source of Truth 宣告

本 spec 為 iRehab Brief Schema 的**唯一公開來源**：

- 任何商業 / 學術 / 整合用途的引用、實作、衍生，**以本 spec 為準**
- 後續變更走 `CHANGELOG.md` + 版本號（見 §12 Spec 變更管理）
- v0.x 期間保留破壞性變更權利；v1.0 起遵守 forward compatibility 承諾

---

## 2. Top-Level Template Structure

### 2.1 Document shape

```json
{
  "specVersion": "0.1.0",
  "id": "ortho_generic_v1",
  "version": "1.0",
  "templateName": {
    "zh-TW": "骨科一般初診",
    "en": "Orthopedics generic intake"
  },
  "specialty": "orthopedics",
  "summaryStyle": "soap_oneliner",
  "vocabularySeeds": ["pain", "swelling", "stiffness"],
  "redFlagModuleSuggestions": ["fracture_warning_module"],
  "questions": [ ... ],
  "authoringDisclaimer": {
    "zh-TW": "本模板為平台 anchor 起點；publish 時你仍是該題版本的臨床作者。",
    "en": "Platform anchor starter; you remain clinical author of any draft you publish."
  },
  "patientDisclaimer": {
    "zh-TW": "本問卷為診前資料蒐集工具，非醫療建議。實際診斷由您的醫師進行。",
    "en": "This intake form is a pre-visit data collection tool, not medical advice. Diagnosis by your physician."
  },
  "clinicalReviewer": {
    "doctorId": "dr_cwlin",
    "doctorName": "林佳緯",
    "specialty": "orthopedics",
    "attestedAt": "2026-05-05T00:00:00+08:00",
    "attestationId": "atst_xxx"
  },
  "reviewStatus": "internal_test_only",
  "createdAt": "2026-05-05T00:00:00+08:00",
  "updatedAt": "2026-05-05T00:00:00+08:00"
}
```

### 2.2 Required top-level fields

| Field | Type | 說明 |
|---|---|---|
| `specVersion` | string | 本 spec 版本號（如 `"0.1.0"`），用於 forward-compat 判斷 |
| `id` | string (slug) | 模板唯一識別，pattern `^[a-z][a-z0-9_]*$` |
| `version` | string | 模板自身版本（與 specVersion 不同） |
| `templateName` | localized object | 模板顯示名稱 |
| `questions` | array | 1-20 個 question objects |

### 2.3 Optional top-level fields

| Field | Type | 說明 |
|---|---|---|
| `specialty` | string | 科別 slug（自由值，建議參照 ICD/Nice categorization） |
| `summaryStyle` | enum | `"soap_oneliner"` \| `"narrative"` \| `"bulleted"`（提示後端 SOAP 處理風格） |
| `vocabularySeeds` | string[] | 詞彙提示，協助下游 NLP（不是強制） |
| `redFlagModuleSuggestions` | string[] | 此 template 建議綁定的 red flag modules |
| `authoringDisclaimer` | localized object | **給醫師看的** authoring-context 免責聲明（例如：「你是 publish 該版本的臨床作者」）|
| `patientDisclaimer` | localized object | **給病人看的**問卷免責聲明（如：「本問卷非醫療建議」）。每個 red flag module 觸發時可能附加 module-specific disclaimer，見 §5.1。 |
| `clinicalReviewer` | object | 臨床審閱者元資料（見 §2.4） |
| `reviewStatus` | enum | `"draft"` \| `"internal_test_only"` \| `"approved"` \| `"deprecated"` |
| `createdAt` | string (ISO 8601) | |
| `updatedAt` | string (ISO 8601) | |

### 2.4 `clinicalReviewer` object

```json
{
  "doctorId": "dr_cwlin",
  "doctorName": "林佳緯",
  "specialty": "orthopedics",
  "attestedAt": "2026-05-05T00:00:00+08:00",
  "attestationId": "atst_xxx",
  "credentials": "Orthopedic surgeon, MD"
}
```

| Field | Required when reviewer present |
|---|---|
| `doctorName` | ✅ |
| `specialty` | ✅ |
| `attestedAt` | ✅（ISO 8601） |
| `doctorId` | ❌（平台 ID） |
| `attestationId` | ❌（平台簽署 ID） |
| `credentials` | ❌ |

---

## 3. Question Object

### 3.1 Common fields (all types)

| Field | Type | Required | 說明 |
|---|---|---|---|
| `id` | string (slug) | ✅ | Stable slug `^[a-z][a-z0-9_]*$`，於 template 內唯一 |
| `type` | enum | ✅ | `text` \| `select` \| `multiselect` \| `boolean` \| `scale` \| `date` |
| `label` | localized object | ✅ | 題目顯示文字 |
| `helpText` | localized object | ❌ | 提示說明 |
| `required` | boolean | ❌ | default `false` |
| `redFlagModuleRefs` | string[] | ❌ | 引用 red flag module IDs |
| `source` | enum | ❌ | `"anchor"`（spec 內建）\| `"anchor_edited"`（醫師微調 anchor）\| `"custom_added"`（醫師新增）\| `"custom_edited"`（從 custom 再修）\| `"imported"`（從外部 spec import）\| `"unknown"`（無法判定） |
| `originalQuestionId` | string (slug) | ❌ | 若 `source = anchor_edited / custom_edited`：原 question 的 `id`，供 audit & SOAP fallback |
| `originalLabel` | localized object | ❌ | 若 `source = anchor_edited / custom_edited`：原始 label，便於追蹤改動 |
| `editedAt` | string (ISO 8601) | ❌ | 最後一次 edit 時間（適用 edited / imported） |

> **設計筆記：** `source` enum 採六值（不是二元 anchor/custom）以保留 provenance 資訊。audit / attribution / 後端 SOAP fallback 都依賴此資訊，過早 collapse 會在下游造成不可逆損失。

### 3.2 Type-specific fields

#### 3.2.1 `text`

自由文字輸入。

| Field | Type | Required | Default | 說明 |
|---|---|---|---|---|
| `maxChars` | integer | ❌ | 200 | 字數上限（建議 ≤ 500） |
| `multiline` | boolean | ❌ | false | 是否多行輸入框 |
| `formatHint` | enum | ❌ | — | 提示前端輸入類型（不參與 hard validation）：`"date"` \| `"time"` \| `"datetime"` \| `"email"` \| `"uri"` \| `"phone"` \| `"id"`。主要用於 §3.2.6 forward-compat：importer 不支援 `date` type 時 degrade 為 `text` + `formatHint:"date"`。 |

**範例**：
```json
{
  "id": "q_chief_complaint",
  "type": "text",
  "label": { "zh-TW": "主訴", "en": "Chief complaint" },
  "required": true,
  "maxChars": 200
}
```

#### 3.2.2 `select`

單選。

| Field | Type | Required | 說明 |
|---|---|---|---|
| `options` | array | ✅ | 選項陣列，每元素為 `{value, label}` 物件（見下） |

**Option object shape**：

| Field | Type | Required | 說明 |
|---|---|---|---|
| `value` | string (slug) | ✅ | Stable machine value，pattern `^[a-z][a-z0-9_]*$`；於同一 question 內唯一；用於 payload / analytics / SOAP mapping，**永不顯示給病人** |
| `label` | localized object | ✅ | 顯示文字（見 §4.1） |

**為何要 stable value**：localized label 改字（包括 i18n 修訂）不應影響 downstream payload。Patient 答案以 `value` 存儲，避免 store display string 或 array index 造成的 drift。

**範例**：
```json
{
  "id": "q_onset",
  "type": "select",
  "label": { "zh-TW": "症狀起始", "en": "Onset" },
  "options": [
    { "value": "acute", "label": { "zh-TW": "急性（< 1 週）", "en": "Acute (<1wk)" } },
    { "value": "subacute", "label": { "zh-TW": "亞急性（1-6 週）", "en": "Subacute (1-6wk)" } },
    { "value": "chronic", "label": { "zh-TW": "慢性（> 6 週）", "en": "Chronic (>6wk)" } }
  ]
}
```

#### 3.2.3 `multiselect`

多選。結構同 `select`（含 `{value, label}` option shape），但 patient 可選多項。

#### 3.2.4 `boolean`

是 / 否。無額外欄位。前端應顯示為兩個選項按鈕。

**範例**：
```json
{
  "id": "q_trauma_history",
  "type": "boolean",
  "label": { "zh-TW": "近期外傷史", "en": "Recent trauma history" }
}
```

#### 3.2.5 `scale`

數值評分（NRS、Likert 等）。

| Field | Type | Required | 說明 |
|---|---|---|---|
| `min` | integer | ✅ | 最小值（如 0） |
| `max` | integer | ✅ | 最大值（如 10） |
| `step` | integer | ❌ | default `1` |
| `labels` | object | ❌ | 端點 / 中點標籤（見下） |

`labels` 結構：
```json
{
  "min": { "zh-TW": "不痛", "en": "No pain" },
  "max": { "zh-TW": "最痛", "en": "Worst pain" },
  "midpoint": { "zh-TW": "中等", "en": "Moderate" }    // optional
}
```

**範例（NRS pain）**：
```json
{
  "id": "q_pain_severity",
  "type": "scale",
  "label": { "zh-TW": "疼痛強度", "en": "Pain intensity" },
  "min": 0,
  "max": 10,
  "labels": {
    "min": { "zh-TW": "不痛", "en": "No pain" },
    "max": { "zh-TW": "最痛", "en": "Worst imaginable" }
  }
}
```

#### 3.2.6 `date`

日期輸入。

| Field | Type | Required | Default | 說明 |
|---|---|---|---|---|
| `format` | string | ❌ | `"YYYY-MM-DD"` | ISO 8601 子格式 |
| `approximate` | boolean | ❌ | false | 是否允許 `YYYY-MM` 或 `YYYY` 模糊輸入 |
| `min` | string (date) | ❌ | — | 最早允許日期 |
| `max` | string (date) | ❌ | — | 最晚允許日期（可用 `"today"` 字面值） |
| `timezone` | string | ❌ | patient local | IANA timezone（如 `"Asia/Taipei"`）；影響 `"today"` 字面值之解算 |

**Timezone 規範：**

- `"today"` 字面值**必須以病患本地時區評估**，不是 server UTC
- Reference validator 應強制 `min ≤ max`（chronological rule）
- 若 question 涉及術前禁食、術後天數計算等臨床敏感日期，建議顯式設定 `timezone` 欄位避免午夜跨日誤判
- 預設 fallback：若 patient device 無時區資訊，validator 採 `Asia/Taipei`（iRehab 主要市場）；外部 implementer 可重設預設

**範例**：
```json
{
  "id": "q_lmp_or_edd",
  "type": "date",
  "label": { "zh-TW": "末次月經 (LMP) 或預產期 (EDD)", "en": "LMP or EDD" },
  "approximate": true,
  "helpText": {
    "zh-TW": "懷孕中請填預產期；非懷孕請填上次月經日期",
    "en": "If pregnant, enter EDD; otherwise enter LMP"
  }
}
```

> **設計筆記：** `date` type 在 v0.1 spec 中保留為 first-class type；implementer 若尚未支援可依下一段 forward-compat 規則 degrade。

**Forward-compat 強制行為（v0.1）**：

任何 importer / consumer 若**尚未支援** `date` type，**MUST** degrade 為 `text` 並附 `formatHint: "date"`，**不得** reject 整份 template：

```json
{
  "id": "q_lmp_or_edd",
  "type": "text",
  "formatHint": "date",
  "label": { "zh-TW": "末次月經 (LMP) 或預產期 (EDD)", "en": "LMP or EDD" }
}
```

此 degradation 為 importer 端自動處理，作者不需重寫 template。Validator 驗證時仍以原 `date` type 為準（不視為錯誤）。`formatHint` 為 informational hint，不參與 hard validation。

---

## 4. i18n & Localization

### 4.1 Localized object 格式

任何顯示字串（label / helpText / option / labels）都使用 localized object：

```json
{
  "zh-TW": "繁體中文文字",
  "en": "English text",
  "ja": "日本語"
}
```

**v0.1 Locale whitelist**（schema 強制）：

| Code | 語言 | v0.1 狀態 |
|---|---|---|
| `zh-TW` | 繁體中文 | ✅ Required |
| `en` | English | ✅ Required |
| `ja` | 日本語 | 🟡 Optional |
| `ko` | 한국어 | 🟡 Optional |
| `es` | Español | 🟡 Optional |
| `vi` | Tiếng Việt | 🟡 Optional |
| `id` | Bahasa Indonesia | 🟡 Optional |

**為何收斂為 whitelist 而非完整 BCP 47**（v0.1 設計決策）：

- 確保 `unknown_locale_key` 錯誤可被 validator deterministic 報出
- 對應 iRehab Patient PWA 既有 7 locales（與 `~/Downloads/IRehab-Patient-PWA/public/locales/` 同步）
- 未來 v0.2 spec 可放寬至完整 BCP 47（含 `zh-CN` / `zh-HK` / 其他語言）；屆時將是 minor version bump，不破壞既有 templates

> **v0.1 Locale Whitelist：** v0.1 採 7 個 whitelisted locale（zh-TW、en、ja、ko、es、vi、id）。未來 minor 版本擴充其他 BCP 47 locale。

### 4.2 Required languages

v0.1 規範：

- **`zh-TW` + `en` 為必填**（任一缺失 → validator hard fail，error code `locale_zh_tw_missing` / `locale_en_missing`）
- 上述其他 5 個 whitelist locale optional
- 任何不在 whitelist 的 locale key → validator hard fail（error code `unknown_locale_key`）

### 4.3 Fallback chain（authoring vs rendering 分層）

**Authoring stage**（template 撰寫時，validator 強制）：

- v0.1：`zh-TW` 與 `en` **同時** required；任一缺失 → validator hard fail
- 其他語言 optional；缺失 → warning 不 fail
- Locale key 必須在 v0.1 whitelist 內（`zh-TW`, `en`, `ja`, `ko`, `es`, `vi`, `id`；任何其他 key → hard fail `unknown_locale_key`）

**Rendering stage**（病患/醫師端 PWA 顯示時，runtime fallback）：

當 UI 請求某 locale（如 `vi`）但該 question 無此語言：

1. 請求的 locale（`vi`）
2. `en` fallback
3. `zh-TW` fallback
4. 若仍缺：應在 authoring 時已被 validator 攔下，不應到 runtime；若到 runtime 視為資料異常

> **重要：** fallback chain 僅適用於**渲染時**，不適用於 authoring。Authoring 必須 zh-TW + en 雙語完整；不允許「單語草稿等系統 fallback」。如未來 spec v0.2 想放寬至「至少一語」（minProperties: 1），需顯式 spec 變更。

---

## 5. Red Flag Module References

> ⚠️ **CRITICAL — public reading**：本章定義之 red-flag modules 為**catalog metadata only**。它們描述「臨床安全主題」與「期望輸入形狀」，但**不包含**可執行 triage logic、threshold rules 或 escalation 指令。任何臨床警示路由必須由採用方（醫療機構 / 廠商）獨立實作與治理；本 spec 不背書任何特定 emergency triage 行為。

### 5.1 接口設計

**公開（spec 中定義）**：

| 元素 | 說明 |
|---|---|
| `redFlagModuleRefs: string[]` | 引用 module ID 陣列 |
| Module catalog（另份 doc） | 列出所有 public modules + 其 contract |
| Module contract | clinical intent / expected input question types / possible alert levels (P0/P1/P2/none) / patient-facing disclaimer 語意鍵 |

**不公開（De Novo 商業資產）**：

| 元素 | 為何不公開 |
|---|---|
| Threshold（閾值） | 涉及 specialty-specific tuning 與 outcome data |
| Combination logic | 多題組合判斷的權重 |
| Specialty-specific weighting | 同一 module 在不同科別之 calibration |
| SOAP insertion behavior | 觸發後 LLM 如何寫入 SOAP |
| Escalation copy | 給醫師看的警示文案 |

### 5.2 Dev preview mode

外部開發者在驗證 schema 時，可使用 mock red flag handler：

- 接收 `redFlagModuleRefs` + question payload
- 回傳 `{ "moduleId": "...", "alertLevel": "P0|P1|P2|none", "message": "(mock)" }`
- 不執行真實判斷

實際 production 觸發必須走 iRehab pipeline。

### 5.3 Module ID 命名規範

`^[a-z][a-z0-9_]*_module$`，例：
- `stroke_warning_module`
- `suicide_risk_module`
- `obstetric_emergency_module`

### 5.4 Module catalog（另份 doc）

完整 module 清單與 contract → [`spec/v0.1/red-flag-modules.md`](red-flag-modules.md)（v0.1，10 modules / 5 領域）。

### 5.5 Module Immutability Rule

**規範**：catalog 中已發佈之 module ID **不可變更語意**。

具體：

- 若需要降低/提升某 module 的 alert level、變更 expected input、改變 disclaimer 語意 → **必須發佈新 ID**（如 `stroke_warning_v2_module`），不可在原 ID 上 in-place 修改
- 舊 ID 標記 `deprecated: true` 並指向繼任 ID，保留至少 6 個月
- **理由**：已 approved 之 templates 透過靜態字串引用 module ID；in-place 修改會造成已部署 templates 的 runtime 行為不可預期改變（臨床風險）
- **僅允許 in-place 修改**：typo 修正、文字微調（不變更語意）、新增其他語言 disclaimer——任何疑問時請拆 v2


### 5.6 Disclaimer 語意鍵

Catalog 中之 `patient-facing disclaimer requirements` 採**語意鍵**而非 opaque A/B/C，定義於 [`spec/v0.1/red-flag-modules.md`](red-flag-modules.md) §1.2：

| 語意鍵 | 臨床意涵 | UI 預期 |
|---|---|---|
| `immediate_er_eval` | 立即急診評估 | 全螢幕紅色警示 + 119 按鈕 |
| `urgent_clinic_visit` | 當日 / 當週優先門診 | 醫師端 P1 notification |
| `routine_monitoring` | 例行追蹤 | 醫師端 todo flag |
| `general_safety_note` | 通用安全提示（不警示） | footer 提示 |

具體文字（多語）由各採用方（HIS / 醫院）自行綁定。Spec 只規範 semantic key，不規範文字內容。

---

## 6. Validation Rules

### 6.1 Hard rules（validator MUST enforce, fail PR）

1. **Document shape**
   - `specVersion`、`id`、`version`、`templateName`、`questions` 必填
   - `specVersion` 對應已知 spec 版本
   - `id` 符合 slug pattern `^[a-z][a-z0-9_]*$`

2. **Questions array**
   - 1 ≤ length ≤ 20
   - `id` 於 template 內唯一
   - `type` ∈ whitelist `{text, select, multiselect, boolean, scale, date}`

3. **i18n**
   - 所有顯示字串為 localized object（不是 plain string）
   - `zh-TW` 與 `en` 必須同時存在於每個 localized object
   - Locale key 必須在 v0.1 whitelist：`{zh-TW, en, ja, ko, es, vi, id}`（其他 key → `unknown_locale_key`）

4. **Type-specific**
   - `select` / `multiselect` 必須有 `options`，1 ≤ length ≤ 30
   - 每個 option 必須是 `{value, label}` 物件
     - `value` 符合 slug pattern `^[a-z][a-z0-9_]*$`
     - `value` 於同一 question 內唯一
     - `label` 為 localized object 含 zh-TW + en
   - `scale` 必須有 `min`、`max`，且 `min < max`
   - `date.min` / `date.max` 為合法日期或字面值（如 `"today"`）

5. **Red flag**
   - `redFlagModuleRefs` 中的 ID 必須存在於已發佈 module catalog
   - Module ID 符合 pattern `^[a-z][a-z0-9_]*_module$`

6. **安全性**
   - 任何顯示字串不得含 HTML tag（`<script>`、`<iframe>`、`<img>` 等）
   - 任何顯示字串不得含 JavaScript URL（`javascript:`）

7. **PROM 自查**（heuristic warning, see §7）
   - 顯示字串不應含 PROM 識別性 wording（PHQ-9 specific items, KOOS specific items 等）

### 6.2 Soft rules（validator SHOULD warn, not fail）

- `text` type 建議含 `maxChars`
- `text` type 若 `maxChars > 200` 建議含 `helpText`
- `scale` 建議含 `labels.min` 與 `labels.max`
- `date` 用於懷孕/手術等模糊回憶情境時建議 `approximate: true`
- 任何 question 的 `label` 長度建議 ≤ 80 字
- Template 建議含 `clinicalReviewer`（v0.1 不強制）

---

## 7. PROM 自查 Heuristic

依 `PROM license audit (internal)` 結論，validator 應實作 PROM 識別性 wording lint：

### 7.1 PROM 黑名單關鍵字（中文 + 英文）

Validator 應 **warn**（不是 fail）若顯示字串包含以下：

| PROM | 識別性 wording 範例 |
|---|---|
| ASES | "Sleep on painful side" / "Wash back / hook bra" / "Reach high shelf" |
| PHQ-9 | "Better off dead" / "Little interest or pleasure in doing things" |
| GAD-7 | "Feeling nervous, anxious, or on edge" |
| KOOS | "How often is your knee painful?" with 5-level Likert |
| ODI | "Pain intensity / Personal care / Lifting / Walking" 10 sections |
| EQ-5D | "Mobility / Self-care / Usual activities" 5 dimensions |

### 7.2 Warning message 範例

```
[PROM-LINT] question "q_xyz" label contains wording resembling DASH/QuickDASH item.
            Suggest rewording or document attribution. See PROM license audit (internal).
```

---

## 8. Example: Minimal

最少必備欄位的合法 template：

```json
{
  "specVersion": "0.1.0",
  "id": "minimal_demo_v1",
  "version": "1.0",
  "templateName": {
    "zh-TW": "最小範例",
    "en": "Minimal example"
  },
  "questions": [
    {
      "id": "q_chief_complaint",
      "type": "text",
      "label": {
        "zh-TW": "主訴",
        "en": "Chief complaint"
      },
      "required": true,
      "maxChars": 200
    }
  ]
}
```

---

## 9. Example: Complete (Orthopedics)

完整骨科一般初診模板（對齊 v0.1 spec、type 命名通用化）：

```json
{
  "specVersion": "0.1.0",
  "id": "ortho_generic_v1",
  "version": "1.0",
  "templateName": {
    "zh-TW": "骨科一般初診",
    "en": "Orthopedics generic intake"
  },
  "specialty": "orthopedics",
  "summaryStyle": "soap_oneliner",
  "vocabularySeeds": ["pain", "swelling", "stiffness", "trauma"],
  "redFlagModuleSuggestions": ["fracture_warning_module", "neurovascular_compromise_module"],
  "questions": [
    {
      "id": "q_chief_complaint",
      "type": "text",
      "label": { "zh-TW": "主訴", "en": "Chief complaint" },
      "required": true,
      "maxChars": 200
    },
    {
      "id": "q_pain_site",
      "type": "select",
      "label": { "zh-TW": "疼痛部位", "en": "Pain site" },
      "options": [
        { "value": "shoulder", "label": { "zh-TW": "肩", "en": "Shoulder" } },
        { "value": "elbow", "label": { "zh-TW": "肘", "en": "Elbow" } },
        { "value": "wrist", "label": { "zh-TW": "腕", "en": "Wrist" } },
        { "value": "hip", "label": { "zh-TW": "髖", "en": "Hip" } },
        { "value": "knee", "label": { "zh-TW": "膝", "en": "Knee" } },
        { "value": "ankle", "label": { "zh-TW": "踝", "en": "Ankle" } },
        { "value": "back", "label": { "zh-TW": "背 / 腰", "en": "Back" } },
        { "value": "other", "label": { "zh-TW": "其他", "en": "Other" } }
      ],
      "required": true
    },
    {
      "id": "q_pain_severity",
      "type": "scale",
      "label": { "zh-TW": "疼痛強度", "en": "Pain intensity" },
      "min": 0,
      "max": 10,
      "labels": {
        "min": { "zh-TW": "不痛", "en": "No pain" },
        "max": { "zh-TW": "最痛", "en": "Worst imaginable" }
      }
    },
    {
      "id": "q_onset",
      "type": "select",
      "label": { "zh-TW": "症狀起始", "en": "Onset" },
      "options": [
        { "value": "acute", "label": { "zh-TW": "急性（< 1 週）", "en": "Acute (<1wk)" } },
        { "value": "subacute", "label": { "zh-TW": "亞急性（1-6 週）", "en": "Subacute (1-6wk)" } },
        { "value": "chronic", "label": { "zh-TW": "慢性（> 6 週）", "en": "Chronic (>6wk)" } }
      ]
    },
    {
      "id": "q_trauma_history",
      "type": "boolean",
      "label": { "zh-TW": "近期外傷史", "en": "Recent trauma history" }
    },
    {
      "id": "q_aggravating",
      "type": "multiselect",
      "label": { "zh-TW": "加重因素", "en": "Aggravating factors" },
      "options": [
        { "value": "weight_bearing", "label": { "zh-TW": "負重", "en": "Weight bearing" } },
        { "value": "movement", "label": { "zh-TW": "動作", "en": "Movement" } },
        { "value": "rest", "label": { "zh-TW": "休息", "en": "Rest" } },
        { "value": "cold", "label": { "zh-TW": "受涼", "en": "Cold" } },
        { "value": "stairs", "label": { "zh-TW": "上下樓梯", "en": "Stairs" } },
        { "value": "sleep_posture", "label": { "zh-TW": "睡姿", "en": "Sleep posture" } }
      ]
    },
    {
      "id": "q_red_flags",
      "type": "multiselect",
      "label": { "zh-TW": "警示症狀", "en": "Red flag symptoms" },
      "options": [
        { "value": "fever", "label": { "zh-TW": "發燒", "en": "Fever" } },
        { "value": "weight_loss", "label": { "zh-TW": "體重減輕", "en": "Weight loss" } },
        { "value": "night_pain", "label": { "zh-TW": "夜間痛", "en": "Night pain" } },
        { "value": "neuro_deficit", "label": { "zh-TW": "神經缺損", "en": "Neurologic deficit" } },
        { "value": "incontinence", "label": { "zh-TW": "大小便失禁", "en": "Loss of bladder/bowel control" } }
      ],
      "redFlagModuleRefs": ["fracture_warning_module", "neurovascular_compromise_module"]
    }
  ],
  "authoringDisclaimer": {
    "zh-TW": "本模板由 林佳緯 醫師作為平台 anchor 起點審閱。你 publish 任何 draft 時仍是該題版本的臨床作者。",
    "en": "Reviewed by Dr 林佳緯 (orthopedics) as platform anchor. You remain clinical author of any draft you publish."
  },
  "patientDisclaimer": {
    "zh-TW": "本問卷為診前資料蒐集工具，非醫療建議。實際診斷由您的醫師進行。",
    "en": "Pre-visit data collection tool, not medical advice. Diagnosis by your physician."
  },
  "clinicalReviewer": {
    "doctorId": "dr_cwlin",
    "doctorName": "林佳緯",
    "specialty": "orthopedics",
    "attestedAt": "2026-05-05T00:00:00+08:00",
    "attestationId": "atst_ortho_generic_v1_seed",
    "credentials": "Orthopedic surgeon, MD"
  },
  "reviewStatus": "approved",
  "createdAt": "2026-04-28T00:00:00+08:00",
  "updatedAt": "2026-05-05T00:00:00+08:00"
}
```

---

## 10. JSON Schema Reference

完整 JSON Schema (Draft 2020-12) reference 將於 v0.2 release 加入 `spec/v0.1/schema/brief-template.schema.json`（規劃中）。本 v0.1 release 為 spec-only，consumer 可自行根據本 spec 撰寫 schema validator。

JSON Schema 實作本 spec §2-§6 之所有 hard rules，可被任何 standard JSON Schema validator 驗證。

**已知 JSON Schema 表達極限**：以下 hard rules 標準 JSON Schema 2020-12 無法完全 enforce，需 reference validator 補強：

- `option.value` 於同一 question 內 unique（schema 只能驗 array shape，不能 cross-element compare）
- `question.id` 於 template 內 unique
- `redFlagModuleRefs` 引用之 ID 必須存在於已發佈 module catalog
- `scale.min < scale.max`（sibling field comparison）

Reference validator 將以 Apache-2.0 釋出於本 repo，實作這些 cross-field rules 作為 hard rules 之第二層 enforcement（規劃中，目標 v0.2）。

範例 brief instance 將於後續 release（v0.2+）加入 `examples/` 目錄。本 v0.1 release 為 spec-only。

---

## 12. Spec 變更管理

### 12.1 Versioning

`specVersion` 為 `MAJOR.MINOR.PATCH`：

- **MAJOR**：breaking change（如新增必填欄位、移除既有欄位、type 名稱改名）
- **MINOR**：additive change（新增 optional 欄位、新增 type、新增 locale）
- **PATCH**：clarifying / non-functional change

### 12.2 Forward compatibility

- v0.x 期間保留 breaking change 權利（preview 階段）
- v1.0 後 MAJOR change 必須有 6 個月 deprecation 期 + adapter 支援

### 12.3 變更流程

1. 提案於 GitHub issue / PR
2. 至少 2 名 maintainer review
3. 有 breaking change 時需 community feedback 期 ≥ 4 週
4. Sign-off → release

---

## 13. Out-of-scope

明確**不在本 spec 範圍**的項目：

| 項目 | 處理位置 |
|---|---|
| Patient submission payload format | 規劃中，將以另一份公開 spec 釋出 |
| Red flag module 內部邏輯（threshold / 權重 / escalation copy）| 永不開源（見 [LICENSE_STRATEGY §6](../../LICENSE_STRATEGY.md)）|
| Composer UX 設計 / AI SOAP 處理 / 病患 PII pipeline | 永不開源（見 [LICENSE_STRATEGY §6](../../LICENSE_STRATEGY.md)）|
| Webhook 接收端 reference implementation | 規劃中（`examples/webhook-receiver/`，未來 release）|
| Specialty pack examples | 規劃中（`examples/`，未來 release，歡迎 community 貢獻）|
| Compatibility Mark / Trademark policy | 詳見 [LICENSE_STRATEGY §4](../../LICENSE_STRATEGY.md) |
| HIS integration patterns | 規劃中（`docs/his-integration.md`，未來 release）|

---

## 14. Maturity Status

本 v0.1.0 為 **預備 release**：

- ✅ Spec 結構穩定 — 無計畫破壞性變更
- ✅ Type 系統穩定 — 6 個 type（text/select/multiselect/boolean/scale/date）為 v1.0 凍結基礎
- ✅ i18n 規範穩定 — 7 個 whitelisted locale + fallback chain
- ✅ Red flag module 接口穩定 — `redFlagModuleRefs` 字串接口為公開 contract
- ⚠️ JSON Schema reference implementation — 規劃中（v0.2 目標）
- ⚠️ Reference validator（Apache-2.0）— 規劃中（v0.2 目標）
- ⚠️ Examples / specialty packs — 規劃中（歡迎 community 貢獻）

v0.x 期間保留破壞性變更權利，但承諾每次破壞性變更走 14 天 RFC（見 §12）。v1.0 後遵守 forward compatibility 承諾。

---

## 15. Changelog

詳見 [`CHANGELOG.md`](../../CHANGELOG.md)。

---

## 16. References

- [`README.md`](../../README.md) — 入口與整體介紹
- [`LICENSE_STRATEGY.md`](../../LICENSE_STRATEGY.md) — 雙授權邊界與 Forever-Closed Boundary
- [`CONTRIBUTING.md`](../../CONTRIBUTING.md) — 貢獻流程與 governance
- [`docs/governing-law.md`](../../docs/governing-law.md) — 準據法（中華民國法律）
- [`docs/brief-vs-prom.md`](../../docs/brief-vs-prom.md) — Brief 與 PROM 的概念區分
- [`spec/v0.1/red-flag-modules.md`](red-flag-modules.md) — Red Flag Module Catalog（接口名稱，不含內部邏輯）
