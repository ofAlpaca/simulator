onsensus

## 組員
* 江紹賢 P76074444
* 孫祥恩 P76084708
* 王宇哲 P76084758

## 動機
在現實生活中，每個人可能都有自己的舒適圈、同溫層，但你所認識的好 友真的有值得如此信賴嗎？在表面與你稱兄道弟、同甘共苦，也許其實是笑裡 藏刀、推你火坑的偽君子。同樣地，在區塊鏈上就好比一個小型社會，以狼人殺為例，每個節點就像是玩家，心中都有自己對於其他人的懷疑程度，然而這種懷疑其實是很 fuzzy 的，也許在彼此討論的過程中，自己就會像牆頭草一樣一直改變立場。只是在我們預期的機制中，懷疑就變成了信賴程度，要處死的狼人也就變成了出塊者，我們希望改變傳統區塊鏈的共識機制，引入 fuzzy 來決定結論，也許能更貼近真實世界的情況

## 流程設計
在 Ripple 的設計中，參與共識的所有節點會各自維護一個信任節點的列表，稱為 Unique Node List (UNL)，節點會根據這些 UNL 的意見產生共識。而我們主要是修改此列表的設計，引入 fuzzy 的概念，將每個 UNL 的節點都額外加上一個參數，稱為信任程度抑或是權重。也就是在原本的共識基礎上多考慮了 fuzzy 的信賴關係，我們稱此 UNL 為 FuzzyUNL


整體的執行步驟如下：
*  系統參數
    * `NUM_NODES`：最直接影響整體效率
        * `CONSENSUS_PERCENT`：達成共識所需相同意見的比例
	    * `UNL_MIN/MAX`：FuzzyUNL 個數的上下界
	        * `TRUST_MIN/MAX`：信賴程度的上下界
		*  初始化
		    * 每個節點根據設定的上下界，產生各自的 FuzzyUNL，包含 `index` 與 `trustness`
		        * 每個節點提出最初的意見，預設兩種意見各半 (+/-)
			    * 開始廣播各自的意見，進行最初的投票
			    *  投票
			        * 每個節點會接收各自 FuzzyUNL 的廣播訊息
				    * 將收集的意見與信賴程度進行 multiperson decision making 後，會得到 + 或 - 的意見
				        * 自身意見會根據計算出的值決定是否更動，並廣播出去
					    * 系統會收集整體的意見，計算該輪相同意見是否有超過比例
					        * 達成共識就結束投票，反之則進行下一輪投票
						* 達成共識
						    * 根據共識決定候選區塊提出與否

						    ![](https://i.imgur.com/yq6n7td.png)

						    ## Multiperson Decision Making
						    每個節點都會有自己的 FuzzyUNL (Fuzzy Unique Node List)，節點對於 UNL 成員有相對應的 trustness preference，再透過下方公式來計算 fuzzy relationship $S$ : 

						    $$S(x_i,x_j)=\begin{equation}  
						    \left\{ 
						           \begin{array}{cc}
							                 1 & if& \frac{|T(x_i)|-|T(x_j)|}{n} > 0 \\
									               0 && otherwise
										               \end{array}
											       \right.  
											       \end{equation}$$

											       * where $x_i$ and $x_j$ denote as possible consensus values (+/-),
											       $T(x_i)$ denotes as the trustness of the $i$ node multiples its value $x_i$,
											       $n$ is the total size of one's FuzzyUNL

											       下方舉例，假設某節點的 FuzzyUNL (value, trustness) 為 $\{(+1, 0.3), (-1, 0.9), (+1, 0.4)\}$

											       $$S(+1, -1)= \frac{|1*0.3+1*0.4|-|-1*0.9|}{3}=-0.2=>0$$
											       $$S(-1, +1)= \frac{|-1*0.9|-|1*0.3+1*0.4|}{3}=0.2=>1$$
											       故可以得此 fuzzy preference relation $S$ :
											       $$S=
											       \begin{bmatrix}
											            & + & - \\
												        + & 0 & 0 \\
													    - & 1 & 0 \\
													    \end{bmatrix}
													    $$
													    由於本共識演算法只有 +1 與 -1 兩種意見可選擇，因此省略 $\alpha-cut$ 的部分，最終此「節點所認知的共識」結果為 -1。

													    ## 運作畫面
													    節點數 200
													    共識門檻 80%
													    意見 +/- 兩種
													    信賴程度為亂數生成

													    Time 可以看成投票的輪數，後方的數字是各自意見的節點數
													    以 20 個節點為一條顯示意見分布
													    <img src="https://i.imgur.com/MRq02Ze.png" width=50%>

													    直到相同意見的節點數超過整體的 80% 後，即達成共識，在此共識為 -1
													    最後我們也會計算整個流程中，網路的訊息量與平均節點的訊息量
													    <img src="https://i.imgur.com/ZPxh4qN.png" width=80%>

													    ## 未來方向
													    * 加入候選區塊的信任程度
													    * 加入更多樣的意見
													    * 更加 fine-grained 的 UNL
													    * fuzzy decision 考慮的面向可以更多
													    * 比較修改後的機制的優劣

													    ## 結論
													    在此我們基於 Ripple 的設計，展示了簡易的 fuzzy 共識流程，成功將 fuzzy logic 套用到區塊鏈的共識機制上。我們相信引入信賴程度的設計，可以更貼近現實的運作情況，讓區塊鏈的共識機制能夠變得更加可靠

													    ## 參考連結
													    * [期末 project](https://github.com/ofAlpaca/simulator)
													    * https://github.com/ripple/simulator
