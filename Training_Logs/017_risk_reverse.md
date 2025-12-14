# 風險逆轉 (Risk Reversal) 與偏斜交易


* **Focus:** 利用風險逆轉 (Risk Reversal) 結構交易波動率曲面的「斜率 (Skew)」，理解宇稱不守恆 (Parity Violation) 下的定價邏輯與陷阱。

---

## Section 1: 物理直覺：宇稱不守恆 (Parity Violation)

### 1.1 對稱性破缺 (Symmetry Breaking)
* **BS 模型 (真空態):** 假設股價對數報酬服從常態分佈，此時 Call 和 Put 的定價是鏡像對稱的 ($\text{Skew} = 0$)。這對應於物理學中的宇稱守恆。
* **真實市場 (受激態):** 市場是一個傾斜的位能井。
    * **負偏斜 (Equity Skew):** 下跌恐懼導致 Put 隱含波動率 (IV) 遠高於 Call IV。這是對稱性自發破缺的結果。
    * **Risk Reversal (RR):** 我們不再交易波動率的「總能量」(Vega Level)，而是交易波動率場的 **梯度 (Gradient)**。

### 1.2 交易員視角：滑動摩擦與流體傳導
* **期貨 (Futures):** 是剛體傳導 ($\Delta = 1$)。
* **Risk Reversal:** 是流體傳導。
    * 當你做 **Long RR** (Long Call / Short Put) 時，你是在一個「坡度會隨高度改變」的滑梯上。
    * 隨著股價上漲，恐慌消退 (Vol 下降)，流體的黏滯係數改變 (Vanna 效應)，導致你的 Delta 傳導效率自動衰減。這就是 **Vanna Drag**。

---

## Section 2: 量化推導：奇模態濾波器 (Odd-Mode Filter)

Risk Reversal 被稱為「偏斜獵人」，數學上是因為它是一個針對波動率曲面擾動的 **奇模態濾波器 (Odd-Mode Filter)**。我們將通過泰勒展開證明這一點。

### 2.1 定義與參數化
令 $x = \ln(K/F)$ 為 Log-Moneyness。定義波動率曲面的擾動函數為 $\delta\sigma(x)$。
Risk Reversal ($RR$) 定義為一對對稱 Moneyness 上的 Call 與 Put 的價格差（或隱含波動率差）：

$$
RR(x) \equiv \sigma_{imp}(x) - \sigma_{imp}(-x)
$$

### 2.2 擾動的奇偶分解 (Parity Decomposition)
任何波動率擾動 $\delta\sigma(x)$ 都可以唯一分解為偶函數分量 (Even) 與奇函數分量 (Odd)：

$$
\delta\sigma(x) = \delta\sigma_{even}(x) + \delta\sigma_{odd}(x)
$$

其中：
* **偶分量 (Level/Curvature):** $\delta\sigma_{even}(x) = \frac{\delta\sigma(x) + \delta\sigma(-x)}{2}$ (對應平行的 Vega 升降或對稱的笑臉變化)。
* **奇分量 (Skew/Rotation):** $\delta\sigma_{odd}(x) = \frac{\delta\sigma(x) - \delta\sigma(-x)}{2}$ (對應曲面的旋轉)。

### 2.3 敏感度分析 (Sensitivity Analysis)
考慮 Risk Reversal 價值的變化 $\Delta RR$。利用一階泰勒展開：

$$
\Delta RR \approx \frac{\partial RR}{\partial \sigma(x)} \delta\sigma(x) + \frac{\partial RR}{\partial \sigma(-x)} \delta\sigma(-x)
$$ 

由於 $RR$ 涉及買 Call 賣 Put，其對應的 Vega 敏感度為：
$$
\Delta RR \approx \text{Vega}_{Call}(x) \cdot \delta\sigma(x) - \text{Vega}_{Put}(-x) \cdot \delta\sigma(-x)
$$

**關鍵對稱性：** 在 Black-Scholes 模型中，對於對稱的 Log-Moneyness，Vega 是近似偶對稱的：

$$
\text{Vega}_{Call}(x) \approx \text{Vega}_{Put}(-x) \equiv \nu
$$

代入後得到：

$$
\Delta RR \approx \nu \cdot [\delta\sigma(x) - \delta\sigma(-x)]
$$

### 2.4 濾波證明 (The Proof)
將 $\delta\sigma$ 的奇偶形式代入括號中：

