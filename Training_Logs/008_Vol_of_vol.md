#  Vomma (Vol of Vol) —— 波動率的慣性與凸性陷阱

* **Date:** 2025-11-29
* **Focus:** 二階希臘字母 Vomma 的物理意義、Black-Scholes 的時空對稱性 ($\mathcal{V} \leftrightarrow \Gamma$)，以及在極端行情下的凸性陷阱 (Convexity Trap)。

---

## Section 1: 物理直覺：波動率的慣性 (The Inertia of the Field)

### Vomma 的物理定義
如果說 Vega ($\mathcal{V}$) 是波動率場的「位能」（波動率改變 1% 對價格的影響），那麼 Vomma ($\text{Vol of Vol}$) 就是場的 **「慣性」 (Inertia)** 或 **「動能累積」**。

* **數學定義：** $\text{Vomma} = \frac{\partial \mathcal{V}}{\partial \sigma} = \frac{\partial^2 P}{\partial \sigma^2}$
* **物理類比：**
    * **低 Vomma (短天期/ATM)：** 輕量級拳擊手。波動率變動對部位的影響是線性的，打一拳痛一下。
    * **高 Vomma (長天期/OTM)：** 滿載的貨運火車。一旦波動率開始移動（加速），它會積蓄巨大的動量。波動率越高，你的 Vega 部位本身會變得越長（Longer Vega），形成正回饋。

### 凸性陷阱 (The Convexity Trap)
對於 Short Vega 的交易員（如賣出寬跨式部位 Short Strangle）：
* **靜態視角：** 認為 $\text{PnL} \approx \text{Vega} \times \Delta \sigma$。
* **動態視角 (Vomma Effect)：** 當 $\sigma$ 暴漲，Vomma (通常為正) 會導致 Short Vega 的部位規模 **自動膨脹**。
    * 你原本 Short 10,000 Vega。
    * VIX 翻倍後，因為 Vomma 效應，你變成了 Short 50,000 Vega。
    * **結果：** 虧損呈指數級放大 (Explosive Loss)，這就是「保證金螺旋」的原因。

---

## Section 2: 量化推導：時空對稱性 (Space-Time Symmetry)

### Black-Scholes 的隱藏對稱
在 Black-Scholes 擴散方程中，「空間曲率 (Gamma)」與「時間/能量擴散 (Vega)」存在著精確的互換關係。

**推導關係式：**
利用 BS 公式中的 $\mathcal{V} = S \phi(d_1) \sqrt{\tau}$ 與 $\Gamma = \frac{\phi(d_1)}{S \sigma \sqrt{\tau}}$，可得：
$$\frac{\mathcal{V}}{\Gamma} = \frac{S \phi(d_1) \sqrt{\tau}}{\frac{\phi(d_1)}{S \sigma \sqrt{\tau}}} = S^2 \sigma \tau$$
整理後得到 **「時空互換方程」**：
$$\mathcal{V} = \Gamma \cdot \sigma \cdot S^2 \cdot \tau$$

### 物理詮釋
1.  **Vega 是積分後的 Gamma：** Vega 代表的是將當下的空間曲率 ($\Gamma$)，沿著剩餘時間 ($\tau$) 和擴散能量 ($\sigma$) 進行積分後的總風險量。
2.  **Vomma 的本質：**
    $$\text{Vomma} = \frac{\partial \mathcal{V}}{\partial \sigma} \approx \Gamma \cdot S^2 \cdot \tau$$
    這表明 Vomma 大致上是 **「時間加權後的 Gamma」**。
    * **長天期 (Long $\tau$):** 即使 Gamma 很小，由於 $\tau$ 很大，Vomma 極大（對 Vol 變化極度敏感）。
    * **短天期 (Short $\tau$):** 即使 Gamma 很大 (Pin Risk)，Vomma 卻很小（Vega 比較「剛性」）。

---

## Section 3: 實戰陷阱：Calendar Spread 的長尾幻覺

### 陷阱描述
許多交易員認為 Calendar Spread (Short Near / Long Far) 是低風險的 "Long Vega" 策略。
* **理論：** Net Vega > 0 (因為 Far Vega > Near Vega)。
* **現實 (The Trap)：** 在崩盤時，市場並不是平行移動。

### 崩盤物理學 (Crash Physics)
1.  **期限結構倒掛 (Inversion):** 恐慌導致近月 IV 呈現「奇點」式的爆發。$\Delta \sigma_{near} \gg \Delta \sigma_{far}$。
2.  **雙重打擊 (Double Whammy):**
    * **Vega PnL:** 儘管近月 Vega 數值較小，但乘上巨大的 $\Delta \sigma$，造成 Short Leg 虧損遠大於 Long Leg 獲利。
    * **Gamma PnL:** 隨著現貨價格崩跌，Short Near Put 迅速變成 ATM，Gamma 呈指數上升。交易員被迫在低點拋售期貨避險 (Short Gamma Hedging)，鎖定實質虧損。

**結論：** 在 VIX 飆升初期，Calendar Spread 的表現往往像是一個 **Short Position**，而非 Long Position。

---

## Section 4: Python Lab Concept (虛擬程式碼)

**目標：** 視覺化 Gamma 與 Vomma 在不同期限 (Term Structure) 下的消長，並模擬 Inversion 情境。

```python
import numpy as np
import scipy.stats as si

def analyze_calendar_risk(S, K, T_near, T_far, r, sigma_near, sigma_far):
    """
    模擬 Calendar Spread 在期限結構倒掛時的真實 PnL
    """
    # Calculate Vegas
    vega_near = calculate_bs_vega(S, K, T_near, r, sigma_near)
    vega_far = calculate_bs_vega(S, K, T_far, r, sigma_far)
    
    net_vega_initial = vega_far - vega_near # Usually Positive
    
    # Crash Scenario: Inversion
    # Near vol spikes +40%, Far vol spikes +10%
    shock_near = 0.40
    shock_far = 0.10
    
    pnl_impact = (vega_far * shock_far) - (vega_near * shock_near)
    
    return {
        "Net_Vega_Exposure": net_vega_initial,
        "Realized_PnL_Impact": pnl_impact # Often Negative!
    }

# Key Takeaway:
# Net Vega 只是靜態指標。在非平行移動的衝擊下，
# 較小的 Near Vega * 巨大的 Shock 會造成毀滅性打擊。
