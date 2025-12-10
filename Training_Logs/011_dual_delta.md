# Dual Delta (Spot Delta vs. Forward Delta)


* **Focus:** 解析 Spot Delta 與 Forward Delta 的物理參照系差異，推導利率 ($r$) 對避險比率的扭曲，以及識別「幽靈虧損 (Phantom PnL)」與「降息陷阱」。

---

## Section 1: 物理直覺：慣性系 vs. 共動系 (Frames of Reference)

### 1. 坐標系的相對論
在物理學中，觀察者的運動狀態決定了測量結果。選擇權定價亦同：
* **Spot Price ($S$):** **實驗室坐標系 (Lab Frame/Inertial Frame)**。
    * 這是我們肉眼看到的絕對價格。它是靜態的快照。
* **Forward Price ($F$):** **共動坐標系 (Comoving Frame)**。
    * 它是隨著無風險利率 ($r$) 的膨脹而移動的參考系。
    * 宇宙膨脹方程式：
    $$F_T = S_0 e^{(r-q)T}$$

### 2. 幽靈虧損 (The Phantom Bleed)
當你身處高利率環境 ($r \gg 0$)，如果你堅持使用 Spot Delta 進行中性避險，你會遇到「鬼」。
* **現象：** 即使股價 $S$ 完全不動，你的 Delta Neutral 組合每天都在虧損，且虧損幅度大於 Theta 預期。
* **物理機制：**
    * 你站在原地 ($S$ 不變)，但宇宙在膨脹 ($F$ 隨時間 $t$ 漂移向上)。
    * Call Option 的價值本質上是追蹤 $F$ 的。
    * **相對運動：** 如果 $S$ 不動，相對於本該上漲的 $F$，現實中的 $S$ 其實是在「相對下跌」。
    * **結果：** 你的 Long Call 部位因為「沒有跟上膨脹速度」而產生了 **Drift Decay**。

---

## Section 2: 金融機制：鏈式法則 (The Chain Rule Derivation)

### 1. 數學推導
我們需要找到兩個 Delta 之間的精確轉換因子。
假設選擇權價值 $V$ 透過 $F$ 依賴於 $S$，即 $V = V(F(S))$。
根據微積分鏈式法則：

$$\Delta_S = \frac{\partial V}{\partial S} = \frac{\partial V}{\partial F} \cdot \frac{\partial F}{\partial S}$$

計算 Forward 對 Spot 的偏導數：

$$F = S e^{(r-q)T} \quad \Rightarrow \quad \frac{\partial F}{\partial S} = e^{(r-q)T}$$

代入後得出關鍵轉換公式：

$$\Delta_S(r) = e^{(r-q)T} \cdot \Delta_F$$

### 2. 避險比率的不對稱性 (Hedge Ratio Asymmetry)
在典型環境 ($r > q$) 下，放大因子 $e^{(r-q)T} > 1$。這導致了避險規模的差異：

$$|\Delta_S| > |\Delta_F|$$

* **Spot Hedge (現貨避險):** 你需要賣出 **較多** 的單位 ($-\Delta_S$)。因為你必須額外對沖掉「持有現貨的時間成本」。
* **Futures Hedge (期貨避險):** 你需要賣出 **較少** 的單位 ($-\Delta_F$)。因為期貨本身已經內含了利率漂移。

> **實戰法則：** 如果你用期貨對沖 SPX 選擇權，卻錯誤地使用了看盤軟體上的 Spot Delta，你會系統性地 **Over-hedge (過度避險)**，導致長期 PnL 拖累。

---

## Section 3: 實戰陷阱：降息衝擊 (The Rate Shock Trap)

### 1. 物理坍縮 (The Collapse)
當央行突然降息 ($r \downarrow$)，這不僅僅影響 Rho (價格)，更會通過鏈式法則瞬間改變 Delta (結構)。
* **初始條件：** $S_t = S_0$ (股價未動)，$\sigma$ 固定。
* **事件：** $r_{new} < r_{old}$ (例如降息 50bps)。
* **Forward 坍縮：** 宇宙膨脹率下降，遠期價格瞬間回落。
    $$F_{new} = S_0 e^{(r_{new}-q)T} < F_{old}$$

