**HashMap**

# 基础知识讲解

基本属性

```java
//默认初始容量为16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
// hashmap 最大容量值
static final int MAXIMUM_CAPACITY = 1 << 30;
//默认负载因子为0.75
static final float DEFAULT_LOAD_FACTOR = 0.75f;
// 存储的负载因子
final float loadFactor;
// 链表长度大于该参数转红黑树
static final int TREEIFY_THRESHOLD = 8;
//当树的节点数小于等于该参数退化为链表
static final int UNTREEIFY_THRESHOLD = 6; 
//Hash数组(在resize()中初始化) ,总是 2 的倍数
transient Node<K,V>[] table;
//表示当前HashMap包含的键值对数量
transient int size;
//表示当前HashMap修改次数
transient int modCount;
//容量阈值(数组 table 中元素填充的数量，一旦超过这个数量 HashMap 就会进行扩容)
int threshold;
//负载因子，用于扩容
final float loadFactor; 
```

总结

- 默认初始容量为`16`，默认负载因子为`0.75`
- `threshold = 数组长度 * loadFactor`，当元素个数大于等于`threshold(容量阈值)`时，HashMap会进行扩容操作
- table数组中存放指向链表的引用

这里需要注意的一点是**table数组并不是在构造方法里面初始化的，它是在resize(扩容)方法里进行初始化的**。

这里说句题外话：可能有刁钻的面试官会问**为什么默认初始容量要设置为16？为什么负载因子要设置为0.75？**我们都知道HashMap数组长度被设计成2的幂次方(下面会讲)，那为什么初始容量不设计成4、8或者32....其实这是JDK设计者经过权衡之后得出的一个比较合理的数字，，如果默认容量是8的话，当添加到第6个元素的时候就会触发扩容操作，扩容操作是非常消耗CPU的，32的话如果只添加少量元素则会浪费内存，因此设计成16是比较合适的，负载因子也是同理。

**java 与或亦或运算符  |  &  ^ **

```java
a |= b 等价于 a = a|b
a &= b 等价于 a = a&b
a ^= b 等价于 a = a^b
|=：两个二进制对应位都为0时，结果等于0，否则结果等于1。两个位只要有一个为1，那么结果就是1，否则就为0
&=：两个二进制的对应位都为1时，结果为1，否则结果等于0。两个操作数中位都为1，结果才为1，否则结果为0
^=：两个二进制的对应位相同，结果为0，否则结果为1。两个操作数的位中，相同则结果为0，不同则结果为1
~ 如果位为0，结果是1，如果位为1，结果是0
```

**Java 中的位移运算符 >> , <<  , >>>**

对位移移动后不是特别熟悉，以int e = 12345 为例进行解析

