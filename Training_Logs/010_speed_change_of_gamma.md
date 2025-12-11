# Speed (Change of Gamma) 


* **Focus:** 深入解析選擇權的三階希臘值 Speed (DgammaDspot)，利用泰勒展開式的立方項 (Cubic Term) 重新定義極端行情下的 PnL 歸因，並揭示 Short Gamma 策略在熔斷 (Melt-up) 時的「死亡螺旋」物理機制。

---

## Section 1: 物理直覺：急動度 (The Physics of Jerk)

### 從加速度到急動度 (Jerk)
在經典力學與微分幾何中，我們對物體運動的描述存在階層性，這與選擇權風險維度完美對應：

1.  **位置 (Position, $S$):** 標的資產價格。
2.  **速度 (Velocity, $V$):** **Delta ($\Delta$)**，位置的變化率 ($dS$)。
3.  **加速度 (Acceleration, $A$):** **Gamma ($\Gamma$)**，速度的變化率 ($d\Delta$)。
4.  **急動度 (Jerk, $J$):** **Speed**，加速度的變化率 ($d\Gamma$)。

### 駕駛艙視角：變化的靈敏度
如果說 **Gamma** 代表油門的靈敏度（踩一下油門，速度增加多少），那麼 **Speed** 就代表 **「靈敏度的變化率」**。
* **Low Speed:** 油門踩到底，推背感 (Gamma) 是恆定的。
* **High Speed:** 油門每踩深 1mm，推背感就呈指數級增強。

在交易實戰中，Speed 衡量的是 **「風險幾何結構的變形速度」**。當 Speed 數值極大時，市場微小的位移 $dS$ 會導致 Gamma 發生劇烈改變。這解釋了為什麼在暴漲/暴跌中，線性避險 (Linear Hedging) 總是失效——你試圖用線性的工具 (Futures) 去修復一個正在發生三階形變的非線性系統。

---

## Section 2: 金融機制：泰勒展開的立方修正 (The Cubic Correction)

### 量化推導：超越 Delta-Gamma 近似
大多數風險模型基於泰勒展開的二階近似 (Second-order Approximation)，這在 $dS \to 0$ 的連續時間下是足夠的。但在 $dS$ 較大的跳躍行情中，必須引入三階修正項。

期權價值 $V(S)$ 在 $S_0$ 處的展開：
$$dV \approx \frac{\partial V}{\partial S} dS + \frac{1}{2} \frac{\partial^2 V}{\partial S^2} (dS)^2 + \mathbf{\frac{1}{6} \frac{\partial^3 V}{\partial S^3} (dS)^3} + \dots$$

轉換為希臘字母語言：
$$dV \approx \Delta \cdot dS + \frac{1}{2} \Gamma \cdot (dS)^2 + \mathbf{\frac{1}{6} \text{Speed} \cdot (dS)^3}$$

其中：
* $\text{Speed} = \frac{\partial \Gamma}{\partial S}$ (Gamma 對 Spot 的偏微分)

