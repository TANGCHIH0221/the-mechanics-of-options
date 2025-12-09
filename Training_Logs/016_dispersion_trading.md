# 離散度交易 


* **Focus:** 利用 Jensen 不等式捕捉市場結構的「去相干 (Decoherence)」，透過數學推導識別「骯髒相關性 (Dirty Correlation)」的定價誤區，並量化 Mark-to-Market 的流動性陷阱。

---

## Section 1: 物理直覺：熱力學與熵 (Thermodynamics & Entropy)

### 宏觀態 vs. 微觀態
* **Index (宏觀態):** 是一個低熵、高度有序的「相干疊加態 (Coherent State)」。個體波動透過破壞性干涉相互抵銷。
* **Components (微觀態):** 是高熵、混亂的粒子集合。
* **Dispersion Trading:** 本質上是在 **做多熵 (Long Entropy)**。
    * **Long Component Vol:** 賭微觀粒子發生劇烈運動。
    * **Short Index Vol:** 賭宏觀系統保持相對平靜。
    * **獲利核心:** 只要粒子運動方向不一致 (Dispersed)，$\sum \sigma_i > \sigma_{index}$，我們就能提取其中的能量差。

### 數學本質：Jensen 不等式 (The Convexity Bet)
Dispersion 策略的獲利基礎建立在凸性幾何上。即便在相關性為 1 的極限情況下，數學仍保證了成分股 Gamma 的總和優勢：
$$\sum_{i=1}^N w_i (\Delta S_i)^2 \ge \left( \sum_{i=1}^N w_i \Delta S_i \right)^2$$
這意味著「平方的加權平均」永遠大於等於「加權平均的平方」。

---

## Section 2: 金融機制：骯髒相關性 (Dirty Correlation)

### 1. 理想氣體假設 (Flat Vol World)
在 Black-Scholes 平坦波動率的世界裡，指數變異數可以被拆解為「獨立項 ($A$)」與「相關項 ($B$)」：
令 $A = \sum w_i^2 \sigma_i^2$ (各股獨立波動貢獻)
令 $B = \sum_{i \neq j} w_i w_j \sigma_i \sigma_j$ (交叉項權重因子，顯然 $B>0$)

則平坦世界下的指數變異數為：
$$\sigma_{index, flat}^2 = A + \rho_{true} B$$
此時反推的隱含相關性是乾淨的：
$$\rho_{imp, flat} = \frac{\sigma_{index, flat}^2 - A}{B} = \rho_{true}$$

### 2. 真實世界的污染 (Skew Contamination)
現實中，指數的 Skew (Downside Crash Fear) 遠比個股陡峭。我們觀察到的市場價格隱含了額外的尾部溢價 $\Delta \sigma_{skew}^2$：
$$\sigma_{index, mkt}^2 = \sigma_{index, flat}^2 + \Delta \sigma_{skew}^2 = (A + \rho_{true} B) + \Delta \sigma_{skew}^2$$

### 3. 骯髒相關性公式 (Dirty Correlation Derivation)
當交易員習慣性地將市場觀察到的 $\sigma_{index, mkt}$ 直接代入平坦公式時，得到的 $\rho_{imp}$ 實際上是：
$$\rho_{imp} = \frac{\sigma_{index, mkt}^2 - A}{B} = \frac{A + \rho_{true} B + \Delta \sigma_{skew}^2 - A}{B}$$
化簡後得到關鍵公式：
$$\rho_{imp} = \rho_{true} + \underbrace{\frac{\Delta \sigma_{skew}^2}{B}}_{\text{Tail Risk Contamination}}$$

* **結論:** 由於 $\Delta \sigma_{skew}^2 > 0$ 且 $B > 0$，市場報價的 $\rho_{imp}$ 被系統性 **高估**。
* **致命誤區:** 當你看到 $\rho_{imp} = 0.8$ 而進場做空相關性時，真實相關性可能只有 $0.6$。剩下的 $0.2$ 其實是 **Crash Premium**。你以為在做統計套利，其實是在裸賣尾部風險。

---