![](https://gitee.com/ZeTing/UploadImg/raw/main/img/c1.png)



12345二进制表达.png

**1、左移运算符 <<**

如果 e << 1 ,左位移1位：

![](https://gitee.com/ZeTing/UploadImg/raw/main/img/c2.png)

左位移1位1.png

位移后十进制数值变成：24690，刚好是12345的二倍，所以有些人会用左位移运算符代替乘2的操作，但是这并不代表是真的就是乘以2，很多时候，我们可以这样使用，但是一定要知道，位移运算符很多时候可以代替乘2操作，但是这个并不代表两者是一样的(这一点需要格外注意，很多人都存在这样的误解)，接着往下看：

这里要注意了，左位移18位后，二进制首位为1，如下图所示：

![](https://gitee.com/ZeTing/UploadImg/raw/main/img/c4.png)

左位移18位.png

此时二进制表达首位为1，此时数值为 -1058799616，同理，如果继续位移，左位移20位，则值为 59768832 又变成了正数

所以根据这个规则，如果任意一个十进制的数左位移32位，右边补位32个0，十进制岂不是都是0了？当然不是！！！ 当int 类型的数据进行左移的时候，当左移的位数大于等于32位的时候，位数会先求余数，然后再进行左移，也就是说，如果真的左移32位 e << 32 的时候，会先进行位数求余数，即为 e<<(32%32) 相当于 e<< 0 ，所以e<< 33 的值和e<<1 是一样的，都是 24690

## 2、右移运算符 >>

同样，还是以12345这个数值为例，
 e右移1位： e>>1

![](https://gitee.com/ZeTing/UploadImg/raw/main/img/c5.png)

12345右移1位.png



右移后得到的值为 6172 和int 类型的数据12345除以2取整所得的值一样，所以有些时候也会被用来替代除2操作

但是如果继续右移，直接右移14位，即为e>>14，则结果为0 ，其过程和左移相似，就不一一演示了；另外，对于超过32位的位移，和左移运算符一样，，会先进行位数求余数

## 3、无符号右移运算符：>>>

无符号右移运算符和右移运算符是一样的，不过无符号右移运算符在右移的时候是补0的，而右移运算符是补符号位的

一下是 -12345 二进制表达式

![](https://gitee.com/ZeTing/UploadImg/raw/main/img/c6.png)

-12345二进制表达.png

对于源码、反码、补码不熟悉的同学，请自行学习，这里就不再进行补充了讲解了，这里提醒一下，在右移运算符中，右移后补0，是由于正数 12345 符号位为0 ，如果为1 则应补1

![](https://gitee.com/ZeTing/UploadImg/raw/main/img/c7.png)

无符号右移运算符.png

## 最后补充一下：

###### 1、原码、反码和补码

一个数可以分成符号位（0正1负）+ 真值，原码是我们正常想法写出来的二进制。由于计算机只能做加法，负数用单纯的二进制原码书写会出错，于是大家发明了反码（正数不变，负数符号位不变，真值部分取反）；再后来由于+0， -0的争端，于是改进反码，变成补码（正数不变，负数符号位不变，真值部分取反，然后+1）。二进制前面的0都可以省略，所以总结来说：计算机里的负数都是用补码（符号位1，真值部分取反+1）表示的。

###### 2、位运算符和2的关系

位运算符和乘2 除2 在大多数时候是很相似的，可以进行替代，同时效率也会高的多，但是两者切记不能混淆 ；
 很多时候有人会把两者的概念混淆，尤其是数据刚好是 2、4、6、8、100等偶数的时候，看起来就更相似了，但是对于奇数，如本文使用的12345 ，右移之后结果为6172 ，这个结果就和数学意义上的除以2不同了，不过对于int 类型的数据，除2 会对结果进行取整，所以结果也是6172 ，这就更有迷惑性了

 

![image-20201119134243625](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20201119134243625.png)



**hash 算法原理**

```java
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

***<u>^=：两个二进制的对应位相同，结果为0，否则结果为1。两个操作数的位中，相同则结果为0，不同则结果为1</u>***

实际上就是把二进制数像右移了 16 位，然后进行异或计算，可以使高位也参加地位运算

```java
处理数据 key             ：zzzzzz                                     
hashCode值              ：1101 0111 0001 1111 1011 1101 1100 0000 
(h >>> 16)位移值         ：0000 0000 0000 0000 1101 0111 0001 1111  
(h ^ (h >>> 16)) 位移值  ：1101 0111 0001 1111 0110 1010 1101 1111  
```

**tab[(n - 1) & hash]   中的 (n - 1) & hash 原理**

<u>***&=：两个二进制的对应位都为1时，结果为1，否则结果等于0。两个操作数中位都为1，结果才为1，否则结果为0***</u>

```java
Object kk = "z";
int kkHash = hash(kk);
kk hashCode值                     ：0000 0000 0000 0000 0000 0000 0111 1010 
128  hashCode值                   ：0000 0000 0000 0000 0000 0000 1000 0000 
n-1 hashCode值                    ：0000 0000 0000 0000 0000 0000 0111 1111 
(n - 1) & kkHash)值               ：122
(n - 1) & kkHash)二进制值          ：0000 0000 0000 0000 0000 0000 0111 1010 

(16 - 1) & kkHash) hashCode值    ：10      
(32 - 1) & kkHash) hashCode值    ：26     相当于10+16
(64 - 1) & kkHash) hashCode值    ：58     相当于26+32
(128 - 1) & kkHash) hashCode值   ：122    相当于58+64
```

**为什么扩容因子为0.75**

1.负载因子是0.75的时候，空间利用率比较高，而且避免了相当多的Hash冲突，使得底层的链表或者是红黑树的高度比较低，提升了空间效率。

2.0.75 是一个比较有意思的数字，下面是我的一些想法

```java
final Node<K,V>[] resize() {
    ......
if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
         oldCap >= DEFAULT_INITIAL_CAPACITY)
    newThr = oldThr << 1; // double threshold
    ......
}
```

在扩容代码中扩大桶的容量用的是左移1的方式

那么那么在最大容量阀值的地方也要保存这么快速计算，也可以使用左移的方式来计算

如果想左移计算那么扩容的最大值应该是整数，如果计算出来是double 类型，这样位移计算会丢失进度，

容量都是2的倍数，在1到0里面能乘以2并且是整数的只有0.5 ，如果容量是4，有3个0.25，0.5，0.75

如果容量是8 ，存在0.125 ， 0.25，0.375 ，0.5 ，0.625，0.75，0.875 以此类推

这样计算，比如容量16来计算 因子0.75计算，通过位移可以使每一次容量和阀值都乘以2后依旧保持因子为0.75

16 ：12

32：24

**扩容计算方式tableSizeFor**

```java
static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

扩容实际操作，就是不停的右移，直到每一位到时1的时候，在n+1 的到2的n次幂的某一位数

