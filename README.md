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

> **本專案聚焦於拆解策略背後的底層驅動：
> 
> Microstructure、Volatility Surface、Term Structure 與 Greeks Dynamics  
> 如何共同影響 P/L 的生成與風險暴露。**

---

##  核心目標 (Core Objectives)

本專案的目標是逐步建立一套，專注於**結構性錯誤定價（Structural Mispricing）**的交易體系：

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

##  專案結構與訓練模組 (Project Structure & Modules)

本專案以「訓練模組（Modules）」的方式組織內容，並搭配每日訓練紀錄。

### Module 1：造市商的重力場 (Microstructure & Gamma)

- **研究重點**：  
  - Open Interest 分佈  
  - Dealer Gamma / Vega 曝險  
  - Hedging flows 對價格路徑的影響
- **典型問題**：  
  - 為什麼價格常黏在整數關卡（pinning effect）附近？  
  - 如何判斷目前環境偏向「正 Gamma（抗跌、抗漲）」或「負 Gamma（助跌、助漲）」？
  - * [x] **[Session 001: 波動率曲面動態與結構化偏斜狩獵](Training_Logs/0251126vol_dynamic_and_skew.md)**
    * **重點：** 崩盤時的 Sticky Strike 識別、Vanna 陷阱、以及 1x2 Ratio Spread 的 Gamma Flip 風險。

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
- **典型問題**：  
  - Contango → Flatten → Backwardation 的過程中，Calendar Spread 如何從「理論上 Long Vega」變成「實務上虧損」？  
  - 當標的價格沿著 Skew 下滑時，ATM → OTM 的 IV 遷移如何影響部位價值？

---

### Module 4：結構化策略設計 (Strategy Architecture)

- **研究重點**：  
  - 以「物理情境」為前提設計選擇權結構  
  - 分析各種策略的 failure modes（失效模式）
- **典型問題**：  
  - 何時適合使用 Calendar / Diagonal / Iron Condor？  
  - 在不同 vol regime 下，如何調整履約價間距與部位大小來控制 tail risk？

---

##  訓練日誌與自我修正 (Training Logs & Iterative Learning)

本專案不只是展示成功案例，也包含：

- 錯誤判斷（例如：低估 Skew 對 Calendar Spread 的影響）  
- 對虧損情境的**事後拆解與重建推理**  
- 將「直覺」轉寫成「可以檢驗的假說與模擬」  

所有這些會整理在 `Training_Logs/` 中。  
我認為：

> **修正錯誤的過程，比單純得出正確答案，更接近 Trader 真實的工作形態。**

---

## 技術與工具 (Tech Stack)

- Python（模擬與資料處理）
- NumPy / pandas（數值運算與時間序列）
- Matplotlib（簡易可視化）
- 之後將逐步加入：  
  - 基礎 QuantLib / 自製定價模組  
  - 簡單 backtesting / scenario simulation 工具

---

##  長期願景 (Long-term Vision)

- 建立一套屬於自己的 **“選擇權物理學” 筆記**  
- 系統性整理各策略的**結構風險與失效模式**  
- 強化對 Vol、Greeks 與 Microstructure 的直覺  
- 以此為基礎，成為能獨立設計、檢驗與執行策略的 **衍生品交易員（Options / Vol Trader）**

---

