Kotlin 记录

在 Kotlin 中，灵活运用集合操作符可以大幅减少 `for` 循环和 `if-else` 的嵌套。为了方便记忆与复习，我们将最常用的操作符分为三大类：**判断与查找**、**过滤与转换**、**统计与分组**。

## ① 判断与查找类（返回 Boolean 或单个元素）

这类操作符的核心特点是：**用于验证集合状态**或**精准提取某一个元素**。通常具有“短路”特性（一旦找到结果，立即停止遍历）。

|**操作符**|**核心含义**|**逻辑特性 / 备注**|**代码举例（业务场景）**|
|---|---|---|---|
|**`all`**|**全部满足**|只有集合里**所有**元素都满足条件，才返回 `true`。|`list.all { it.age > 18 }`<br><br>  <br><br>_(检查：是否全部成年)_|
|**`none`**|**全不满足**|只有集合里**没有一个**元素满足条件，才返回 `true`。|`list.none { it.isExpired }`<br><br>  <br><br>_(检查：是否全都没有过期)_|
|**`find`**|**找到第一个**|返回满足条件的**第一个**元素，找不到就返回 `null`。|`list.find { it.id == 102 }`<br><br>  <br><br>_(查找：ID为102的商品)_|
|**`firstOrNull`**|**找到第一个**|功能和 `find` **完全一样**，但语义更清晰，Kotlin 官方更推荐。|`list.firstOrNull { it.id == 102 }`|

### 💡 深度辨析：`any` vs `all` vs `none`

- `any { ... }`：只要有 **1个** 满足就支持。

- `all { ... }`：必须 **100%** 满足才行。

- `none { ... }`：必须 **0%** 满足（全部不满足）才行。


## ② 过滤与转换类（返回新集合）

这类操作符的特点是：**不会修改原集合**，而是处理后**生成并返回一个全新的集合**。

|**操作符**|**核心含义**|**逻辑特性 / 备注**|**代码举例（业务场景）**|
|---|---|---|---|
|**`filter`**|**过滤（保留）**|保留所有**满足**条件的元素，组合成新列表。|`list.filter { it.price > 100 }`<br><br>  <br><br>_(筛选：价格大于100的高价商品)_|
|**`filterNot`**|**反向过滤（剔除）**|**剔除**满足条件的元素，保留剩下的元素。|`list.filterNot { it.isExpired }`<br><br>  <br><br>_(清洗：剔除所有过期的优惠券)_|
|**`map`**|**转换 / 映射**|把集合里的每条数据，**转换成另一种形式**。|`userList.map { it.name }`<br><br>  <br><br>_(提取：把用户对象列表变成名字字符串列表)_|
|**`mapNotNull`**|**转换并去空**|转换的同时，**自动把返回 `null` 的结果剔除掉**，避免新集合中含有 `null`。|`list.mapNotNull { it.toDetailOrNull() }`<br><br>  <br><br>_(安全转换：只保留转换成功的数据)_|

## ③ 统计与分组类（聚合操作）

这类操作符用于对集合进行**整体维度的归纳、数量统计或分门别类**。

|**操作符**|**核心含义**|**逻辑特性 / 备注**|**代码举例（业务场景）**|
|---|---|---|---|
|**`count`**|**计数**|统计满足条件的元素一共有多少个。|`list.count { it.status == 1 }`<br><br>  <br><br>_(统计：当前在线的用户总数)_|
|**`groupBy`**|**分组**|按照某个特征把集合拆分成一个 `Map<Key, List>`。|`list.groupBy { it.awardType }`<br><br>  <br><br>_(分类：按奖励类型将红包分组)_|

## 进阶避坑与性能优化指南（面试高频）

### 1. 别用 `filter { ... }.size`，请用 `count { ... }`

- ❌ `list.filter { it.age > 18 }.size`

    - **缺点**：`filter` 会在内存中**创建一个临时的新列表**，把满足条件的人放进去，最后再数个数。浪费内存。

- `list.count { it.age > 18 }`

    - **优点**：直接在底层内部通过计数器累加，**不创建新集合**，性能大幅提升。


### 2. 别用 `map { ... }.filterNotNull()`，请用 `mapNotNull { ... }`

- ❌ `list.map { it.toDetailOrNull() }.filterNotNull()`

    - **缺点**：进行了两轮遍历，第一轮生成带 `null` 的列表，第二轮再去空。

- `list.mapNotNull { it.toDetailOrNull() }`

    - **优点**：一轮遍历同时搞定转换和去空，代码更少，效率更高。



# 协程



# Flow

## 操作符

### map

### onEach

### flowOn



### onStart

### catch
### onCompletion

### collect

### conflate

### collect



### transform

### take

### reduce

### fold



### zip



### combine



### flattenMerge



### flatMapConcat



### flatMapMerge



### flatMapLatest
