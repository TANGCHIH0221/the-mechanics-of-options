# Options as Physical Systems: 衍生品交易的結構與實戰模擬
### A Trader's Journal: Bridging Theoretical Physics and Market Microstructure

> 「市場不僅僅是隨機漫步，它是一個受制於資金流動、結構限制與自適應的複雜物理系統。」

---

##  關於本專案 (About This Project)

本專案是一份**持續更新的衍生品交易研究與訓練日誌**。

作為一名具有物理背景的學生，我希望跳脫只依賴 Black–Scholes 這類**靜態封閉解模型**的思維，  
改用「第一原理思考（First Principles Thinking）」與「物理直覺」，去理解：

- **隱含波動率曲面如何變形？**
- **造市商與資金流如何影響價格路徑？**
- **為什麼名義上 Long Vega 的部位，遇到崩跌時仍然會虧損？**


---

##  核心目標 (Core Objectives)

本專案的目標是逐步建立一套，專注於**結構性錯誤定價**的交易體系：

1. **觀察能量**  
   - 研究波動率曲面（Volatility Surface），視為市場的「位能分佈」  
   - 找出市場對某些履約價 / 期限「過度恐懼或過度樂觀」的區域  

2. **理解場論**  
   - 以造市商 Gamma / Vega 部位，來理解價格附近的「引力／斥力」  
   - 探討 dealer hedging flows 如何形成短期價格「重力場」  

3. **掌握流體**  
   - 將 Charm、Vanna 等視為「流體力學」中的流速與剪切力  
   - 分析時間流逝與波動率變動如何改變部位的方向與槓桿

---

##  核心哲學：物理 × 金融 (Physics Meets Finance)

我相信金融市場與物理系統有著高度類似的結構。  
以下是本專案常用的類比框架：

| 物理概念 (Physics)      | 金融對應 (Finance)                            | 實戰應用 (Trading Edge) |
|-------------------------|-----------------------------------------------|-------------------------|
| 位能井 (Potential Well) | 造市商 Gamma 擁擠 (Dealer Long Gamma)        | 識別價格被「釘住」的區間，設計 Iron Butterfly / Short Straddle 收 Theta。 |
| 相變 (Phase Transition) | 負凸性爆發、Vol Regime Shift                  | 預判平靜 → 湍流的臨界點，避免 Short Gamma / Short Vega 的尾部風險。 |
| 阻尼運動 (Damping)      | 均值回歸 (Mean Reversion)                    | 觀察資金流如何「抑制」或「加速」波動，選擇 Reversion vs Breakout 策略。 |
| 熵與時間 (Entropy)      | Charm (∂Δ/∂t)、時間衰減                      | 利用週末 / 假期效應，評估 OTM 選項 Delta 衰減帶來的隱含槓桿變化。 |

這些不是為了華麗比喻，而是用來**建立可重複的直覺與決策框架**。

---
##  Field Guide: The Physical Operators (Greeks Reimagined)

在實戰中，我將標準希臘字母視為物理系統中的**算符 (Operators)**。
以下速查表修正了傳統教科書的靜態觀點，重點關注 $T \to 0$ 的奇異點行為與高階交互作用：

