Merge sort 的時間複雜度可以用以下遞迴關係表示：

$T(n)=2T(n2)+Θ(n)T(n) = 2T\left(\frac{n}{2}\right) + \Theta(n)$

這裡：

- $a=2$：表示分成兩個子問題。
    
- $b=2$：表示每個子問題的大小為原來的一半。
    
- $f(n)=Θ(n)f(n) = \Theta(n)$：表示合併兩個已排序序列所需要的時間為線性時間。
    

接下來，我們利用 Master Theorem 來證明時間複雜度。

### 步驟 1：計算 $n^{\log_b a}$

首先，我們計算指數部分：

$n^{\log_b a}=n{\log⁡_2 2}=n1=n^{\log_b a} = n^{\log_2 2} = n^1 = n$

### 步驟 2：比較 $f(n)$ 與 $n^{\log_b a}$

我們發現 $f(n)=Θ(n)f(n) = \Theta(n)$ 與 $nlog⁡ba=Θ(n)n^{\log_b a} = \Theta(n)$ 屬於相同階 (即 $f(n)=Θ(nlog⁡ba)f(n) = \Theta(n^{\log_b a}))$。

根據 Master Theorem 的第三種情形（又稱為「平衡情形」）：

- 如果 $f(n)=Θ(nlog⁡balog⁡kn)f(n) = \Theta\left(n^{\log_b a}\log^k n\right)$ 且 $k=0$，則：
    
    $T(n)=Θ(nlog⁡balog⁡k+1n)T(n) = \Theta\left(n^{\log_b a}\log^{k+1} n\right)$

代入我們的數值，有：

$T(n)=Θ(nlog⁡n)T(n) = \Theta(n \log n)$

### 結論

利用 Master Method，我們證明了 merge sort 的時間複雜度為：

$Θ(nlog⁡n)\Theta(n \log n)$

這就是 merge sort 的漸進上界，證明過程中我們將遞迴式的分解與合併部分分別進行分析，再應用 Master Theorem 得出結果。


### 為什麼 k = 0
在 Master Theorem 中，我們常用以下形式來表示 f(n)f(n)：

$f(n)=Θ(nlog⁡balog⁡kn)f(n)=\Theta\left(n^{\log_b a} \log^k n\right)$

其中 k 是一個非負整數，用來描述 f(n) 與 $n^{\log_b a}$ 的相對關係。

在 Merge Sort 的情況下，遞迴關係是：

$T(n)=2T(n2)+Θ(n)T(n)=2T\left(\frac{n}{2}\right)+\Theta(n)$

這裡 a=2a=2 與 b=2b=2，所以：

$nlog⁡ba=nlog⁡22=nn^{\log_b a}=n^{\log_2 2}=n$

而我們的合併成本 f(n) 為 $\Theta(n)$，可以寫成：

$Θ(n)=Θ(n⋅1)=Θ(n⋅log⁡0n)\Theta(n)=\Theta\left(n\cdot 1\right)=\Theta\left(n\cdot\log^0 n\right)$

因為 $log⁡0n=1\log^0 n=1$ 對所有 n 都成立，所以在這裡 k=0。

換句話說，k=0 的原因是我們並沒有額外的對數因子（例如 $\log n$ 或 $\log^2 n$），而 $f(n)$ 只是單純的線性項，與 $n^{\log_b a}$ 完全一致，並沒有額外的對數增長。