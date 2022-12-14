## 2.4	浮点数

浮点表示对形如 $V=x\times 2^y$ 的有理数进行编码。它对执行涉及非常大的数字与非常接近于 0 的数字，以及更普遍地作为实数运算的近似值的计算是很有用的。
### 2.4.1	二进制小数

理解浮点数的第一步是考虑含有小数值的二进制数字。

数字权的定义与小数点有关，左边位的权是 2 的正幂，右边位的权是 2 的负幂。

### 2.4.2	IEEE 浮点表示

直接表示数字的定点表示法不能很有效地表示非常大的数字 —— 要想表示 $5\times 2^{100}$ 你得用 101 后面跟着 100 个零的位模式来表示。相反，我们希望通过给定 $x$ 和 $y$ 的值，来表示形如 $x\times 2^y$ 的数。

IEEE 浮点标准用 $V=(-1)^s\times M\times 2^E$ 的形式来表示一个数。

- **符号（sign）** $s$ 决定这数是负数（$s=1$）还是正数（$s=0$），而对于数值 0 的符号位解释作为特殊情况处理。
- **尾数（significand）** $M$ 是一个二进制小数，它的范围是 $1\sim 2-\epsilon$，或者是 $0\sim 1-\epsilon$。
- **阶码（exponent）** $E$ 的作用是对浮点数加权，这个权重是 2 的 $E$ 次幂（可能是负数）。

将浮点数的位表示划分为三个字段，分别对这些值进行编码：

- 一个单独的符号位 $s$ 直接编码符号 $s$。
- $k$ 位的阶码字段 $exp=e_{k-1}...e_1e_0$ 编码阶码 $E$。
- $n$ 位小数字段 $frac=f_{n-1}...f_1f_0$ 编码尾数 $M$，但是编码出来的值也依赖于阶码字段的值是否等于 0。

![](/notes/img/2.4.1.png)

给定位表示，根据 `exp` 的值，被编码的值可以分为三种不同的情况（最后一种情况有两个变种）。下图说明了单精度格式的情况

![](/notes/img/2.4.2.png)

#### 情况 1：规格化的值

这是最普遍的情况。当 exp 的位模式既不全为 0 或 1 时，都属于这类情况。在这种情况中，阶码字段被解释为以**偏置**形式表示的有符号整数。阶码的值是 $E=e-Bias$，其中 $e$ 是无符号数，$Bias$ 是一个等于 $2^{k-1}-1$ （单精度是 127，双精度是1023）的偏置值。由此产生的指数的取值范围对于单精度是 $-126\sim +127$，对于双精度是 $-1022\sim +1023$。

小数字段 $frac$ 被解释为描述小数值 $f$，其中 $0\leq f<1$，其二进制表示为 $0.f_{n-1}...f_1f_0$。也就是二进制小数点在最高有效位的左边。尾数定义为 $M=1+f$，也就是我们把 $M$ 看做 $1.f_{n-1}f_{n-1}...f_0$ 的数字。这种方式也叫做 **隐含的以 1 开头的（implied leading 1）** 表示。

#### 情况 2：非规格化的值

当阶码域为全 0 时，所表示的数是**非规格化**形式。在这种情况下，阶码值是 $E=1-Bias$，而尾数的值是 $M=f$，也就是小数字段的值，不包含隐含的开头的 1.

**非规格化值为什么要这样设置偏置值？** 我们将在后文看到，使阶码值为 $1-Bias$ 而非 $-Bias$ 提供了一种从非规格化值平滑转换到规格化值的方法。

非规格化数有两个用途：

- 它提供了一种表示数值 0 的方法。因为使用规格化数，我们我们必须总是使 $M\geq 1$，因此我们就不能表示 0。实际上，$+0.0$ 的浮点表示的位模式为全 0：符号位是 0，阶码字段全为 0（表明是一个非规格化值），而小数域也全为 0，这就得到了 $M=f=0$。当符号位为 1，其他域全为 0 的时候，我们得到值 $-0.0$。根据 IEEE 的浮点格式，值 $+0.0$ 和 $-0.0$ 在某些方面被认为是不同的，而在其他方面是相同的。
- 非规格化数的另一个功能是表示那些非常接近于 0.0 的数。它们提供了一种属性，称为**逐渐下溢（gradual underflow）**，其中可能的数值分布均匀地接近于 0.0。

#### 情况3：特殊值

最后一类数值是当阶码全为 1 的时候出现的。当小数域全为 0 时，得到的值表示无穷，当 $s=0$ 时是 $+\infin$，$s=1$ 是 $-\infin$。当我们把两个非常大的数相乘或者除以零的时候，无穷能够表示溢出的结果。当小数域为非零时，结果值被称为 ”NaN"，即 “Not a Number"。一些运算的结果不能是实数或无穷时就会返回这样的值，如 $\sqrt{-1}$ 和 $\infin - \infin$。在某些应用中它也用于表示未初始化的数据。

### 2.4.4	舍入

对于浮点数的加法和乘法来说，我们可以先计算出准确值，然后转换到合适的精度。在这个过程中可能会溢出，也可能需要舍入来满足 frac 的精度。舍入方式有四种:

|    方式    | 1.40 | 1.60 | 1.50 | 2.50 | -1.50 |
| :--------: | :--: | :--: | :--: | :--: | :---: |
| 向偶数舍入 |  1   |  2   |  2   |  2   |  -2   |
|  向零舍入  |  1   |  1   |  1   |  2   |  -1   |
|  向下舍入  |  1   |  1   |  1   |  2   |  -1   |
|  向上舍入  |  2   |  2   |  2   |  3   |  -1   |

**向偶数舍入** 一个看似随意却很棒的做法，有效解决了单调舍入方式造成的统计偏差。

### 2.4.5	浮点运算

IEEE 标准指定了一个简单的规则，来确定诸如加法和乘法这样的算数运算的结果。把浮点值 x 和 y 看成实数，而某个运算 $\odot$ 定义在实数上，计算将产生 $Round(x\odot y)$，这是对实际运算的精确结果进行舍入后的结果。在实际中，浮点单元的设计者会使用一些技巧来 “在避免执行这种精确的计算” 的前提下保证得到一个正确的舍入结果。

**浮点加法和乘法不具有结合性，这是缺少的最重要的群属性。** 例如若一个编译器给定如下代码片段：

```c
x = a + b + c;
y = b + c + d;
```

编译器可能试图通过产生下列代码来省去一个浮点加法：

```c
t = b + c;
x = a + t;
y = t + d;
```

**浮点加法具有单调性：如果 $a\ge b$，那么对于任何 $x\neq NaN$，都有 $x + a\ge x + b$。浮点乘法的单调性则需要考虑符号的影响。无符号和补码的乘法则没有这些单调性属性。**

### 2.4.6	C 语言的浮点数

- `int->float` 数字不会溢出，但是可能被舍入
- `int->double float->double` 能够保留精确数值（double范围更大）
- `double->float` 因为范围要小一些，所以值可能会溢出。同时可能被舍入（精确度较小）
- `float->int double->int` 值将会向零舍入
