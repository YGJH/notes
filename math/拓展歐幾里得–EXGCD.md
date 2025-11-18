- 對於不定方程 $ax + by = m$ 的整數解，有解的充要條件是 $gcd(a,b)$ 是 m 的因數
- 若 $a , b$ 是整數，且$gcd(a ,b) = d$，那麼對於任意的整數 $x , y , ax + by$ 都一定是 d 的倍數，一定存在整數$x  ,y$ ，使得 ax + by = m 有解

## 證明
[[裴蜀定理證明]]


## exgcd 的用途
於是，知道 $ax+by=m$ 的整數解其有解的充要條件為
$gcd(a,b)|m$ 後，就可以回到正題，要如何在已知a , b的情況求解式子裡的x , y
設當前的式子為
$ax_1+by_1=gcd(a,b)$，根據歐幾里得算法，存在式子
```cpp
int exgcd(int a,int b,long long &x,long long &y) { 
	if(b == 0){x=1,y=0;return a;} 
	int now=exgcd(b,a%b,y,x);
	y-=a/b*x;
	return now;
}
```

exgcd 還有甚麼用途呢?可以用來求逆元。  
求 a 在模p 意義下的逆元  
倘若使用費馬小定理則需要保證p為質數，使用 exgcd 則不需要。
設x 為a 在模p 意義下的逆元，那麼滿足式子：  
$ax≡1(mod p)$
那麼有：$ax+my=1$
然后用 exgcd 求出 x 即可
```cpp
long long inv(long long a,long long m){ 
	long long x,y;
	long long d=exgcd(a,m,x,y); 
	if(d==1) return (x+m)%m; else return -1; //-1為無解 
}
```

[[中國剩餘定理]]