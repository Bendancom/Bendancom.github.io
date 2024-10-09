# 多索引泛型容器类的实现


`C&#43;&#43;`中实现了单映射`STL`容器`std::map`，但实际业务中需要的是多映射类型的对任意数据的快速查找。虽可以使用多个`std::map`，但这对空间浪费过大，因而作者自行实现一个简易的多索引泛型容器库。
&lt;!--more--&gt;

## 实现思路

使用关联容器或自定义关联容器存储索引
使用容器存储数据表，容器类型可根据`IO`特性自行调整

* 键：数据的引用，以减少重复数据。
* 值：数据在总表中的位置(指针)。

容器是不能存储引用的，使用`std::reference_wrapper`来包装引用并存储
使用 `std::tuple` 封装数据。

## 标准库使用

* `&lt;type_traits&gt;`
* `&lt;ranges&gt;`
* `&lt;tuple&gt;`
* `&lt;vector&gt;`
* `&lt;concepts&gt;`

## 泛型

### 索引类的键值对类型

#### 键

`std::reference_wrapper` 的本质是指针，即在内存中占用`sizeof(size_t)`大小字节，由此可进行分类：

* $\geqslant$`sizeof(size_t)`的使用 `std::reference_wrapper`
* $&lt;$`sizeof(size_t)`的使用其本身

那么类型别名模板如下：

```C&#43;&#43;
template &lt;typename T&gt;
using map_key = std::conditional_t&lt;sizeof(T) &lt; sizeof(size_t), T, std::reference_wrapper&lt;T&gt;&gt;;
```

#### 值

* 泛型容器，其索引值为该泛型容器的迭代器
* 随机访问容器，其索引值可优化为内存地址

为泛型选择了第一种。

```C&#43;&#43;
template &lt;template &lt;typename T, typename... Args&gt; typename container, typename... Args&gt;
using map_value = std::ranges::iterator_t&lt;container&lt;std::tuple&lt;Args...&gt;&gt;&gt;;
```

### 索引容器包装/解封

#### `map_wrapper`

对单个索引容器的包装

##### `map_wrapper`封装

```C&#43;&#43;
template &lt;template &lt;typename, typename, typename... Args&gt; typename T&gt;
struct map_wrapper {};
```

##### `map_wrapper_expand`

运用模板特化读取封装的单个索引容器类型，后接两个类型：

* `K`: 键类型
* `V`: 值类型

直接组成一个完好的索引类。

```C&#43;&#43;
template &lt;typename T, typename K, typename V&gt;
struct map_wrapper_expand;
template &lt;
    template &lt;template &lt;typename, typename, typename... Args&gt; typename args&gt; typename T,
    template &lt;typename, typename, typename... Args&gt; typename Other,
    typename K,
    typename V&gt;
struct map_wrapper_expand&lt;T&lt;Other&gt;, K, V&gt;
{
    using type = Other&lt;K, V&gt;;
};
```

#### `maps_wrapper`

对不同索引的整体封装

##### `maps_wrapper` 封装

```C&#43;&#43;
export template &lt;template &lt;typename, typename, typename... Args&gt; typename... Others&gt;
struct maps_wrapper {};
```

##### `maps_wrapper_expand`

通过对 `maps_wrapper`的展开，对其中的每个索引容器单独使用 `map_wrapper` 封装，后组合成 `std::tuple` 的二层封装结构

```C&#43;&#43;
template&lt;typename T&gt;
struct maps_wrapper_expand;
template&lt;template&lt;template&lt;typename, typename, typename... Args&gt;typename... args&gt;typename T, template&lt;typename, typename, typename... Args&gt;typename...Others&gt;
struct maps_wrapper_expand&lt;T&lt;Others...&gt;&gt; {
    using type = std::tuple&lt;map_wrapper&lt;Others&gt;...&gt;;
};
```

##### 踩坑

```C&#43;&#43;
template&lt;typename T&gt;
struct maps_wrapper_expand;
template&lt;template&lt;template&lt;typename, typename, typename... Args&gt;typename... args&gt;typename T, template&lt;typename, typename, typename... Args&gt;typename...Others&gt;
struct maps_wrapper_expand&lt;T&lt;Others...&gt;&gt; {
    template&lt;typename V,typename... K&gt;
    using type = std::tuple&lt;Others&lt;K,V&gt;...&gt;;
};
```