### 2. Delta 縮水與部位失衡
由於 Call Option 的 Moneyness 是由 $F$ 決定的，當 $F$ 下跌：
1.  **Spot Delta 變小：**
    $$|\Delta_S^{new}| = e^{(r_{new}-q)T}|\Delta_F(F_{new})| < |\Delta_S^{old}|$$
2. **淨部位變動 (Net Position Shift):**
    你手中的 Short Stock 數量是根據舊的（較大的） $\Delta_S^{\text{old}}$ 建立的。

   $$
    \text{Net Delta} = \underbrace{\Delta_S^{\mathrm{new}}}_{\mathrm{Smaller~Long}} - \underbrace{\Delta_S^{\mathrm{old}}}_{\mathrm{Fixed~Short}} < 0
    $$

### 3. 結局：技術性做空 (Technical Net Short)
* **狀態：** 你瞬間變成了 **Net Short**。
* **災難場景：** 降息通常伴隨市場大漲 (Risk-on Rally)。
    * 市場： $\uparrow$ 暴漲。
    * 你的部位： $\downarrow$ 做空 (因數學公式變動導致的避險過剩)。
* **對策：** 在降息發生的瞬間，必須機械式地執行 **Buy Spot** 以重新平衡 Delta，否則將面臨雙重擊殺 (Whipsaw)。

---

## Section 4: Python Lab Concept (Divergence Visualization)

*模擬高利率環境下，隨著到期日 ($T$) 拉長，使用錯誤 Delta 進行避險的誤差範圍。*

```python
import numpy as np
import scipy.stats as si
import matplotlib.pyplot as plt

def dual_deltas(S, K, T, r, q, sigma):
    # Standard d1 calculation
    d1 = (np.log(S / K) + (r - q + 0.5 * sigma ** 2) * T) / (sigma * np.sqrt(T))
    
    # Spot Delta (Lab Frame): Sensitivity to Spot Price
    # This is what most retail platforms display
    delta_spot = si.norm.cdf(d1) * np.exp(-q * T)
    
    # Forward Delta (Comoving Frame): Sensitivity to Forward Price
    # Derived from Chain Rule: Delta_F = Delta_S * exp(-(r-q)T)
    # This is the correct hedge ratio if using Futures
    delta_fwd = delta_spot * np.exp(-(r - q) * T)
    
    return delta_spot, delta_fwd

# Parameters: High Rate Environment Simulation
S = 100
K = 110 # OTM Call
T_years = np.linspace(0.1, 2, 50) # From short term to LEAPS (2y)
r = 0.05 # 5% Interest Rate
q = 0.0
sigma = 0.2

spots = []
fwds = []

for t in T_years:
    ds, df = dual_deltas(S, K, t, r, q, sigma)
    spots.append(ds)
    fwds.append(df)

# Visualization
plt.figure(figsize=(10, 6))
plt.plot(T_years, spots, label='Spot Delta (Spot Hedge Size)', linewidth=2)
plt.plot(T_years, fwds, label='Forward Delta (Futures Hedge Size)', linewidth=2, linestyle='--')

plt.fill_between(T_years, spots, fwds, alpha=0.1, color='red', label='Over-Hedge Risk Zone')

plt.title(f'The Divergence: Spot vs Forward Delta (r={r*100}%)')
plt.xlabel('Time to Maturity (Years)')
plt.ylabel('Delta Value')
plt.legend()
plt.grid(True, alpha=0.3)
plt.show()

# Insight:
# Notice how the "Over-Hedge Risk Zone" expands significantly for LEAPS.
# For a 2-year option, the difference can be >10% of the notional.
# Using Spot Delta to size a Futures hedge guarantees a bleed.