## Section 3: 實戰陷阱：死於黎明前 (Death by Repricing)

### 情境：特質性崩盤 (The Idiosyncratic Crash)
假設權重股 Amazon 暴跌 20% ($\Delta S_{AMZN} = -0.2$)，權重 $w=5\%$，而其他成分股不動，導致指數下跌約 1% ($\Delta S_{index} = -0.01$)。

### 1. Gamma 層面：數學上的絕對勝利 (Realized PnL)
根據 Jensen 不等式，Long Component 的 Gamma 獲利遠大於 Short Index 的 Gamma 損失。
* **Component Gain (Amazon):** $$PnL_{comp} \approx \frac{1}{2} \Gamma_{comp} (\Delta S)^2 \propto w (\Delta S)^2 = 0.05 \times (0.20)^2 = \mathbf{0.002}$$
* **Index Loss:** $$PnL_{index} \approx -\frac{1}{2} \Gamma_{index} (\Delta S_{index})^2 \propto -1 \times (0.01)^2 = \mathbf{-0.0001}$$
* **結果:** Gamma 獲利是損失的 **20 倍**。在純物理路徑上，Dispersion 完勝。

### 2. MTM 層面：流動性的絕對死亡 (Unrealized PnL)
真正的殺手不是已實現的路徑，而是 **市場對未來的重新定價 (Repricing)**。
* **Correlation Spike:** 市場恐慌，認為這是系統性風險的前兆，$\rho_{imp}$ 從 40% 瞬間飆升至 80%。
* **Vega Explosion:** 你的 Short Index Straddle (Short Vega) 對 $\rho$ 高度敏感，價格暴漲。
* **PnL 歸因 (The Killer):**
  $$\Delta PnL_{vega} \approx -\nu_{index} \times (\text{Vol Spike}) \ll 0$$
  $$(\text{Vega Loss: } -0.04 \sim -0.10) \gg (\text{Gamma Profit: } +0.002)$$
* **結局:** 在你能從 Gamma 賺到錢之前，你會先因為帳面巨額浮虧 (Mark-to-Market Loss) 而收到 **Margin Call (追繳保證金)** 並被強制平倉。

> **Survival Rule:** 你不會以 Gamma 結算，你會以 MTM 結算。Dispersion 是一個「用流動性換取幾何優勢」的遊戲。

---

## Section 4: Python Lab Concept

*模擬骯髒相關性與 MTM 衝擊*

```python
import numpy as np

def calculate_dispersion_pnl(shock_size, w_target, rho_shock, vol_shock):
    """
    Simulating the clash between Realized Gamma and Unrealized Vega
    shock_size: percent drop in target stock (e.g., 0.20)
    rho_shock: increase in implied correlation (e.g., +0.40)
    vol_shock: increase in index implied vol (e.g., +0.10)
    """
    # 1. Realized Gamma (Jensen's Logic / Cash in Pocket)
    # Simplified proportional gain approximation
    gamma_gain_component = w_target * (shock_size ** 2)
    gamma_loss_index = (w_target * shock_size) ** 2
    net_gamma_pnl = gamma_gain_component - gamma_loss_index
    
    # 2. Unrealized Vega (Repricing Logic / Margin Call)
    # Assume we are Short Index Vega with sensitivity 'nu'
    nu = 1.0 # arbitrary unit per 1% vol change
    vega_loss = -nu * (vol_shock + rho_shock * 0.5) # rho spike feeds into index vol repricing
    
    return net_gamma_pnl, vega_loss

# Case Study: Amazon Crash (-20%)
# Market Panic: Correlation spikes +40%, Index Vol spikes +5%
gamma, vega = calculate_dispersion_pnl(0.20, 0.05, 0.40, 0.05)

print(f"Realized Gamma Profit (Cash): {gamma:.5f}")
print(f"Unrealized Vega Loss (MTM):  {vega:.5f}")
print(f"Survival Ratio (Gain/Loss): {abs(gamma/vega):.2%}")
# Insight: Ratio is typically < 5%. 
# Conclusion: You are right mathematically, but dead financially.