$$
\delta\sigma(x) - \delta\sigma(-x) = [\delta\sigma_{even}(x) + \delta\sigma_{odd}(x)] - [\delta\sigma_{even}(-x) + \delta\sigma_{odd}(-x)]
$$

利用偶函數性質 $\delta\sigma_{even}(-x) = \delta\sigma_{even}(x)$ 和奇函數性質 $\delta\sigma_{odd}(-x) = -\delta\sigma_{odd}(x)$：

$$
= [\delta\sigma_{even}(x) + \delta\sigma_{odd}(x)] - [\delta\sigma_{even}(x) - \delta\sigma_{odd}(x)]
$$
$$
= 2 \cdot \delta\sigma_{odd}(x)
$$

**最終結論：**

$$
\boxed{\Delta RR \approx 2\nu \cdot \delta\sigma_{odd}(x)}
$$

* 若市場發生平行震盪 ($\delta\sigma_{odd} = 0$)，RR 價值不變 $\rightarrow$ **Vega Neutral**。
* RR 的 PnL 幾乎完全來自於波動率曲面的 **奇模態旋轉 (Skew Rotation)**。

---

## Section 3: 陷阱與對策 (The Trap & Mechanics)

### 3.1 Vanna Drag：多頭的隱形損耗
**情境:** Long RR (Long Call + Short Put) 試圖建立零成本多頭。
**陷阱:** 隨著 Spot 上漲，Skew 通常會變平 (Flattening)。
* **物理過程:** $S \uparrow \Rightarrow \sigma_{put} \downarrow$。
* **Vanna 效應:** $\text{Vanna} = \partial \Delta / \partial \sigma > 0$ (對於 Put 而言，意味著 Delta 變得更不負)。
* **結果:** Short Put 的 Delta 貢獻度隨著恐慌消退而迅速歸零。你預期的 Delta 是 1.0，但實際持有的是一個 Delta 隨價格上漲而衰減的部位 ($1.0 \to 0.6$)。**你在最需要 Delta 的時候失去了它們。**

### 3.2 Skew Inversion：空頭的雙重加速 (Dual Acceleration)
**情境:** Short RR (Short Call + Long Put) 用於下檔避險。
**陷阱:** 遭遇 **Short Squeeze / M&A**，市場從負偏斜瞬間翻轉為正偏斜 (Inversion)。

**災難解剖:**
1.  **Delta Blowout (線性項):** Spot 暴漲，Short Call $\Delta \to -1$。
2.  **Skew Blowout (非線性項):**
    * 你 Short 的是 Skew ($\delta\sigma_{odd}$)。
    * 在 Inversion 中，$\delta\sigma_{odd}$ 發生符號反轉並劇烈放大。
    * Call IV 暴漲 (Vega 虧損) + Put IV 不動/下跌 (Vega 無獲利)。
    * **雙殺局面:** Long Leg (Put) 失效，Short Leg (Call) 爆炸。這比單純 Short Call 更危險，因為 Long Put 給予了交易員錯誤的安全感。

---

## Section 4: Python Lab Concept

*模擬 Skew 旋轉對 RR 組合的非線性 PnL 衝擊*

```python
import numpy as np
from scipy.stats import norm

def risk_reversal_parity_check(S, K_call, K_put, T, r, sigma_atm, skew_rotation):
    """
    Analyzes the PnL impact of purely rotating the skew (Odd-Mode perturbation).
    
    skew_rotation: scalar, represents the magnitude of the odd-function perturbation.
                   positive = more negative skew (normal equity market)
                   negative = inversion (squeeze/M&A)
    """
    # Base Volatility
    vol_call = sigma_atm - skew_rotation  # Assuming linear skew model locally
    vol_put  = sigma_atm + skew_rotation
    
    # Calculate Vegas (approx equal)
    d1 = (np.log(S/K_call) + (r + 0.5*sigma_atm**2)*T) / (sigma_atm*np.sqrt(T))
    vega_atm = S * norm.pdf(d1) * np.sqrt(T)
    
    # Theoretical PnL from Skew change (derived in Section 2)
    # Delta_RR ~ 2 * Vega * Delta_Sigma_Odd
    # Note: If skew_rotation changes, that IS the delta_sigma_odd
    
    estimated_skew_pnl = 2 * vega_atm * skew_rotation
    
    return estimated_skew_pnl

# Key Takeaway: 
# The PnL is linear to Vega but heavily dependent on the regime of the rotation.
# A sudden sign flip in 'skew_rotation' causes massive drawdowns for Short RR.