![扩容的计算方法](https://gitee.com/ZeTing/UploadImg/raw/main/img/20201124175433.png)

```java
扩容机制   ：1111 =二进制：10001010111
扩容机制   ：1110 =二进制：10001010110 = n >>> 1 ：1000101011
扩容机制 1 ：1663 =二进制：11001111111 = n >>> 2 ：110011111
扩容机制 2 ：2047 =二进制：11111111111 = n >>> 4 ：1111111
扩容机制 4 ：2047 =二进制：11111111111 = n >>> 8 ：111111
扩容机制 8 ：2047 =二进制：11111111111 = n >>> 16：0
扩容机制 16：2047 =二进制：11111111111
扩容机制 最后结果：2048 二进制输出：100000000000
扩容机制 结果 ：2048
```

# put方法

```java
public V put(K key, V value) {
    //key 值进行 hash 计算
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0) // 如果 table 为空或长度为 0 时
        n = (tab = resize()).length; //对 HashMap 进行扩容
    if ((p = tab[i = (n - 1) & hash]) == null) //索引数组对应位置元素为空
        tab[i] = newNode(hash, key, value, null); //创建新节点插入数组中
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            // table 中的键值对 key 和要存入的键值对 key 一致
            e = p;
        else if (p instanceof TreeNode)
            //table 中的节点为红黑树
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //table 中为链表
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    //遍历到表尾也没有找到 key 值相同节点，插入新节点
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        //长度超过树化阈值转化为红黑树
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 链表中包含相同 key 的元素
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            // map 中存在相同 key 元素
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                //替换元素的 value 值
                e.value = value;
            afterNodeAccess(e);
            //返回旧的 value
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        //数组元素长度超过临界值，进行扩容
        resize();
    afterNodeInsertion(evict);
    return null;
}
```
总结来说就是几步：

1. **判断数组是否初始化，未初始化则进行扩容操作。**
2. **hash 算法计算节点在数组中的索引位，若数组元素为空则新建节点插入元素到数组中。**
3. **判断数组元素和插入元素 key 值是否相同，相同则替换 value 值。**
4. **数组中的节点是否为红黑树结构，若为红黑树则进行红黑树插入操作。**
5. **遍历节点链表，查找到相同 key 值节点则替换 value 值，若不存在则新建节点插入链表。**
6. **链表长度是否大于树化阈值，大于则转换链表为红黑树；数组元素长度是否超过临界值，超过则进行扩容。**



 红黑树的插入操作：

```java
final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                               int h, K k, V v) {
    Class<?> kc = null;
    // 是否被查找过
    boolean searched = false;
    //查找根节点
    TreeNode<K,V> root = (parent != null) ? root() : this;
    for (TreeNode<K,V> p = root;;) {//遍历红黑树
        int dir, ph; K pk;
        if ((ph = p.hash) > h) //当前节点 hash 值大于要插入的节点 hash 值
            dir = -1; //往左孩子查找
        else if (ph < h) //当前节点 hash 值小于要插入的节点 hash 值
            dir = 1;//往右孩子查找
        else if ((pk = p.key) == k || (k != null && k.equals(pk))) //查找到相同 key 值得节点
            return p;
        else if ((kc == null &&
                (kc = comparableClassFor(k)) == null) ||
                (dir = compareComparables(kc, k, pk)) == 0) { //节点元素未实现 compare 方法
            if (!searched) {
                TreeNode<K,V> q, ch;
                searched = true;
                if (((ch = p.left) != null &&
                        (q = ch.find(h, k, kc)) != null) || //遍历左孩子
                        ((ch = p.right) != null &&
                                (q = ch.find(h, k, kc)) != null)) //遍历右孩子
                    //查询到相同 key 值节点
                    return q;
            }
            //节点元素实现了 compare 方法，计算遍历方向
            dir = tieBreakOrder(k, pk);
        }

        TreeNode<K,V> xp = p;
        if ((p = (dir <= 0) ? p.left : p.right) == null) { // 遍历到末尾节点还未查找到相同元素
            Node<K,V> xpn = xp.next;
            //创建新节点
            TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
            if (dir <= 0) //插入左孩子
                xp.left = x;
            else //插入右孩子
                xp.right = x;
            //把节点的指针连接起来
            xp.next = x;
            x.parent = x.prev = xp;
            if (xpn != null)
                ((TreeNode<K,V>)xpn).prev = x;
            //balanceInsertion 插入操作平衡红黑树，并把根节点和原来数组中的元素进行交换位置
            moveRootToFront(tab, balanceInsertion(root, x));
            return null;
        }
    }
}
//移动根节点到前面数组中
static <K,V> void moveRootToFront(Node<K,V>[] tab, TreeNode<K,V> root) {
    int n;
    if (root != null && tab != null && (n = tab.length) > 0) {
        int index = (n - 1) & root.hash; //计算根节点位置
        TreeNode<K,V> first = (TreeNode<K,V>)tab[index]; //取出数组的第一个元素
        if (root != first) { //根节点和数组的元素不相同
            Node<K,V> rn;
            tab[index] = root; //根节点放到数组中
            TreeNode<K,V> rp = root.prev; //取出根节点的上一个元素
            if ((rn = root.next) != null)  //根节点下一个元素不为空
                ((TreeNode<K,V>)rn).prev = rp; //根节点下一个元素前指针指向根节点上一个元素
            if (rp != null) //根节点上一个元素不为空
                rp.next = rn;//根节点上一个元素后指针指向根节点下一个元素
            if (first != null) //数组元素存在
                first.prev = root; //数组元素的前指针指向根节点
            root.next = first;//根节点的后指针指向取出的数组元素
            root.prev = null; //端口根节点的前指针
        }
        assert checkInvariants(root);
    }
}

```

上面就是红黑树插入的代码，逻辑并不难，总结一下就是通过遍历红黑树，用节点的 hash 值和树中的节点进行比较，计算出要插入节点在树中的方向，是左边还是右边。因为红黑树也是一颗二叉排序树，具有二叉排序树的特点，值小的放左边，大的放右边，找到合适的位置后把节点指针连上就把数据插入到树中了。

 

接下来最麻烦的就是插入数据后进行二叉树的平衡操作，因为数据插入当前的树可能就不满足红黑树的特点了，为了让插入数据的树也是一颗红黑树，就要进行平衡操作。我们先来看看插入数据平衡时要经过哪些步骤：

![红黑树插入逻辑](https://gitee.com/ZeTing/UploadImg/raw/main/img/untitled.png)

```java
//插入平衡操作
static <K,V> TreeNode<K,V> balanceInsertion(TreeNode<K,V> root,
                                            TreeNode<K,V> x) {
    x.red = true;//把节点染红
    for (TreeNode<K,V> xp, xpp, xppl, xppr;;) {
        if ((xp = x.parent) == null) { //插入的节点就是根节点
            x.red = false;//节点涂黑
            return x;//返回当前节点
        }
        else if (!xp.red || //父节点是黑节点
                (xpp = xp.parent) == null) //祖父节点为空，即是根节点的孩子                
            return root; //根节点无变化
        if (xp == (xppl = xpp.left)) { //父节点是祖父节点的左孩子
            if ((xppr = xpp.right) != null && xppr.red) { //祖父节点的右孩子（叔叔节点）是红色
                xppr.red = false;//将当前节点的叔叔节点涂黑
                xp.red = false;//父节点涂黑
                xpp.red = true; //祖父节点涂红
                x = xpp;//把当前节点指向祖父节点，从新的当前节点开始算法
            }
            else {//祖父节点的右孩子（叔叔节点）是黑色
                if (x == xp.right) { //当前节点是其父节点的右孩子
                    //当前节点的父节点做为新的当前节点，以新当前节点为支点左旋。
                    root = rotateLeft(root, x = xp);
                    //当前节点的父节点不为空时祖父节点指向父节点，否则祖父节点赋空
                    xpp = (xp = x.parent) == null ? null : xp.parent;
                }
                if (xp != null) {//当前节点是其父节点的左孩子
                    xp.red = false; //父节点涂黑
                    if (xpp != null) { //祖父节点不为空
                        xpp.red = true; //祖父节点变红色
                        root = rotateRight(root, xpp);//以祖父节点为支点进行右旋
                    }
                }
            }
        }
        else {//父节点是祖父节点的右孩子；和上面一样，左变右
            if (xppl != null && xppl.red) {//祖父节点的左孩子（叔叔节点）是红色
                xppl.red = false;//将当前节点的叔叔节点涂黑
                xp.red = false;//父节点涂黑
                xpp.red = true;//祖父节点涂红
                x = xpp;//把当前节点指向祖父节点，从新的当前节点开始算法
            }
            else {//祖父节点的左孩子（叔叔节点）是黑色
                if (x == xp.left) {//当前节点是其父节点的左孩子
                    //当前节点的父节点做为新的当前节点，以新当前节点为支点右旋。
                    root = rotateRight(root, x = xp);
                    //当前节点的父节点不为空时祖父节点指向父节点，否则祖父节点赋空
                    xpp = (xp = x.parent) == null ? null : xp.parent;
                }
                if (xp != null) {//当前节点是其父节点的右孩子
                    xp.red = false;//父节点涂黑
                    if (xpp != null) {//祖父节点不为空
                        xpp.red = true;//祖父节点变红色
                        root = rotateLeft(root, xpp);//以祖父节点为支点进行左旋
                    }
                }
            }
        }
    }
}
```

HashMap 中的旋转代码：

```java
//左旋转操作
static <K, V> TreeNode<K, V> rotateLeft(TreeNode<K, V> root,
                                        TreeNode<K, V> p) {
    TreeNode<K, V> r, pp, rl;
    if (p != null && (r = p.right) != null) { //当前节点不为空并且右孩子不为空
        if ((rl = p.right = r.left) != null) //当前节点右孩子的左孩子不为空
            //p.right = r.left 是通过左旋把右孩子的左孩子变为当前节点的右孩子
            rl.parent = p;//将右孩子的左孩子父亲指向当前节点
        if ((pp = r.parent = p.parent) == null)//当前节点没有父节点
            // r.parent = p.parent 通过左旋把右孩子作为根节点，父指针和当前节点断开
            (root = r).red = false; //将右孩子作为根节点并涂黑
        else if (pp.left == p) //当前节点是父亲的左孩子节点
            pp.left = r; //左旋把父节点的左孩子指向当前节点的右孩子
        else //当前节点是父亲的右孩子节点
            pp.right = r;//左旋把父节点的右孩子指向当前节点的右孩子
        r.left = p;//左旋后把当前节点作为右孩子的左孩子
        p.parent = r;//左旋后把当前节点父指针指向右孩子
    }
    return root;
}

//右旋转操作
static <K, V> TreeNode<K, V> rotateRight(TreeNode<K, V> root,
                                         TreeNode<K, V> p) {
    TreeNode<K, V> l, pp, lr;
    if (p != null && (l = p.left) != null) {//当前节点不为空并且左孩子不为空
        if ((lr = p.left = l.right) != null)//当前节点左孩子的右孩子不为空
            //p.left = l.right 是通过右旋把左孩子的右孩子变为当前节点的左孩子
            lr.parent = p;//将左孩子的右孩子父亲指向当前节点
        if ((pp = l.parent = p.parent) == null)//当前节点没有父节点
            // l.parent = p.parent 通过右旋把左孩子作为根节点，父指针和当前节点断开
            (root = l).red = false;//将左孩子作为根节点并涂黑
        else if (pp.right == p)//当前节点是父亲的右孩子节点
            pp.right = l;//右旋把父节点的右孩子指向当前节点的左孩子
        else//当前节点是父亲的左孩子节点
            pp.left = l;//右旋把父节点的左孩子指向当前节点的左孩子
        l.right = p;//右旋后把当前节点作为左孩子的右孩子
        p.parent = l;//右旋后把当前节点父指针指向左孩子
    }
    return root;
}
```

# get方法

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) { //取出数组中索引位对应的节点元素
        if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
            //数组中的元素接口 key 值和查询的 key 值一致
            return first;
        if ((e = first.next) != null) { // 节点的 next 指针不为空
            if (first instanceof TreeNode) //是红黑树节点
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                    //查询到 key 值一致的元素节点
                    return e;
            } while ((e = e.next) != null); //遍历链表
        }
    }
    return null;
}

