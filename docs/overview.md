# Overview

> 這份文件給第一次進來這 repo 的人 — 5 分鐘搞清楚「這是什麼、不是什麼、給誰用、為什麼長這樣」。

---

## What is iRehab Brief Schema?

**一個診前症狀問卷的 authoring format。**

當醫師想設計一份「病患看診前先填的症狀問卷」時，他需要一個**結構化的格式**來描述問卷 — 哪些題、什麼類型、選項是什麼、哪些題會觸發紅旗判斷。

iRehab Brief Schema 就是這個格式。**它定義「怎麼寫」，不定義「怎麼判斷」。**

舉個具體例子：

```json
{
  "id": "main_complaint",
  "type": "text",
  "label": {
    "zh-TW": "您最不舒服的部位 / 症狀",
    "en": "Primary symptom or affected area"
  },
  "required": true,
  "redFlagModuleRefs": ["fracture_warning_module"]
}
```

這是一題。`type: "text"` 說它是文字輸入，`label` 是給病人看的題目（雙語），`required` 是必答，`redFlagModuleRefs` 引用了一個叫做 `fracture_warning_module` 的紅旗判斷模組。

**Schema 公開：題目、類型、選項、紅旗 module 的「名字」**
**Schema 不公開：紅旗 module 的「內部判斷邏輯」（這條是 De Novo 的商業 IP）**

兩條線分開，是這個 spec 整個設計的核心。

---

## What it is NOT

避免誤解：

- **不是 SaaS / 平台** — 這個 repo 沒有產品，沒有 UI，沒有 backend。它只是文件 + JSON Schema。
- **不是醫療器材** — 純粹的 authoring format，不做診斷判斷，不歸 TFDA / FDA / NMPA SaMD 範疇。
- **不是 PROM 量表庫** — 跟 PROM（Patient-Reported Outcome Measures，例如 ASES / KOOS / WOMAC）是不同的東西。詳見 [`brief-vs-prom.md`](brief-vs-prom.md)。
- **不是 FHIR Questionnaire 的 fork** — FHIR 是另一個 ecosystem 的 standard，本 schema 是獨立設計，但目標市場有重疊。
- **不是 iRehab Composer 編輯器** — Composer 是 De Novo 的商業產品（醫師端編 brief 用的 GUI），永不開源。Schema 只規範格式，任何人都可以自己做編輯器。

---

## Why open source this?

簡單版：

> 「**格式」開源換廣度與正當性，「邏輯」留 closed 守護核心 IP。**

詳細版：

如果一個格式只有 De Novo 一家用，HIS 廠商要原生支援得簽授權、付錢、過法務 — 這是高 friction。沒人會主動做。

開源後變成：

- HIS 廠商可以**直接讀 schema 做原生整合**（沒 NDA、沒授權金）
- 第三方平台可以做**格式 reciprocity**（彼此問卷可互讀）
- 外部醫師可以**透過 PR 貢獻 specialty pack**（婦泌、神內、身心、復健…）
- 學術論文可以**引用本 schema 為 citable standard**

這些事情在 proprietary 下**結構性**地不會發生 — 不是「比較慢」，是「根本不可能」。

而 De Novo 的核心 IP（紅旗 pipeline 邏輯、AI SOAP 整合、override corpus、Composer 編輯器）**永遠不會**進到本 repo（詳見 [`LICENSE_STRATEGY.md` §6](../LICENSE_STRATEGY.md)）。

格式是分發器，邏輯是引擎；引擎非公開，分發器可以免費。

---

## Designed for whom

主要給以下幾類人用：

### 1. HIS / EHR 廠商

要在自家系統裡加診前問卷功能，不想自己從 0 設計格式。
→ 直接讀 spec、寫 native importer/exporter，跟 iRehab 雲端整合或不整合都行。

### 2. 第三方平台 / SaaS 業者

掛號平台、健保資訊整合者、醫材廠的服務模組。
→ 可以做 brief schema reciprocity — 你的問卷我能讀，我的問卷你能讀。

### 3. 研究員 / 學術論文作者

想用一個 standardized 結構描述「臨床問診前的結構化資料採集」。
→ 引用本 schema 為方法學 citable artifact。

### 4. 外部醫師 / specialty 領域 contributor

