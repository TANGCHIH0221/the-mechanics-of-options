# Session 018: Ratio Backspreads

* **Date:** 2025-11-28  
* **Focus:** 利用凸性 (Convexity) 捕捉極端行情的逃逸速度計算，以及波動率阻尼 (Vol Dampening) 對 Long Vega 策略的影響。

---

## Section 1: The Physics (逃逸速度與相變)

Ratio Backspread 的本質是構建一個非線性的動能場，模擬從「重力井」發射火箭的物理過程。

* **重力井 (Gravity Well)：** 由 Short ATM Option 構成。這是負 Gamma 與 Theta 衰減最強的區域 (S ≈ K1)。
* **逃逸速度 (Escape Velocity)：** 為了獲利，資產價格 S_t 必須具備足夠的動能 (Realized Volatility)，在 Theta 耗盡燃料之前，衝破 Break-even Point (BEP)。
* **相變 (Phase Transition)：** 當價格穿越 Gamma Flip 點時，部位性質發生質變：
  * **Bound State (S < S_flip)：** Net Short Gamma（受困於引力，時間是敵人）。
  * **Free State (S > S_flip)：** Net Long Gamma（擺脫引力，動能轉化為指數級獲利）。

---

## Section 2: The Financial Mechanics (The Whiteboard Proof)

我們需要說清楚 Gamma 曲線如何翻轉，以及為什麼需要足夠的峰度 (Kurtosis) 才能逃離死亡谷。

### 1. The Gamma Flip (相變點推導)

部位結構（1x2 Call Ratio）：

> V(S) = -C(S, K1) + 2 * C(S, K2)

淨 Gamma 為二階導數的疊加：

> Gamma_net(S) = -Gamma(S, K1) + 2 * Gamma(S, K2)

Gamma 分佈 Gamma(S, K) 在 Black–Scholes 下是近似鐘形曲線（在 ATM 附近最大）：

* 當 S ≈ K1 (ATM) 時：  
  Gamma(S, K1) >> Gamma(S, K2)  
  ⇒ Gamma_net(S) < 0  → **Risk Zone**（死亡谷左側）。

* 當 S → K2 (原本 OTM 的 Long Legs 變 ATM) 時：  
  Gamma(S, K2) 上升且權重加倍  
  ⇒ Gamma_net(S) > 0  → **Profit Zone**（凸性開始爆發）。

臨界點 (Flip Point) S\* 滿足：

> Gamma(S\*, K1) = 2 * Gamma(S\*, K2)

這個點就是 Gamma Flip，相當於 P&L 曲線的「曲率由負轉正」的拐點。

---

### 2. The Break-Even Approximation (逃逸距離)

假設初始為 Zero Cost（淨權利金 ≈ 0），到期時 S_T > K2 的 payoff：

> Payoff(S_T) = S_T - 2K2 + K1

令 Payoff = 0，可得到上方損益兩平點：

> S_BEP ≈ 2K2 - K1

**物理意義：**

* 如果 Strike 差距 ΔK = K2 - K1 = 10，  
  則從 K2 開始，價格還需要再漲約 10 點才進入正報酬區間。
* 這代表 Ratio Backspread 的「逃逸距離」很長：  
  必須依賴尾部大行情（Fat Tail）來跨越死亡谷。  
* 在近似常態、波動不大的世界中，價格多半卡在 K1 和 K2 一帶，  
  長時間承受 Theta Burn，策略的長期期望值傾向變差。

---

## Section 3: The Trap & Strategy (波動率悖論)

> **關鍵認知：方向看對 (Spot ↑) 不代表賺錢，環境參數 (Vol, Skew) 的變形可能是致命風險。**

---

### 1. The "Vol-Dampening" Paradox (波動率阻尼)

這是交易員最常誤判的陷阱之一。

* **誤區：**  
  以為自己「賣了昂貴的 ATM Call (High Vega)」、而且又「買了便宜的 OTM Call」，  
  所以主觀認為自己是 Short Vega，會從 Vol Crush 裡獲利。

* **真相（典型 1x2 結構）：**  
  如果 K2 不離 K1 太遠，兩張 OTM Call 的 Vega 加總往往略大於一張 ATM Call 的 Vega：  
  > Vega_net ≈ -Vega_ATM + 2 * Vega_OTM  
  > 在很多實務參數下，Vega_net 其實是「略為正值」 → **Net Long Vega**

* **災難：**  
  當市場暴漲 (Spot Rally) 同時伴隨整體隱含波動率下跌 (Vol Crush)：  
  > PnL_vega = Vega_net * Δsigma  
  > 若 Vega_net > 0 且 Δsigma < 0 ⇒ PnL_vega < 0

你做對了 Delta（方向賺錢），  
但因為其實是 Net Long Vega，在 Vol Crush 之下反而被 Vega 殺死。  
從路徑角度看，這會讓「有效 BEP」動態往右移動，  
變成：價格永遠在追一個不斷後退的終點線。

---

### 2. Skew Flattening (偏斜率攤平)

* **機制：**  
  在極端上漲行情中，恐慌情緒消退，市場從「保護下檔」轉為「追逐上漲」：  
  * Put 端 IV 大幅下跌  
  * ATM IV 通常回落  
  * Call wing（高履約價）IV 上升有限或僅略為修正  
  整體表現為 **Skew 變平（Flattening）**。

* **後果：**  
  你原本以為：  
  > 「賣貴的 ATM、買便宜的 OTM → 我穩賺 skew edge。」  

  但在實際動態下可能發生：
  * ATM IV 下跌很多，你的 Short ATM Call 的 Vega 損失很大（如果實際 Vega 分佈更偏重這一腿）。  
  * OTM Call IV 上升有限，且 Vega 本來就比較小，補不回來。  

  結果是：  
  * Delta 面向上，你的方向是對的；  
  * 但 **Vega + Skew 的動態效應** 把這些方向獲利吃掉，甚至變成帳面虧損。

---

## Section 4: Python Lab Concept (動態模擬)

驗證死亡谷與 Vol Crush / Skew 動態的模擬思路（概念版）：

```python
import numpy as np

def simulate_death_valley_dynamics(S_range, K1, K2, vol_regime):
    """
    S_range: price array
    K1, K2: strikes
    vol_regime: 'constant', 'vol_crush', 'skew_flatten'
    """
    # 1. 到期 intrinsic payoff：1x2 Call Ratio 結構
    intrinsic = -np.maximum(S_range - K1, 0) + 2 * np.maximum(S_range - K2, 0)

    # 2. 這裡本來可以加入時間價值 (Time Value) 近似，
    #    用簡化的 Black-Scholes 或預先計算好的 option table。
    #    目標：畫出 T-30 / T-15 / T-5 的 P&L 曲線，
    #    強調 K1~K2 區間的「Valley of Death」凹陷。

    # 3. 模擬不同 Vol Regime 對 P&L 的影響
    if vol_regime == 'vol_crush':
        # 當 Spot 上漲，整體隱含波動率同步下滑 (Δsigma < 0)
        # 結合 Vega_net 估算 Vega P&L 拖累
        pass
    elif vol_regime == 'skew_flatten':
        # 模擬：ATM IV 下跌較多，高履約價 Call IV 小幅上升
        # 用 weighted vega（各腿 vega * 各自 ΔIV）計算淨影響
        pass

    # 4. 最終：畫出
    # Line A: constant vol 下的理論 BEP 位置
    # Line B: vol_crush / skew_flatten 下 BEP 向右移動後的實際 P&L 曲線
    return intrinsic  # 實作時可改成回傳多條曲線資料
