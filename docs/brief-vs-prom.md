# Brief vs PROM — 概念辨別與 Audit Rationale

**版本**：v0.1
**日期**：2026-05-05
**作者**：林佳緯（Tom）+ Claude（Haiku 4.5）+ Codex（review）
**目的**：說明 iRehab Brief 與標準化 PROM 的概念差別，以及為什麼開源 Brief examples 前必須跑 PROM audit。

---

## 1. 兩個概念的本質差別

| 概念 | 是什麼 | 例子 | 版權狀態 |
|---|---|---|---|
| **Brief** | iRehab 自己設計的**症狀分流問卷** | 「主訴」「疼痛部位」「警示症狀」 | 你寫的，你的 |
| **PROM** | 國際**標準化驗證量表** | KOOS、PHQ-9、ODI、ASES | **作者保留版權**，多數需授權 |

### 核心差別

- **Brief** = 自由設計，目的是**看診前資訊收集**與分流。題目本身沒有臨床驗證的 reference range，也沒有 cross-study comparison 的功能。
- **PROM** = 經過統計驗證、有 reference range、可比較跨研究——多數有授權限制，Bibliographic citation 是必要的。

---

## 2. 為什麼開源 Brief 前要跑 PROM audit？

兩個真實風險：

### 2.1 開發過程**無意夾帶**

開發者設計 Brief 題目時可能：
- 「啟發自」KOOS / ODI / DASH 等 PROM
- 從醫學 KB / 文獻 review 中潛意識複製題目結構
- 改寫量表項目但保留識別性 wording

**法律風險**：即使是改寫，也可能被認定為**衍生作品**（derivative work）。版權法保護的是「表達」（expression），不只「逐字 copy」。

iRehab 平台同時內部維護 ~20 個 PROM instruments（屬商業模組，**不在本 schema 範圍**）。Brief 的問卷題庫應與 PROM 模組嚴格分離，但開源前必須做 audit 確認沒有 PROM 詞句意外混入 Brief 公開素材。

### 2.2 開源後**外部貢獻者夾帶**

醫師發 PR 貢獻 specialty pack 時可能：
- 直接把 PHQ-9 / GAD-7 整套題目塞進來
- 認為「臨床常用就是 public domain」（錯誤觀念）
- 缺乏 license 自查意識

**對策**：PR template 必須要求貢獻者**自查並簽署**未夾帶受版權保護量表。

---

## 3. PROM Audit 的真正意義

PROM 清洗 ≠ 「清掉一堆 PROM 題目」。

實際意義是：**audit trail**——「我們做過自查，舉證 Brief examples 是自製，不是衍生 PROM」的書面紀錄。

### 這份 audit 的價值

| 對象 | 用途 |
|---|---|
| **律師** | 開源 license 確認時的書面佐證（不是請律師研究 PROM 法律狀態，而是 confirm 自查邏輯合理） |
| **醫院 IT** | 採用 Brief Schema 前的合規查核，敢蓋章 |
| **未來 PR 貢獻者** | 示範「自查標準」，作為 PR template 的範本 |
| **De Novo 自身** | 防衛性紀錄，未來若被指控版權侵犯有 due diligence 證據 |

### 工作量預估

如果 Brief 設計乾淨（純 iRehab 自製），audit 可在 1-2 小時內完成：
- ~30 分鐘：grep + 比對 14 個 PROM 特徵題目
- ~30-60 分鐘：撰寫 audit doc
- 結論若全 PASS → 結案
- 若有 medium/high risk → 再花 0.5-1 天評估改寫或申請授權

**這跟 Codex 第一輪估計的 1-2 天 PROM 清洗工作量不同**——因為 Codex 假設 Brief 含大量 PROM 痕跡，而實際應該乾淨。

---

## 4. 14 個重點 PROM 名單（audit 範圍）

依 Codex 建議，以下為主要疑似來源（含縮寫、原作者、版權狀態）：

| PROM | 領域 | 原作者 / 機構 | 版權概況 |
|---|---|---|---|
| **PHQ-9** | 憂鬱 | Spitzer/Williams/Kroenke (Pfizer 1999) | US public domain；台灣版授權狀態未明 |
| **GAD-7** | 焦慮 | Spitzer (Pfizer) | US public domain；台灣版未明 |
| **KOOS** | 膝關節 | Roos/Lohmander 1998 | 學術免費；商業需授權 |
| **HOOS** | 髖關節 | KOOS-derivative | 同 KOOS |
| **DASH** | 上肢 | IWH 1996 | 商業使用需授權 |
| **QuickDASH** | 上肢簡版 | IWH | 同 DASH |
| **ASES** | 肩肘 | Richards 1994 | 學會所有 |
| **ODI** | 下背 | Fairbank 1980 | 商業需授權 |
| **NDI** | 頸部 | Vernon/Mior 1991 | 商業需授權 |
| **EQ-5D** | 通用 QoL | EuroQol Group 1990 | 商業需授權（學術免費） |
| **PROMIS** | 多領域題庫 | NIH | Public domain（HealthMeasures） |
| **WOMAC** | 膝/髖 OA | Bellamy 1988 | 商業需授權 |
| **Oxford scores** | 關節（OKS/OHS/OSS） | Oxford 大學 | 商業需授權 |
| **Disease-specific** | 各專屬 | 各 specialty | 須個案查證 |

---

## 5. Audit 結論的三種可能

| 結論 | 意義 | 後續行動 |
|---|---|---|
| **A. 完全乾淨** | Brief 系統獨立，PROM 走別的模組 | 半小時 audit 結案 |
| **B. 部分啟發** | 某些題目 wording 接近 PROM | 改寫或加 attribution |
| **C. 整段引用** | 有題組是直接從 PROM copy | 必須移除或申請授權 |

De Novo Orthopedics 對 iRehab Brief 已進行內部 PROM license audit，結論為情境 A（完全乾淨 — Brief 與 PROM 模組嚴格分離）。本文件公開的目的，是讓未來貢獻 specialty pack 的醫師意識到「PROM 是高風險地雷區」，不要在 PR 中引入 PROM 改寫題目。

---

## 6. Related Documents

- [`README.md`](../README.md) — Repo 入口
- [`spec/v0.1/authoring-spec.md`](../spec/v0.1/authoring-spec.md) — Authoring format spec
- [`LICENSE_STRATEGY.md`](../LICENSE_STRATEGY.md) — 雙授權邊界
- [`CONTRIBUTING.md`](../CONTRIBUTING.md) — 貢獻流程
