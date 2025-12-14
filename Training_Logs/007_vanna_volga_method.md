#  Vanna-Volga 方法

* **Focus:** 利用微擾理論 (Perturbation Theory) 修正 Black-Scholes 的平坦假設，透過靜態複製 (Static Replication) 捕捉 Vanna (Skew) 與 Volga (Convexity) 的市場溢價。

---

## Section 1: 物理直覺：場的微擾 (Field Perturbations)

### 1.1 理想氣體 vs. 真實氣體
* **Black-Scholes 模型 (零階近似):** 假設波動率空間是平坦的 ($\sigma = \text{const}$)，類似於理想氣體方程式，忽略了粒子間的交互作用。
* **真實市場 (高階修正):** 是一個充滿「位能」的場。當價格移動，波動率場會發生形變。
    * **一階微擾 (Skew):** $S$ 與 $\sigma$ 耦合 $\rightarrow$ 產生 Vanna。
    * **二階微擾 (Curvature):** $\sigma$ 本身具有慣性 $\rightarrow$ 產生 Volga (Vomma)。

### 1.2 交易員視角：避險路徑的「摩擦係數」
OTM 選擇權的溢價 ($P_{market} - P_{BS}$)，本質上是造市商為了抵消「動態避險誤差」而收取的預付費用。

* **Vanna ($d\Delta/d\sigma$): 座標系的扭曲**
    當 Spot 移動時，隱含波動率隨之改變，導致 Delta 出現額外的漂移。交易員被迫在價格已經移動後進行追單 (Buy High / Sell Low)。
    > **物理意義:** 這是為了抵抗「滑動摩擦力」所做的功。

* **Volga ($d^2V/d\sigma^2$): 質量的變化**
    當波動率上升，選擇權的 Gamma (質量) 變大。這意味著部位的慣性增加，改變方向需要更大的能量（資金）。
    > **物理意義:** 這是為了改變「系統動能」所做的功。

---

## Section 2: 金融機制與數學推導 (The Mathematics of Replication)

Vanna-Volga (VV) 方法的核心思想是 **靜態複製 (Static Replication)**。既然 BS 模型無法預測 Vanna/Volga 的損耗，我們就構建一個投資組合 $\Pi$，用市場上流動性最好的工具來「合成」出目標選擇權的風險特徵。

### 2.1 複製空間的基底 (Basis Instruments)
我們選擇三個流動性最好的工具作為線性空間的基底向量：
1.  **ATM Straddle ($I_1$):** 捕捉 Vega 水位 (Level)。
2.  **25-Delta Risk Reversal ($I_2$):** 捕捉 Skew (Vanna)。
3.  **25-Delta Butterfly ($I_3$):** 捕捉 Convexity (Volga)。

### 2.2 線性方程組 (The System)
假設目標選擇權為 $O_{target}$。我們尋找一組權重向量 $\mathbf{w} = [w_1, w_2, w_3]^T$，使得投資組合 $\Pi = \sum w_i I_i$ 在關鍵希臘字母上與 $O_{target}$ 完全匹配。

我們忽略 Delta (可通過期貨對沖) 和 Theta，專注於波動率相關的高階風險。建立線性方程組 $A\mathbf{w} = \mathbf{b}$：

$$
\underbrace{
\begin{bmatrix}
\mathcal{V}(I_1) & \mathcal{V}(I_2) & \mathcal{V}(I_3) \\
\text{Vanna}(I_1) & \text{Vanna}(I_2) & \text{Vanna}(I_3) \\
\text{Volga}(I_1) & \text{Volga}(I_2) & \text{Volga}(I_3)
\end{bmatrix}
}_{\mathbf{A}: \text{Basis Risk Matrix}}
\cdot
\underbrace{
\begin{bmatrix}
w_1 \\
w_2 \\
w_3
\end{bmatrix}
}_{\mathbf{w}: \text{Weights}}
=
\underbrace{
\begin{bmatrix}
\mathcal{V}(O_{target}) \\
\text{Vanna}(O_{target}) \\
\text{Volga}(O_{target})
\end{bmatrix}
}_{\mathbf{b}: \text{Target Risk Vector}}
$$