//取出红黑树中的节点
final TreeNode<K,V> getTreeNode(int h, Object k) {
    return ((parent != null) ? root() : this).find(h, k, null);
}

//遍历查找红黑树中的元素
final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
    TreeNode<K,V> p = this;
    do {
        int ph, dir; K pk;
        TreeNode<K,V> pl = p.left, pr = p.right, q;
        if ((ph = p.hash) > h) //当前节点 hash 值大于 key 的hash 值
            p = pl; //查找左孩子
        else if (ph < h) //当前节点 hash 值小于 key 的hash 值
            p = pr; //查找右孩子
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            //当前节点 key 值和查询的 key 值一致，返回当前节点
            return p;
        else if (pl == null) //左孩子为空
            p = pr; //查找右孩子
        else if (pr == null)//右孩子为空
            p = pl;//查找左孩子
        else if ((kc != null ||
                (kc = comparableClassFor(k)) != null) &&
                (dir = compareComparables(kc, k, pk)) != 0) // 元素实现了 comparable 类
            //通过 comparable 比对结果获取查找的方向
            p = (dir < 0) ? pl : pr;
        else if ((q = pr.find(h, k, kc)) != null) //遍历右孩子
            return q;
        else
            p = pl; //遍历左孩子
    } while (p != null);
    return null;
}
```

get步骤：

1. 通过 key 的 hash 值计算出数组索引位。
2. 取出索引位节点判断是否命中，若 key 值相同则直接返回。
3. 若未命中并且索引位上存在 hash 冲突的链表，则从头到尾遍历链表查找相同 key 值的 value。
4. 如果索引位上的是红黑树，则通过遍历树查找相同 key 值的 value。

# remove 方法

remove 就是移除当前 key 的数据，首先通过 key 的 hash 索引从表中查到对应的 value 值，移动数据的指针断开移除数据和哈希表中的数据引用，就可以把数据从 map 中移除。

```java
public V remove(Object key) {
    Node<K,V> e;
    //计算 hash 值并移除节点
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
}

