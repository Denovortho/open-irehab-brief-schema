# iRehab Brief Schema — Red Flag Module Catalog

**Spec version:** v0.1.0
**Release date:** 2026-05-06
**Companion doc:** [`authoring-spec.md`](authoring-spec.md)
**Maintainer:** De Novo Orthopedics 谷盺生物科技股份有限公司
**License:** CC BY 4.0（詳見 [`LICENSE_STRATEGY.md`](../../LICENSE_STRATEGY.md)）

---

## 0. TL;DR

本文件列出 iRehab Brief Schema v0.1 中 **公開**之 red flag module 接口（contract）。每個 module 公開 5 個元素：

1. Module ID
2. Clinical intent（臨床用途）
3. Expected input（哪類 question 應 reference 此 module）
4. Possible alert levels（P0/P1/P2/none）
5. Patient-facing disclaimer requirements（觸發時須附加之病患告知）

**不公開**：閾值、組合判斷邏輯、specialty-specific weighting、SOAP insertion、escalation copy（保留為 De Novo 商業資產，見 spec §5.1）。

---

## 1. 通用設計原則

### 1.1 Alert level 定義

| Level | 意義 | UX 對應 |
|---|---|---|
| **P0** | Critical — 病患應立即就醫 / 撥 119 | 紅色全螢幕警示 + 119 按鈕 + 告知病患「請勿等候線上回覆」 |
| **P1** | High — 醫師應在當日聯繫病患 | 醫師端 notification + 病患端「醫師將盡快聯繫您」 |
| **P2** | Moderate — 應於下次門診優先處理 | 醫師端 todo flag |
| **none** | 無觸發 | 不顯示警示 |

> **Standardization note**：catalog 中**每個 module** 之 "Possible alert levels" 必須包含 `none`（即可不觸發），確保 validator 與 reference handler 一致對應。如某 module 永遠至少 P2（不存在 none），需於該 module entry 顯式註明理由。

### 1.2 Patient-facing disclaimer 語意鍵

每個 module 觸發時必須附帶以下**語意鍵**之一或多個，對應病患端 UI 預期行為：

| 語意鍵 | 臨床意涵 | 預期 UX | 建議文字（中文範本，HIS 可自由覆寫） |
|---|---|---|---|
| `immediate_er_eval` | 立即急診評估 | 紅色全螢幕警示 + 119 按鈕 + 攔截病患繼續填表 | 「您勾選的症狀建議立即急診評估。請撥 119 或前往急診。本問卷不能取代急診判斷。」 |
| `urgent_clinic_visit` | 當日 / 當週優先門診 | 黃色 banner + 醫師端 P1 notification | 「請勿等候線上回覆；如本次問卷顯示警示症狀，請直接聯繫您的醫療團隊。」 |
| `routine_monitoring` | 例行追蹤 | 醫師端 todo flag | 「您的回答將由醫師在下次門診優先處理。」 |
| `general_safety_note` | 通用安全提示 | footer 隱性提示 | 「醫師會以您填寫的內容優先安排對談，您的回答受醫療隱私保護。」 |

**重要**：本 catalog 僅定義**語意鍵**，具體文字（含多語）由各採用方（HIS / 醫院 / 廠商）自行綁定。語意鍵為穩定 contract；新增 disclaimer 類型時擴充本表（minor version bump）。

> **設計筆記：** disclaimer 採 4 個語意鍵而非 opaque A/B/C enum — HIS 廠商看到 `immediate_er_eval` 比 `A` 直觀，未來新增類型也不會 break 既有 consumer。

### 1.3 Dev preview mode

外部開發者可使用 mock handler 驗證 schema：

```typescript
function mockRedFlagHandler(moduleId: string, payload: any): RedFlagResult {
  return {
    moduleId,
    alertLevel: "P2",  // mock: 永遠回傳 P2
    message: "(mock) module triggered for development preview",
    patientFacingDisclaimers: ["general_safety_note"]  // semantic key (見 §1.2)
  };
}
```

實際 production 觸發必須走 iRehab pipeline。

---

## 2. Module Catalog（v0.1）

10 個 module，分 5 個臨床領域。

### 2.1 精神科

#### `suicide_risk_module`

