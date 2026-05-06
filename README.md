# iRehab Brief Schema

> An open authoring format for pre-consult symptom intake questionnaires.
> 一個開放的診前症狀問卷格式標準。

[![License: Apache-2.0 (code)](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![License: CC BY 4.0 (content)](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey.svg)](LICENSE-CONTENT)
[![Version](https://img.shields.io/badge/spec-v0.1.0-green.svg)](spec/v0.1/)

---

## What is this?

**iRehab Brief Schema** 是一個開放的診前問卷格式 — 一個標準化的「醫師可以怎麼設計病患診前症狀問卷」的 authoring format。它不是一個產品、不是一個 SaaS、不是一個 app。它是格式本身。

**為什麼開源這個？**

格式（`questions[]` 結構、type 系統、i18n、red flag 接口）是「分發器」，**任何 HIS 廠商、第三方平台、外部醫師都可以原生支援**，不需要跟任何單一供應商簽授權。

商業價值（紅旗判斷邏輯、AI SOAP 整合、override corpus）留給 iRehab Composer 平台 — 兩層分開，各自走各自的競爭軸線。

---

## 30 秒看 Schema 長什麼樣

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
  "questions": [
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
  ]
}
```

完整格式請見 [spec/v0.1/authoring-spec.md](spec/v0.1/authoring-spec.md)。

---

## License

本 repo 採雙授權：

| 內容 | 授權 |
|---|---|
| Schema 檔（`spec/v0.1/schema/*.json`）、validator 程式碼、CI 工具 | [Apache-2.0](LICENSE) |
| Spec 文件、範例、教學、本 README 等內容 | [CC BY 4.0](LICENSE-CONTENT) |

**準據法（Governing Law）：** 本 repo 的授權條款效力以**中華民國（台灣）法律**為準據，並以**臺灣臺中地方法院**為第一審管轄法院。詳見 [`docs/governing-law.md`](docs/governing-law.md)。

> **重要：** 雙授權的「界線」與哪些東西**永遠不會被加入本 repo** 都明確記載於 [`LICENSE_STRATEGY.md`](LICENSE_STRATEGY.md)。請任何想用本 schema 做商業整合的人**先讀那份**。

---

## What's NOT in this repo (forever closed)

下列元件**永遠不會**加入本 repo（屬 De Novo Orthopedics / 谷盺生物科技商業 IP）：

1. **Red flag pipeline 內部邏輯**（threshold、權重、escalation copy 等）— 醫療判斷邏輯
2. **AI SOAP 整合 prompts + tuning** — 商業 prompt engineering
3. **病患 PII 處理程式碼** — 個資合規碰不得
4. **iRehab Doctor PWA 的 UI/UX 程式碼**
5. **Outcome registry 資料管線**
6. **Composer 編輯器（GUI for schema authoring）**

詳見 [`LICENSE_STRATEGY.md` §6](LICENSE_STRATEGY.md)。本 repo 只開放**格式（schema）+ red flag module 的接口（reference）**。

---

## Repo 結構

```
.
├── README.md
├── LICENSE                    Apache-2.0（schema / validator code）
├── LICENSE-CONTENT            CC BY 4.0（docs / spec / examples）
├── LICENSE_STRATEGY.md        雙授權邊界 + Forever-closed boundary §6
├── CONTRIBUTING.md            如何貢獻 + governance
├── CHANGELOG.md
├── spec/
│   └── v0.1/
│       ├── authoring-spec.md           authoring format 主 spec
│       ├── red-flag-modules.md         red flag module catalog（公開接口）
│       └── schema/
│           └── brief-template.schema.json  JSON Schema (Draft 2020-12)
├── examples/
│   ├── orthopedics-minimal.json         4-question minimal example
│   └── orthopedics-complete.json        8-question complete example
└── docs/
    ├── overview.md            5-min 高層次介紹（建議先看）
    ├── governing-law.md       台灣法準據法宣告
    └── brief-vs-prom.md       Brief 與 PROM 概念區分
```

**規劃中（v0.2+，歡迎 community 貢獻）：**
- Reference validator（Apache-2.0，跑 cross-field rules）
- 其他 specialty pack examples（婦泌 / 神內 / 身心 / 復健 / 急診…）
- HIS integration patterns 文件
- Webhook receiver reference implementation

---

## Getting Started

1. **5 分鐘了解全貌：** [`docs/overview.md`](docs/overview.md)
2. **看 spec 細節：** [`spec/v0.1/authoring-spec.md`](spec/v0.1/authoring-spec.md)
3. **看 example brief 怎麼長：** [`examples/orthopedics-complete.json`](examples/orthopedics-complete.json)
4. **驗證自己的 brief template：** 用 [`spec/v0.1/schema/brief-template.schema.json`](spec/v0.1/schema/brief-template.schema.json)（任何 JSON Schema Draft 2020-12 validator，例如 [`ajv`](https://ajv.js.org/)）
5. **想貢獻 specialty pack？** 先看 [`CONTRIBUTING.md`](CONTRIBUTING.md)

### 快速驗證範例（Node.js + Ajv）

```bash
npm install ajv ajv-formats
```

```js
const Ajv = require('ajv/dist/2020');
const addFormats = require('ajv-formats');
const fs = require('fs');

const ajv = new Ajv({allErrors: true, strict: false});
addFormats(ajv);

const schema = JSON.parse(fs.readFileSync('spec/v0.1/schema/brief-template.schema.json'));
const example = JSON.parse(fs.readFileSync('examples/orthopedics-complete.json'));

const validate = ajv.compile(schema);
console.log(validate(example) ? 'PASS ✓' : validate.errors);
```

---

## Maintainer

- **De Novo Orthopedics 谷盺生物科技股份有限公司** — Taiwan
- **Lead clinical reviewer:** 林佳緯 醫師
- **Contact:** issues only via this repo（不接受 email PR；governance 詳見 CONTRIBUTING）

---

## Citation

If you use iRehab Brief Schema in research, please cite:

```
De Novo Orthopedics. (2026). iRehab Brief Schema v0.1: An open authoring format
for pre-consult symptom intake questionnaires.
https://github.com/Denovortho/open-irehab-brief-schema
```
