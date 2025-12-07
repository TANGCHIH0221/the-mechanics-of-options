# Iron Butterfly vs. Straddle — 峰度交易與幾何陷阱


* **Focus:** 解析 Short Straddle 與 Iron Butterfly 的物理本質差異，識別 Gamma 反轉點 (Gamma Flip) 的對沖變奏，以及量化 Skew 形狀對結構化策略的損益歸因。

---

## Section 1: 物理微觀：位能井的邊界條件 (Potential Wells)

我們將兩種策略視為對「粒子（價格）被束縛在原點」的賭注，但兩者的物理邊界條件截然不同。

### 1. 無限深位能井 (Short Straddle)
* **物理模型:**這是一個 **開放邊界 (Open Boundary)** 系統。
    * 你建立了一個引力場 (Short Gamma)，試圖將粒子拉回原點。
    * **風險特徵:** 粒子一旦獲得足夠動能 ($dS$ 夠大) 逃逸，系統能量損失趨近於無限。沒有牆壁可以吸收衝擊。
* **對沖動力學 (Hedging Dynamics):**
    * $\Gamma_{total} \ll 0$ 恆成立。
    * **行為:** 永遠處於恐懼中。對沖操作永遠是 **順勢 (Pro-cyclical)** 的——上漲被迫追買，下跌被迫追賣。

### 2. 反射壁位能井 (Short Iron Butterfly)
* **物理模型:** 近似於量子力學中的 **有限深位能井 (Finite Square Well)**。
    * **井底 (Body):** 粒子束縛區。由 Short ATM Options 主導 ($\Gamma < 0$)。
    * **反射壁 (Wings):** 邊界條件。由 Long OTM Options 主導 ($\Gamma > 0$)。
* **關鍵機制: The Gamma Flip (符號反轉)**
    當價格從 ATM 漂移至 Breakeven 點附近時，結構的總 Gamma 會發生相變：
    
    $$\Gamma_{\text{total}} = \underbrace{\Gamma_{\text{short ATM}}}_{\text{衰退}} + \underbrace{\Gamma_{\text{long OTM}}}_{\text{爆發}} \approx 0 \text{ or } > 0$$

* **戰術變奏:**
    * **在中心:** 行為類似 Straddle，順勢對沖。
    * **在邊界 (The Wall):** Long Wings 提供的正 Gamma 開始產生「反作用力」。對沖節奏從「恐慌追價」轉為「觀察等待」，甚至可以利用局部的 Long Gamma 進行 **逆勢 Scalping (Mean Reversion Trading)**。

---

## Section 2: 金融機制：希臘字母的幾何合成

### 1. 公式的幾何拆解
不要將 IB 視為「有保護的 Straddle」，而應視為「兩個不同方向的波動率賭注」的疊加：

$$\text{Iron Butterfly} \approx \underbrace{\text{Short Straddle}}_{\text{Short Vega}} + \underbrace{\text{Long Strangle}}_{\text{Long Vega (Convexity)}}$$

### 2. Vega 結構與 Vomma (Vol of Vol)
* **Short Straddle:** 純粹的 **Short Vega**。
    * 幾何圖形：在 ATM 處有一根巨大的負 Vega 柱子。
    * 對波動率曲率不敏感。
* **Iron Butterfly:** **Net Short Vega + Long Vomma**。
    * **Vomma Analysis:**
        * ATM 部分：Short Vomma (IV 暴漲時，Vega 負得更多)。
        * Wings 部分：**Long Vomma** (IV 暴漲時，Wing Vega 變大)。
    * **物理意義:** Iron Butterfly 本質上是在 **Short Volatility Level** 的同時 **Long Kurtosis (峰度)**。