想為某個自己擅長的科別（婦科、身心、神內、家醫…）設計一份範例 brief，貢獻給社群。
→ 走 PR 流程，看 [`CONTRIBUTING.md`](../CONTRIBUTING.md)。

### 5. iRehab 生態系內部使用者

iRehab 平台醫師會用 Composer 編輯 brief，最終產出物符合本 spec。Composer 是商業產品（不開源），但**產出格式公開**讓你的資料隨時可以遷出。

---

## Design trade-offs

幾個關鍵決定，記下背後思考避免後人重複討論：

### 為什麼題數限制 1-20

一份 brief 是「診前快速採集」，不是「完整病史」。20 題上限強制設計者去取捨：什麼是真正不可少的、什麼是 nice-to-have。

> 如果你發現需要 30 題，多半是兩個 brief 合在一起；考慮拆。

### 為什麼 type 只有 6 種（text / select / multiselect / boolean / scale / date）

少即是多。複雜 type（matrix / file upload / signature）會讓 spec 立即變沉重，consumer 實作門檻飆高。6 種覆蓋 95% 的問診場景；剩下 5% 不該由本 spec 解決。

### 為什麼 i18n 強制至少 2 語

`zh-TW` + `en` 雙語**強制**才能 publish。讓「臨時用單語草稿」這種陋習從 spec 層級擋住。如果你連 en label 都沒寫，問卷不該流到 production。

### 為什麼 option.value 與 label 分開

`label` 給人看，`value` 給機器看。修 label 文字（甚至改 i18n 翻譯）**不應**影響下游 payload / SOAP mapping。不分開的話，一改翻譯就破壞 5 年來累積的資料。

### 為什麼 red flag module 「reference」公開、「邏輯」永遠 closed

紅旗判斷是醫療判斷邏輯：什麼答案在什麼組合下要 escalate 成「立即急診」？這個邏輯如果開源，會：

- 失去 quality gate（任何人可以改 threshold，造成漏診 / 過度警示）
- 失去 De Novo 的差異化（這是公司唯一不容易被複製的部分）

所以紅旗 module 接口公開（讓 HIS 廠商知道有哪些 module，每個是做什麼的），但內部邏輯永遠 closed。詳見 [`spec/v0.1/red-flag-modules.md`](../spec/v0.1/red-flag-modules.md) + [`LICENSE_STRATEGY.md` §6](../LICENSE_STRATEGY.md)。

### 為什麼 forever-closed boundary 寫進 LICENSE_STRATEGY

沒這條，open-core 策略容易被質疑「你只是還沒開源更多而已」。明確列出 6 項商業 IP 永不開源（紅旗邏輯 / AI SOAP / 病患 PII / Composer UI / Outcome registry / B2B pipeline），讓商業整合對象一次看清「界線在哪」。

### 為什麼準據法是台灣

授權人是台灣公司（De Novo Orthopedics 谷盺生物科技）；商標、責任、爭議解決都在台灣處理最直接。Apache-2.0 + CC BY 4.0 條文本身國際通用、不修改；準據法宣告是附加陳述。詳見 [`docs/governing-law.md`](governing-law.md)。

---

## Roadmap

| Version | Status | 主要內容 |
|---|---|---|
| **v0.1.0** | Released 2026-05-06 | 純 spec + 法律框架（你現在看的）|
| v0.1.1 | Planned | + JSON Schema reference + 2 examples + 此 overview |
| v0.2.0 | Planned 2026 Q3 | + Reference validator (Apache-2.0) + 多科別 examples |
| v1.0.0 | Target 2026 Q4 | Stable open standard + forward-compat 承諾啟動 |

詳見 [`CHANGELOG.md`](../CHANGELOG.md)。

---

## Where to go next

- **想看 spec 細節：** [`spec/v0.1/authoring-spec.md`](../spec/v0.1/authoring-spec.md)
- **想看 red flag module 接口：** [`spec/v0.1/red-flag-modules.md`](../spec/v0.1/red-flag-modules.md)
- **想看授權邊界：** [`LICENSE_STRATEGY.md`](../LICENSE_STRATEGY.md)
- **想貢獻：** [`CONTRIBUTING.md`](../CONTRIBUTING.md)
- **想看一份完整 brief 長什麼樣：** [`examples/orthopedics-complete.json`](../examples/orthopedics-complete.json)
- **法律問題 / 商業整合：** `service@denovortho.com`