这个方法看起来可以直接一步到位，只要如下使用即可：

```C&#43;&#43;
maps_wrapper_expand&lt;
    maps_wrapper&lt;
        std::map,
        std::map&gt;&gt;
::type&lt;
        long double,
        int,
        int
&gt;   maps; 
```

以上确实能通过编译。

但在引入模板后，不能通过编译。
模板如下：

```C&#43;&#43;
template&lt;typename Maps,typename V,typename... K&gt;
using maps_tuple_t =maps_wrapper_expand&lt;Maps&gt;::type&lt;V,K...&gt;; 
```

探查下来问题大概出在可变参数模板类内的类型别名模板定义，不能在可变参数模板类中嵌套类型别名模板。

#### `maps_tuple_before`

原理是将两种可变参数分别按顺序封装到 `std::tuple` 中，再利用 `size_t... N` 统一进行展开并拼装为完整的索引类，最后封装至`std::tuple`中。
使用`map_wrapper_expand`来进行拼装

```C&#43;&#43;
template &lt;typename T, typename K, typename V, typename Index&gt;
struct maps_tuple_before;
template &lt;typename T, typename K, typename V, size_t... N&gt;
struct maps_tuple_before&lt;T, K, V, std::index_sequence&lt;N...&gt;&gt;
{
    using type = std::tuple&lt;typename map_wrapper_expand&lt;
        std::tuple_element_t&lt;N, typename maps_wrapper_expand&lt;T&gt;::type&gt;,
        std::tuple_element_t&lt;N, K&gt;,
        V&gt;::type...&gt;;
};
```

#### `maps_tuple`

这就是最终的对索引容器处理的类，是对`maps_tuple_before`进行一个模板简化。
在其上增添了自动生成 `std::index_sequence&lt;N...&gt;`

```C&#43;&#43;
template &lt;
    template &lt;typename T, typename... container_args&gt; typename container,
    typename associative_containers,
    typename... args&gt;
struct maps_tuple
{
    using type = maps_tuple_before&lt;
        associative_containers,
        std::tuple&lt;map_key&lt;args&gt;...&gt;,
        map_value&lt;container, args...&gt;,
        std::make_index_sequence&lt;sizeof...(args)&gt;&gt;::type;
};
```

### `multi_index`

使用约束与模板进行泛型匹配
应有如下类型

* 数据容器
* 索引容器包装，其中含有各数据所需索引容器类型
* 数据

#### 模板

``` C&#43;&#43;
template &lt;
    template &lt;typename T, typename... container_args&gt; typename container,
    typename associative_containers,
    typename... args&gt;
requires requires (......)
{
    ......
}
class multi_index{
    ......
};
```

#### 约束

由于泛型的需求，其`requires`要求应为最普适，即：

* 数据表容器应为 `common_range`&amp;`sized_range`类型
* 数据量应与索引容器量一致
* 索引表容器应为 `common_range`类型，并可被`std::get`函数解包，且有如下函数：
  * `find`
  * `insert`
  * `erase`
  * `count`

而索引总类是由 `std::tuple`进行包装的，因而须函数展开进行约束计算。