### 2.3 求解與定價公式
權重向量的解為 $\mathbf{w} = \mathbf{A}^{-1}\mathbf{b}$。

一旦求出權重，VV 模型的價格即為 BS 價格加上「複製成本的溢價」：

$$
P_{VV}(K) = P_{BS}(K, \sigma_{ATM}) + \underbrace{\sum_{i=1}^{3} w_i \left[ P_{mkt}(I_i) - P_{BS}(I_i, \sigma_{ATM}) \right]}_{\text{Vanna-Volga Correction Premium}}
$$

這條公式的物理含義極為深刻：
* **$P_{BS}$:** 真空中的理論價格。
* **Correction Term:** 我們利用市場對 RR (Skew) 和 Fly (Curvature) 的報價，推算出當前環境下 Vanna 和 Volga 的「單位價格」，並將其加回到目標選擇權上。

### 2.4 簡化版 (Diagonal Approximation)
在實務上，若假設矩陣 $\mathbf{A}$ 對角線佔優 (Diagonally Dominant)，我們可以將權重簡化為直接比率（這是大多數量化庫的實作方式）：

$$
P_{VV}(K) \approx P_{BS}(K) + \frac{\text{Vanna}(K)}{\text{Vanna}_{RR}} \cdot \text{Cost}_{RR} + \frac{\text{Volga}(K)}{\text{Volga}_{Fly}} \cdot \text{Cost}_{Fly}
$$

這直觀地展示了：**OTM 價格 = BS 價格 + (Skew 敏感度 $\times$ Skew 價格) + (凸性敏感度 $\times$ 凸性價格)。**

---

## Section 3: 陷阱與對策 (The Trap: Extrapolation)

### 3.1 邊界條件失效 (The Wall)
Vanna-Volga 本質上是基於平滑曲面的 **內插法 (Interpolation)**。它隱含了「平移不變性」假設。

* **失效場景:** **Barrier Options (障礙選擇權)** 或 Deep OTM (5 Delta / One-Touch)。
* **物理崩潰點:**
    Barrier Option 類似於量子力學中的無限深位能井。在障礙價格 $H$ 處，機率密度函數 (PDF) 必須強制為零：
    $$\lim_{S \to H} P(S_T) = 0$$
    然而，VV 模型只是基於 ATM 附近的泰勒展開，它「不知道」遠處有一堵牆。它會錯誤地將波動率曲面延伸穿過障礙，導致巨大的定價誤差。

### 3.2 實戰法則
1.  **安全區 (Inner Core):** 25 Delta ~ ATM。VV 模型快速、魯棒，適合大量報價 vanilla options。
2.  **危險區 (Outer Rim):** 10 Delta 以外或含 Barrier。
    * **禁止:** 使用 VV 推估 One-Touch 價格。
    * **必須:** 切換至 **Local Volatility (Dupire)** 或 **Stochastic Volatility (Heston)** 模型。這些模型是通過求解偏微分方程 (PDE) 運作的，天然滿足邊界條件。

---

## Section 4: Python Lab Concept

*計算 VV 權重並定價 OTM Option 的核心邏輯*

```python
import numpy as np

def solve_vv_weights(target_greeks, instrument_matrix):
    """
    Solves for the weights of the replicating portfolio using Linear Algebra.
    
    Args:
        target_greeks: np.array [Vega, Vanna, Volga] of the target option
        instrument_matrix: 3x3 np.array of Greeks for [Straddle, RR, Fly]
        
    Returns:
        weights: np.array [w_straddle, w_rr, w_fly]
    """
    # System: Aw = b  =>  w = A^(-1)b
    try:
        weights = np.linalg.solve(instrument_matrix, target_greeks)
        return weights
    except np.linalg.LinAlgError:
        print("Matrix Singularity: Instruments are linearly dependent.")
        return None

def pricing_kernel(bs_price, weights, market_costs):
    """
    P_VV = P_BS + w * (P_Market - P_BS)
    """
    # market_costs = [Cost_Straddle_Premium, Cost_RR_Premium, Cost_Fly_Premium]
    # These costs are the difference between Market Price and BS(ATM_Vol) Price
    
    correction = np.dot(weights, market_costs)
    return bs_price + correction
