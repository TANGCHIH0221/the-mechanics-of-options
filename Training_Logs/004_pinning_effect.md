#  Pinning Effect 

* **Date:** 2025-11-27
* **Focus:** 理解選擇權到期日，市場微觀結構如何因造市商的 Gamma 避險而產生價格吸力，以及不同結算機制下的風險差異。

## Section 1: The Physics (重力井與簡諧運動)

在到期日 (Expiration) 臨近時，市場動力學從隨機遊走 (Brownian Motion) 轉變為受迫簡諧運動 (Damped Harmonic Oscillator)。

* **相變 (Phase Transition):** 當市場存在巨大的 Open Interest 且造市商 (Market Makers) 持有 **淨長 Gamma (Net Long Gamma)** 時，履約價 $K$ 附近會形成一個「位能井」。
* **恢復力 (Restoring Force):** 價格一旦偏離 $K$，就會受到一個反向力 $F = -k(S-K)$。這個 $k$ (彈性係數) 正比於市場的總 Gamma 量。
* **能量守恆:** Gamma Scalping 將市場的「動能 (Price Velocity)」轉化為造市商的「內能 (Realized P&L)」，同時通過反向操作消耗了市場的波動性，導致系統冷卻 (**Vol Crush**)。

## Section 2: The Financial Mechanics (The Whiteboard Proof)

為什麼造市商的避險會創造重力？我們利用物理極限與泰勒展開來證明 $F = -kx$ 的存在。

### 1. Gamma 的奇異點 (The Singularity)
Black-Scholes Gamma 公式為：
$$\Gamma = \frac{N'(d_1)}{S \sigma \sqrt{T-t}}$$

當 $S \approx K$ 且 $t \to T$ (到期) 時：
* 分子 $N'(d_1) \approx \frac{1}{\sqrt{2\pi}}$ (常數)
* 分母 $\sqrt{T-t} \to 0$

因此：

$$
\lim_{t \to T} \Gamma(S, t) = 
\begin{cases} 
\infty & \text{if } S = K \\
0 & \text{if } S \neq K 
\end{cases}
$$

**物理意義：** 這正是物理學中 **狄拉克 $\delta$ 函數 (Dirac Delta Function)** 的定義。這意味著在 $K$ 點，市場的「慣性質量」趨近無窮大。

### 2. 負回饋循環 (The Negative Feedback Loop)
假設造市商 (MM) 處於 **Net Long Gamma** 狀態 ($\Gamma_{MM} > 0$)。為了保持 Delta Neutral，MM 的避險單 (Hedge) 必須抵消選擇權部位的 Delta 變化：

$$\Delta_{Net} = \Delta_{Option} + \Delta_{Hedge} = 0$$
$$\Rightarrow d(\Delta_{Hedge}) = -d(\Delta_{Option})$$

利用泰勒展開 (Taylor Expansion) 展開 $d(\Delta_{Option})$：
$$d(\Delta_{Option}) \approx \frac{\partial \Delta}{\partial S} dS = \Gamma_{MM} dS$$

代回上式，得到避險動力學公式：

$$\underbrace{d(\Delta_{Hedge})}_{\text{MM Trading Flow}} = - \underbrace{\Gamma_{MM}}_{\text{Spring Constant } k} \cdot \underbrace{dS}_{\text{Displacement } x}$$

* **物理結論:** 造市商的交易流 (Force) 與價格位移 ($dS$) 方向永遠相反，且大小正比於 Gamma。
* 當 $\Gamma \to \infty$ (如前述)，回復力 $F \to \infty$。這就是為什麼價格會被「死死釘在」Strike 上。

## Section 3: The Trap & Strategy 

**關鍵認知：市場結構決定風險型態 (Boundary Conditions matter)。**

### 1. 美股/個股選擇權 (Physical Settlement)
* **風險名稱:** **Pin Risk (釘選風險)**
* **機制:** 收盤時間 (16:00) 與 行權截止時間 (17:30) 存在時間差。
* **薛丁格的貓:** 如果收盤價 $S = K$ (Perfect Pin)，而在盤後價格微幅跳動，你無法確定對手是否會行權。
* **災難:** 你可能在週一開盤時發現自己被指派了「裸部位 (Naked Position)」，若此時發生跳空，將面臨無限風險。
* **對策:** **Don't be a hero.** 永遠在收盤前平倉 ATM 短部位。

### 2. 台指期/VIX (Cash Settlement)
* **風險名稱:** **Settlement Gaming & Gap Risk**
* **機制:** 沒有實物交割，直接現金結算。
* **現象:** 雖然沒有被指派股票的風險，但會面臨 **"Gaming the Fix"**。大戶會在結算計算區間（如最後 30 分鐘 VWAP）利用現貨權值股猛烈攻擊，試圖操縱結算價。
* **災難:** 你的避險模型如果是基於即時價格，可能會與最終的「結算價」產生巨大偏差。此外，留倉過夜則面臨 **Gap Risk** (如 0206 事件)，因無法盤後避險，Gamma/Vega 雙殺。

## Section 4: Python Lab Concept (重力模擬)

驗證 Pinning Effect 的模擬思路：

```python
import numpy as np

def simulate_pinning(S0, K, T, steps, gamma_strength):
    """
    S0: Initial Price
    K: Strike Price (Pinning Center)
    gamma_strength: Representing the Net Long Gamma inventory of the Street
    """
    dt = T / steps
    S = S0
    path = [S]
    
    for i in range(steps):
        time_left = T - i*dt
        if time_left <= 0: break
        
        # 1. Standard Brownian Motion (Thermal Noise)
        dW = np.random.normal(0, 1) * np.sqrt(dt)
        noise = 0.2 * S * dW  # 20% Volatility
        
        # 2. Pinning Force (The Hooke's Law)
        # Force is proportional to Gamma ~ 1/sqrt(time)
        # Force direction is opposite to displacement (S - K)
        gamma_factor = gamma_strength / (np.sqrt(time_left) + 1e-4)
        restoring_force = - gamma_factor * (S - K) * dt
        
        dS = noise + restoring_force
        S += dS
        path.append(S)
        
    return path