| Operator (Greek) | 物理定義 (BSM 視角) | 時間尺度 ($T$) | 波動率尺度 ($\sigma$) | 空間分佈 (Moneyness) |
| :--- | :--- | :--- | :--- | :--- |
| **Delta ($\Delta$)** | **Velocity (速度)**<br>$N(d_1)$ | **Step Function**<br>$T \to 0$ 時二元化 (0 或 1) | **Smoothing**<br>$\sigma \uparrow$ 使曲線變平緩 | ATM $\approx$ 0.5 |
| **Gamma ($\Gamma$)** | **Curvature**<br>$\frac{\partial^2 C}{\partial S^2}$ | **Singularity**<br>$\propto \frac{1}{\sqrt{T}}$ (**反比**)<br>快到期時 ATM 爆炸 | **Concentration**<br>$\propto \frac{1}{\sigma}$ (**反比**)<br>低波時能量更集中 | **ATM 最大**<br>向兩側迅速衰減 |
| **Theta ($\Theta$)** | **Entropy (熵增)**<br>$\frac{\partial C}{\partial t}$ | **Acceleration**<br>$\propto \frac{1}{\sqrt{T}}$ (**反比**)<br>最後 30 天加速腐爛 | **Cost of Convexity**<br>$\propto \sigma$ (**正比**)<br>波動越大，租金越貴 | **ATM 最大** (絕對值)<br>OTM 衰減較慢 |
| **Vega ($\nu$)** | **Temperature Sensitivity**<br>$\frac{\partial C}{\partial \sigma}$ | **Time Weight**<br>$\propto \sqrt{T}$ (**正比**)<br>遠月對 Vol 最敏感 | **Complex**<br>$\sigma$ 改變鐘形曲線寬度 | **ATM 最大**<br>深 ITM/OTM 趨近 0 |
| **Vanna** | **Slope of Vega (斜率流)**<br>$\frac{\partial \Delta}{\partial \sigma} = \frac{\partial \nu}{\partial S}$ | **Flow**<br>$\propto \sqrt{T}$ (不顯著)<br>隨時間穩定 | **Complex**<br>主要影響 Skew 的斜率 | **ATM $\approx$ 0**<br>**Wings (OTM/ITM) 最大** |
| **Vomma** | **Vol Convexity (波動凸性)**<br>$\frac{\partial^2 C}{\partial \sigma^2}$ | **Long Duration**<br>$\propto T$ (**強正比**)<br>長天期才有大 Vomma | **Inverse**<br>低波時 Vomma 較大 | **ATM 最大** |
> **Physics Note:** 注意 $\Gamma$ 與 $\Theta$ 的對稱性 ($\Theta \approx -\frac{1}{2}\sigma^2 S^2 \Gamma$)。這代表在物理上，我們無法創造能量，只能在「時間耗損」與「價格凸性」之間進行交換。

---

##  專案結構與訓練模組 (Project Structure & Modules)

本專案以「訓練模組（Modules）」的方式組織內容，並搭配每日訓練紀錄。

### Module 0：市場作為決策系統 (Decision Theory & Physics)

**WHY：** 所有的 Alpha 最終都源於決策的品質，而非運氣。
本模組建立交易的「公理體系」，透過賽局理論與統計物理，重新定義何謂「正確的決策」。

- **研究重點**：
  - **遍歷性 (Ergodicity)**：為何正確的決策可能導致破產？(時間平均 vs 系統平均)
  - **熵增與同質化 (Entropy)**：為何資訊擁擠會導致 EV 歸零？
  - **相變 (Phase Transition)**：如何利用凸性 (Convexity) 達到階級躍遷的「逃逸速度」？

- **討論問題 (The Axioms)**：
  - * [x] **[射龍門賽局與階級流動](Training_Logs/000_Decision_Theory_and_Class_Mobility.md)**
     * **重點 0.1 (-EV 的本質)：** 選擇比結果重要。富人與賭鬼的決策邏輯可能相同，差別在於部位規模管理 (Kelly Criterion)
   
---  

### Module 1：微觀結構與重力場(Microstructure & Gamma)

- **研究重點**：  
  - Open Interest 分佈  
  - Dealer Gamma / Vega 曝險  
  - Hedging flows 對價格路徑的影響
- **討論問題**：
-  * [x] **[ Pining Effect](Training_Logs/004_pinning_effect.md)**
    * **重點：** Gamma 重力井的狄拉克奇異點 (Dirac Delta)、動態避險的負回饋循環 、以及結算日的 Pin Risk 。  


---

### Module 2：希臘字母的流體力學 (Advanced Greeks)

- **研究重點**：  
  - 超越 Delta / Gamma / Theta 的二階希臘值：Charm, Vanna, Vomma …  
  - 在不同市場情境中，這些二階效應如何主導 P/L
- **典型問題**：  
  - Charm：為何週末時間流逝，會讓原本「看似中性」的部位，方向性偏移？  
  - Vanna：在崩盤時，Vol 上升 + Spot 下跌如何共同放大 Delta 變化？

---

### Module 3：恐懼的幾何形狀 (Volatility Surface)

- **研究重點**：  
  - 期限結構（Term Structure）與微笑曲線（Skew / Smile）的變形  
  - 不同 Regime 下，前端與後端 IV 反應速度的差異
