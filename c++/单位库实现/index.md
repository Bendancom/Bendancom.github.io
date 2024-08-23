# C&#43;&#43;单位库实现


在现实世界中的物理规律都需要物理单位，相较于直接进行数字的计算，带有物理量的计算可进行量纲校验与单位换算。其中有两类，其一是静态实现，在编译期进行校验，另一种是动态实现，在运行期进行校验。为将两种方法的优点进行统一，便自行开发单位库。

&lt;!--more--&gt;

## 前置需求

- 线性代数（关于线性空间的变换）
- 一丢丢英语
- C&#43;&#43;23

## 数学理解

对于相同指数下的单位的变换，可认为是一个基于一种单位基底的线性空间到另一种单位基底的线性空间的线性变换
若对基进行变换，由于绝大部分物理单位间的变换都是线性变换，只有一个量纲下的物理量是仿射变换(说的就是你，温度)，先考虑绝大多数，那么可直接乘以一个变换矩阵，得到由新的基组成的单位。
但该前提是所有量纲的指数都是 \(1\)，或者说，在一个量纲中，不同的指数有不同的尺规。
如 \(\mathrm{in}\) 到 \(\mathrm{m}\) 的尺规是 0.0254，但 \(\mathrm{in}^{2}\) 到 \(\mathrm{m}^{2}\) 的尺规是 \(6.4516(2.54^{2})\)。
同时单位制词头的不同也带来了不同的尺规，\(\mathrm{m}\) 到 \(\mathrm{km}\) 的尺规为 \(1000\)，\(\mathrm{m}^{2}\) 到 \(\mathrm{km}^{2}\) 的尺规为 \(1000,000\)

\[
    \begin{align}
    \boldsymbol{x}&#39; &amp;= \operatorname{diag}(m_{1,1}^{p_{1,1}}c_{1,1}^{p_{1,1}}x_{1,1},\cdots,m_{n,n}^{p_{n,n}}c_{n,n}^{p_{n,n}}x_{n,n})\notag \\ 
    &amp;= \operatorname{diag}[(m_{1,1}c_{1,1})^{p_{1,1}},\cdots,(m_{n,n}c_{n,n})^{p_{n,n}}] \boldsymbol{x}\notag
    \end{align}
\]
其中：

- \(\boldsymbol{p}\) 用对角矩阵表示的各量纲的指数，其中元素为 \(p_{i,j}\)
- \(\boldsymbol{x}\) 原量纲单位的对角矩阵，其中元素为 \(x_{i,j}\)
- \(\boldsymbol{m}\) 单位制词头的对角矩阵，在量纲指数为 \(1\) 下的尺规，其中元素为 \(m_{i,j}\)
- \(\boldsymbol{c}\) 变换的对角矩阵，在量纲指数为 \(1\) 下的尺规，其中元素为 \(c_{i,j}\)
- \(\boldsymbol{x}&#39;\) 现量纲的对角矩阵，其中元素为 \(x_{i,j}^{\prime}\)
- \(n\) 代表维度数

对其求行列式可得：

\[
    \begin{align}
    |\boldsymbol{x}&#39;| &amp;= |\operatorname{diag}[(m_{1,1}c_{1,1})^{p_{1,1}},\cdots,(m_{n,n}c_{n,n})^{p_{n,n}}]| \cdot |\boldsymbol{x}| \notag \\
    \frac{|\boldsymbol{x}&#39;|}{|\boldsymbol{x}|} &amp;= |\operatorname{diag}[(m_{1,1}c_{1,1})^{p_{1,1}},\cdots,(m_{n,n}c_{n,n})^{p_{n,n}}]| \notag\\
    \frac{|\boldsymbol{x}&#39;|}{|\boldsymbol{x}|} &amp;= \prod_{i=1}^{n}(m_{i,i}c_{i,i})^{p_{i,i}} \notag
    \end{align}
\]

现在再来考虑温度，在温度量纲指数为 \(1\) 时，温度单位的变换为不过原点的一次函数。因而最好的就是只有一个单位：开尔文(K)。

- 在温度仅表示温差这一含义时，可忽略温差，只看温标。
- 在仅有温度这一量纲时，需同时看温标与温差。

因而，在只有温度量纲且指数为\(1\) 时，允许从开尔文 \(\Longleftrightarrow\) 其他温度单位的转换。

那么有两种实现方法：

1. 将量纲线性空间扩展为 \(n&#43;1\) 维。
2. 单列在一次量纲的温度变换，，在温度量纲中只有开尔文，如同物质的量只有摩尔一样。

本库采用方法 2

## 实现思路

单位库应包含如下内容

- 量纲
- 数

因而难点在于量纲，单位库的一切都是围绕量纲的，数只不过是附带的内容(虽然在计算中这才是主要的)。

### 头文件使用

- `&lt;concepts&gt;`
- `&lt;stdfloat&gt;`

### 量纲

量纲是复杂的，尤其是将静态与动态相结合的同时要有高扩展性。

#### 量纲核心

- 量纲类型
- 指数

##### 基类

基类是一切扩展性的根源，要保留有最高的扩展性，因而使用模板进行定义：
其中应至少含有

- 尺规(`magnitude`)，要求范围为有理数\(\mathbb{Q}\)
- 指数(`power`)，要求范围为整数\(\mathbb{Z}\)[^1]
- 简名(`short_name`)，要求为字符串
- 量纲简名(`dimension_short_name`)，要求为字符串

[^1]:实际在物理中是真有指数为有理数的量纲操作的，但这样不好计算，详情参照 [小时百科-物理量和单位转换-例3](https://wuli.wiki/online/Units.html)

为此定义约束(`concept`)：

```C&#43;&#43;
template&lt;typename T&gt;
concept rational = requires (T x, T y) {
    {x &#43; y} -&gt; std::convertible_to&lt;T&gt;;
    {x - y}  -&gt; std::convertible_to&lt;T&gt;;
    {x* y} -&gt; std::convertible_to&lt;T&gt;;
    {x / y} -&gt; std::convertible_to&lt;T&gt;;
    {-x} -&gt; std::same_as&lt;T&gt;;
    x = y;
    requires T(3) / T(2) == T(1.5);
    requires std::totally_ordered&lt;T&gt;;
};
```

至于为什么不用`C&#43;&#43;`自带的`std::floating_point`与`std::integral`，因为这两个追溯到根源的实现是结构的模板特化，也就是说如果有新的自定义数据类型，可以表达数与进行数运算但不会被识别为数，是不严谨的。

```C&#43;&#43;
template&lt;rational magnitude_t,rational power_t&gt;
struct basic_dimension{
    magnitude_t magnitude;
    power_t power;
};
```

从基类中我们定义最通用的内存空间最小类：

```C&#43;&#43;
struct dimension : basic_dimension&lt;std::float16_t,std::float16_t&gt; {};
```

参考资料：  
[1] [addis.物理量和单位转换[EB/OL].小时百科.https://wuli.wiki/online/Units.html](https://wuli.wiki/online/Units.html)  
[2] [lilong.CommonUnits[CP/OL].gitee.(2021-04-19)[2024-05-22].https://gitee.com/llongww/common-units](https://gitee.com/llongww/common-units)


---

> 作者:   
> URL: http://localhost:1313/c&#43;&#43;/%E5%8D%95%E4%BD%8D%E5%BA%93%E5%AE%9E%E7%8E%B0/  

