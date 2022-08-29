## 2.3	整数运算

### 2.3.1	无符号加法

我们将操作 $+^u_w$描述为整数和 $x+y$ 截断为 $w$ 位的结果 ：

 对满足 $0 \leq x,y<2^w$ 的 $x$ 和 $y$ 有：

$$x+^u_wy=\begin{cases}x+y,\quad x+y< 2^w(正常) \\x+y-2^w,\quad 2^w\leq x+y\leq 2^{w+1}(溢出) \end{cases}$$

*当 $x+y<2^w$ 时，和的$2^{w+1}$ 位即最高位为0，所以丢弃它并不会影响结果。当 $x+y$  的值超过这个值（即溢出）时，和的$2^{w+1}$ 为1，而丢弃它就等价于从和中减去$2^w$。* 说一个算术运算溢出，是指完整的整数不能放到数据类型的字长限制中。

C 语言怎么检测溢出呢？

```c
// 0 <= x, y <= UMax_w
int isOverflow(unsigned int x, unsigned int y) {
    unsigned res = x + y;
    return res < x; // or res < y
}
```

*看一眼最初的公式中 x 和 y 的取值，我们会发现正常情况下必定有 `res >= x && res >= y`，而在溢出时，由于`x < pow(2, w) && y < pow(2, w)`，所以与 x 或 y 相加的必定是个负数，即 `res < x && res < y`。*

#### 原理：无符号数求反

当 $x=0$ 时，对满足 $0\leq x<2^w$ 的任意 $x$，其 $w$ 位的无符号逆元 $-^u_wx$ 由下式给出：

$$-^u_wx=\begin{cases}x,\quad x=0 \\2^w-x,\quad x>0 \end{cases}$$

*$(x+2^w-x)$ mod $2^w$ = $2^w$ mod $2^w$ = 0*

### 2.3.2	补码加法

#### 原理：补码加法

对满足 $-2^{w-1}\leq x,y\leq 2^{w-1}-1$ 的整数 $x$ 和 $y$，有：

$$x+^u_wy=\begin{cases}x+y-2^w,\quad 2^{w-1}\leq x+y（正溢出 \\x+y,\quad -2^{w-1} \leq x+y\leq 2^{w-1} (正常) \\ x+y+2^w, \quad x + y<-2^{w-1}(负溢出) \end{cases}$$

*既然补码加法与无符号数加法有相同的位级表示，我们就可以按如下步骤表示运算$+^t_w$：将其参数转换为无符号数，执行无符号数加法，再将结果转换为补码：*

$$x+^t_wy\doteq U2T_w(T2U_w(x)+^u_wT2U_w(y))$$

由之前内容，$$T2U_w(x)=x_{w-1}2*w+x$$

**一个值得注意的结论是， `x >> k` 等价于 `x / 2^k`，`x << k` 等价于 `x * 2^k`**