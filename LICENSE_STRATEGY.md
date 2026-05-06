# License Strategy

> 本文件說明 **iRehab Brief Schema** 的雙授權結構、邊界、以及哪些東西**永遠不會**加入本 repo。

---

## §1 雙授權結構

| 內容類型 | 授權 | 範圍 |
|---|---|---|
| **Schema 檔** (`spec/v0.1/schema/*.json`) | Apache-2.0 | 機器可讀格式定義（含 patent grant）|
| **Validator 程式碼**（未來加入時）| Apache-2.0 | 任何隨 schema 提供的驗證工具 / CI helper |
| **Spec 文件**（`spec/`、`docs/`、`README.md`、`CONTRIBUTING.md` 等）| CC BY 4.0 | 文件、教學、範例 |
| **範例 brief instance**（`examples/`）| CC BY 4.0 | 可直接用於商業整合的範例 |

---

## §2 為什麼分兩種授權

- **Apache-2.0（程式碼）**：含**明確的專利授權**（patent grant），對 HIS 廠商、第三方平台採用 schema validator 時提供強保護
- **CC BY 4.0（文件 / 內容）**：規定**標示來源**即可商用、改編、再散布，是國際通用的開放內容標準

兩個授權**不互通** — schema 檔不適用 CC BY 4.0，文件不適用 Apache-2.0。引用時請依檔案類型適用對應條款。

---

## §3 準據法（Governing Law）

本 repo 的所有授權條款效力以**中華民國（台灣）法律**為準據，並以**臺灣臺中地方法院**為第一審管轄法院。

此宣告**不修改 Apache-2.0 / CC BY 4.0 條文本身**（兩者皆禁止本文修改），而是放在本 repo 與 `README.md` 中作為授權人的**準據法選擇陳述**（governing law statement）。詳見 [`docs/governing-law.md`](docs/governing-law.md)。

---

## §4 商標

「**iRehab**」、「**愛復健**」、「**De Novo Orthopedics**」、「**谷盺生物科技**」是 **De Novo Orthopedics 谷盺生物科技股份有限公司** 在中華民國（台灣）的註冊商標：

- iRehab 愛復健及圖：申請號 109004530，註冊號公告 — 涵蓋 Nice 第 9、10、35、44 類
- 谷盺生物科技 / DE NOVO ORTHOPEDICS：申請號 113069840、註冊號 02477131 — 涵蓋 Nice 第 5、9、10、35、42、44 類

**Apache-2.0 / CC BY 4.0 授權不授予商標使用權**。在 fork、衍生、商業整合 schema 時：

- ✅ 可以說「本產品支援 iRehab Brief Schema 格式」、「相容於 iRehab Brief Schema」
- ✅ 可以引用本 schema 進行學術論文、blog 寫作（依 CC BY 4.0 attribution）
- ❌ 不可將「iRehab」、「愛復健」、「De Novo」用作自己產品的名稱、logo、品牌
- ❌ 不可暗示自己的 fork / 衍生品「**是**」 iRehab 官方版本

商標問題請來信 `service@denovortho.com`。

---

## §5 Spec 變更管理

| 變動類型 | Major | Minor | Patch |
|---|---|---|---|
| Breaking（破壞性，無向後相容）| ✓ | | |
| 新增 type / 新增可選欄位 | | ✓ | |
| 文件 / 範例修正 | | | ✓ |

每次 release tag `v<MAJOR>.<MINOR>.<PATCH>`，變更紀錄寫入 [`CHANGELOG.md`](CHANGELOG.md)。

破壞性變更需在 [Issues](https://github.com/Denovortho/open-irehab-brief-schema/issues) 開 RFC 討論串，至少 14 天 community comment period。

---

## §6 Forever-Closed Boundary

下列元件**永遠不會**加入本 repo，永遠不開源。它們是 **De Novo Orthopedics 商業 IP**，是公司護城河：

### 6.1 Red flag pipeline 內部邏輯

具體包含：

- 紅旗 threshold（每題的觸發門檻數值）
- 權重 / scoring 算法
- Multi-question 組合判斷邏輯
- Escalation copy（觸發後的訊息文案）
- Severity tier 判定
- 後端 LLM 二階段判斷整合

**為什麼 closed**：這是醫療判斷邏輯，open 後失去 quality gate。同時是 De Novo 真正的護城河。

### 6.2 AI SOAP 整合 prompts + tuning

具體包含：

- Brief → SOAP 整理用的 LLM prompts
- Few-shot examples（含內部 case 資料）
- Output post-processing rules
- Model selection / tuning policies

**為什麼 closed**：Prompt engineering 是持續演進的商業 IP，與 De Novo 醫師端工作流深度耦合。

### 6.3 病患 PII 處理程式碼

具體包含：

- PHI 隔離 / 加密層
- 同意框（consent framework）程式碼
- TTL / 自動刪除機制實作
- Audit log 結構

**為什麼 closed**：個資處理碰不得。任何 OS 變動都會引發合規 audit，風險不對等。

### 6.4 iRehab Doctor PWA 的 UI / UX

具體包含：

- Doctor 端工作流程設計
- Composer 編輯器 UI
- AI SOAP review screen
- Override 操作流程

**為什麼 closed**：工作流程設計是商業差異化。

### 6.5 Outcome registry 資料管線

具體包含：

- Implant outcome registry pipeline（若實作）
- 醫材廠 / 商業險 partnership data flow
- B2B revenue stream 相關程式碼

**為什麼 closed**：B2B 收入流核心。

### 6.6 Composer 編輯器（GUI for schema authoring）

具體包含：

- Visual editor 程式碼
- Drag-drop UI
- Authoring shortcuts
- Specialty pack 切換 UX

**為什麼 closed**：醫師體驗的關鍵差異化。**本 schema 開源不等於編輯器開源** — 任何人可以拿 spec 自己做 editor，但 De Novo 自家 Composer 是商業產品。

---

## §7 Contribution Boundary

外部 contributor 可貢獻的範圍：

- **可以**：specialty pack 範例 brief template（會 review）、文件修正、範例補充、i18n locale 翻譯、JSON Schema validator 修正
- **不可以**：red flag module 的「內部邏輯」（只能貢獻**接口參考名稱** `redFlagModuleRefs`，不能貢獻實作）、Composer / PWA UI、後端 SOAP 處理

詳見 [`CONTRIBUTING.md`](CONTRIBUTING.md)。

---

## §8 Versioning + EOL

- v0.x 為**預備 release**，不保證向後相容
- v1.0+ 才開始 forward-compat 承諾
- EOL policy：每個 major 版本至少維護 24 個月
- 任何 deprecation 都會在 minor release 提前公告至少 1 個 minor cycle

---

## Questions?

關於授權、商標、forever-closed boundary 任何疑問：

- 一般問題 → [GitHub Issues](https://github.com/Denovortho/open-irehab-brief-schema/issues)
- 商業整合 / 授權 → `service@denovortho.com`
- 商標問題 → `service@denovortho.com`

---

## Audit Trail

本 license strategy 由 **De Novo Orthopedics** 委任律師確認（2026-05-06，26/26 預擬結論全綠燈），不可單方變更核心結構。任何 §1 / §6 重大調整需重新法律 review。