//移除节点
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {//取出数组中索引位对应的节点元素
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
            //查找到 key 值相同的节点
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode) //节点是红黑树节点
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key); //从红黑树中取出对应 key 的节点
            else {
                do {
                    if (e.hash == hash &&
                            ((k = e.key) == key ||
                                    (key != null && key.equals(k)))) {//链表中查到 key 值相同的节点
                        node = e;
                        break;
                    }
                    p = e;//指针移位
                } while ((e = e.next) != null); //遍历链表
            }
        }
        if (node != null && (!matchValue || (v = node.value) == value ||
                (value != null && value.equals(v)))) { //查到到有节点存在
            if (node instanceof TreeNode) //查找到的节点为树节点
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);//移除树节点
            else if (node == p) //查找到的节点为数组中的节点
                tab[index] = node.next; //把节点下一指针数据移到数组中
            else //把链表要移除元素的上一个元素指针指向移除元素的下一指针
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

从红黑树中移除数据

```java
//移除树节点
final void removeTreeNode(HashMap<K,V> map, Node<K,V>[] tab,
                          boolean movable) {
    int n;
    if (tab == null || (n = tab.length) == 0)
        return;
    int index = (n - 1) & hash; //计算数组中索引位
    TreeNode<K,V> first = (TreeNode<K,V>)tab[index], root = first, rl; //取出数组中索引位对应的节点元素
    TreeNode<K,V> succ = (TreeNode<K,V>)next, pred = prev;
    if (pred == null)//移除的节点为数组中的节点
        tab[index] = first = succ; //把移除节点的下一节点移动到数组中
    else //移除的节点存在上一个节点
        pred.next = succ; //把上一个节点的指针指向下一节点
    if (succ != null) //如果移除节点的下一节点不为空
        succ.prev = pred; //把下一节点的指针指向上一节点
    if (first == null) //数组索引位上没有元素
        return;
    if (root.parent != null) // 数组位上节点的父节点不为空
        root = root.root(); //取出根节点
    if (root == null || root.right == null || //根节点或根节点的右孩子为空
            (rl = root.left) == null || rl.left == null) { //根节点左孩子或根节点左孩子的左孩子为空
        tab[index] = first.untreeify(map);  // too small
        return;
    }
    TreeNode<K,V> p = this, pl = left, pr = right, replacement;
    if (pl != null && pr != null) { //当前节点有左孩子和右孩子
        TreeNode<K,V> s = pr, sl;
        while ((sl = s.left) != null) // find successor
            s = sl; //移动指针到右孩子最左边孩子
        // p 节点和 s 节点交换颜色
        boolean c = s.red; s.red = p.red; p.red = c; // swap colors
        TreeNode<K,V> sr = s.right;
        TreeNode<K,V> pp = p.parent;
        if (s == pr) { // p was s's direct parent
            //右孩子没有左孩子
            p.parent = s;// 当前节点和右孩子交换位置
            s.right = p;
        }
        else { //右孩子有左孩子
            TreeNode<K,V> sp = s.parent; // 取出右孩子最左边孩子的父节点
            if ((p.parent = sp) != null) { //sp 节点不为空
                //把当前节点 p 和 s 节点交换位置
                if (s == sp.left) // s 是左孩子
                    sp.left = p; //p 和左孩子交换位置
                else // s 是右孩子
                    sp.right = p; // p 和右孩子交换位置
            }
            if ((s.right = pr) != null) // s 节点的右孩子和 p 节点右孩子不为空
                pr.parent = s; //p 节点的右孩子 pr 作为 s 节点的右孩子
        }
        p.left = null; // p 节点的左孩子赋空
        if ((p.right = sr) != null) //s 节点的右孩子不为空
            sr.parent = p; //把 s 节点的右孩子作为 p 节点的左孩子 
        if ((s.left = pl) != null) //p 节点的左孩子不为空
            pl.parent = s; // 把 p 节点的左孩子作为 s 节点的左孩子
        if ((s.parent = pp) == null)//当前节点的父节点为空
            root = s; //把 s 节点作为根节点
        else if (p == pp.left) // 当前节点是父节点的左孩子
            pp.left = s; //把 s 节点作为父节点的左孩子
        else // 当前节点是父节点的右孩子
            pp.right = s;//把 s 节点作为父节点的右孩子
        if (sr != null) //s 节点的右孩子不为空
            replacement = sr; // 替换 s 节点的右孩子
        else //s 节点的右孩子为空
            replacement = p;// 替换当前节点 p
    }
    else if (pl != null) //当前节点只有左孩子
        replacement = pl; // 替换左孩子
    else if (pr != null) //当前节点只有右孩子
        replacement = pr; //替换右孩子
    else // 当前节点是叶子节点
        replacement = p; //替换当前节点
    if (replacement != p) { //如果替换的不是当前节点
        //取得当前节点的父节点，并把替换节点是父指针指向当前节点的父节点
        TreeNode<K,V> pp = replacement.parent = p.parent;
        if (pp == null) //如果父节点为空
            root = replacement; //替换的节点作为根节点
        else if (p == pp.left) //当前节点是左孩子
            pp.left = replacement; //替换节点作为父亲节点的左孩子
        else //当前节点是右孩子
            pp.right = replacement; //替换节点作为父亲节点的右孩子
        p.left = p.right = p.parent = null; //断开当前节点的指针
    }
    //如果删除的是黑节点，则进行平衡操作
    TreeNode<K,V> r = p.red ? root : balanceDeletion(root, replacement);
    //如果替换的是当前节点，即是叶子节点
    if (replacement == p) {  // detach
        TreeNode<K,V> pp = p.parent; //取到当前节点的父亲节点
        p.parent = null; //断开当前节点的父指针
        if (pp != null) { //如果父亲不为空
            if (p == pp.left) //如果移除的节点是左孩子
                pp.left = null; //断开父节点的左指针
            else if (p == pp.right) //如果移除节点是右孩子
                pp.right = null; //断开父节点的右指针
        }
    }
    if (movable) //移动根节点位置
        moveRootToFront(tab, r);
}
```

