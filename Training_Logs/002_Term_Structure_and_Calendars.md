# Session 002: 時間的物理學 - 期限結構、遠期波動率與 Charm
**Date:** 2025-11-27
**Focus:** 深入解析「時間 ($t$)」維度對交易的影響——從宏觀的期限結構 (Term Structure) 到微觀的二階希臘字母 (Charm) 漂移。

---

## 1. 時間的宏觀幾何：遠期波動率 (Forward Volatility)

### 變異數的可加性 (The Logic of Variance Additivity)
交易員常誤以為平坦的期限結構 (Flat Term Structure) 代表市場預期未來波動率恆定。這是錯誤的直覺。
在物理上，**變異數 (Variance, $\sigma^2 T$)** 才是可加的物理量（類似能量），而標準差（波動率）不是線性疊加的。

**情境分析 (Scenario):**
* $T_1$ (1個月後到期) IV = 15%
* $T_2$ (2個月後到期) IV = 20%

**物理推導 (The Calculation):**
為了將整體的平均波動率從 15% 拉升至 20%，市場暗示第二個月必須蘊含巨大的波動能量。
$$\sigma_{fwd} = \sqrt{\frac{\sigma_2^2 T_2 - \sigma_1^2 T_1}{T_2 - T_1}}$$
$$\sigma_{fwd} = \sqrt{\frac{0.04 \times 2 - 0.0225 \times 1}{1}} \approx 24\%$$

**實戰結論 (Takeaway):**
看似溫和的期限結構 (20%)，其實隱藏了第二個月高達 **24%** 的事件風險。忽略 Forward Vol 會導致在風暴前夕錯誤地低價賣出波動率。

---

## 2. 結構性陷阱：Calendar Spread 與 Gamma 失配

### 交易設置 (The Setup - Backwardation)
* **情境:** 財報發布 (Binary Event) 或 危機爆發。
* **結構:** 近月 IV (150%) > 遠月 IV (50%)。
* **常見錯誤:** 做多 Calendar Spread (賣近月 / 買遠月) 試圖套利。

### 為什麼會失敗？ (The Physics of Failure)
這個策略在理論上是 Long Vega，但在「事件驅動」的物理環境下，它面臨兩個致命的不對稱性 (Mismatch)：

1.  **Gamma Mismatch (動力學失配):**
    * $\Gamma \propto 1/\sqrt{T}$。近月 Gamma 是極尖銳的峰值，遠月是平緩的山丘。
    * **淨部位:** 巨大的 **Net Short Gamma**。
    * **後果:** 若財報導致跳空 (Gap)，近月部位 Delta 瞬間飆升至 1.0 (被行權)，遠月部位反應不及，導致巨額虧損。

2.  **Vol Dynamics (位能失配):**
    * 事件後會發生 **Vol Crush**。雖然近月跌得多，但遠月 IV 也會下跌。
    * 由於你是 **Long Back Month Vega**，遠月 IV 的下跌會直接侵蝕你的資產淨值 (NAV)。

**修正策略:** **Iron Condor (鐵兀鷹)** —— 利用 Short Vega 吃掉波動率崩跌，同時利用翅膀 (Wings) 鎖住 Gamma 風險。

---

## 3. 時間的微觀力學：Charm (Delta Decay)

### 什麼是 Charm？
Charm ($\partial \Delta / \partial t$) 描述的是 **Delta 對時間的敏感度**。
在 Calendar Spread 中，我們不只面臨 Gamma Mismatch，還面臨 **Charm Mismatch**。

* **物理直覺:** 隨著 $T \to 0$ (接近到期)，機率波函數坍縮。
    * 虛值 (OTM) 選項的 Delta 會加速歸零。
    * 實值 (ITM) 選項的 Delta 會加速歸一。

### 實戰應用：週末效應 (The Weekend Trap)
假設你在週五持有一個 Delta Neutral 的 Calendar Spread (Sell Front / Buy Back)。
* **近月 (Short Leg):** 剩餘時間極短 $\to$ Charm 極大 $\to$ Delta 隨時間漂移極快。
* **遠月 (Long Leg):** 剩餘時間長 $\to$ Charm 小 $\to$ Delta 穩定。
* **週一開盤:**
    即使股價不動，因為經過了週末 (2天)，近月 Delta 大幅衰減，遠月 Delta 幾乎沒變。你的部位會自動產生 **Delta 偏差 (Drift)**，迫使你進行額外的避險交易 (Re-hedging)，這就是時間對結構的「侵蝕」。

---

## 4. Python Lab: Forward Vol Visualization
*Calculating the implied forward volatility from the term structure.*

```python
import numpy as np
import matplotlib.pyplot as plt

def calc_forward_vol(t_near, vol_near, t_far, vol_far):
    """
    Calculate Forward Volatility between t_near and t_far
    t: Time in years
    vol: Annualized Volatility (decimal)
    """
    var_near = (vol_near**2) * t_near
    var_far = (vol_far**2) * t_far
    
    # Forward Variance = (Total Variance - Near Variance) / Time Diff
    forward_var = (var_far - var_near) / (t_far - t_near)
    
    if forward_var < 0:
        return 0 # Arbitrage / Anomaly
        
    return np.sqrt(forward_var)

# Example: The hidden risk
print(f"Implied Vol for Month 2: {calc_forward_vol(1/12, 0.15, 2/12, 0.20):.2%}")
