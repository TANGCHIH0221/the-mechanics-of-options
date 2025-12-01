# Session 012: 波動率錐：均值回歸幾何學與相變

* **Date:** 2025-12-01
* **Focus:** 波動率錐 (Volatility Cone) 的物理構造、從位移到路徑積分的資訊熵升維、以及事件視界下的條件機率修正。

---

## Section 1: 位能井與相變 (Potential Wells & Phase Transitions)

### 1. 波動率的虎克定律 (Hooke's Law of Volatility)
Volatility Cone 本質上描繪了一個 **「位能井 (Potential Well)」**。
* **物理模型：** 市場具有均值回歸 (Mean Reversion) 的重力屬性。就像一個被拉伸的彈簧，當隱含波動率 (IV) 遠離長期均值時，市場會產生一股「恢復力 (Restoring Force)」將其拉回。
* **數學描述：** 在彈性限度內，恢復力與偏離程度成正比：
$$F_{restore} = -k \cdot (IV_{current} - IV_{mean})$$
其中 $k$ 是市場的均值回歸速度 (Speed of Mean Reversion)。

### 2. 臨界點：彈性限度 (The Elastic Limit)
大多數交易員死於盲目相信均值回歸，忽略了物理系統存在「斷裂點」。
* **彈性體制 (Elastic Regime):** IV 越高，回歸力越強（Short Vol 的期望值越高）。此時線性恢復力成立。
* **相變 (Phase Transition):** 當市場發生結構性崩潰（如 2008, 2020），彈簧被拉斷了。
    * 此時 IV 突破 100th Percentile 並不代表它會回頭，反而可能進入 **「自由逃逸 (Free Expansion)」** 模式。
    * **警訊：** 若 IV 高檔且 **VVIX (Vol of Vol)** 爆炸性上升，代表位能井失效，禁止做空波動率。

---

## Section 2: 資訊熵與路徑積分 (Entropy & Path Integrals)

### 1. 位移 vs. 路徑 (Displacement vs. Distance)
傳統的 `Close-to-Close` 波動率計算只使用了 1 個自由度（收盤價），這在物理上等同於只測量粒子的「位移」，而忽略了它實際行走的「路徑長」。

**資訊熵階層 (Hierarchy of Information Entropy):**
* **Close-to-Close (Low Entropy):** 忽略了盤中高點與低點。如果價格劇烈震盪後收在平盤，此模型會誤判為「零波動」。
* **Parkinson / Garman-Klass (High Entropy):** 利用 OHLC (Open, High, Low, Close) 資訊。**Range ($H - L$)** 代表了粒子在該時段內的 **Maximum Excursion**，這直接對應到布朗運動的路徑積分能量。

### 2. 量化公式：Garman-Klass Estimator
我們使用高熵估計量來構建更敏銳的 Vol Cone：
$$\sigma_{GK}^2 = \frac{1}{2} \left( \ln \frac{H_t}{L_t} \right)^2 - (2\ln 2 - 1) \left( \ln \frac{C_t}{O_t} \right)^2$$
**物理意義：** 對於造市商而言，Range 決定了被掃出場 (Stop-out) 的機率，因此比收盤價更真實。此估計量的效率是傳統標準差的 7-8 倍。

---

## Section 3: 策略陷阱

### 1. 疊加態的謬誤 (Fallacy of Superposition)
傳統 Vol Cone 是基於歷史數據構建的 **「無條件分佈 (Unconditional Distribution)」**，它錯誤地混合了兩種截然不同的物理狀態：
* $95\%$ 的擴散日 (Diffusion Days) - 平靜的布朗運動。
* $5\%$ 的跳躍日 (Jump/Event Days) - 財報或重大數據公佈。

### 2. 條件機率修正 (Conditional Probability Adjustment)
當我們面臨財報 (Earnings) 時，我們處於 $P(\text{Jump}) = 1$ 的狀態。使用傳統 Vol Cone 來評估此時的 IV 會導致嚴重的 **假陽性 (False Positive)** —— IV 看起來處於歷史高點，但相對於「事件分佈」可能僅是中位數。

**正確的定價模型：擴散 + 跳躍**
我們必須將總變異數分解，不能直接混用：

$$\text{Total Variance} \approx \underbrace{\sigma_{diff}^2 \times T}_{\text{Diffusion Part}} + \underbrace{E[J^2]}_{\text{Jump Part}}$$

### 3. 雙軌錐體對策 (Dual Cones Strategy)
* **Clean Cone (Baseline):** 剔除所有 Earnings/Event 窗口的數據。用於評估日常市場的 Rich/Cheap。
* **Event Cone (Jump Regime):** 僅包含歷史上 Earnings 發生時的 Realized Move 或 IV。
    * **交易訊號:** 只有當當前 IV 高於 **Event Cone** 的 90th Percentile 時，才考慮 Short Event Vol (Selling Straddles/Iron Condors)。
    * **數據清洗:** 計算 Ex-Earnings IV 來還原真實的市場情緒底噪。

---

## Section 4: Python Lab Concept (事件感知型錐體)

**目標：** 建立一個能區分「擴散體制」與「跳躍體制」的波動率錐構建器。

```python
import numpy as np
import pandas as pd

class SmartVolCone:
    def __init__(self, price_data, event_dates):
        self.data = price_data
        self.events = event_dates

    def garman_klass_vol(self, row):
        """
        High-Entropy Estimator:
        利用 Range (H-L) 捕捉路徑能量，比 Close-to-Close 更精確
        """
        log_hl = np.log(row['High'] / row['Low'])
        log_co = np.log(row['Close'] / row['Open'])
        return 0.5 * log_hl**2 - (2*np.log(2)-1) * log_co**2

    def build_cones(self, window_days=[5, 20, 60]):
        # Separate the universe into 'Peace' (Diffusion) and 'War' (Jump)
        # 標記每一天是否在事件窗口內 (e.g., T-1 to T+1)
        self.data['is_event'] = self.data.index.isin(self.events)
        
        # 1. Clean Cone (Diffusion only)
        # 排除事件噪音，建立純粹的擴散基底
        clean_data = self.data[~self.data['is_event']]
        clean_vol = clean_data.rolling(window_days).apply(self.garman_klass_vol)
        
        # 2. Event Cone (Jump regime)
        # 僅捕捉歷史財報窗口的實際波動 (Realized Moves)
        event_data = self.data[self.data['is_event']]
        
        # Returns quantified percentiles for both regimes
        return clean_cone_quantiles, event_cone_quantiles