```C&#43;&#43;
requires requires(
    maps_tuple_t&lt;container, associative_containers, args...&gt; maps,
    map_key&lt;args&gt;... keys,
    map_value&lt;container, args...&gt; value) {
    std::ranges::common_range&lt;container&lt;std::tuple&lt;args...&gt;&gt;&gt;;
    std::ranges::sized_range&lt;container&lt;std::tuple&lt;args...&gt;&gt;&gt;;
    sizeof...(args) == std::tuple_size_v&lt;decltype(maps)&gt;;
    // common_range
    [&amp;maps]&lt;size_t... N&gt;(std::index_sequence&lt;N...&gt;) -&gt; bool
    {
        return (... &amp;&amp; std::ranges::common_range&lt;std::tuple_element_t&lt;N, decltype(maps)&gt;&gt;);
    }(std::make_index_sequence&lt;sizeof...(args)&gt;{});
    // find
    [&amp;maps]&lt;size_t... N&gt;(std::index_sequence&lt;N...&gt;, auto &amp;&amp;...keys) -&gt; bool
    {
        return (... &amp;&amp; std::same_as&lt;
                           decltype(std::ranges::begin(std::get&lt;N&gt;(maps))),
                           decltype(std::get&lt;N&gt;(maps).find(keys))&gt;);
    }(std::make_index_sequence&lt;sizeof...(args)&gt;{}, keys...);
    // count
    [&amp;maps]&lt;size_t... N&gt;(std::index_sequence&lt;N...&gt;, auto &amp;&amp;...keys) -&gt; bool
    {
        return (... &amp;&amp; std::same_as&lt;decltype(std::ranges::begin(std::get&lt;N&gt;(maps))),
                                    decltype(std::get&lt;N&gt;(maps).count(keys))&gt;);
    }(std::make_index_sequence&lt;sizeof...(args)&gt;{}, keys...);
    // erase
    [&amp;maps]&lt;size_t... N&gt;(std::index_sequence&lt;N...&gt;) -&gt; bool
    {
        return (... &amp;&amp; std::same_as&lt;
                           decltype(std::ranges::begin(std::get&lt;N&gt;(maps))),
                           decltype(std::get&lt;N&gt;(maps).erase(std::ranges::begin(std::get&lt;N&gt;(maps))))&gt;);
    }(std::make_index_sequence&lt;sizeof...(args)&gt;{});
    // insert
    [&amp;maps, &amp;value]&lt;size_t... N&gt;(std::index_sequence&lt;N...&gt;, auto &amp;&amp;...keys) -&gt; bool
    {
        return (... &amp;&amp; std::same_as&lt;
                           decltype(std::ranges::begin(std::get&lt;N&gt;(maps))),
                           decltype(std::get&lt;N&gt;(maps).insert({keys, value}))&gt;);
    }(std::make_index_sequence&lt;sizeof...(args)&gt;{}, keys...);
    // can decompress by function std::get&lt;0&gt;
    [&amp;maps]&lt;size_t... N&gt;(std::index_sequence&lt;N...&gt;, auto &amp;&amp;...keys) -&gt; bool
    {
        return (... &amp;&amp; std::same_as&lt;
                           decltype(keys),
                           decltype(std::get&lt;0&gt;(*std::ranges::begin(std::get&lt;N&gt;(maps))))&gt;);
    }(std::make_index_sequence&lt;sizeof...(args)&gt;{}, keys...);
    // can decompress by function std::get&lt;1&gt;
    [&amp;maps, &amp;value]&lt;size_t... N&gt;(std::index_sequence&lt;N...&gt;) -&gt; bool
    {
        return (... &amp;&amp; std::same_as&lt;
                           decltype(value),
                           decltype(std::get&lt;1&gt;(*std::ranges::begin(std::get&lt;N&gt;(maps))))&gt;);
    }(std::make_index_sequence&lt;sizeof...(args)&gt;{});
}
```

## 函数实现

### 辅助函数

模板约束中并未对索引容器要求`clear`与`swap`函数，因而需`requires`检验

```C&#43;&#43;
template &lt;size_t... N&gt;
constexpr void make_indexs(iterator data, std::index_sequence&lt;N...&gt;)
{
    (std::get&lt;N&gt;(map).insert({std::get&lt;N&gt;(*data), data}), ...);
}
template &lt;size_t... N&gt;
constexpr void erase_indexs(const iterator &amp;iter, std::index_sequence&lt;N...&gt;)
{
    (std::get&lt;N&gt;(map).erase(std::get&lt;N&gt;(map).find(std::get&lt;N&gt;(*iter))), ...);
}
template &lt;size_t... N&gt;
constexpr void clear_indexs(std::index_sequence&lt;N...&gt;)
    requires requires(multi_index_t th) { (std::get&lt;N&gt;(th).clear(), ...); }
{
    (std::get&lt;N&gt;(map).clear, ...);
}
template &lt;size_t... N&gt;
constexpr void swap_indexs(multi_index_t &amp;other, std::index_sequence&lt;N...&gt;)
    requires requires(multi_index_t th, multi_index_t other) { (std::get&lt;N&gt;(th).swap(std::get&lt;N&gt;(other)), ...); }
{
    (std::get&lt;N&gt;(map).swap(std::get&lt;N&gt;(other)), ...);
}
```

