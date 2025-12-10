# Iron Butterfly vs. Straddle


* **Focus:** 利用 Iron Butterfly 構建有限深度的位能井 (Potential Well)，解析 Gamma Flip 的非線性動態，以及崩盤時「偏斜陡峭化」對 PnL 的非直觀影響。

---

## Section 1: 物理直覺：量子侷限 (Quantum Confinement)

### 1. 無限 vs. 有限位能井
在選擇權物理學中，Short Volatility 策略等同於設置一個「引力場」來束縛粒子（價格）。
* **Short Straddle (自由粒子):** 你施加了引力，但如果粒子動能 ($Realized Vol$) 超過你的位能深度 ($Premium$)，粒子將逃逸至無限遠。風險 $\to \infty$。
* **Iron Butterfly (有限位能井):** 我們在 $K_{L}$ 與 $K_{H}$ 處建立了無限高的位能壁（Long Wings）。
    * **物理本質:** 我們不再賭粒子「完全靜止」，而是賭粒子的能量不足以發生 **穿隧效應 (Tunneling)** 穿過我們的保護邊界。
    * **代價:** 為了建立這道牆，系統總能量 (Max Profit) 降低。

### 2. 峰度 (Kurtosis) 的幾何學
Iron Butterfly (IBF) 的定價本質上是關於波動率曲面的 **曲率 (Curvature)**。

$$\text{Value}_{IBF} \approx \text{Short Straddle}_{ATM} + \text{Long Strangle}_{OTM}$$

* **平坦曲面 (Gaussian):** 峰度 $\kappa = 3$。
* **高凸曲面 (Leptokurtic):** 峰度 $\kappa > 3$。這意味著機率質量從「肩膀」移向「中心」和「尾部」。
* **數學意義:** 峰度上升等價於波動率 Smile 變陡峭 (Steepening)。
    $$\frac{\partial^2 \sigma_{imp}}{\partial K^2} \gg 0$$
* **交易結論:** 當市場預期峰度上升，OTM Wings 的價格相對於 ATM 暴漲。因此，**在高 Kurtosis 環境下建倉 Iron Butterfly，你的 Expected Edge (Credit) 會顯著被壓縮**。

---

## Section 2: 金融機制：Gamma 的奇異點 (The Singularity)

### 1. Gamma 的「W」型結構
不同於 Straddle 的單一倒鐘形，IBF 的 Gamma Profile 呈現複雜的符號反轉：

$$\Gamma_{Net} = \Gamma_{Short}^{ATM} + \Gamma_{Long}^{Wing}$$

* **Zone 1 (ATM):** $\Gamma \ll 0$ (Short Gamma 主導)。
* **Zone 2 (Near Wing):** $\Gamma \gg 0$ (Long Gamma 主導)。
* **The Flip:** 兩者交界處，Net Gamma 穿越零軸。

### 2. 數學推導：指數級爆發
為何反轉如此劇烈？觀察 Gamma 公式中的核：

$$\Gamma \propto \frac{1}{S\sigma\sqrt{T}} e^{-\frac{d_1^2}{2}}$$

當價格從 ATM ($S \approx K_{ATM}$) 移向 Wing ($S \to K_{Wing}$)：
1.  **ATM Term:** $d_1$ 變大， $e^{-d_1^2}$ 衰減至 0。
2.  **Wing Term:** $d_1$ 接近 0， $e^{-d_1^2}$ 衝向峰值 1。
由於指數函數的性質，這種「權力交接」不是線性的，而是呈現 **相變 (Phase Transition)** 特徵。

### 3. 對沖行為的精神分裂 (Hedging Schizophrenia)
這是交易員最容易犯錯的「死亡區域」。
* **Flip 之前 (Short Gamma):** **逆勢 (Mean Reversion)**。跌買漲賣。
* **Flip 之後 (Long Gamma):** **順勢 (Momentum)**。跌要追空 (Short More)，漲要追多。
* **實戰陷阱:** 如果你在穿越 Flip 點後，大腦還停留在「跌深買進」的 Short Gamma 慣性中，你會在 Long Gamma 區域逆勢接刀，導致瞬間被市場碾壓。