代码很长，但是理解原理就不是很难了，如果删除的是叶子节点，则断开叶子节点和父节点之前的指针连接就可以把数据从数组删除了，如果不是叶子节点则把要删除的节点移动到叶子节点再进行删除。注意一点的是，如果删除节点后元素个数达到退化为链表的阈值，就要把树转化为链表。而且删除之后的树可能又不是一颗红黑树了，所以还要进行平衡操作，还是先来看下删除的平衡操作要经过哪些步骤

![红黑树删除步骤](https://gitee.com/ZeTing/UploadImg/raw/main/img/%E7%BA%A2%E9%BB%91%E6%A0%91%E5%88%A0%E9%99%A4%E6%AD%A5%E9%AA%A4.png)



```java
//删除节点的平衡操作
static <K,V> TreeNode<K,V> balanceDeletion(TreeNode<K,V> root,
                                           TreeNode<K,V> x) {
    for (TreeNode<K,V> xp, xpl, xpr;;)  { //遍历红黑树
        if (x == null || x == root) // 替换节点为空或者是根节点
            return root; //不用平衡操作
        else if ((xp = x.parent) == null) { // 替换当前节点没父亲节点
            x.red = false; //把节点染成黑色
            return x;
        }
        else if (x.red) {//如果当前节点是红色
            x.red = false; //直接染黑，结束
            return root;
        }
        else if ((xpl = xp.left) == x) { //如果当前节点是左孩子
            if ((xpr = xp.right) != null && xpr.red) {//如果存在兄弟节点并且为红色
                xpr.red = false; //兄弟节点染黑
                xp.red = true; //父节点涂红
                root = rotateLeft(root, xp); //以父节点作为支点进行左旋转操作
                //父节点不为空则取出兄弟节点
                xpr = (xp = x.parent) == null ? null : xp.right;
            }
            if (xpr == null) //不存在兄弟节点
                x = xp; //设置x的父节点为新的x节点
            else { //存在兄弟节点
                TreeNode<K,V> sl = xpr.left, sr = xpr.right;
                if ((sr == null || !sr.red) &&
                        (sl == null || !sl.red)) {//兄弟节点的两个孩子都为黑色
                    xpr.red = true; //将x的兄弟节点设为红色
                    x = xp; //设置x的父节点为新的x节点
                }
                else {
                    if (sr == null || !sr.red) {//兄弟节点的右孩子不存在或右孩子为黑节点
                        if (sl != null) //兄弟节点有左孩子
                            sl.red = false; //把兄弟节点的左孩子染黑
                        xpr.red = true; //把兄弟节点涂红
                        root = rotateRight(root, xpr); //以兄弟节点作为支点进行右旋转操作
                        xpr = (xp = x.parent) == null ?
                                null : xp.right;//存在父亲则取出兄弟节点
                    }
                    if (xpr != null) { //存在兄弟节点
                        //把兄弟节点染成当前节点父节点颜色
                        xpr.red = (xp == null) ? false : xp.red;
                        if ((sr = xpr.right) != null) //兄弟节点有右孩子
                            sr.red = false; //把兄弟节点的右孩子染黑
                    }
                    if (xp != null) { // 存在父亲节点
                        xp.red = false; // 把当前节点父节点染成黑色
                        root = rotateLeft(root, xp); //以父亲节点作为支点进行左旋操作
                    }
                    x = root;
                }
            }
        }
        else { // symmetric 删除的是右孩子
            if (xpl != null && xpl.red) { //如果存在兄弟节点并且为红色
                xpl.red = false; //左兄弟节点染黑
                xp.red = true;//父节点涂红
                root = rotateRight(root, xp);//以父节点作为支点进行右旋转操作
                //父节点不为空则取出左兄弟节点
                xpl = (xp = x.parent) == null ? null : xp.left;
            }
            if (xpl == null)//不存在兄弟节点
                x = xp;//和父节点交换
            else {//存在左兄弟节点
                TreeNode<K,V> sl = xpl.left, sr = xpl.right;
                if ((sl == null || !sl.red) &&
                        (sr == null || !sr.red)) { //兄弟节点的两个孩子都为黑色
                    xpl.red = true;//将x的兄弟节点设为红色
                    x = xp;//设置x的父节点为新的x节点
                }
                else {
                    if (sl == null || !sl.red) { //兄弟的左孩子是黑色
                        if (sr != null) //兄弟节点存在右孩子
                            sr.red = false;//把兄弟节点的右孩子染黑
                        xpl.red = true;//把兄弟节点涂红
                        root = rotateLeft(root, xpl);//以兄弟节点作为支点进行左旋转操作
                        //父节点不为空则取出兄弟节点
                        xpl = (xp = x.parent) == null ?
                                null : xp.left;
                    }
                    if (xpl != null) {//存在兄弟节点
                        //把兄弟节点染成当前节点父节点颜色
                        xpl.red = (xp == null) ? false : xp.red;
                        if ((sl = xpl.left) != null)//兄弟节点有左孩子
                            sl.red = false; //兄弟节点左孩子染成黑色
                    }
                    if (xp != null) {
                        xp.red = false;//把当前节点父节点染成黑色
                        root = rotateRight(root, xp);//以当前节点的父节点为支点进行右旋，算法
                    }
                    x = root;
                }
            }
        }
    }
}
```

# 扩容机制

```java
//扩容
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length; //取出老数组的长度
    int oldThr = threshold; //当前阈值作为老的阈值
    //新的容量值，新的扩容阀界值
    int newCap, newThr = 0;
    if (oldCap > 0) { //原数组长度大于 0 
        if (oldCap >= MAXIMUM_CAPACITY) { // 原数组已是最大容量
            threshold = Integer.MAX_VALUE;// 阈值设置为最大容量
            return oldTab; //返回原来长度，不用扩容
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && //新的长度在原来基础上增大 2 倍
                oldCap >= DEFAULT_INITIAL_CAPACITY)  //原来的长度大于默认长度
            newThr = oldThr << 1; // double threshold 新的阈值也翻倍
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY; //使用默认容量
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY); //使用默认阈值
    }
    if (newThr == 0) { // 新阈值为 0 
        float ft = (float)newCap * loadFactor; //使用新容量和负载因子计算阈值
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? //新长度不超过最大容量则使用计算的阈值
                (int)ft : Integer.MAX_VALUE); //新长度超过则阈值为最大值
    }
    threshold = newThr; //把新计算的阈值作为当前阈值
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap]; //创建新长度的数组
    table = newTab; //当前使用的数组指向新数组
    if (oldTab != null) { // 老的数组不为空
        for (int j = 0; j < oldCap; ++j) {//遍历老数组
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {//取出数组中的值
                oldTab[j] = null; //回收老数组的指向
                if (e.next == null) // 取出的值只有一个元素
                    newTab[e.hash & (newCap - 1)] = e; //重新计算位置，并把值填充到新数组中
                else if (e instanceof TreeNode) //取出的值是树节点
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order 存在哈希冲突的链表结构
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next; // 移位到下一个位置
                        //与老数组长度计算决定元素是放在原索引处还是新索引
                        if ((e.hash & oldCap) == 0) { //老索引位
                            if (loTail == null) //低位尾结点为空
                                loHead = e; //当前节点作为低位头结点
                            else //低位尾结点不为空
                                loTail.next = e; //低位尾结点下一个指针指向当前节点
                            loTail = e; //当前节点作为低位尾结点
                        }
                        else {//放到新索引位
                            if (hiTail == null)//高位尾结点为空
                                hiHead = e;//当前节点作为高位头结点
                            else//高位尾结点不为空
                                hiTail.next = e;//高位尾结点值一个指针指向当前节点
                            hiTail = e;//当前节点作为高位尾结点
                        }
                    } while ((e = next) != null);//遍历链表
                    if (loTail != null) {//低位链表有数据
                        loTail.next = null;
                        newTab[j] = loHead;//把新的链表存到新数组的原来位置上
                    }
                    if (hiTail != null) {//高位链表有数据
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;//把新的链表存到新数组的 index + 老长度位置
                    }
                }
            }
        }
    }
    return newTab;
}
```





















# 参考出处

https://blog.csdn.net/guizhou_tiger_chen/article/details/108329677