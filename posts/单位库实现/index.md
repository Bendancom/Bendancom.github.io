# 单位库实现


在现实世界中的物理规律都需要物理单位，相较于直接进行数字的计算，带有物理量的计算可进行量纲校验与单位换算。其中有两类，其一是静态实现，在编译期进行校验，另一种是动态实现，在运行期进行校验。为将两种方法的优点进行统一，便自行开发单位库。

&lt;!--more--&gt;

## 前置需求

- 线性代数（关于线性空间的变换）
- 一些英语，有翻译也可（编程必须，不然容易看不懂变量）
- C&#43;&#43;23

## 数学理解

对于相同指数下的单位的变换，可认为是一个基于一种单位基底的线性空间到另一种单位基底的线性空间的线性变换。  
若对基进行变换，由于绝大部分物理单位间的变换都是线性变换，只有一个量纲下的物理量是仿射变换(说的就是你，温度)，先考虑绝大多数，那么可直接乘以一个变换矩阵，得到由新的基组成的单位。  
但该前提是所有量纲的指数都是 $1$，或者说，在一个量纲中，不同的指数有不同的尺规。
如 $\mathrm{in}$ 到 $\mathrm{m}$ 的尺规是 0.0254，但 $\mathrm{in}^{2}$ 到 $\mathrm{m}^{2}$ 的尺规是 $6.4516(2.54^{2})$。  
同时单位制词头的不同也带来了不同的尺规，$\mathrm{m}$ 到 $\mathrm{km}$ 的尺规为 $1000$，$\mathrm{m}^{2}$ 到 $\mathrm{km}^{2}$ 的尺规为 $1000,000$

$$
    \begin{align}
    \boldsymbol{x}&#39; &amp;= \operatorname{diag}(m_{1,1}^{p_{1,1}}c_{1,1}^{p_{1,1}}x_{1,1},\cdots,m_{n,n}^{p_{n,n}}c_{n,n}^{p_{n,n}}x_{n,n})\notag \\ 
    &amp;= \operatorname{diag}[(m_{1,1}c_{1,1})^{p_{1,1}},\cdots,(m_{n,n}c_{n,n})^{p_{n,n}}] \boldsymbol{x}\notag
    \end{align}
$$
其中：

- $\boldsymbol{p}$ 用对角矩阵表示的各量纲的指数，其中元素为 $p_{i,j}$
- $\boldsymbol{x}$ 原量纲单位的对角矩阵，其中元素为 $x_{i,j}$
- $\boldsymbol{m}$ 单位制词头的对角矩阵，在量纲指数为 $1$ 下的尺规，其中元素为 $m_{i,j}$
- $\boldsymbol{c}$ 变换的对角矩阵，在量纲指数为 $1$ 下的尺规，其中元素为 $c_{i,j}$
- $\boldsymbol{x}&#39;$ 现量纲的对角矩阵，其中元素为 $x_{i,j}^{\prime}$
- $n$ 代表维度数

对其求行列式可得：

