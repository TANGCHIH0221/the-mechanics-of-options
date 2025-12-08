# Color (Gamma Decay)


* **Focus:** 三階希臘值 Color ($\partial \Gamma / \partial t$) 的物理意義。推導 ATM Gamma 的 $T^{-3/2}$ 發散性質，以及為何用短期選擇權對沖長期 Gamma 會導致結構性的「自動做空陷阱」。

---

## Section 1: 物理直覺：放射性衰變與波函數坍縮

### 1. 波動力學 (Wave Mechanics)
隨著時間 $t \to T$，價格的機率波函數 (PDF) 會發生坍縮 (Collapse)。
* **OTM (真空衰變):** 對於虛值期權，機率密度消散。Gamma 逐漸「熄滅」，風險歸零。
* **ATM (奇異點形成):** 對於價平期權，機率密度向中心無限集中。PDF 變得無限陡峭，導致 Gamma **爆炸性增長** (Singularity)。

### 2. 週末的幽靈 (The Weekend Ghost)
當持有 ATM Short Gamma 部位度過週末時（假設股價不變）：
* **Theta:** 你收到了時間價值 (PnL 增加)。
* **Color:** 你的 Gamma 風險變得更尖銳、更不穩定。
* **物理結論:** 雖然賺了租金，但房子結構變得更脆弱。週一開盤時，你對價格微擾 (Perturbation) 的敏感度顯著上升。

---

## Section 2: 金融機制：Color 的標度律與推導

### 1. 數學推導：從 Gamma 到 Color
我們從 ATM Gamma 的 Black-Scholes 近似公式出發（設 $S \approx K, r=0$）：

$$
\Gamma(T) \approx \frac{1}{S \sigma \sqrt{T}} = \frac{1}{S \sigma} T^{-\frac{1}{2}}
$$
其中 $T$ 為剩餘到期時間 (Time to Maturity)。

**Step A: 對剩餘時間 $T$ 微分**

$$
\frac{\partial \Gamma}{\partial T} = \frac{1}{S \sigma} \cdot \left(-\frac{1}{2}\right) T^{-\frac{3}{2}}
$$

$$
\frac{\partial \Gamma}{\partial T} = -\frac{1}{2T} \cdot \underbrace{\left( \frac{1}{S \sigma \sqrt{T}} \right)}_{\Gamma} = -\frac{\Gamma}{2T}
$$

**Step B: 對日曆時間 $t$ 微分 (定義 Color)**
由於 $T = T_{expiry} - t$，根據鏈鎖律 $\frac{dT}{dt} = -1$：

$$
Color \equiv \frac{\partial \Gamma}{\partial t} = \frac{\partial \Gamma}{\partial T} \cdot \frac{dT}{dt}
$$

$$
Color = \left( -\frac{\Gamma}{2T} \right) \cdot (-1) = \frac{\Gamma}{2T}
$$

### 2. $T^{-3/2}$ 災難 (The Scale of Divergence)
這個公式揭示了 Color 最可怕的物理性質——它的發散速度遠快於 Gamma：

$$
\text{Magnitude of Color} \propto \frac{1}{T \sqrt{T}} = T^{-3/2}
$$
* **物理意義：** 當 $T \to 0$ 時，Gamma 以 $1/\sqrt{T}$ 速度增長，而 Color 以 $1/T^{1.5}$ 速度爆發。
* **實戰後果：** 在極短天期 (0DTE/Weekly) 選項中，Gamma 的衰變/增長速率極快。任何基於靜態快照 (Static Snapshot) 的 Gamma 對沖，在幾分鐘內就會失效。

---

## Section 3: 陷阱與對策：對沖錯配 (The Hedge Trap)

### 1. 毀滅方程 (The Equation of Drift)
考慮一個 **Gamma Neutral** 的期曆組合 (Calendar Spread)：
* **Long:** 長期 ATM 選擇權 ($T_{long} = 1 \text{ Year}$)
* **Short:** 短期 ATM 選擇權 ($T_{short} = 1 \text{ Week}$)
* **初始狀態:** $\Gamma_{net} = N_L \Gamma_L - N_S \Gamma_S = 0$

隨著時間流逝 $dt$ (價格 $S$ 不動)，組合的 Net Gamma 變化率由 Net Color 決定：
$$
Color_{net} = N_L \cdot Color_L - N_S \cdot Color_S
$$
代入 $Color = \frac{\Gamma}{2T}$ 與初始條件 $N_L \Gamma_L = N_S \Gamma_S = G$：
$$
Color_{net} = G \left( \frac{1}{2T_{long}} - \frac{1}{2T_{short}} \right)
$$

### 2. 自動做空的物理機制
由於 $T_{short} \ll T_{long}$，則 $\frac{1}{T_{short}} \gg \frac{1}{T_{long}}$，導致：
$$
Color_{net} \ll 0 \quad (\text{Deeply Negative})
$$

這意味著：
1.  **自然漂移：** 隨著每一秒過去，$\Gamma_{net}$ 會自動從 $0$ 往 **負值 (Net Short Gamma)** 漂移。
2.  **強制交易：** 為了維持 Gamma Neutral，你必須不斷 **買回 (Buy to Cover)** 短期選擇權的 Short Gamma。
3.  **隱形損耗：** 即使市場完全不動，你也被迫進行交易。你是在支付 Spread 與手續費，來對抗「時間造成的 Gamma 結構失衡」。

---

## Section 4: Python Lab Concept 

*模擬：Gamma Neutral Calendar Spread 隨時間的 Gamma 漂移路徑*

```python
import numpy as np

def gamma_drift_simulation(S, sigma, T_long_start, T_short_start):
    """
    Simulates the 'Natural Drift' of a Gamma-Neutral portfolio.
    展示為何 Calendar Spread 會隨著時間自動變成 Net Short Gamma。
    """
    # Simulate 5 days passing (dt steps)
    t_pass = np.linspace(0, 5/365, 100) 
    
    # Gamma Function (ATM approximation: 1 / (S*vol*sqrt(T)))
    def get_gamma(T):
        # Prevent division by zero
        return 1 / (S * sigma * np.sqrt(np.maximum(T, 1e-6)))

    # 1. Initial Hedge Ratio setup at t=0
    gamma_L_0 = get_gamma(T_long_start)
    gamma_S_0 = get_gamma(T_short_start)
    # To be neutral: N_long * Gamma_L = N_short * Gamma_S
    # Normalize: N_long = 1, then N_short = Gamma_L / Gamma_S
    n_short = gamma_L_0 / gamma_S_0 
    
    net_gammas = []
    
    # 2. Time evolution loop
    for t in t_pass:
        # Remaining time decreases
        T_L = T_long_start - t
        T_S = T_short_start - t
        
        # Recalculate Gammas
        gamma_L = get_gamma(T_L) # Changes slowly
        gamma_S = get_gamma(T_S) # Explodes quickly
        
        # Net Gamma Drift
        # Since we held the position constant (n_short is fixed),
        # Net Gamma = Gamma_L - n_short * Gamma_S
        current_net_gamma = gamma_L - n_short * gamma_S
        net_gammas.append(current_net_gamma)
        
    return t_pass, net_gammas

# Key Takeaway:
# The result will show a curve diving deeply into NEGATIVE territory.
# The physics: Short-term Gamma grows (via Color) much faster than Long-term Gamma.