| 元素 | 內容 |
|---|---|
| **Clinical intent** | 篩查自殺意念 / 計畫之存在；觸發後優先安排心理 / 精神科對談 |
| **Expected input** | `multiselect` question with options describing self-harm intent or plan |
| **Possible alert levels** | P0（具體計畫）/ P1（意念但無計畫）/ none |
| **Patient-facing disclaimer** | `immediate_er_eval` + `general_safety_note`（必附）；P0 額外顯示 1925（自殺防治專線）/ 1995（生命線） |
| **Reference example** | `psychiatry_v1` pack `q_psych_red_flags`（option `"想結束自己生命的念頭"` / `"已經想過具體怎麼做"`） |

#### `acute_psychosis_module`

| 元素 | 內容 |
|---|---|
| **Clinical intent** | 篩查急性精神病症狀（幻覺 / 妄想 / 失眠 / 衝動冒險） |
| **Expected input** | `multiselect` with options describing hallucinations, sleep deprivation, impulsive behavior, or substance loss-of-control |
| **Possible alert levels** | P1（單一症狀）/ P2（輕微）/ none |
| **Patient-facing disclaimer** | `general_safety_note`（必附） |
| **Reference example** | `psychiatry_v1` pack `q_psych_red_flags` |

### 2.2 神經科

#### `stroke_warning_module`

| 元素 | 內容 |
|---|---|
| **Clinical intent** | FAST screening — 突發性單側無力 / 臉歪 / 言語困難 / 視力 |
| **Expected input** | `multiselect` with options describing sudden unilateral weakness, facial droop, speech difficulty, vision loss |
| **Possible alert levels** | P0（疑似急性中風）/ P1（亞急性 TIA）/ none |
| **Patient-facing disclaimer** | `immediate_er_eval` + `urgent_clinic_visit`（必附）；P0 強烈建議撥 119 |
| **Reference example** | `neurology_v1` pack `q_neuro_red_flags` |

#### `seizure_alert_module`

| 元素 | 內容 |
|---|---|
| **Clinical intent** | 抽搐 / 失去意識 / 突發劇烈頭痛 |
| **Expected input** | `multiselect` with options describing seizure events, loss of consciousness, thunderclap headache |
| **Possible alert levels** | P0（持續抽搐 / SAH 疑似）/ P1（單次發作）/ none |
| **Patient-facing disclaimer** | `immediate_er_eval` + `urgent_clinic_visit`（必附） |
| **Reference example** | `neurology_v1` pack `q_neuro_red_flags` |

### 2.3 婦產科

#### `obstetric_emergency_module`

| 元素 | 內容 |
|---|---|
| **Clinical intent** | 產科急症（大量出血 / 胎動消失 / 早產徵兆 / 嚴重腹痛） |
| **Expected input** | `multiselect` with options describing heavy bleeding, decreased fetal movement, preterm labor signs, severe abdominal pain；應與 `select` pregnancy_status 配合（gating） |
| **Possible alert levels** | P0（胎動消失 + 孕中）/ P1（大量出血 / 早產徵兆）/ none |
| **Patient-facing disclaimer** | `immediate_er_eval` + `urgent_clinic_visit`（必附）；P0 強烈建議至婦產科急診 |
| **Reference example** | `obstetrics_v1` pack `q_ob_red_flags` |

#### `preeclampsia_warning_module`

| 元素 | 內容 |
|---|---|
| **Clinical intent** | 子癲前症警示（視力模糊 + 劇烈頭痛 + 妊娠期）— 依 ACOG guideline |
| **Expected input** | `multiselect` with options describing vision change + severe headache；需 pregnancy_status = pregnant |
| **Possible alert levels** | P0（多重症狀 + 妊娠期）/ P1（單一症狀）/ none |
| **Patient-facing disclaimer** | `immediate_er_eval` + `urgent_clinic_visit`（必附） |
| **Reference example** | `obstetrics_v1` pack `q_ob_red_flags` |

### 2.4 泌尿婦科

#### `urinary_retention_module`

| 元素 | 內容 |
|---|---|
| **Clinical intent** | 急性尿滯留（完全尿不出來）— 需立即導尿 |
| **Expected input** | `multiselect` with option indicating complete urinary retention |
| **Possible alert levels** | P0（完全 retention）/ none |
| **Patient-facing disclaimer** | `immediate_er_eval` + `urgent_clinic_visit`（必附） |
| **Reference example** | `urogynecology_v1` pack `q_uroyn_red_flags` |

