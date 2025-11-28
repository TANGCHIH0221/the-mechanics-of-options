# Session 018: Ratio Backspreads

* **Date:** 2025-11-28
* **Focus:** 利用凸性 (Convexity) 捕捉極端行情的逃逸速度計算，以及波動率阻尼 (Vol Dampening) 對 Long Vega 策略的影響。

## Section 1: The Physics (逃逸速度與相變)

Ratio Backspread 的本質是構建一個非線性的動能場，模擬從「重力井」發射火箭的物理過程。

* **重力井 (Gravity Well):** 由 Short ATM Option 構成。這是負 Gamma 與 Theta 衰減最強的區域 ($S \approx K_{1}$)。
* **逃逸速度 (Escape Velocity):** 為了獲利，資產價格 $S_{t}$ 必須具備足夠的動能 (Realized Volatility)，在 Theta 耗盡燃料之前，衝破 Break-even Point (BEP)。
* **相變 (Phase Transition):** 當價格穿越 **Gamma Flip** 點時，部位性質發生質變：
    * **Bound State ($S < S_{flip}$):** Net Short Gamma (受困於引力，時間是敵人)。
    * **Free State ($S > S_{flip}$):** Net Long Gamma (擺脫引力，動能轉化為指數級獲利)。

## Section 2: The Financial Mechanics (The Whiteboard Proof)

我們需要證明 Gamma 曲線如何翻轉，以及需要多少峰度 (Kurtosis) 才能逃離死亡谷。

### 1. The Gamma Flip (相變點推導)
部位結構：$V = -C(K_{1}) + 2C(K_{2})$ (假設 1x2 Call Ratio)。
淨 Gamma 為二階導數的疊加：

$$
\Gamma_{net}(S) = -\Gamma(S; K_{1}) + 2\Gamma(S; K_{2})
$$

由於 Gamma 分佈 $\Gamma(S) \propto e^{-d_{1}^{2}/2}$ 是鐘形曲線：
* 當 $S \approx K_{1}$ (ATM): $\Gamma(K_{1}) \gg \Gamma(K_{2}) \implies \Gamma_{net} < 0$ (Risk Zone).
* 當 $S \to K_{2}$ (OTM): $\Gamma(K_{2})$ 上升且權重加倍 $\implies \Gamma_{net} > 0$ (Profit Zone).

臨界點 (Flip Point) $S^{*}$ 滿足：

$$
\Gamma(S^{*}; K_{1}) = 2\Gamma(S^{*}; K_{2})
$$

### 2. The Break-Even Approximation (逃逸距離)
忽略權利金淨收支 (Assuming Zero Cost)，損益兩平點近似為：

$$
S_{BEP} \approx K_{1} + 2(K_{2} - K_{1}) = 2K_{2} - K_{1}
$$

**物理意義：** 這展示了極高的位能壁壘。若 Strike 差距 $\Delta K = 10$，價格必須從 $K_{2}$ 再漲 10 點才能開始獲利。這證明了此策略高度依賴 **Fat-tail (肥尾)** 事件，常態分佈 (Normal Distribution) 的假設在此策略下期望值為負。

## Section 3: The Trap & Strategy (波動率悖論)

**關鍵認知：方向看對 (Spot $\uparrow$) 不代表賺錢，環境參數 (Vol $\downarrow$) 可能是致命的。**

### 1. The "Vol-Dampening" Paradox (波動率阻尼)
這是交易員最常誤判的陷阱。

* **誤區:** 以為賣了昂貴的 ATM Call (High Vega)，所以自己是 Short Vega。
* **真相:** 1x2 結構通常是 **Net Long Vega**。

$$
\text{Vega}_{net} = -\text{Vega}_{ATM} + 2 \times \text{Vega}_{OTM} > 0
$$

* **災難:** 當市場暴漲 (Spot Rally)，通常伴隨隱含波動率崩跌 (Vol Crush)。

$$
\text{PnL}_{Vega} = \text{Vega}_{net} \times \Delta \sigma < 0
$$

你做對了 Delta，但被 Vega 殺死。這種現象會導致 BEP 動態向右移動，讓你永遠追不到獲利點。

### 2. Skew Flattening (偏斜率攤平)
* **機制:** 在極端上漲行情中，恐慌情緒消退，Skew (OTM IV - ATM IV) 往往會變平。
* **後果:** 你原本指望 OTM Call 的 IV 維持高檔 (Sticky Strike)，結果它跌得比 ATM 還快。這是雙重打擊。

## Section 4: Python Lab Concept (動態模擬)

驗證死亡谷與 Vol Crush 的模擬思路：

```python
import numpy as np

def simulate_death_valley_dynamics(S_range, K1, K2, vol_regime):
    """
    S_range: Price array
    vol_regime: 'constant', 'inverse_to_spot' (Vol Crush)
    """
    # 1. 計算標準 Payoff (Intrinsic Value)
    intrinsic = -np.maximum(S_range - K1, 0) + 2 * np.maximum(S_range - K2, 0)
    
    # 2. 計算 T-30, T-15 的 PnL 曲線 (含 Time Value)
    # 重點: 展示在 100-110 (Strike間) 的凹陷 (Valley)
    
    # 3. 模擬 Vol Crush 效應
    if vol_regime == 'inverse_to_spot':
        # 當 Spot 上漲，強制降低 Vol
        # 計算此時 Net Vega 對 PnL 的拖累
        pass
        
    # Visual Output:
    # 繪製兩條曲線：
    # Line A: Constant Vol (BEP @ 120)
    # Line B: Realized Vol Dampening (BEP shifts to 125) -> The Trap
    
    return plot
