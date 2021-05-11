## 基本概念及证明

Barrett约减是一种高效计算$r = z \mod q$的约减方法，它基于一种低成本的求商运算，估算出一个近似于$q = \lfloor z/p \rfloor$的值$\hat{q}$，这样使得$z-\hat{q}p=r+np$，其中$n$是一个很小的数，因此最终的结果$r$可以通过很少次数的减法计算得出。该低成本的求商运算依赖于一个适当选择的基$b$(如$b = 2^L$， $L$根据模数$p$来选择)的幂，并且计算过程中涉及到计算一个与模数相关的量$\lfloor b^{2k}/p \rfloor$，因此适合对**同一个模进行多次约减**的运算。

下面是Barrett约减的算法描述

![](img/BarrettReduction.bmp)

参照算法描述，我们来验证其正确性，

首先我们要确定$\hat{q}$是$q$的一个好的估计值，即二者非常接近，也就是能使得最后一步做的减法次数非常少。
$$
\lfloor \lfloor z/b^{k-1} \rfloor\cdot \mu /b^{k+1}\rfloor = \lfloor \lfloor \frac{z}{b^{k-1}}\rfloor \cdot \lfloor \frac{b^{2k}}{p}\rfloor \cdot \frac{1}{b^{k+1}} \rfloor \leq \lfloor  \frac{z}{b^{k-1}} \cdot \frac{b^{2k}}{p} \cdot \frac{1}{b^{k+1}} \rfloor = \lfloor\frac{z}{p} \rfloor
$$
即
$$
\hat{q} \leq q
$$
再设
$$
\alpha = \frac{z}{b^{k-1}} - \lfloor \frac{z}{b^{k-1}} \rfloor \\
\beta = \frac{b^{2k}}{p} - \lfloor \frac{b^{2k}}{p} \rfloor
$$
那么就有$0 \leq \alpha < 1, 0 \leq \beta < 1$，此外有
$$
q = \lfloor  \frac{z}{b^{k-1}} \cdot \frac{b^{2k}}{q} \cdot \frac{1}{b^{k+1}}\rfloor =\lfloor \frac{(\lfloor\frac{z}{b^{k-1}}\rfloor + \alpha)(\lfloor\frac{b^{2k}}{p}\rfloor + \beta)}{b^{k+1}}\rfloor \leq \lfloor\hat{q} + \frac{\lfloor\frac{z}{b^{k-1}}\rfloor+\lfloor\frac{b^{2k}}{p}\rfloor+1}{b^{k+1}}\rfloor
$$
由于$z<b^{2k}$，所以$\lfloor \frac{z}{b^{k-1}} \rfloor \leq b^{k+1}-1$；又因为$k = \lfloor \log_b{p}\rfloor + 1$，所以$p \geq b^{k-1}$，也就是说$\lfloor \frac{b^{2k}}{q}\rfloor \leq b^{k+1}$，综合上面两点，该不等式可转化为
$$
q \leq \lfloor \hat{q} + \frac{b^{k+1}-1+b^{k+1}+1}{b^{k+1}}\rfloor = \lfloor \hat{q} + 2 \rfloor
$$
综上，可以证明经由算法第一步计算得到的$\hat{q}$满足以下条件
$$
q-2 \leq \hat{q} \leq q
$$
即$\hat{q}$的确是$q$的一个很好的估计值。

进而，我们可以得到
$$
0 < z - \hat{q}p \leq z - (q-2)p
$$
又$r = z-qp < p$，所以
$$
z-\hat{q}p < 3p
$$
上文中我们提到，$p < b^k$，并且$b \geq 3$，也就是说$3p < 3b^k \leq b^{k+1}$，也就是说步骤2中计算的结果就等价于$r = z-\hat{q}p\mod {b^{k+1}}$，进而能够得到$r < 3p(r = r\mod p, p+r\mod p, 2p+ r \mod p)$这样一个结论，在这一结论下步骤4的减法操作最多需要两次。督导这里可能会想，步骤2不是多此一举吗，直接计算不行吗？如果我理解的没错，这样做也是为了简化计算，假设我们选择的$b$值为$2^L$次方，那么对$b^k$取模的操作只需要去掉该计算数的高$L*k$位即可，同理，步骤一中的$/b^{k+1}$的操作也可以直接通过右移$k+1$位来实现，那这样整个约减操作中就不需要进行除法操作，大大提高了计算效率(唯一需要的除法操作为$\mu = \lfloor b^{2k}/p \rfloor$可以通过预计算的方式为多次同一模数的模约减服务)。