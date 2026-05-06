# Contributing to iRehab Brief Schema

謝謝考慮對本 schema 貢獻。本文件說明貢獻流程、governance 模式、以及哪些東西可以 / 不可以貢獻。

---

## TL;DR

- **歡迎** specialty pack 範例、文件修正、i18n locale、JSON Schema validator 修正
- **不接受** red flag 內部邏輯、Composer UI、後端 SOAP 處理（這些屬 forever-closed，詳見 [`LICENSE_STRATEGY.md` §6](LICENSE_STRATEGY.md)）
- **Governance：** review-only — 所有 PR 由 De Novo Orthopedics 維護團隊審查，48 小時內回應目標
- **重大變更（破壞性 / 結構性）** 需先開 RFC issue 討論至少 14 天

---

## 你可以貢獻什麼

### ✅ 歡迎的貢獻類型

#### 1. Specialty pack 範例 brief template

不同科別的範例 brief（婦科、神經科、身心科、復健科、家醫、急診、兒科、心臟科…）都歡迎以 `examples/<specialty>-<scenario>.json` 形式提交。

要求：
- 完全合 spec（用 `spec/v0.1/schema/brief-template.schema.json` 驗證）
- 不超過 20 題
- 至少 zh-TW + en 兩個 locale
- 在 PR 描述附 5-10 行的「設計考量」說明
- **不要** 在 example 中嵌入紅旗 module 內部邏輯（只填 `redFlagModuleRefs` 接口參考）

審核時間目標：14 天內完成 first-pass review。

#### 2. 文件修正 / 範例補充 / 教學

`docs/`、`README.md`、`spec/v0.1/README.md` 等文件的錯字、語句、舉例補充都歡迎。

審核時間目標：48-72 小時。

#### 3. i18n locale 翻譯

文件、範例 brief 的非主流 locale（vi、es、id、ja、ko）翻譯。

要求：
- 必須是 native speaker 或具備等同水準
- 提交時附簡短自我介紹（PR description 即可）

#### 4. JSON Schema validator 修正

`spec/v0.1/schema/brief-template.schema.json` 的 bug 修正、邊界條件補強。

要求：
- 必須附 test case（至少 1 個 valid + 1 個 invalid）
- 修正必須**向後相容** — 否則需走 RFC

---

### ❌ 不接受的貢獻類型

下列內容是 **De Novo Orthopedics 商業 IP**，不會接受 PR（請見 [`LICENSE_STRATEGY.md` §6](LICENSE_STRATEGY.md)）：

- 任何 red flag module 的**內部邏輯**（threshold / 權重 / escalation copy / scoring 算法）— **只能**貢獻接口名稱（`redFlagModuleRefs` 字串）
- Composer 編輯器 UI（任何 GUI for authoring）
- 後端 SOAP / LLM 處理 pipeline
- 病患 PII 處理層
- iRehab Doctor PWA 的工作流設計
- Outcome registry 資料管線

如果你 PR 包含上述內容，會被 close。

---

## Governance — 怎麼運作

本 repo 採 **review-only governance**：

- 所有變更走 **PR-based**
- **無外部 maintainer / committer 角色**（v0.1 階段）
- PR 審查者 = De Novo Orthopedics 維護團隊
- 重大架構變更（spec breaking change、license boundary 調整、§6 forever-closed 修改）= **單方決策權保留 De Novo**

**為什麼這樣設計：** schema 是醫療判斷的入口，其品質直接影響臨床安全。外部 unfettered commit 會損害這個保證。

### 未來治理路徑（roadmap）

- **v0.x**：review-only by De Novo
- **v1.x**：考慮加 trusted committer 名單（active contributor + 2 年以上 commit history）
- **v2.x+**：若 schema 成為廣泛使用標準，考慮成立 governance foundation（可能 fork 出獨立基金會持有 schema 治理權）

---

## PR 流程

1. **先開 issue 討論**（特別是 spec 變更）— PR 前確保方向對
2. Fork repo
3. Branch 命名建議：`example/<specialty>` / `docs/<topic>` / `i18n/<locale>` / `validator/<bug-id>`
4. Commit message：用 imperative mood（"add foo" 不是 "added foo"）
5. **每個 PR 一件事** — 不要混合 example + i18n + spec 修正
6. 在 PR description 列出：
   - 改了什麼
   - 為什麼改
   - 如何驗證（特別是 example PR 需附驗證指令輸出）
7. 提交 PR → 等 review

審核時間：
- 文件修正 / 範例補充：48-72 小時
- Specialty pack：14 天內 first-pass
- Spec 變更：先 RFC（至少 14 天 comment period）

---

## RFC（Request for Comments）

**何時需要 RFC：**

- Spec 結構變動（新增 type、修改既有 type 行為）
- 破壞性變更
- 任何影響 §6 forever-closed boundary 的提案
- 大規模重構建議

**RFC 流程：**

1. 開一個新 issue，title 用 `[RFC] <title>` 格式
2. 內文包含：
   - **Motivation**（為什麼）
   - **Proposed change**（具體要怎麼改）
   - **Backward compatibility**（向後相容性影響）
   - **Alternatives considered**（替代方案 + 為什麼不選）
3. 至少 14 天 comment period
4. De Novo 維護團隊在 comment period 結束後做最終決定
5. 決策結果回貼到 issue，並進 next minor release 的 changelog

---

## Code of Conduct

- 對事不對人
- 接受 review，不接受 personal attack
- 不傳播違反醫療倫理的內容（例如：宣稱可診斷的紅旗判斷 / 違反 PHI 規範的範例）
- 違反者直接 ban，無申訴

---

## Disclosures / Conflict of Interest

如果你是醫療器材廠商、HIS 廠商、第三方平台 vendor 提交 specialty pack PR，請在 PR description 揭露：

- 你的雇主 / 商業背景
- 該 specialty pack 是否準備整合進你公司的商業產品

這些**不會**影響 PR 是否被接受（schema 開源初衷就是給商業整合用），但揭露可幫助 reviewer 評估 conflict of interest。

---

## License of Contributions

提交 PR 即視為**同意以本 repo 雙授權方式公開貢獻內容**：

- 如果你貢獻的是程式碼（schema / validator）→ Apache-2.0
- 如果你貢獻的是文件 / 範例 → CC BY 4.0

無需簽 CLA，但請保證：

1. 你是貢獻內容的著作權人，或已得授權
2. 內容不違反第三方權利
3. 你同意以本 repo 授權公開

---

## Questions?

- 一般問題 → [GitHub Issues](https://github.com/Denovortho/open-irehab-brief-schema/issues)
- 商業整合 / 授權 → `service@denovortho.com`