#### `pelvic_emergency_module`

| 元素 | 內容 |
|---|---|
| **Clinical intent** | 骨盆腔急症（嚴重脫垂 + UTI 高燒 + 突發大量陰道出血） |
| **Expected input** | `multiselect` with options describing severe prolapse, fever + flank pain, sudden vaginal bleeding |
| **Possible alert levels** | P0（高燒 + 腰痛 = 上行性 UTI）/ P1（其他組合）/ none |
| **Patient-facing disclaimer** | `immediate_er_eval` + `general_safety_note`（必附） |
| **Reference example** | `urogynecology_v1` pack `q_uroyn_red_flags` |

### 2.5 骨科

#### `fracture_warning_module`

| 元素 | 內容 |
|---|---|
| **Clinical intent** | 骨折 / 病理性骨折警示（夜間痛 + 體重減輕 + 發燒組合 → 腫瘤 / 感染） |
| **Expected input** | `multiselect` with options describing fever, weight loss, night pain；應與 `boolean` trauma_history 配合 |
| **Possible alert levels** | P1（多重 systemic flags）/ P2（單一 flag）/ none |
| **Patient-facing disclaimer** | `general_safety_note`（必附） |
| **Reference example** | `ortho_generic_v1` pack `q_red_flags` |

#### `neurovascular_compromise_module`

| 元素 | 內容 |
|---|---|
| **Clinical intent** | 馬尾症候群（cauda equina）/ 周邊神經血管壓迫 — 大小便失禁為強指標 |
| **Expected input** | `multiselect` with option describing bladder/bowel incontinence + neurologic deficit |
| **Possible alert levels** | P0（incontinence + 急性發作）/ P1（單一神經缺損）/ none |
| **Patient-facing disclaimer** | `immediate_er_eval` + `urgent_clinic_visit`（必附）；P0 建議至急診評估 MRI |
| **Reference example** | `ortho_generic_v1` pack `q_red_flags` |

---

## 3. Module 開發治理

### 3.1 新增 module

外部貢獻者**不可**直接新增 module（不公開判斷邏輯設計者）。新增流程：

1. 提案於 GitHub issue 描述 clinical intent
2. De Novo 內部 spec module logic（threshold / weighting / escalation copy）
3. 審查通過後新增 catalog entry（公開接口）
4. v0.x 期間採此 review-only 模式；v1.0 後考慮開放部分 module 為 community-contributable

### 3.2 棄用 module

- 棄用前 deprecation period ≥ 6 個月
- 棄用通知於 catalog `deprecated` field 註記
- Catalog 永遠保留歷史 module ID（不可重用）

---

## 4. 與 spec v0.1 對應

本文件支援 spec §5（Red Flag Module References）所定義之接口。重點對應：

| Spec §5 | 本文件 |
|---|---|
| 公開 module ID | §2 全 10 個 |
| 公開 clinical intent | §2 各 module 之 "Clinical intent" |
| 公開 expected input | §2 各 module 之 "Expected input" |
| 公開 alert levels | §1.1 + 各 module 表 |
| 公開 disclaimer requirements | §1.2 + 各 module 表 |
| 不公開 threshold | ✅ 不揭露 |
| 不公開 combination logic | ✅ 不揭露 |
| 不公開 SOAP insertion behavior | ✅ 不揭露 |

---

## 5. Validator 整合

Spec §6.1 hard rule 5 規範：

> `redFlagModuleRefs` 中的 ID **必須存在於已發佈 module catalog**

實作上 validator 應：

1. Load 本 catalog 為 known module list（v0.1 為以上 10 個）
2. 對 template 內每個 question 之 `redFlagModuleRefs` 執行 lookup
3. 若 reference 不存在 → fail PR
4. 若 module deprecated → warn

---

## 6. Sign-off Checklist

本 catalog v0.1 已對齊 [`authoring-spec.md`](authoring-spec.md) §5 之 module reference 接口。

---

## 7. Changelog

詳見 [`CHANGELOG.md`](../../CHANGELOG.md)。

---

## 8. References

- 主 spec：[`authoring-spec.md`](authoring-spec.md)
- License 結構：[`LICENSE_STRATEGY.md`](../../LICENSE_STRATEGY.md)（特別是 §6 Forever-Closed Boundary 中關於紅旗 pipeline 內部邏輯的條款）