$$
    \begin{align}
    |\boldsymbol{x}&#39;| &amp;= |\operatorname{diag}[(m_{1,1}c_{1,1})^{p_{1,1}},\cdots,(m_{n,n}c_{n,n})^{p_{n,n}}]| \cdot |\boldsymbol{x}| \notag \\
    \frac{|\boldsymbol{x}&#39;|}{|\boldsymbol{x}|} &amp;= |\operatorname{diag}[(m_{1,1}c_{1,1})^{p_{1,1}},\cdots,(m_{n,n}c_{n,n})^{p_{n,n}}]| \notag\\
    \frac{|\boldsymbol{x}&#39;|}{|\boldsymbol{x}|} &amp;= \prod_{i=1}^{n}(m_{i,i}c_{i,i})^{p_{i,i}} \notag
    \end{align}
$$

现在再来考虑温度，在温度量纲指数为 $1$ 时，温度单位的变换为不过原点的一次函数。因而最好的就是只有一个单位：开尔文(K)。

- 在温度仅表示温差这一含义时，可忽略温差，只看温标。
- 在仅有温度这一量纲时，需同时看温标与温差。

因而，在只有温度量纲且指数为$1$ 时，允许从开尔文 $\Longleftrightarrow$ 其他温度单位的转换。

那么有两种实现方法：

1. 将量纲线性空间扩展为 $n&#43;1$ 维。
2. 单列在一次量纲的温度变换，在温度量纲中只有开尔文，如同物质的量只有摩尔一样。

本库采用方法 2

## 实现思路

单位库应包含如下内容

- 量纲
- 数

因而难点在于量纲，单位库的一切都是围绕量纲的，数只不过是附带的内容(虽然在计算中这才是主要的)。

### 头文件使用

- `&lt;concepts&gt;`
- `&lt;stdfloat&gt;` 可选，若觉得`float`过大可使用`float16_t`

### 量纲

量纲是复杂的，尤其是将静态与动态相结合的同时要有高扩展性。
在这里本人是将单位制词头与单位拆分，这样便于扩展，不用再实现同一单位的不同单位制词头。

- 量纲类型
  - 单位制词头
  - 量纲单位
- 指数

## 实现

### 数学

#### 约束

定义数的约束(`concept`)：

```C&#43;&#43;
template&lt;typename T&gt;
concept rational = requires (T x, T y) {
    {x &#43; y} -&gt; std::convertible_to&lt;T&gt;;
    {x - y}  -&gt; std::convertible_to&lt;T&gt;;
    {x * y} -&gt; std::convertible_to&lt;T&gt;;
    {x / y} -&gt; std::convertible_to&lt;T&gt;;
	{x &#43;= y} -&gt; std::convertible_to&lt;T&gt;;
	{x -= y}  -&gt; std::convertible_to&lt;T&gt;;
	{x *= y} -&gt; std::convertible_to&lt;T&gt;;
	{x /= y} -&gt; std::convertible_to&lt;T&gt;;
    {-x} -&gt; std::same_as&lt;T&gt;;
    x = y;
    requires T(3) / T(2) == T(1.5);
    requires std::totally_ordered&lt;T&gt;;
};
```

```C&#43;&#43;
template&lt;typename T&gt;
concept integral = requires (T x, T y) {
	{x &#43; y} -&gt; std::convertible_to&lt;T&gt;;
	{x - y}  -&gt; std::convertible_to&lt;T&gt;;
	{x * y} -&gt; std::convertible_to&lt;T&gt;;
	{x / y} -&gt; std::convertible_to&lt;T&gt;;
	{x &#43;= y} -&gt; std::convertible_to&lt;T&gt;;
	{x -= y}  -&gt; std::convertible_to&lt;T&gt;;
	{x *= y} -&gt; std::convertible_to&lt;T&gt;;
	{x /= y} -&gt; std::convertible_to&lt;T&gt;;
	{-x} -&gt; std::convertible_to&lt;T&gt;;
	x = y;
	requires T(3) / T(2) == T(1) || T(3) / T(2) == T(2);
	requires std::totally_ordered&lt;T&gt;;
};
```

至于为什么不用`C&#43;&#43;`自带的`std::floating_point`与`std::integral`，因为这两个追溯到根源的实现是结构的模板特化，也就是说如果有新的自定义数据类型，可以表达数与进行数运算但不会被识别为数，是不严谨的。

#### 幂函数

在C&#43;&#43;23中标准库仍有大部分未实现编译期常量的数学函数，其中就有`pow`，且所有数学函数都未实现泛型，因而要自己实现。
如下是简易的实现，便于演示没用快速幂算法。

```C&#43;&#43;
template&lt;rational T&gt;
constexpr T pow(rational auto x,integral auto p){
    if (p &gt; 0){
        T temp = 1;
        for (size_t i=0;i&lt;p;i&#43;&#43;)
            temp = temp * x;
        return temp;
    }
    else if (p == 0){
        if (x != 0)
            return 1;
        else
            throw std::exception();
    }
    else {
        T temp = 1;
        for (size_t i=0;i&lt;-p;i&#43;&#43;)
            temp = temp / x;
        return temp;
    }
}
```

### 基类（量纲单位）

基类是一切扩展性的根源，要保留有最高的扩展性，因而使用模板进行定义：

#### 基类数据

其中应至少含有

- 尺规(`magnitude`)，要求范围为有理数$\mathbb{Q}$
- 指数(`power`)，要求范围为整数$\mathbb{Z}$[^1]
- 简名(`short_name`)，要求为字符串
- 量纲简名(`dimension_short_name`)，要求为字符串

[^1]:实际在物理中是真有指数为有理数的量纲操作的，但这样不好计算，详情参照 [小时百科-物理量和单位转换-例3](https://wuli.wiki/online/Units.html)


如下为基类中数据的定义：

```C&#43;&#43;
template&lt;rational magnitude_t,integral power_t&gt;
struct basic_dimension{
    magnitude_t magnitude;
    power_t power;
    std::string_view short_name;
    std::string_view dimension_short_name;
};
```

其中`std::string_view`还可被优化为`const char* const`但除非是超大量的单位存储，否则这优化是没啥必要的（因为不如`std::string_view`类方便，在理解意义上的）

从基类中我们定义最通用的内存空间最小类：

```C&#43;&#43;
using dimension = basic_dimension&lt;std::float,std::int8_t&gt;;
```

#### 基类函数

在基类中应有完全比较函数即`operator==()`函数，比较全部四个数据；
也应有类型比较函数`type_equal`，比较幂指数与唯一单位名；
还应有模板转换函数，为特殊超大单位提供无损转换

```C&#43;&#43;
template&lt;rational magnitude_t,integral power_t&gt;
struct basic_dimension{
    constexpr operator==(...) const;
    constexpr type_equal(...) const;
    static constexpr basic_dimension&lt;...&gt; convert(...) const;
};
```
要达成以上条件，其中的约束为：

- 为基类的任意派生
- 二者有可隐式转换的类型

```C&#43;&#43;
template&lt;template&lt;rational R,integral I&gt; typename dimension_t,rational R,integral I&gt;
requires requires {
    requires std::derived_from&lt;dimension_t&lt;R,I&gt;,basic_dimension&lt;R,I&gt;&gt;; 
    requires std::common_with&lt;magnitude_t,R&gt;;
    requires std::common_with&lt;power_t,I&gt;;
}
constexpr bool operator==(const dimension_t&lt;R,I&gt;&amp; dimen) const {
    ...
}
```

### 单位制词头结构

其中有两种数据类型，这导致该结构更像是对基类（量纲单位）的包装

- 单位制词头
- 基类（量纲单位）

```C&#43;&#43;
template&lt;rational magnitude_t,integral power_t&gt;
struct basic_MP_dimension {
    MP metric_prefix;
    basic_dimension&lt;magnitude_t, power_t&gt; dimen;
};
```

#### 单位制词头

两种方式

- 采用对数表示，运用`char`数据类型存储
  - 优：空间换时间（但不至于这么省，又不是极限单片机），可能大概也许比较易懂
  - 劣：浪费计算时间
- 枚举类型 &#43; 固定数组存储
  - 优：时间换空间
  - 劣：每多一个词头多占用`4`字节（默认`float = float32_t`，占用 32`bit`= 4`byte`）

选用第二种方法——枚举类型 &#43; 固定数组存储。
采用`std::array`，会多8字节。
```C&#43;&#43;
enum class MP : char{
    Q,
    R,
    Y,
    Z,
    E,
    P,
    T,
    G,
    M,
    k,
    h,
    da,
    null,
    d,
    c,
    m,
    μ,
    n,
    p,
    f,
    a,
    z,
    y,
    r,
    q,
};

auto MP_array = std::to_array&lt;float&gt;({
    1e30,
    1e27,
    1e24,
    1e21,
    1e18,
    1e15,
    1e12,
    1e9,
    1e6,
    1e3,
    1e2,
    1e1,
    1,
    1e-1,
    1e-2,
    1e-3,
    1e-6,
    1e-9,
    1e-12,
    1e-15,
    1e-18,
    1e-21,
    1e-24,
    1e-27,
    1e-30
});
```
#### 结构函数

与基类（量纲单位）类似

```C&#43;&#43;
template&lt;rational magnitude_t,integral power_t&gt;
struct basic_MP_dimension {
    constexpr bool operator==(...) const;
    constexpr bool type_equal(...) const;
    static constexpr basic_dimension&lt;...&gt; convert(...) const;
};
```

约束略有不同，只需确保泛型模板参数之间可以隐式转换即可：
```C&#43;&#43;
template&lt;rational R,integral I&gt;
requires requires {
    requires std::common_with&lt;magnitude_t,R&gt;;
    requires std::common_with&lt;power_t,I&gt;;
}
constexpr bool operator==(const basic_MP_dimension&lt;R,I&gt;&amp; MP_dimen) const {
    ...
}
```

### 单位类

前面的可以说是都是基础，接下来到了重点：单位类

```C&#43;&#43;
template&lt;rational value_t,rational magnitude_t,integral power_t&gt;
class basic_unit;
```

#### 类型别名

使用的类型别名

- `V`，一种类型，可转换为`value_t`，其约束为`std::convertible_to&lt;V,value_t&gt;`
- `M`，一种类型，可转换为`magnitude_t`，其约束为`std::convertible_to&lt;M,magnitude_t&gt;`
- `P`，一种类型，可转换为`power_t`，其约束为`std::convertible_to&lt;P,power_t&gt;`

以下省略`V`，`M`，`P`类型的约束

```C&#43;&#43;
using dimension_t = basic_MP_dimension&lt;magnitude_t, power_t&gt;;
using MP_Less_t = MP_dimension_less&lt;magnitude_t, power_t&gt;;
```

其中 `MP_dimension_less` 为：
```C&#43;&#43;
template&lt;rational M,integral P&gt;
struct MP_dimension_less {
    constexpr bool operator()(const basic_MP_dimension&lt;M, P&gt;&amp; ldimen,const basic_MP_dimension&lt;M, P&gt;&amp; rdimen) const {
        return ldimen.dimen.dimension_name &lt; rdimen.dimen.dimension_name;
    }
};
```

#### 数据类型

- `value`，类型为`value_t`
- `dimens`，类型为`std::set&lt;dimension_t,MP_Less_t&gt;`

#### 模板类型转换

要注意因为是任意匹配类型，要进行模板类型转换才能插入
使用 `basic_MP_dimension::convert`

#### 初始化

- 空初始化
- 拷贝初始化
- 值初始化
- 值与单个单位类型初始化
- 值与复数单位类型初始化

```C&#43;&#43;
constexpr basic_unit() {}
constexpr basic_unit(const basic_unit&lt;V, M, P&gt;&amp; unit);
constexpr basic_unit(const value_t&amp; value);
constexpr basic_unit(const value_t&amp; value,const basic_MP_dimension&lt;M, P&gt;&amp; dimen);
template&lt;std::ranges::input_range Rng&gt;
constexpr basic_unit(const value_t&amp; value, Rng&amp;&amp; range);
```

##### 初始化约束

因为对容器里存储类型的判断无法直接泛型，所以只能由同名变量来判断了。

```C&#43;&#43;
template&lt;std::ranges::input_range Rng&gt;
requires requires {
    requires std::same_as&lt;decltype(std::ranges::range_value_t&lt;Rng&gt;::metric_prefix), MP&gt;;
    requires std::convertible_to&lt;decltype(std::ranges::range_value_t&lt;Rng&gt;::dimen.magnitude),magnitude_t&gt;;
    requires std::convertible_to&lt;decltype(std::ranges::range_value_t&lt;Rng&gt;::dimen.power),power_t&gt;;
    requires std::same_as&lt;decltype(std::ranges::range_value_t&lt;Rng&gt;::dimen.short_name), std::string_view&gt;;
    requires std::same_as&lt;decltype(std::ranges::range_value_t&lt;Rng&gt;::dimen.dimension_name), std::string_view&gt;;
}
constexpr basic_unit(const value_t&amp; value, Rng&amp;&amp; range);
```

### 尺度计算

```C&#43;&#43;
value_t MagnitudeCalculateFrom(const basic_unit&lt;V, M, P&gt;&amp; u) const {
    value_t all_magnitude = 1;
    for(auto i : dimens) {
        value_t magni = 1;
        auto dimen = basic_MP_dimension&lt;M, P&gt;::convert(i);
        if (auto iter = u.getDimensions().find(dimen); iter != u.getDimensions().end()){
            magni *= pow&lt;value_t&gt;(iter-&gt;dimen.magnitude / i.dimen.magnitude, iter-&gt;dimen.power);
            magni *= pow&lt;value_t&gt;(MP_array[(char)iter-&gt;metric_prefix] / MP_array[(char)i.metric_prefix],iter-&gt;dimen.power);
        }
        all_magnitude *= magni;
    }
    return all_magnitude;
}
```

### 错误定义

在这里作者并未自定义错误，全部采用`std::exception`错误（毕竟是演示）

### 加减


对于加减的计算，先是要比较其量纲单位是否匹配，后是尺度转换，再是相加
因而其返回值为`std::expected`。
同时应返回的`basic_unit`要取二者共有的类型，即`std::common_type_t&lt;magnitude_t,M&gt;`等

#### 加减函数

以加法函数为例，错误在返回值中传出而非直接抛出
以下为函数声明[^2]：

[^2]:在以下情形中，`V`、`M`、`P`类型都是由`std::common_with`进行约束
```C&#43;&#43;
constexpr std::expected&lt;
    basic_unit&lt;
        std::common_type_t&lt;value_t,V&gt;,
        std::common_type_t&lt;magnitude_t,M&gt;,
        std::common_type_t&lt;power_t,P&gt;
    &gt;,
    std::exception
&gt; plus(const basic_unit&lt;V,M,P&gt;&amp; unit) const noexcept;
```

而在函数中为比较量纲，那么

1. 容器大小相同，即`size`相同
2. 由于使用的容器`std::set`为有序容器，即可直接按顺序比较即可。因为顺序不同时二者一定不同

最终返回公共类型

以下为函数实现[^3]：

```C&#43;&#43;
if (dimens.size() == unit.getDimensions().size()) {
    auto i = dimens.begin();
    auto j = unit.getDimensions().begin();
    for (;i != dimens.end() &amp;&amp; j != unit.getDimensions().end();) {
        if (!i-&gt;type_equal(*j))
            return std::unexpected(std::exception());
        i&#43;&#43;;
        j&#43;&#43;;
    }
    return basic_unit&lt;
        std::common_type_t&lt;value_t,V&gt;,
        std::common_type_t&lt;magnitude_t,M&gt;,
        std::common_type_t&lt;power_t,P&gt;
    &gt;(value &#43; MagnitudeCalculateFrom(unit) * unit.getValue(),dimens);
}
else return std::unexpected(std::exception());
```

[^3]:不同模板之间无法访问私有变量，所以只能从公开接口获取

同理可得到减法函数

#### 加减运算符

套用加减函数并抛出可能的错误
以加法运算符为例

```C&#43;&#43;
auto u = plus(unit);
if (u.has_value())
    return u.value();
else throw u.error();
```

减法运算符同理

### 乘

对于乘法，其并没有任何的错误可被抛出

#### 乘法函数

以下为乘法的函数声明[^2]：

```C&#43;&#43;
constexpr basic_unit&lt;
    std::common_type_t&lt;value_t,V&gt;,
    std::common_type_t&lt;magnitude_t,M&gt;,
    std::common_type_t&lt;power_t,P&gt;
&gt; multiply(const basic_unit&lt;V, M, P&gt;&amp; unit) const noexcept;
```

对于乘法而言无需去比较量纲，但需要去尝试去找出相对应的量纲并在指数上相加，如果找不到就插入该找不到的量纲。

以下为函数实现[^3]：

```C&#43;&#43;
std::set&lt;
    basic_MP_dimension&lt;
        std::common_type_t&lt;magnitude_t,M&gt;,
        std::common_type_t&lt;power_t,P&gt;
    &gt;,
    MP_dimension_less&lt;
        std::common_type_t&lt;magnitude_t,M&gt;,
        std::common_type_t&lt;power_t,P&gt;
    &gt;
&gt; tmp_dimens;
for (auto i : dimens){
    auto dimen = basic_MP_dimension&lt;M, P&gt;::convert(i);
    basic_dimension&lt;
        std::common_type_t&lt;magnitude_t,M&gt;,
        std::common_type_t&lt;power_t,P&gt;
    &gt; tmp_dimen {
        i.dimen.magnitude,
        i.dimen.power,
        i.dimen.short_name,
        i.dimen.dimension_name,
    };
    if (auto iter = unit.getDimensions().find(dimen); iter != unit.getDimensions().end())
        tmp_dimen.power &#43;= iter-&gt;dimen.power;
    tmp_dimens.insert(
        basic_MP_dimension&lt;
            std::common_type_t&lt;magnitude_t,M&gt;,
            std::common_type_t&lt;power_t,P&gt;
        &gt; {
            i.metric_prefix,
            tmp_dimen
        }
    );
}
```

#### 乘法运算符

直接使用乘法函数即可

### 除

对于除法，与乘法相比需判断值是否为`0`

#### 除法函数

以下为除法的函数声明[^2]：

```C&#43;&#43;
constexpr std::expected&lt;
    basic_unit&lt;
        std::common_type_t&lt;value_t,V&gt;,
        std::common_type_t&lt;magnitude_t,M&gt;,
        std::common_type_t&lt;power_t,P&gt;
    &gt;,
    std::exception
&gt; devide(const basic_unit&lt;V, M, P&gt;&amp; unit) const noexcept;
```

以下为函数实现[^3]：

```C&#43;&#43;
if (unit.getValue() == 0)
    return std::unexpected(std::exception());
std::set&lt;
    basic_MP_dimension&lt;
        std::common_type_t&lt;magnitude_t,M&gt;,
        std::common_type_t&lt;power_t,P&gt;
    &gt;,
    MP_dimension_less&lt;
        std::common_type_t&lt;magnitude_t,M&gt;,
        std::common_type_t&lt;power_t,P&gt;
    &gt;
&gt; tmp_dimens;
for (auto i : dimens){
    auto dimen = basic_MP_dimension&lt;M, P&gt;::convert(i);
    basic_dimension&lt;
        std::common_type_t&lt;magnitude_t,M&gt;,
        std::common_type_t&lt;power_t,P&gt;
    &gt; tmp_dimen {
        i.dimen.magnitude,
        i.dimen.power,
        i.dimen.short_name,
        i.dimen.dimension_name,
    };
    if (auto iter = unit.getDimensions().find(dimen); iter != unit.getDimensions().end())
        tmp_dimen.power -= iter-&gt;dimen.power;
    tmp_dimens.insert(
        basic_MP_dimension&lt;
            std::common_type_t&lt;magnitude_t,M&gt;,
            std::common_type_t&lt;power_t,P&gt;
        &gt; {
            i.metric_prefix,
            tmp_dimen
        }
    );
}
return basic_unit&lt;
    std::common_type_t&lt;value_t,V&gt;,
    std::common_type_t&lt;magnitude_t,M&gt;,
    std::common_type_t&lt;power_t,P&gt;
&gt;(value / (MagnitudeCalculateFrom(unit) * unit.getValue()),tmp_dimens);
```

#### 除法运算符

使用并抛出（如有）除法函数中的错误

## TODO

- [ ] 添加错误分类
- [ ] 重载`std::formatter`
- [ ] 四则运算的运算至函数与运算符（e.g. `operator&#43;=`与`plusto`）
- [ ] 从字符串中转化（估计要等到C&#43;&#43;26了，静态反射）

参考资料：  
[1] [addis.物理量和单位转换[EB/OL].小时百科.https://wuli.wiki/online/Units.html](https://wuli.wiki/online/Units.html)


---

> 作者: Bendancom  
> URL: https://bendancom.github.io/posts/%E5%8D%95%E4%BD%8D%E5%BA%93%E5%AE%9E%E7%8E%B0/  

