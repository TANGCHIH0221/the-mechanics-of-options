# 射龍門賽局與階級流動的物理學 (The Physics of Decision Theory)


* **Source Inspiration:** JIM (Threads)
* **Focus:** 市場微觀結構中的賽局理論、Alpha 衰變、以及為何「不冒險」是物理學上最大的風險。

> **"Risk is not just a number; it is the price of admission to a moving reference frame."**

---

## Section 1: 思想實驗 (The Gedanken Experiment)

為了理解市場的本質，我們引用一個來自社群 (Credit: JIM) 的簡化賽局模型——「射龍門 (Red Dog)」。

### 實驗 A：共識的陷阱 (The Trap of Consensus)
**設定：**
* **玩家：** $N=1000$ 人。
* **情境：** 底牌翻出 AK (極高勝率區間)。
* **規則：** 猜中的人平分池底 $P = 1,000,000$ 元。沒猜中無懲罰。
* **變數：** 1000 個人擁有相同的資訊（都知道 AK 勝率高）。

**結果：**
如果 1000 個人都做出了「正確」的判斷（猜會過關），每個人分到的錢是 $1000$ 元。
如果考慮時間成本或交易手續費，這個名義上 $Win Rate \approx 95\%$ 的交易，其實際 $EV$ (期望值) 可能趨近於 0 甚至為負。

**[Quant's Logic] Alpha 衰變 (Alpha Decay)**
這解釋了為什麼散戶容易賠錢：
1.  **資訊同質性 (Information Homogeneity)：** 當一個信號對所有人可見時，它就不再是信號，而是雜訊。
2.  **擁擠交易 (Crowded Trades)：** 物理學上稱為「阻抗匹配 (Impedance Matching)」失衡。太多負載掛在同一個電源上，電壓（超額報酬）瞬間驟降。

---

### 實驗 B：座標系的相對性 (Relativity of Wealth)
**設定修正：**
* **成本：** 每玩一次需付 $C = 1,000$ 元入場費。
* **本金：** 每個人手上有 $W_0 = 100,000$ 元。
* **選項：** 你可以選擇「過關」、「撞柱」或「不猜」。不猜的人省下入場費。

**觀察：**
1.  **不猜的人 (The Risk-Averse)：** 本金維持 $100,000$。看似保本。
2.  **猜的人 (The Risk-Takers)：** 結局分裂為「富人」與「賭鬼」。

**結論：**
社會最終是按 **「相對排名」 (Relative Rank / Purchasing Power)** 進行結算。
* **前 10 名 (富人)：** 資產大幅增值。
* **前 200 名 (精英)：** 資產小幅增值。
* **不猜的人 (社畜)：** 雖然名目本金沒少，但因為前 200 名的資產膨脹，導致購買力與社會地位相對暴跌。
* **後 200 名 (賭鬼)：** 破產。

---

## Section 2: 物理學詮釋 (The Physics of Alpha)

### 1. 慣性參考系的謬誤 (Inertial Reference Frames)
選擇「不猜」的人，犯了物理學上最基本的錯誤：**選錯參考系**。
* 他們以為參考系是 **現金 (Cash)**，相對於現金，他們靜止不動 ($v=0$)，所以覺得安全。
* 真實的參考系是 **全社會財富總量 (Global Wealth Cap)**，這個座標系正以 $v_{inflation} + v_{growth}$ 的速度加速遠離。
* **物理結論：** 站在原地不動的人，相對於真實座標系，正以 $-v$ 的速度後退。這就是為什麼「社畜」永遠無法翻身——因為他們拒絕了維持相對位置所需的動能。

### 2. 遍歷性破壞 (Ergodicity Breaking)
在這個實驗中，「富人」與「賭鬼」往往做出了**完全相同**的決策（在 +EV 時入場）。
* **區別：** 僅在於路徑依賴 (Path Dependence) 與運氣。
* **定義：**
    * **賭鬼：** 在單次路徑中遇到「吸收壁 (Absorbing Barrier)」（如：撞柱、爆倉），導致遊戲終止。
    * **富人：** 擁有足夠的資本深度的或部位控管，使其能存活到「時間平均」收斂至「系統期望值」。

> **交易員的直覺：**
> 被市場淘汰的往往不是「做錯決定」的人，而是「槓桿開太大」導致無法撐過變異數 (Variance) 的人。

---

## Section 3: 策略啟示 (Strategic Implications)

這給予了我們後續訓練模組 (Module 1-4) 的核心理由：

### 為什麼我們需要選擇權 (Options)？
在線性資產 (Stocks/Futures) 中，要追上加速移動的財富座標系，你往往需要開槓桿，但槓桿會增加碰到「吸收壁」(破產) 的機率。

選擇權提供了 **「凸性 (Convexity)」**：
$$Payoff \propto S^2 \quad (\text{Gamma})$$
它允許我們定義最大虧損 (Defined Risk)，同時保留了向上的爆發力。這是在物理上唯一能提供 **「逃逸速度 (Escape Velocity)」** 且不至於在起飛前解體的工具。

### 下一步 (Next Step)
既然我們接受了「不冒險就是最大風險」的公理，接下來的任務就是學習**如何精確地冒險**。

* **Module 1:** 識別市場何時擁擠 (Microstructure)。
* **Module 2:** 利用二階效應保護部位 (Advanced Greeks)。
* **Module 3:** 在波動率曲面上找到位能差 (Vol Surface)。

---