### 數學驗證：Black-Scholes 下的 Speed 解析解
為了判斷 Speed 的符號與特性，我們從 Gamma 公式出發：
$$\Gamma = \frac{N'(d_1)}{S \sigma \sqrt{T}}$$
對 $S$ 微分可得 Speed 的解析形式：
$$\text{Speed} = \frac{\partial \Gamma}{\partial S} = - \frac{\Gamma}{S} \left( \frac{d_1}{\sigma \sqrt{T}} + 1 \right)$$

### 幾何符號分析 (Sign Geometry)
這條公式揭示了 Speed 的符號取決於 $d_1$ 的位置（即 Moneyness）：

1.  **Long OTM Call ($S \ll K$):**
    * 此時 $d_1$ 為負值且絕對值較大。
    * 在鐘形曲線 (Bell Curve) 左側，隨著 $S$ 增加逼近 $K$，Gamma 顯著上升。
    * $\frac{\partial \Gamma}{\partial S} > 0$，故 **Speed > 0**。
    * **物理意義：** 這是「凸性紅利」。隨著價格上漲，你的 Gamma 越來越大，獲利加速能力增強。

2.  **Long ITM Call ($S \gg K$):**
    * 此時 $d_1$ 為正值。
    * 在鐘形曲線右側，隨著 $S$ 繼續增加，Option 轉變為單純的 Delta 1 曝險，Gamma 衰退。
    * $\frac{\partial \Gamma}{\partial S} < 0$，故 **Speed < 0**。

---

## Section 3: 實戰陷阱：Short Speed 與死亡螺旋 (The Death Spiral)

### 場景：Short OTM Call vs. Melt-up (熔斷式暴漲)
最致命的場景發生在持有 **Short Deep OTM Call** 時，市場出現向上的極端跳躍 ($dS \gg 0$)。

**1. 部位物理狀態：**
* **Short Gamma ($\Gamma_{pos} < 0$):** 逆風而行。
* **Short Speed ($\text{Speed}_{pos} < 0$):** 因為 Call 本身 Speed > 0，你持有 -1 單位，故部位 Speed 為負。

**2. 災難公式 (The PnL Equation):**
將符號代入三階 PnL 公式：

$$
PnL_{total} \approx \underbrace{\Delta_{hedge}}_{\approx 0} + \underbrace{\frac{1}{2}(-|\Gamma|)(dS)^2}_{\text{Gamma Loss}} - \mathbf{\underbrace{\frac{1}{6}|\text{Speed}|(dS)^3}_{\text{Speed Loss}}}
$$

* 注意三階項： $-\text{Speed} \times (+dS)^3$。
* 由於 $dS$ 為正且很大，$(dS)^3$ 是巨大的正數。
* **結論：** Speed 項會造成額外的、巨大的負 PnL。虧損不是按平方增長，而是按 **立方 (Cubic)** 爆炸。

**3. 動態避險的潰敗 (Hedging Lag):**
為什麼你補期貨永遠補不夠？
* **$t_0$ 時刻：** 你的 Delta 模型告訴你 Short 10 口期貨。
* **$t_1$ 時刻 ($S \to S + \delta$):**
    * 由於 Speed 極大，Gamma 瞬間從 -50 變成 -200。
    * 新的 Delta 缺口瞬間擴大。
    * 你剛成交的避險單，是基於 $t_0$ 的幾何結構，但市場已經移動到了 $t_1$ 的幾何結構。
    * **Result:** You are perpetually under-hedged. 你在追趕一個加速逃逸的目標。

---

## Section 4: Python Lab Concept (虛擬程式碼)

*目標：計算 Speed 造成的「殘差 PnL」，量化模型低估風險的程度。*

```python
import numpy as np
from scipy.stats import norm

def analyze_speed_impact(S, K, T, r, sigma, dS_shock):
    """
    Quantifying the 3rd order PnL impact (Speed/Jerk)
    量化三階 Speed 對 PnL 的衝擊：比較二階預測與三階預測的差異
    """
    # 1. Calculate Standard Greeks
    d1 = (np.log(S/K) + (r + 0.5*sigma**2)*T) / (sigma*np.sqrt(T))
    gamma = norm.pdf(d1) / (S * sigma * np.sqrt(T))
    
    # 2. Calculate Speed (Analytical Derivation)
    # Speed = dGamma/dS
    speed = -gamma * (d1 / (sigma * np.sqrt(T)) + 1/S)
    
    # 3. Simulate PnL Impact for a Short Call Position
    # Position: Short 1 Unit
    pos_gamma = -gamma
    pos_speed = -speed
    
    # Taylor Expansion Components
    pnl_2nd_order = 0.5 * pos_gamma * (dS_shock**2)
    pnl_3rd_order_term = (1/6) * pos_speed * (dS_shock**3)
    
    total_pnl = pnl_2nd_order + pnl_3rd_order_term
    
    print(f"Shock Size: {dS_shock}")
    print(f"Gamma Loss (2nd Order): {pnl_2nd_order:.4f}")
    print(f"Speed Loss (3rd Order): {pnl_3rd_order_term:.4f}")
    print(f"Ratio (Speed/Gamma): {abs(pnl_3rd_order_term/pnl_2nd_order):.2%}")

    return total_pnl

# Key Takeaway:
# 當 dS_shock 很大時 (e.g., +5% gap)，Speed Loss 可能佔總虧損的 30% 以上。
# 這是風險控管模型 (Risk Engine) 常忽略的「隱形殺手」。