### 3. 微笑變陡 (Smile Steepening) 的優勢
假設市場平靜 (ATM IV Flat)，但尾部恐慌導致 Smile 曲線變陡 (Wings IV $\uparrow$)：
* **Straddle PnL:** $\approx 0$ (對尾部變化無感)。
* **Iron Butterfly PnL:** $> 0$。
    $$dP \approx \underbrace{\nu_{ATM} d\sigma_{ATM}}_{\approx 0} + \underbrace{\nu_{Wings} d\sigma_{Wings}}_{> 0 (\text{Profit})}$$
    * IB 會從 Long Wings 的增值中獲利。

---

## Section 3: 陷阱與對策：Skew 的定價陷阱

### 場景：財報後的假象 (The Earnings Trap)
你構建了 IB 賭財報，結果：
1.  **Price:** Perfect Pin at ATM.
2.  **ATM Vol:** Crushed (VIX drop).
3.  **Result:** **PnL 不如預期，甚至虧損。**

### 數學歸因：非均勻坍縮 (Non-Uniform Collapse)
財報後的 IV Crush 並不是整個曲面均勻下降。
* **ATM:** 資訊不確定性消除 $\rightarrow$ IV 暴跌。
* **OTM:** 市場對「黑天鵝」的恐懼往往具有黏性 (Sticky)，甚至會因為後驗機率重新定價而導致 OTM Skew 相對變陡。

$$PnL \approx \underbrace{(\text{Short Vega}_{ATM} \times \text{Crush})}_{\text{Profit}} - \underbrace{(\text{Long Vega}_{Wings} \times \text{Sticky/Rise})}_{\text{Loss}}$$

由於 Wings 有兩顆 (Call+Put)，且在 Pin Risk 下其相對權重上升，昂貴的 Wings 會吃掉大部分 ATM 的利潤。

### 真正的死因：買在山頂 (Buying the Top of the Skew)
Iron Butterfly 的陷阱不在於出場，而在於 **進場 (Entry)**。
* **情境:** 財報前，市場極度恐慌，Skew Percentile > 90% (OTM 非常貴)。
* **邏輯:** 雖然賣了很貴的 ATM，但你也被迫用「史上最貴的價格」去買 OTM 保險。
* **Edge Calculation:**  $$\text{Edge} = \text{ATM Premium}_{\text{sold}} - \text{OTM Premium}_{\text{bought}}$$
    當 OTM 太貴時， Edge 被壓縮至零。

> **The Protocol:** 當 Skew 異常陡峭時，**禁止使用 Iron Butterfly**。此時應轉為 **Short Straddle** (接受無限風險以換取 Edge) 或 **Ratio Spreads** (賣出更多昂貴的 OTM)。永遠不要在保險費率最高的時候買保險。

---

## Section 4: Python Lab Concept (峰度與利潤模擬)

*目標：模擬 Skew 形狀對 Iron Butterfly 期望值的侵蝕效應*

```python
# Iron Butterfly Valuation under Skew Regimes
import numpy as np
from scipy.stats import norm

def iron_fly_edge_analysis(S, K, T, r, atm_vol, skew_factor):
    """
    skew_factor: 代表 OTM IV 相對於 ATM IV 的溢價程度 (Kurtosis Proxy)
    """
    # 1. 定義波動率曲面
    iv_atm = atm_vol
    iv_wing = atm_vol * (1 + skew_factor) # Skew 越陡，Wing IV 越高
    
    # 2. 計算權利金 (Premium)
    # Short Straddle (ATM)
    credit_atm = call_price(S, K, T, r, iv_atm) + put_price(S, K, T, r, iv_atm)
    
    # Long Strangle (Wings at K +/- width)
    width = S * 0.05 # 5% width
    debit_wings = call_price(S, K+width, T, r, iv_wing) + put_price(S, K-width, T, r, iv_wing)
    
    # 3. 淨權利金 (Max Profit Potential)
    net_credit = credit_atm - debit_wings
    
    return net_credit

# Simulation Logic:
# 當 skew_factor 從 10% 上升到 30% (財報前恐慌)，
# net_credit 會顯著下降。這證明了 "Expensive Wings" 吃掉了 Alpha。
