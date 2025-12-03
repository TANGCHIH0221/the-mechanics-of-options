# 變異數交換 (Variance Swaps)


* **Focus:** 從路徑依賴到路徑獨立，詳解 $1/K^2$ 權重的物理來源，以及跳空擴散下的三階項誤差分析。

---

## Section 1: 物理直覺：位移 vs. 能量 (Action Principle)

### 交易維度的升維
* **Delta One / Linear Payoff:** 關注粒子的**位移**。如果你繞了一大圈回到原點，獲利為零。
* **Variance Swap / Convex Payoff:** 關注粒子的**動能積分** (Action Integral)。
$$V = \frac{1}{T} \int_0^T \sigma_t^2 dt$$
即使價格回到原點，只要過程中經歷了劇烈的震盪（高溫），你就能獲得巨大的回報。

### 凸性的暴力 (The Violence of Convexity)
Variance Swap 的本質是對波動率的平方 ($\sigma^2$) 進行交易。這賦予了它強大的凸性：
* **線性增長 (Vol Swap):** Payoff $\propto (\sigma_{realized} - K_{vol})$。
* **指數級增長 (Var Swap):** Payoff $\propto (\sigma_{realized}^2 - K_{var}^2)$。

當市場恐慌爆發，波動率從 $20\%$ 飆升至 $60\%$：
* Vol Swap 獲利：$3$ 倍增長。
* Var Swap 獲利：$3^2 = 9$ 倍增長。
這種**二階導數 ($d\sigma^2/d\sigma = 2\sigma$)** 意味著波動率越高，PnL 的加速度越快。對於 Short 方而言，這是致命的「正反饋迴路」。

---

## Section 2: 金融機制：複製組合的幾何結構 (Mathematical Derivation)

### 1. 為什麼是 Log Contract？(Ito's Lemma 推導)
為了捕捉 $\sigma^2$，我們尋找一個二階導數為常數的函數。考慮對數價格 $f(S) = \ln S$。
假設股價服從幾何布朗運動 (GBM)：
$$\frac{dS_t}{S_t} = (r-q)dt + \sigma_t dW_t$$