---

## Section 3: 實戰陷阱：偏斜膨脹 (The Skew Expansion Trap)

### 1. 崩盤時的非均勻幾何 (Non-Uniform Geometry)
很多 Quant 模型假設 Vol Surface 平行移動 (Level Shift)，但真實崩盤是 **扭曲 (Twist)**。
* **ATM IV:** 上升 (例如 +10%)。
* **OTM Put IV:** 爆炸性上升 (例如 +40%) $\to$ **Skew Steepening**。
* **OTM Call IV:** 可能持平或下降。

### 2. PnL 歸因的幻覺
理論上 IBF 是 Net Short Vega。但在崩盤初期，PnL 常出現詭異的「抗跌」甚至獲利。
**Vega PnL 近似公式：**

$$dV \approx \text{Vega}_{ATM} \cdot d\sigma_{ATM} + \text{Vega}_{Wing} \cdot d\sigma_{Wing}$$

雖然 $|\text{Vega}_{ATM}| > |\text{Vega}_{Wing}|$  ，但是 $d\sigma_{Wing} \gg d\sigma_{ATM}$。

$$\underbrace{\text{Vega}_{Wing}^{Long} \cdot \text{Explosion}}_{\text{Massive Gain}} \approx \underbrace{|\text{Vega}_{ATM}^{Short}| \cdot \text{Rise}}_{\text{Loss}}$$

**結果：** Long Put Wing 的過度反應抵銷了核心部位的虧損。

### 3. 代價：Gamma Trap & Delayed Crush
天下沒有白吃的午餐。這種「Vega 緩衝」伴隨著兩個致命的二階代價：
1.  **Gamma Trap:** 你的部位被暴力拉進了 Long Gamma 區域。這迫使你在市場底部大量 Short Futures 來對沖 Delta。一旦市場 V 型反轉，你將面臨雙重巴掌 (Whipsaw)。
2.  **Delayed Vol Crush:** 當市場反彈，恐慌消退，**Skew Flattening** 的速度極快。Put Wing 的價值會瞬間蒸發 (Crush)，而 ATM IV 往往還維持高檔。
    * **結局:** 你的「保險」在最需要它的時候失效了，留下一堆無用的 Short Vega 曝險。

---

## Section 4: Python Lab Concept

*模擬 Gamma Flip 的位置與強度，識別對沖危險區。*

```python
import numpy as np
import scipy.stats as si
import matplotlib.pyplot as plt

def get_gamma(S, K, T, r, sigma):
    d1 = (np.log(S / K) + (r + 0.5 * sigma ** 2) * T) / (sigma * np.sqrt(T))
    gamma = si.norm.pdf(d1) / (S * sigma * np.sqrt(T))
    return gamma

# 參數設定：模擬還有 7 天到期的 Iron Butterfly
S = np.linspace(90, 110, 200)
K_ATM = 100
K_Wing = 95 # 只看 Put 側
T = 7/365
sigma = 0.2

# Gamma Profile 疊加
# Short ATM Put + Long OTM Put (Put Spread side of the Fly)
gamma_short = -get_gamma(S, K_ATM, T, 0, sigma)
gamma_long  = get_gamma(S, K_Wing, T, 0, sigma)
gamma_net   = gamma_short + gamma_long

# Visualization
plt.figure(figsize=(10, 5))
plt.plot(S, gamma_net, label='Net Gamma', lw=2, color='purple')
plt.axhline(0, color='black', lw=1)
plt.axvline(K_Wing, color='green', ls='--', label='Long Wing')
plt.axvline(K_ATM, color='red', ls='--', label='Short ATM')
plt.title('The Danger Zone: Gamma Flip Dynamics')
plt.ylabel('Net Gamma')
plt.xlabel('Spot Price')
plt.legend()
plt.show()

# Key Observation:
# 注意那條紫色線穿過 0 的位置。
# 那就是你必須從 "Buy the Dip" 切換成 "Sell the Rip" 的數學邊界。
