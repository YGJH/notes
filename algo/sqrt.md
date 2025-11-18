## 經典題

有 n 個正整數 $a1,a2,a3...,an$ 且 $a1+a2+a3+...+an=k$

問是否可以選一些數字 $a_i$ 總合為 x

### 分case 處理
- 把 $a_i$ 分成兩種 case:

	- $a_i≤ \sqrt{k}$，這種 case 的 $a_i$ 種類不超過 k 種
	- $a_i > \sqrt{k}$，**這種 case 的 $a_i$ 不超過k 個**  ($a$1+a2+a3+...+an=k$)
####  case1 : $a_i <= \sqrt{k}$
- 這種可以用有限背包做，統計每個數字出現的次數，$O(k\sqrt{k}\log{a})$
#### case2 : $a_i > \sqrt{k}$
- 由於題目限定$a_1 + a_2 + a_3 ... + a_n = k$
- 這種 case **不超過**$\sqrt{k}$**個**
-> 不超過$\sqrt{k}$的背包問題->$O(k\sqrt{k})$

->如此一來兩種 case 的複雜度分別為  

$$
\begin{cases}
O(k\sqrt{k}×log⁡{a_i})\ \ \ \  a_i≤k\\
O(k\sqrt{k})\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ a_i>k
\end{cases}
$$
而在 $ai≤k$ 的 case 複雜度比較高，因此可以把 k 設小一點  
平衡一下複雜度