- **討論問題**：
-  * [x] **[波動率曲面動態與結構化偏斜](Training_Logs/001_Vol_Dynamics_and_Skew.md)**
    * **重點：** 崩盤時的 Sticky Strike 識別、Vanna 陷阱、以及 1x2 Ratio Spread 的 Gamma Flip 風險。
-  * [x] **[ 期限結構與 Charm](Training_Logs/002_Term_Structure_and_Calendars.md)**
    * **重點：** 遠期波動率 (Forward Vol) 的能量守恆、Calendar Spread 的 Gamma 陷阱、以及週末的 Charm 效應。


---


### Module 4：結構化策略設計 (Strategy Architecture)

- **研究重點**：  
  - 以「物理情境」為前提設計選擇權結構  
  - 分析各種策略的 failure modes（失效模式）
- **討論問題**：
- * [x] **[Ratio Backspreads](Training_Logs/018_ratio_backspread.md)**
    * **重點：**  Ratio Backspread 的死亡谷 (Valley of Death)、Theta–Gamma 對抗、Gamma Flip 相變點、BEP 主方程、Skew Flattening 對 Weighted Vega 的反殺、以及高峰度 (Kurtosis) 在凸性策略中的關鍵角色。。
     

---


## 技術與工具 (Tech Stack)

- Python（模擬與資料處理）
- NumPy / pandas（數值運算與時間序列）
- Matplotlib（簡易可視化）
- 之後將逐步加入：  
  - 基礎 QuantLib / 自製定價模組  
  - 簡單 backtesting / scenario simulation 工具

---
# AI-Assisted Research Framework  
> **AI is not a shortcut — it is an accelerator for deliberate practice.**

本專案同時使用 **人工智慧（AI）作為研究輔助工具**，  
讓訓練流程更高效、更結構化，也更接近真實交易室的推理方式。

---

##  Daily Quant Reasoning Training

我利用 AI 自行生成每日訓練題，包括：

- 市場結構邏輯（Microstructure Reasoning）  
- 波動率體制（Vol Regime）判讀  
- 高階希臘值（Charm / Vanna / Vomma）動態推理  
- 崩跌 / 爆漲時的 P&L 拆解  
- 極端情境下的直覺訓練  

透過 AI，我可以在短時間內接觸大量極端市場案例，  
加速累積交易直覺與錯誤修正能力。

---

##  AI-Driven Simulation & Verification

AI 協助我快速構建 Python 模型與實驗，用於：

- 測試策略的失效模式（Failure Modes）  
- 分析 Greeks 隨 Spot / IV / t 的自動演化  
- 模擬 Sticky-Strike / Sticky-Delta / Sticky-Moneyness  
- 比較 Calendar / Condor / Ratio Spread 在不同 regime 的行為  

我先以物理直覺提出假設（Hypothesis），  
再用 AI + Python 進行數值實驗驗證。

---

##  Meta-Learning：透過對話迭代推理

我將 AI 視為「研究室助理 + PM 教練」：

- 讓 AI 用問題 challenge 我的推理  
- 偵測盲點、提出反例  
- 協助分析虧損背後的 root-cause  
- 模擬真實交易室的 decision-making flow  

透過這種方式，我把直覺轉化成可檢驗的模型與假說。

---

## Market-Facing Readiness

AI 也協助我：

- 快速理解市場新聞與 macro catalyst  
- 分析 vol surface 的動態變化  
- 模擬重大事件（NFP / CPI / Crash）的波動率反應  
- 理解不同 regime 下的 Theta / Vanna / Charm 路徑  

使我能將：

**物理直覺 → 策略設計 → 市場實況**  
三者連結起來。

---

## Why AI is a Structural Edge

> **我不是被動使用 AI，而是把 AI 當成方法論的一部分。**

AI 讓我能：

- 更快 iterate  
- 更快驗證想法  
- 更快理解極端行情的風險暴露  
- 更快累積市場直覺  
- 更快建立屬於自己的交易研究體系  

AI 是我訓練流程中的 最重要的工具之一 。

---

##  長期願景 (Long-term Vision)

- 建立一套屬於自己的 **“選擇權物理學” 筆記**  
- 系統性整理各策略的**結構風險與失效模式**  
- 強化對 Vol、Greeks 與 Microstructure 的直覺  
- 以此為基礎，成為能獨立設計、檢驗與執行策略的 **衍生品交易員（Options / Vol Trader）**

---

