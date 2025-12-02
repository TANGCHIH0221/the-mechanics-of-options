#  Event Volatility 


* **Focus:** 利用 Dirac Delta 函數建模從總變異數中剝離事件波動率 (Event Variance Extraction)、理解波函數坍縮 (IV Crush) 的物理機制，以及低波動率環境下的 Gamma 奇異點風險。

---

## Section 1: 波函數坍縮 (The Physics of Collapse)

### 超級疊加態 (Superposition)
在財報或重大事件前，股價並非處於單一數值的預測路徑上，而是處於一個機率分佈的 **疊加態 (Superposition)**。
* **市場定價對象：** 市場不是在賭「股價會漲到哪」，而是在為這個分佈的 **寬度 (Variance/Energy)** 定價。
* **物理類比：** $\Psi_{market} = \sum c_{i} \psi_{i}$。隱含波動率 (IV) 代表了維持這個疊加態所需的能量。

### 坍縮與能量釋放 (The Collapse)
當事件結果 ($E$) 公佈的瞬間，測量發生了。
* **IV Crush:** 波函數瞬間坍縮為單一特徵值 (Observed Price)。支撐原本寬廣分佈的能量 (IV) 必須瞬間釋放。這就是為什麼 Long Straddle 在方向猜對的情況下仍可能虧損——因為你買的是「不確定性的能量」，而該能量已經耗散。
* **瞬間移動 (Teleportation):** 股價的跳空 (Jump) 是一種非連續位移。這導致 Gamma Scalper 無法透過「路徑積分」獲利 (無法在沿途的每一個價位低買高賣)，只能接受端點的線性位移。

---

## Section 2: 金融機制：變異數萃取 (The Geometry of Variance)

### 物理建模：Dirac Delta 衝擊
我們將波動率視為隨時間變化的函數。平時市場遵循布朗運動擴散，但在財報瞬間，市場經歷一次脈衝。
* **連續時間模型 (Continuous Model):**
$$\sigma^2(t) = \sigma_{diff}^2 + v_E \delta(t - t_E)$$
其中 $\delta(\cdot)$ 為 Dirac Delta 函數，代表在 $t_E$ 時刻注入的瞬時變異數，$v_E$ 為單次事件的變異數 (Dimensionless Variance)。

* **總變異數積分 (Total Variance Integration):**
對於到期時間為 $T$ 的選擇權，其總變異數為路徑積分：
$$\text{TotalVar}(T) = \int_0^T \sigma^2(t) dt = \sigma_{diff}^2 T + v_E$$

* **市場觀測 (Market Observation):**
市場報價的隱含波動率 (IV, $\sigma_{IV}$) 是將上述總能量平均分攤到整個時間段的結果：
$$\sigma_{IV}^2(T) \cdot T = \sigma_{diff}^2 T + v_E$$

### 實戰解算：期限結構聯立求解
為了將 $v_E$ (事件能量) 從背景雜訊中分離，我們利用期限結構 (Term Structure) 建立聯立方程式。假設 $T_1$ (近月) 與 $T_2$ (次月) 皆包含該事件：

1. **系統方程組：**

$$
\begin{cases}
\sigma_{1}^2 T_{1} = \sigma_{diff}^2 T_{1} + v_E \\
\sigma_{2}^2 T_{2} = \sigma_{diff}^2 T_{2} + v_E
\end{cases}
$$

2. **求解背景擴散 ($\sigma_{diff}$):**
利用斜率公式消除常數項 $v_E$：
$$\sigma_{diff} = \sqrt{ \frac{\sigma_{1}^2 T_{1} - \sigma_{2}^2 T_{2}}{T_{1} - T_{2}} }$$

3. **萃取事件變異數 ($v_E$):**
將 $\sigma_{diff}$ 代回任意一式：
$$v_{event} = \sigma_{1}^2 T_{1} - \sigma_{diff}^2 T_{1}$$

4. **轉化為價格移動 %:**
$$\text{Implied Event Move} \approx \sqrt{v_{event}}$$

---

## Section 3: 陷阱與對策 (The Trap & Strategy)

### 低波動率的奇異點 (The Low Vol Singularity)
常犯的錯誤是在 IV Crush 後持有 ATM Short Straddle，認為「波動率低 = 安全」。
* **物理事實：** Gamma ($\Gamma$) 與波動率 ($\sigma$) 成反比。
$$\Gamma_{ATM} \approx \frac{1}{S \sigma \sqrt{2\pi T}}$$
* **危險機制：** 當 $\sigma \to 0$ 且 $T \to 0$ 時，鐘形曲線變得極度尖銳 (High Kurtosis)。Gamma 趨近於 **Dirac Delta Function**。
* **後果：** 即使股價只有微小的擾動，Delta 也會從 0 瞬間跳變為 $\pm 1$。你的部位在「毫無感覺」與「瞬間死亡」之間切換，無法進行有效的動態避險。

### 實戰對策
**Defined Risk:** 絕不在低波動率、短到期日的情況下裸賣 Gamma。使用 **Iron Fly (鐵蝴蝶)** 鎖住兩翼，將無限的 Gamma 風險轉化為有限的 Vega 損失。
2. **Calendar Spreads:** 利用 $v_{event}$ 在近月與遠月的差異。賣出近月 (高 IV, 吸收 Crush)，買入遠月 (低 IV, 保留 Gamma)。

---
####  物理直覺：鐘形曲線的幾何變形
<div align="center">
  <img src="images/pgamma.png" width="90%" alt="Gamma Regimes Comparison">
</div>


---
## Section 4: Python Lab Concept

```python
import numpy as np

def calculate_implied_move(iv_near, iv_far, days_near, days_far):
    """
    Extracts the implied event move from two term structure points.
    從期限結構中剝離事件波動率
    Logic: TotalVar = DiffusiveVar * T + EventVar
    """
    t_near = days_near / 365.0
    t_far = days_far / 365.0
    
    # Total Variance (Dimensionless)
    var_near = (iv_near ** 2) * t_near
    var_far = (iv_far ** 2) * t_far
    
    # 1. Solve for Diffusive Variance (Background Noise)
    # This is essentially the slope of the variance-time plot
    var_diff_annual = (var_far - var_near) / (t_far - t_near)
    
    # 2. Extract Event Variance (The Jump / Intercept)
    # v_event = TotalVar - DiffusiveVar * T
    var_event = var_near - (var_diff_annual * t_near)
    
    # Safety check for negative variance (arb opportunity or bad data)
    if var_event < 0:
        return 0.0, np.sqrt(abs(var_diff_annual))
        
    implied_move_pct = np.sqrt(var_event)
    
    return implied_move_pct, np.sqrt(var_diff_annual)

# Example Usage: NVDA Earnings
# Near: 5 days, 80% IV
# Far: 30 days, 50% IV
move, background_vol = calculate_implied_move(0.80, 0.50, 5, 30)
print(f"Implied Event Move: {move:.2%}")      # Expected output: ~8.01%
print(f"Background Vol: {background_vol:.2%}") # Expected output: ~41.47%
