# Changelog

All notable changes to iRehab Brief Schema will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [v0.1.2] — 2026-05-06

**Doctor onboarding patch.**

### Added

- [`docs/for-doctors.md`](docs/for-doctors.md) — Direct entry point for clinicians:
  4 ready-to-paste AI prompt templates (specialty pack design / paper-to-JSON
  conversion / system integration planning / specialty pack PR proposal) +
  guidance on which spec files to point AI at if it doesn't auto-discover.
- `README.md` opening section "👋 給醫師的快速入口" — three-bullet quick-start
  pointing to the AI workflow + `docs/for-doctors.md`.

### Changed

- Repo structure section in README now lists `for-doctors.md`.

### Why this release

Making the workflow explicit: a doctor can paste the repo URL into their
ChatGPT / Claude / Gemini and have AI design a specialty-specific brief
template from this spec without writing any code.

---

## [v0.1.1] — 2026-05-06

**Patch release: add JSON Schema reference, examples, and overview doc.**

### Added

- JSON Schema (Draft 2020-12) reference at [`spec/v0.1/schema/brief-template.schema.json`](spec/v0.1/schema/brief-template.schema.json)
  - Full structural validation: required fields, locale whitelist, question types, option shape
  - Cross-field rules documented in `$comment` (validator must enforce: question.id uniqueness, scale.min<max, option.value uniqueness)
- 2 example brief templates at [`examples/`](examples/)
  - `orthopedics-minimal.json` — 4-question minimal example
  - `orthopedics-complete.json` — 8-question complete example with red flag module references, clinicalReviewer, etc.
  - Both pass JSON Schema validation against the v0.1 schema
- High-level overview at [`docs/overview.md`](docs/overview.md)
  - "What is / is NOT" / "Why open source" / design trade-offs / roadmap
- README quick-start with Ajv validation snippet

### No spec changes

The `spec/v0.1/authoring-spec.md` itself is unchanged — this is purely an
additive release providing the reference artifacts that v0.1.0 promised.

---

## [v0.1.0] — 2026-05-06

**Initial public release — spec only.**

### Added

- Authoring format spec v0.1（[`spec/v0.1/authoring-spec.md`](spec/v0.1/authoring-spec.md)）
  - Top-level template structure（`questions[]` 為核心結構）
  - Question type 系統：`text` / `select` / `multiselect` / `boolean` / `scale` / `date`
  - i18n localized object 格式 + 7 個 whitelisted locale + 渲染期 fallback chain
  - Red flag module reference 接口（`redFlagModuleRefs` 為公開 contract）
  - Validation hard rules + soft rules
  - PROM 自查 heuristic
  - Forward-compat degradation rule（`date` → `text` + `formatHint`）
- Red flag module catalog v0.1（[`spec/v0.1/red-flag-modules.md`](spec/v0.1/red-flag-modules.md)）
  - 10 公開 module 接口（5 個臨床領域）：clinical intent + alert level + patient-facing disclaimer 語意鍵
  - **內部判斷邏輯（threshold / 權重 / scoring）永不開源** — 詳見 [`LICENSE_STRATEGY.md` §6](LICENSE_STRATEGY.md)
- Documentation（[`docs/`](docs/)）
  - `governing-law.md` — 台灣法準據法宣告（管轄法院：臺灣臺中地方法院）
  - `brief-vs-prom.md` — Brief 與 PROM 概念區分（避免授權混淆）
- Dual licensing：
  - Apache-2.0 for schema / validator code（[`LICENSE`](LICENSE)）— 含 patent grant
  - CC BY 4.0 for spec docs / examples（[`LICENSE-CONTENT`](LICENSE-CONTENT)）
- License strategy（[`LICENSE_STRATEGY.md`](LICENSE_STRATEGY.md)）
  - §6 Forever-Closed Boundary：6 項商業 IP 永不開源
  - §4 商標獨立保護（iRehab 愛復健 + DE NOVO ORTHOPEDICS 雙商標）
- Contribution guide（[`CONTRIBUTING.md`](CONTRIBUTING.md)）— review-only governance

### 規劃中（v0.2 目標）

- JSON Schema (Draft 2020-12) reference（`spec/v0.1/schema/brief-template.schema.json`）
- Reference validator（Apache-2.0）
- Examples directory：`orthopedics-minimal.json` / `orthopedics-complete.json` 起步
- Specialty pack 範例（歡迎 community 透過 PR 貢獻）
- HIS integration patterns 文件
- Webhook receiver reference implementation

### Legal Basis

- v0.1 法律基礎：律師預擬意見書（A 組個資 / B 組醫療法 / C 組開源授權 / D 組其他）共 26 條結論於 2026-05-06 全數律師確認，無修正
- 商標保護獨立於本 repo 授權（見 [`LICENSE_STRATEGY.md` §4](LICENSE_STRATEGY.md)）

### Notes

- v0.x 為**預備 release**，不保證 v1.0 前向後相容
- v1.0 正式 release 時間目標：2026 Q4