### 初始化

#### `default`

```C&#43;&#43;
constexpr multi_index() {}
```

#### `Range`

```C&#43;&#43;
template &lt;std::ranges::input_range Rng&gt;
    requires requires { std::convertible_to&lt;std::ranges::range_reference_t&lt;Rng&gt;, data_list_t&gt;; }
constexpr multi_index(Rng &amp;&amp;t) : data_sheet(std::ranges::to&lt;data_sheet_t&gt;(t))
{
    for (iterator iter = data_sheet.begin(); iter != data_sheet.end(); iter&#43;&#43;)
        make_indexs(iter, std::make_index_sequence&lt;sizeof...(Args)&gt;{});
}
```

#### `copy`/`move`

```C&#43;&#43;
constexpr multi_index(const multi_index&lt;container, associative_container, Args...&gt; &amp;d) : 
    data_sheet(d.data_sheet), map(d.map) {}
```

### 迭代

#### `begin`/`cbegin`

```C&#43;&#43;
constexpr iterator begin()
{
    return std::ranges::begin(data_sheet);
}
constexpr const_iterator cbegin() const
{
    return std::ranges::cbegin(data_sheet);
}
```

#### `end`/`cend`

```C&#43;&#43;
constexpr iterator end()
{
    return std::ranges::end(data_sheet);
}
constexpr const_iterator cend() const
{
    return std::ranges::cend(data_sheet);
}
```

#### `rbegin`/`crbegin`

```C&#43;&#43;
constexpr reverse_iterator rbegin()
    requires requires(data_sheet_t data) { data.rbegin(); }
{
    return data_sheet.rbegin();
}
constexpr const const_reverse_iterator crbegin() const
    requires requires(data_sheet_t data) { data.crbegin(); }
{
    return data_sheet.crbegin();
}
```

#### `rend`/`crend`

```C&#43;&#43;
constexpr reverse_iterator rend()
    requires requires(data_sheet_t data) { data.rend(); }
{
    return data_sheet.rend();
}
constexpr const const_reverse_iterator crend() const
    requires requires(data_sheet_t data) { data.crend(); }
{
    return data_sheet.crend();
}
```

### 查询

#### `size`

```C&#43;&#43;
constexpr size_t size() const
{
    return data_sheet.size();
}
```

#### `empty`

```C&#43;&#43;
constexpr bool empty() const
    requires requires(data_sheet_t data) { data.empty(); }
{
    return data_sheet.empty();
}
```

### 修改

#### `insert`/`insert_range`

```C&#43;&#43;
constexpr iterator insert(const iterator &amp;iter, const data_list_t &amp;data)
{
    iterator tmp = data_sheet.insert(iter, data);
    make_indexs(tmp, std::make_index_sequence&lt;sizeof...(Args)&gt;{});
    return tmp;
}
template&lt;std::input_range Rng&gt;
    requires requires { std::convertible_to&lt;std::ranges::range_reference_t&lt;Rng&gt;, data_list_t&gt;; }
constexpr iterator insert_range(const iterator &amp;iter, Rng &amp;&amp;data)
{
    const size_t size = data.size();
    iterator tmp = data_sheet.insert_range(iter, data);
    iterator i = tmp;
    for (size_t j = 0; j &lt; size; j&#43;&#43;)
    {
        make_indexs(i, std::make_index_sequence&lt;sizeof...(Args)&gt;{});
        i&#43;&#43;;
    }
    return tmp;
}
```

#### `push_back`/`append_range`

```C&#43;&#43;
constexpr void push_back(const data_list_t &amp;data)
{
    data_sheet.push_back(data);
    make_indexs(std::prev(std::ranges::end(data_sheet)), std::make_index_sequence&lt;sizeof...(Args)&gt;{});
    return;
}
template&lt;std::input_range Rng&gt;
    requires requires { std::convertible_to&lt;std::ranges::range_reference_t&lt;Rng&gt;, data_list_t&gt;; }
constexpr void append_range(Rng &amp;&amp;data)
{
    iterator tmp = std::prev(data.end());
    data_sheet.append_range(data);
    for (tmp&#43;&#43;; tmp != data.end(); tmp&#43;&#43;)
        make_indexs(tmp, std::make_index_sequence&lt;sizeof...(Args)&gt;{});
}
constexpr void clear()
    requires requires(data_sheet_t data, maps_tuple_t maps) { data.clear(); maps.clear(); }
{
    data_sheet.clear();
    clear_indexs(std::make_index_sequence&lt;sizeof...(args)&gt;{});
}
```

