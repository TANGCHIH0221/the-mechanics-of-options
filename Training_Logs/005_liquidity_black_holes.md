# Session 005: 流動性黑洞與微觀結構相變


* **Focus:** 識別訂單簿 (LOB) 的相變時刻、推導極端行情下的指數型衝擊成本、以及解析 4 月的成交策略。

---

## Section 1: 微觀結構物理學：從液態到真空 (Phase Transitions)

### 1. 彈性與塑性 (Elasticity vs. Plasticity)
市場微觀結構可以視為一個物理晶格。
* **液態/彈性體制 (Liquid Regime):** 受到外力（市價單）衝擊後，LOB 會迅速回填。這遵循 **虎克定律 (Hooke's Law)**：
$$F_{restore} = -k \cdot \Delta P$$
其中 $k$ 是「訂單簿恢復力 (Resilience)」。只要 $k > 0$，價格就會均值回歸。

* **真空/塑性體制 (Vacuum Regime):** 當 $k \to 0$，市場失去恢復力。此時出現 **Quote Fading (報價消風)**——造市商在成交前瞬間撤單。

### 2. 物理指標：回填率 (Refill Ratio)
我們使用流體力學中的質量守恆概念來監控補充速度：
$$\text{Refill Ratio}(t, \Delta t) = \frac{\text{New Limit Orders inflow}}{\text{Depth Consumed by Market Orders}}$$
當此比率驟降至 1 以下並持續惡化，代表市場正從「液態」相變為「氣態/真空」。

---

## Section 2: 衝擊成本的原因 (The Mechanics of Impact)

### 1. 平方根法則的崩潰 (Breakdown of the Square Root Law)
標準 Barra/Almgren-Chriss 模型假設：
$$I \propto \sigma \cdot \sqrt{\frac{Q}{V}}$$
這隱含了市場是連續介質，且 $V$ (成交量) 代表真實流動性。

### 2. 流動性黑洞模型 (The Black Hole Model)
在崩盤時， $V$ 暴增（恐慌拋售），但真實流動性 $D(t)$（LOB 深度）呈現指數衰減。我們修正模型如下：假設 Maker 的存活率隨時間/恐慌指數呈現指數衰減 $D(x) \approx D_{0} e^{-\alpha x}$。

你的滑價 (Slippage/Impact) 將不再是對數或平方根增長，而是變成**指數增長**：
$$\text{Impact} \approx \int_{0}^{Q} \frac{1}{D(q)} dq \approx \int_{0}^{Q} \frac{1}{D_{0}e^{-\alpha q}} dq$$

$$\text{Impact} \propto \frac{1}{\alpha D_{0}} (e^{\alpha Q} - 1)$$

**物理意義：** 在危機時刻，**Volume 是騙人的，Depth 才是真實的。** 不要被巨大的成交量誤導以為流動性很好。

---

## Section 3: 陷阱與對策：奇異點與掠食者 (Singularity & Predator)

### 1. 數學推導：停損奇異點 (Stop-Loss Singularity)
在 Flash Crash 或 Liquidity Vacuum 中，Bid 端的深度 $L$ 會瞬間蒸發，使得你的停損市價單 (Stop $\to$ Market) 面臨物理上的奇異點。

單筆交易造成的價格滑價 $\Delta P$ 近似於：
$$\Delta P \sim \frac{Q}{L}$$

在條件事件「黑洞發生」下，$L$ 逼近 0，且有正機率落在 0 附近。因此，滑價的條件期望值將會發散：
$$\mathbb{E}\left[ \frac{1}{L} \mid \text{crash} \right] \to \infty$$

* **結論：** **Stop Market Order 在黑洞中的期望損失 = 負無限大 (發散)。** 在此處執行市價單，等同於在黑洞視界邊緣自殺。

### 2. 實戰案例：4 月崩跌的微觀交易者 (The April Predator)
**事件背景：** 4 月份因為川普的關稅事件，早盤累積的大量停損單（位能），並且因為台指期的跌停限制，有許多 margin call 的單無法砍單，也就是說流動性低到不行。觀察到某筆交易利用此進行「雙向獵殺」：

* **Phase 1: 推入真空 (Pushing into the Vacuum)**
    * **觀察：** 偵測到 $k \to 0$ (Refill Ratio 失效) 以及 $L \to 0$。
    * **行動：** 在價格下跌初期，主動拋售（做空）。
    * **邏輯：** 利用 **Liquidity Depletion**。此時拋售 $1$ 單位的衝擊會造成 $n$ 單位的跌幅。

* **Phase 2: 逆轉熵值 (Inverting Entropy)**
    * **觀察：** 當市場進入 **Singularity**，所有人都被迫觸發 Stop Market Order 時。
    * **行動：** 獲利了結空單，並**反手掛出 Limit Buy (做多)**。
    * **邏輯：** 當 $\mathbb{E}[1/L] \to \infty$ 時，他成為了那個唯獨存在的 $L$。他賺取的是巨大的 **流動性溢價 (Liquidity Premium)** —— 市場為了成交，願意支付給他任何價格。

---

## Section 4: Python Lab Concept (偵測流動性耗盡)

**目標：** 建立一個指標，偵測價格下跌時，買盤是否正在撤退 (Liquidity Depletion)。

```python
import numpy as np
import pandas as pd

def detect_liquidity_depletion(trade_tape, lob_snapshots):
    """
    偵測 '推土機效應'：當價格下跌時，下方的 Bid 是否在成交前就消失。
    """
    # 1. 計算成交所造成的價格衝擊 (Realized Impact)
    price_change = trade_tape['price'].diff()
    volume = trade_tape['volume']
    
    # 加上 epsilon 避免除以零
    impact_per_vol = price_change.abs() / (volume + 1e-9)
    
    # 2. 觀測 LOB 深度變化 (Order Book Depth)
    # 取 Best Bid ~ Best Bid - 5 ticks 的總量
    bid_depth = lob_snapshots['bid_depth_5_levels']
    
    # 3. 定義流動性耗盡指標 (Depletion Index)
    # Logic: 
    # 正常市場：Depth 大 -> Impact 小
    # 黑洞前夕：Depth 看似很大 -> Impact 卻突然變大 (代表 Depth 是虛的/一碰就碎)
    
    depletion_index = impact_per_vol * bid_depth
    
    return depletion_index