對 $\ln S_t$ 應用伊藤引理 (Ito's Lemma)：
$$d(\ln S_t) = \frac{1}{S_t}dS_t - \frac{1}{2}\frac{1}{S_t^2}(dS_t)^2$$

代入 $dS_t$ 並利用 $(dS_t)^2 = S_t^2 \sigma_t^2 dt$：
$$d(\ln S_t) = \left( (r-q) - \frac{1}{2}\sigma_t^2 \right) dt + \sigma_t dW_t$$

移項整理 $\sigma_t^2$：
$$\frac{1}{2}\sigma_t^2 dt = (r-q)dt - d(\ln S_t) + \sigma_t dW_t$$

對區間 $[0, T]$ 積分並取期望值 (Risk-Neutral Expectation)，其中 Martingale 項 $\mathbb{E}[\int \sigma dW] = 0$：
$$\mathbb{E} \left[ \int_0^T \sigma_t^2 dt \right] = 2 \left( (r-q)T - \mathbb{E}\left[ \ln \frac{S_T}{S_0} \right] \right)$$

**物理意義：** 實現變異數 (Realized Variance) 等價於「持有 Log Contract 的負 payoff」加上一個漂移項。

### 2. 為什麼權重是 $1/K^2$？(Static Replication)
我們無法直接交易 $\ln S_T$，必須用一籃子歐式選擇權進行靜態複製。
根據 **Carr-Madan Formula**，任意兩次可微函數 $g(S_T)$ 可展開為：
$$g(S_T) = g(F) + g'(F)(S_T - F) + \int_0^F g''(K)(K - S_T)^+ dK + \int_F^\infty g''(K)(S_T - K)^+ dK$$

對於 Log Contract，目標函數為 $g(S) = \ln S$。
計算其二階導數（即所需的 Gamma 密度）：
$$g(S) = \ln S \implies g'(S) = \frac{1}{S} \implies g''(S) = -\frac{1}{S^2}$$

將 $g''(K) = -1/K^2$ 代入積分式：
$$\ln S_T \approx \text{Linear Term} - \int_0^F \frac{1}{K^2} P(K) dK - \int_F^\infty \frac{1}{K^2} C(K) dK$$

**結論 (Inverse Square Law):**
為了合成 Variance Swap，我們必須在每一個履約價 $K$ 上持有 **$\frac{1}{K^2}$ 單位** 的選擇權。
* 這意味著在低履約價 (Low Strikes) 區域，我们需要極大數量的 Put 來模擬 Log 函數急劇增加的曲率。

---

## Section 3: 陷阱與對策：跳空擴散 (High-Order Moments)

### 1. 離散化誤差推導
在連續時間模型中，我們假設 $\sum (\ln \frac{S_i}{S_{i-1}})^2 \approx \int \sigma^2 dt$。
但在存在跳空 (Jump) $x = \frac{\Delta S}{S}$ 時，Log Return 為：
$$r = \ln(1+x)$$

對 $r$ 進行泰勒展開 (Taylor Expansion)：
$$r = x - \frac{x^2}{2} + \frac{x^3}{3} - \dots$$

真實的變異數貢獻 (True Variance Contribution) 是 $r^2$。我們保留至三階項：
$$r^2 \approx \left( x - \frac{x^2}{2} \right)^2 \approx x^2 - x^3 + O(x^4)$$

### 2. 複製組合的缺陷
我們的靜態複製組合（$1/K^2$ Puts）本質上是為了捕捉二階項（Gamma）：
$$\text{Replication PnL} \approx \text{Delta Hedge} + \frac{1}{2}\Gamma (\Delta S)^2 \approx x^2$$

### 3. 凸性缺口 (The Convexity Gap)
比較兩者，當發生向下崩盤 ($x < 0$) 時：
$$\text{Gap} = \text{True Var} - \text{Replicated Var} \approx (x^2 - x^3) - x^2 = -x^3$$

* **情境：向下崩盤 (Crash, $x \ll 0$)**
    * 假設 $x = -0.2$ (跌 20%)。
    * 三階項 $-x^3 = -(-0.2)^3 = +0.008$ (正值)。
    * **結果：** 真實變異數 (Realized Var) 比 複製組合 (Replication) 多出了 $0.008$ 的收益。
* **交易含義：**
    * **Short Variance Swap:** 你以為用 Puts 對沖了，但實際上你會**賠錢** (Under-hedged)。
    * **Long Variance Swap:** 你會獲得比 Gamma 預期更高的收益。這就是為什麼 Variance Swap Strike 往往包含 **Jump Risk Premium**。

---

## Section 4: Python Lab Concept (虛擬程式碼)

此模組用於模擬跳空幅度對複製誤差的影響，量化「凸性缺口」。

```python
import numpy as np

def analyze_replication_error(jump_size):
    """
    Quantifying the gap between Log-Contract and 1/K^2 Replication
    during a market jump.
    """
    x = jump_size  # e.g., -0.15
    
    # 1. Theoretical Log Return (The definition of Variance Swap payoff)
    # r = ln(St / St-1)
    log_return = np.log(1 + x)
    true_variance_payoff = log_return ** 2
    
    # 2. Gamma Replication (What the 1/K^2 strip captures)
    # This captures the quadratic variation only
    replication_payoff = x**2
    
    # 3. Third Order Term Approximation (The analytical error)
    # Error approx ~ -x^3
    approx_error = -(x**3)
    
    print(f"Jump: {x:.2%}")
    print(f"True Var Payoff: {true_variance_payoff:.6f}")
    print(f"Replication PnL: {replication_payoff:.6f}")
    print(f"Gap (Under-hedge): {true_variance_payoff - replication_payoff:.6f}")
    print(f"Approximated Gap (-x^3): {approx_error:.6f}")

# Example Output for x = -0.20:
# Jump: -20.00%
# True Var Payoff: 0.049760  <-- (ln(0.8))^2
# Replication PnL: 0.040000  <-- (-0.2)^2
# Gap: 0.009760              <-- The Short Seller's unaccounted loss
# Approx Gap: 0.008000       <-- -(-0.2)^3 captures most of it