#### `erase`/`pop_back`/`clear`/`pop_front`

```C&#43;&#43;
constexpr iterator erase(const iterator &amp;iter)
    requires requires(data_sheet_t data, iterator iter) { data.erase(iter); }
{
    erase_indexs(iter, std::make_index_sequence&lt;sizeof...(args)&gt;{});
    return data_sheet.erase(iter);
}
constexpr void pop_back()
    requires requires(data_sheet_t data) { data.pop_back(); }
{
    erase_indexs(data_sheet.end()--, std::make_index_sequence&lt;sizeof...(args)&gt;{});
    data_sheet.pop_back();
}
constexpr void clear()
    requires requires(data_sheet_t data, maps_tuple_t maps) { data.clear(); maps.clear(); }
{
    data_sheet.clear();
    clear_indexs(std::make_index_sequence&lt;sizeof...(args)&gt;{});
}
constexpr void pop_front()
    requires requires(data_sheet_t data) { data.pop_front(); }
{
    erase_indexs(data_sheet.begin(), std::make_index_sequence&lt;sizeof...(args)&gt;{});
    data_sheet.pop_front();
}

```

#### `swap`/`resize`

```C&#43;&#43;
constexpr void resize(const size_t &amp;size)
    requires requires(data_sheet_t data, size_t size) { data.resize(size); }
{
    data_sheet.resize(size);
}
constexpr void swap(multi_index&lt;container, associative_containers, args...&gt; &amp;other)
    requires requires(data_sheet_t data, data_sheet_t &amp;other) { data.swap(other); }
{
    data_sheet.swap(other.data_sheet);
    swap_indexs(other.map, std::make_index_sequence&lt;sizeof...(args)&gt;{});
}
```

### 查找

#### `find`

```C&#43;&#43;
template &lt;size_t N&gt;
constexpr std::vector&lt;iterator&gt; find(const std::tuple_element_t&lt;N, data_list_t&gt; &amp;data)
{
    std::vector&lt;iterator&gt; tmp;
    auto iter = std::get&lt;N&gt;(map).find(data);
    if (iter != std::get&lt;N&gt;(map).end())
        for (size_t i = 0; i &lt; std::get&lt;N&gt;(map).count(std::get&lt;0&gt;(*iter)); i&#43;&#43;)
            tmp.push_back(std::get&lt;1&gt;(*&#43;&#43;iter));
    return tmp;
}
```

## 总代码

位于`github` 仓库中 [https://github.com/Bendancom/BlogCodes](https://github.com/Bendancom/BlogCodes)

## TODO

- [ ] `map_wrapper`在单种容器下的自动展开
- [ ] `std::formatter`重载
- [ ] 更简单的初始化，如`std::make_tuple`

## 参考资料  
\[1]:[泡沫o0.【C&#43;&#43; 包装器类 std::reference_wrapper 】全面指南：深入理解与应用C&#43;&#43; std::reference_wrapper——从基础教程到实际案例分析[EB/OL].CSDN.2023/09/26.https://blog.csdn.net/qq_21438461/article/details/131297103](https://blog.csdn.net/qq_21438461/article/details/131297103)  
\[2]:[下学期上五年级.Modern C&#43;&#43; 学习笔记（20）——高级模板[EB/OL].知乎.2023/06/10.https://zhuanlan.zhihu.com/p/636178655](https://zhuanlan.zhihu.com/p/636178655)


---

> 作者: Bendancom  
> URL: https://bendancom.github.io/posts/%E5%A4%9A%E7%B4%A2%E5%BC%95%E6%B3%9B%E5%9E%8B%E5%AE%B9%E5%99%A8%E7%B1%BB%E7%9A%84%E5%AE%9E%E7%8E%B0/  

