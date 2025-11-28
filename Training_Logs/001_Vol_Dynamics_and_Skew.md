# Session 001: 波動率曲面動態與結構化偏斜狩獵

**Date:** 2025-11-26
**Focus:** 崩盤時的市場體制識別 (Regime Identification)、局部波動率 (Local Vol) 的物理直覺、以及構建 Delta 中性的偏斜 (Skew) 策略。

---

## Section 1: 市場微觀結構：黏性體制 (The "Sticky" Regimes)

### 市場情境 (The Scenario)
在市場崩盤或恐慌初期，隱含波動率 (IV) 曲面的行為模式會發生劇烈變化。能否正確識別這種「相變 (Phase Transition)」，決定了避險模型的成敗。

### 恐慌的物理學：黏性履約價 (Sticky Strike)
* **觀察：** 在崩盤時，市場並不會平滑地重新定價機率分佈。相反地，特定價格水準會出現對下檔保護 (Downside Protection) 的「剛性需求」。
* **體制特徵：** **Sticky Strike (黏性履約價)**。
    * 整個 IV Surface 會垂直抬升 (Level Shift)。
    * $IV_{new}(K) \approx IV_{old}(K) + \text{Shock}$。
    * 這與 "Sticky Delta"（Skew 曲線隨著股價沿 Moneyness 滑動）形成強烈對比。

### 避險陷阱：Vanna 曝險 (The Vanna Trap)
對於造市商 (Market Maker) 或持有 Short OTM Puts 的交易員來說，如果在崩盤時錯誤假設了 **Sticky Delta**，將導致災難性的 **避險不足 (Under-Hedge)**。

**數學推導：**
Delta 的總變化量為：
$$d\Delta \approx \frac{\partial \Delta}{\partial S} dS + \frac{\partial \Delta}{\partial \sigma} d\sigma$$
$$d\Delta \approx \Gamma dS + \text{Vanna} d\sigma$$

* **在 Sticky Delta 假設下：** 模型假設 $d\sigma \approx 0$ (相對於 Moneyness 不變)，因此忽略了 Vanna 項。
* **在現實 (Sticky Strike) 中：** $d\sigma$ 是巨大的正值。由於 OTM Put 的 Vanna 不為零（隨著 $\sigma$ 上升，隱含 $\Delta$ 會變得更負），**Vanna 效應會迫使部位變得比模型預測的「更 Long」(Delta 變得更正)**。
* **結果：** 交易員賣出的期貨數量不足，導致交易桌直接暴露在更大的下跌風險中。

---

## Section 2: 局部波動率：恐懼的幾何學 (The Geometry of Fear)

### 物理直覺：平均速度 vs. 瞬時速度
為了理解隱含波動率 (IV) 與局部波動率 (LV) 的關係，我們使用運動學類比：
* **IV ($\sigma_{imp}$):** 粒子從 $t_0$ 到 $T$ 路徑上的 **平均速度**。
* **LV ($\sigma_{loc}$):** 粒子在特定價格 $S$ 與特定時間 $t$ 的 **瞬時速度**。

### 「兩倍斜率」快篩法則 (The "2x Slope" Heuristic)
在典型的股票市場中，呈現 **負偏斜 (Negative Skew/Smirk)** 結構（履約價越低，IV 越高）：
$$\frac{\partial \sigma_{imp}}{\partial K} < 0$$

為了讓 *平均* 速度 (IV) 隨著履約價降低而上升，該低價區的 *瞬時* 速度 (LV) 必須顯著更高，才能把平均值拉上去。

**Dupire 公式的直覺詮釋：**
$$\sigma_{loc}(K) \approx \sigma_{imp}(K) + K \frac{\partial \sigma_{imp}}{\partial K} \times (\text{Slope Adjustment})$$

**實戰經驗法則：**
> **局部波動率 (Local Vol) 曲面的陡峭程度，大約是隱含波動率 (Implied Vol) 曲面的兩倍。**

* **隱含意義：** 如果 IV Skew 很陡峭，代表市場正在定價一種「爆炸性加速」——一旦價格觸及該低點，波動率將會瞬間噴發。

---

## Section 3: 結構化設計：狩獵偏斜 (Hunting the Skew)

### 戰略目標
* **市場狀態：** 崩盤後 (Post-crash)。VIX 處於高檔，Skew 極度陡峭 (OTM Puts 相對於 ATM 非常昂貴)。
* **物理假設：** 恐慌將會消退。預期 VIX 會緩慢下降，但更重要的是 **Skew 會變平 (Flatten)** —— OTM 的權利金消風速度會快於 ATM。
* **任務：** 建構一個 Delta 中性、Vega 中性的結構，專門捕捉 Skew Flattening 的 Alpha。

### 策略選擇：Delta 對沖的 1x2 賣權比例價差 (1x2 Put Ratio Spread)
* **Leg 1:** Buy 1x ATM Put (高 Gamma, 高 Vega)。
* **Leg 2:** Sell 2x OTM Put (高 Skew 溢價)。
* **Hedge:** Long Futures 以中和初始 Delta。

### 獲利原理 (The Physics of P&L)
透過調整口數比例 (1 vs 2)，我們針對 Vol Surface 的平行移動做到了 **Vega Neutral**。我們的 Alpha 來自於 **斜率的變化**：
$$P\&L \approx \text{Vega}_{short} \times (\text{OTM Vol 下降}) - \text{Vega}_{long} \times (\text{ATM Vol 下降})$$
由於我們預期 $\Delta\sigma_{OTM} \gg \Delta\sigma_{ATM}$，即使整體市場不動，該結構也能因曲線變平而獲利。

### 風險分析：Gamma 反轉 (The Gamma Flip)
這個結構包含了一個隱藏的「奇異點 (Singularity)」。
* **局部區域 (Locally):** 我們是 Net Long Gamma (因為 ATM Gamma > OTM Gamma)。
* **尾部風險 (Tail Risk):** 當價格崩跌接近 Short OTM Strikes 時：
    * ATM Gamma 衰退。
    * OTM Short Gamma 呈指數級爆發。
    * **反轉時刻 (The Flip):** Net Gamma 瞬間從正值翻轉為 **巨大的負值 (Massive Negative)**。
* **防禦機制:**
    1.  **動態 Delta 對沖 (DDH):** 在接近「反轉點」時激進地拋售期貨。
    2.  **斷翼蝴蝶 (Broken Wing Butterfly):** 買入更遠虛值的 OTM Put 作為「災難保險」，封鎖無限下跌風險，將未定義的風險轉化為已知的成本。

---

### Section 4: 程式碼視覺化概念 (Python Lab)
*計劃實作：分析 Skew 斜率與 LV 代理變數*

```python
# Conceptual Logic for Skew Slope Analysis
import numpy as np

def calculate_local_vol_proxy(strike, iv_surface):
    """
    Approximating the steepness of LV based on IV slope.
    利用 IV 斜率近似推導 Local Vol 的陡峭程度
    """
    iv_slope = np.gradient(iv_surface, strike)
    # Heuristic: LV acts as a leveraged function of IV slope
    # 經驗法則：LV 是 IV 斜率的槓桿函數
    lv_proxy = iv_surface + 2 * (strike * iv_slope) 
    return lv_proxy

# Takeaway:
# If iv_slope is negative (Smirk), lv_proxy drops much faster/deeper 
# representing the intense instantaneous volatility required mathematically.
