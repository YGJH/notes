好——我把那段 **部分分式分解 + 積分** 的每一步都詳細寫清楚，帶你從代數到最後的反導數驗證。

# 部分分式分解（把 11+x3\dfrac{1}{1+x^3} 拆開）

1. 先因式分解分母（和公式 a3+b3=(a+b)(a2−ab+b2)a^3+b^3=(a+b)(a^2-ab+b^2)）：
    

1+x3=(1+x)(1−x+x2).1+x^3=(1+x)(1 - x + x^2).

2. 設部分分式形式（因為分母有一次因子與二次因子）：
    

11+x3=A1+x+Bx+Cx2−x+1.\frac{1}{1+x^3}=\frac{A}{1+x}+\frac{Bx+C}{x^2-x+1}.

3. 兩邊同乘以 1+x3=(1+x)(x2−x+1)1+x^3=(1+x)(x^2-x+1) 得恆等式：
    

1=A(x2−x+1)+(Bx+C)(1+x).1 = A(x^2-x+1) + (Bx+C)(1+x).

4. 展開並整理係數：
    

1=Ax2−Ax+A+(Bx+C)(1+x)=Ax2−Ax+A+Bx+Bx2+C+Cx=(A+B)x2+(−A+B+C)x+(A+C).\begin{aligned} 1 &= A x^2 - A x + A + (Bx+C)(1+x) \\ &= A x^2 - A x + A + Bx + Bx^2 + C + Cx \\ &= (A+B)x^2 + (-A+B+C)x + (A+C). \end{aligned}

右邊要等於常數 11，即相當於 0⋅x2+0⋅x+10\cdot x^2 + 0\cdot x + 1。所以比較係數得到線性方程組：

{A+B=0−A+B+C=0A+C=1\begin{cases} A + B = 0 \\ - A + B + C = 0 \\ A + C = 1 \end{cases}

5. 解這組方程：
    

- 由 A+B=0A+B=0 得 B=−AB=-A。
    
- 把 B=−AB=-A 代入第二式： −A+(−A)+C=0⇒−2A+C=0⇒C=2A-A + (-A) + C = 0 \Rightarrow -2A + C = 0 \Rightarrow C=2A。
    
- 代入第三式： A+2A=1⇒3A=1⇒A=13A + 2A =1 \Rightarrow 3A=1 \Rightarrow A=\tfrac{1}{3}。  
    所以
    

A=13,B=−13,C=23.A=\tfrac{1}{3},\quad B=-\tfrac{1}{3},\quad C=\tfrac{2}{3}.

6. 得到部分分式結果：
    

11+x3=13⋅11+x+−x+23(x2−x+1)\boxed{\displaystyle \frac{1}{1+x^3} =\frac{1}{3}\cdot\frac{1}{1+x} +\frac{-x+2}{3(x^2-x+1)} }

# 對每一項積分（一步步來）

將結果分成兩部分積分：

∫11+x3 dx=13∫11+x dx  +  13∫−x+2x2−x+1 dx.\int \frac{1}{1+x^3}\,dx = \frac{1}{3}\int\frac{1}{1+x}\,dx \;+\; \frac{1}{3}\int\frac{-x+2}{x^2-x+1}\,dx.

## 第一項

13∫11+x dx=13ln⁡∣1+x∣.\frac{1}{3}\int\frac{1}{1+x}\,dx = \frac{1}{3}\ln|1+x|.

## 第二項（把分子拆成「導數×常數 + 常數」）

注意 (x2−x+1)′=2x−1 (x^2-x+1)' = 2x-1。我們把分子寫成 α(2x−1)+β \alpha(2x-1)+\beta：

−x+2=α(2x−1)+β.-x+2 = \alpha(2x-1)+\beta.

比較係數：

2α=−1⇒α=−12,−α+β=2⇒β=32.2\alpha = -1 \Rightarrow \alpha = -\tfrac{1}{2},\qquad -\alpha + \beta = 2 \Rightarrow \beta = \tfrac{3}{2}.

所以

−x+2x2−x+1=−12⋅2x−1x2−x+1  +  32⋅1x2−x+1.\frac{-x+2}{x^2-x+1} = -\tfrac{1}{2}\cdot\frac{2x-1}{x^2-x+1} \;+\; \tfrac{3}{2}\cdot\frac{1}{x^2-x+1}.

把係數 13\tfrac{1}{3} 帶回去：

13∫−x+2x2−x+1 dx=13[−12∫2x−1x2−x+1dx  +  32∫1x2−x+1dx].\frac{1}{3}\int\frac{-x+2}{x^2-x+1}\,dx = \frac{1}{3}\Big[-\tfrac{1}{2}\int\frac{2x-1}{x^2-x+1}dx \;+\; \tfrac{3}{2}\int\frac{1}{x^2-x+1}dx\Big].

- 第一個子積分：∫2x−1x2−x+1dx=ln⁡(x2−x+1)\displaystyle \int\frac{2x-1}{x^2-x+1}dx=\ln(x^2-x+1)。因此係數為 −16ln⁡(x2−x+1)-\tfrac{1}{6}\ln(x^2-x+1)。
    
- 第二個子積分：需把 x2−x+1x^2-x+1 完全平方化：
    

x2−x+1=(x−12)2+34.x^2-x+1 = \big(x-\tfrac{1}{2}\big)^2 + \tfrac{3}{4}.

所以

∫1x2−x+1dx=∫1(x−12)2+(32)2dx=23arctan⁡ ⁣(x−1232)=23arctan⁡ ⁣(2x−13).\int\frac{1}{x^2-x+1}dx = \int\frac{1}{\big(x-\tfrac12\big)^2 + (\tfrac{\sqrt3}{2})^2}dx = \frac{2}{\sqrt{3}}\arctan\!\bigg(\frac{x-\tfrac12}{\tfrac{\sqrt3}{2}}\bigg) = \frac{2}{\sqrt3}\arctan\!\bigg(\frac{2x-1}{\sqrt3}\bigg).

乘上前面係數 13⋅32=12\frac{1}{3}\cdot\frac{3}{2}=\tfrac{1}{2}，得到這一項合併為

12⋅23arctan⁡ ⁣(2x−13)=13arctan⁡ ⁣(2x−13).\tfrac{1}{2}\cdot\frac{2}{\sqrt3}\arctan\!\bigg(\frac{2x-1}{\sqrt3}\bigg) = \frac{1}{\sqrt3}\arctan\!\bigg(\frac{2x-1}{\sqrt3}\bigg).

## 把所有項合起來

總結果為

∫11+x3 dx=13ln⁡∣1+x∣  −  16ln⁡(x2−x+1)  +  13arctan⁡ ⁣(2x−13)+C\boxed{\displaystyle \int \frac{1}{1+x^3}\,dx = \frac{1}{3}\ln|1+x| \;-\; \frac{1}{6}\ln(x^2-x+1) \;+\; \frac{1}{\sqrt3}\arctan\!\Big(\frac{2x-1}{\sqrt3}\Big) + C }

# 驗算（簡短）

對上式微分會回到 11+x3\dfrac{1}{1+x^3}，你可以把每一項微分、通分後看到恰好回到原來的被積分式（如果要我可以把驗證的每一步也列出來）。

---

如果要我把每一步（乘開、整理、比較係數、積分替換、完成平方、最後微分驗證）的代數計算逐行列成純代數式，我可以再貼出「每行展開的中間式」給你看。想要哪一種更細的呈現？