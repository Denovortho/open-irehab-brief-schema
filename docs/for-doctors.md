# For Doctors — 給醫師的快速入口

> 你不需要會工程。把這個 repo 連結交給你慣用的 AI 助手（ChatGPT / Claude / Gemini），它會根據這份 schema 幫你規劃自己科別的診前問卷。

---

## 30 秒了解這是什麼

**iRehab Brief Schema** 是一份**問卷格式藍圖** — 它告訴 AI / 工程師「一份結構化的診前症狀問卷該怎麼寫」。

- 不是醫療器材（不做診斷判斷）
- 不是 SaaS 產品（沒有要你登入或註冊）
- 是一個**可以被任何人讀、任何人實作**的標準藍圖

把這個 repo 的連結（[`github.com/Denovortho/open-irehab-brief-schema`](https://github.com/Denovortho/open-irehab-brief-schema)）丟給 AI，請它幫你做下面這些事。

---

## 你今天就可以做的 4 件事

### 1. 為自己的科別設計一份結構化診前問卷

複製貼上這個 prompt：

```
我是【你的科別 — 例如「身心科」】醫師，門診常見的主訴是【列 3-5 個 — 例如「焦慮、失眠、情緒困擾」】。

請根據這個 repo 的 spec：
https://github.com/Denovortho/open-irehab-brief-schema

幫我設計一份 8-10 題的結構化診前問卷，輸出符合 brief-template.schema.json 的 JSON 格式。

要求：
- 中英文雙語（zh-TW + en）
- 至少包含 1 題 scale 類型（疼痛 / 嚴重度 / 頻率）
- 至少包含 1 題 boolean 類型（是 / 否）
- 包含 1-2 個紅旗判斷題（redFlagModuleRefs）
- 紙本可填、線上可填都要兼容（題目簡短）
- 不要直接抄已知 PROM 量表（避免授權問題）
```

---

### 2. 把現有的紙本問卷數位化

如果你診所或醫院已有一份紙本問卷想轉成數位版：

```
我這份紙本診前問卷有【N】題（內容附下方 / 圖片）：

【貼上現有題目，或附圖】

請參考這個 repo 的 spec：
https://github.com/Denovortho/open-irehab-brief-schema

把上面這份問卷轉成符合 brief-template.schema.json v0.1 的 JSON 格式。

要求：
- 維持原意，不要新增或刪減題目
- 結構化（找出選項型題目改用 select / multiselect / boolean）
- 補上 zh-TW + en 雙語
- 找出哪些題目應該綁紅旗 module（外傷 / 神經血管 / 嚴重疼痛 / 急性症狀）
```

---

### 3. 規劃如何把問卷接進診所 / 醫院系統

```
我的診所 / 醫院目前用的是【你的系統 — 例如「自家網站」/「HIS 廠商 X」/「LINE OA」】。

我已經有一份符合 iRehab Brief Schema v0.1 的 JSON 問卷（連結 / 內容）。

請幫我規劃：
1. 病患怎麼接觸到這份問卷（QR / 簡訊 / 院內網站連結）
2. 病患填完後資料怎麼回傳到我們
3. 我做為醫師，看診前怎麼看到這份結構化資料
4. 隱私與資料保存（病患沒看診就刪、看診後保存幾年）

請給具體的步驟流程（包括誰要做什麼、用哪個工具）。
```

---

### 4. 為自己的學會 / 醫院群組撰寫一份 specialty pack 提案

如果你想代表自己的學會 / 醫院 / 科別貢獻一份「本科共通」的 brief 範例給社群：

```
我代表【學會名稱 / 科部 / 醫院群】，想為【科別】貢獻一份共通診前問卷模板。

請根據這個 repo：
https://github.com/Denovortho/open-irehab-brief-schema

幫我做兩件事：
1. 設計一份 10-15 題的 specialty pack（已涵蓋本科常見初診情境）
2. 寫一份 PR description，說明：
   - 為什麼選這 10-15 題（臨床根據）
   - 哪些題綁了紅旗 module 接口
   - 在地化 / 國際化考量（i18n）
   - 為什麼某些「常見題」沒選（取捨理由）

PR 流程請參考 CONTRIBUTING.md。
```

提交方式：在 [https://github.com/Denovortho/open-irehab-brief-schema](https://github.com/Denovortho/open-irehab-brief-schema) 開 PR，或直接 email `service@denovortho.com`。

---

## 如果你的 AI 不知道要往哪看

新一代的 AI（ChatGPT 4.x、Claude Sonnet 4.6+、Gemini 2.5+）通常拿到 GitHub URL 就會自己抓 README 跟主要 spec 看。如果你發現 AI 沒抓對重點，可以指引它讀這幾個檔案：

1. [`docs/overview.md`](overview.md) — 5 分鐘整體介紹
2. [`spec/v0.1/authoring-spec.md`](../spec/v0.1/authoring-spec.md) — 完整 schema 規格
3. [`spec/v0.1/schema/brief-template.schema.json`](../spec/v0.1/schema/brief-template.schema.json) — JSON Schema reference
4. [`examples/orthopedics-complete.json`](../examples/orthopedics-complete.json) — 一份完整範例
5. [`spec/v0.1/red-flag-modules.md`](../spec/v0.1/red-flag-modules.md) — 紅旗 module 接口

直接告訴 AI：「請先讀以上 5 個檔案再開始設計。」

---

## 第一次嘗試的成果不會完美，但⋯⋯

讓你**今天就有起步**，而不是等到「有工程師 / 有預算 / 有 IT 部門排期」才開始。

跟 AI 討論幾輪，產出的第一版可以拿去：

- 給你的同事或學弟妹看
- 拿去給你診所 / 醫院 IT 部門當 baseline
- 拿去當論文 method section 的引用對象
- 拿去當合作對象（HIS 廠商 / 第三方平台）的「我希望我的問卷長這樣」溝通工具

完美不是這個 repo 的目標。「**讓你能起步**」才是。

---

## 想反過來：你要的東西本 schema 沒有？

歡迎在 [GitHub Issues](https://github.com/Denovortho/open-irehab-brief-schema/issues) 開 issue 提出。常見的 feedback 類型：

- **缺少 type**：「我需要 file upload / signature / image annotation 類型的題目」
- **缺少 i18n locale**：「我需要泰文 / 越南文版本的官方 example」
- **紅旗 module 缺**：「我覺得本科應該要有 XXX 風險判斷的 module 接口」
- **法規問題**：「在 X 國家用本 schema 有什麼合規問題？」

我們會在 48-72 小時內回應 issue。Spec 變更會走 RFC 流程（≥ 14 天 community comment）。

---

## 商業整合 / 學會合作

學會、醫院群、HIS 廠商、保險公司想做正式合作（institutional license / 跨醫院 standardized brief）：

`service@denovortho.com`

或在 GitHub Issues 開一個 `[partnership]` prefix 的 issue。

---

## 最後

這個 repo 的設計目標是**讓「自己動手」這件事的門檻降到一個下午**。如果你發現有任何環節讓你動不了，那就是我們這份 spec 沒寫清楚 — 歡迎開 issue 罵我們。

去把那份 GitHub URL 丟給你的 AI 吧。
