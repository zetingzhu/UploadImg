**LruCache 使用**

# 1.LruCache 原理

LruCache 其实使用了 LinkedHashMap 双向链表结构

LinkedHashMap 本来不是线程安全的，LruCache 对LinkedHashMap 进行封装，提升读写安全

## LinkedHashMap 简单原理

LinkedHashMap 是继承Hashmap 

accessOrder  排序方式 - true 用于*访问顺序，  false 用于插入顺序

当构造参数 accessOrder 设置为true的时候，get后结果会顺序会发生改变，get的数据会移动到链表的最后端

accessOrder==true

```java

    public void testLinkedHashMap() {
        LinkedHashMap<Integer, Integer> map = new LinkedHashMap<>(0, 0.75f, true);
        map.put(0, 0);
        map.put(1, 1);
        map.put(2, 2);
        map.put(3, 3);
        map.put(4, 4);
        map.put(5, 5);
        map.put(6, 6);
        map.put(7, 7);
        map.get(1);
        printMapLog(map, "1");
        map.get(5);
        printMapLog(map, "2");
        map.put(8, 8);
        map.put(9, 9);
        map.put(10, 10);
        printMapLog(map, "3");
    }
```

输出结果

```java
 System.out: 打印结果 1 :0:0       
 System.out: 打印结果 1 :2:2       
 System.out: 打印结果 1 :3:3       
 System.out: 打印结果 1 :4:4       
 System.out: 打印结果 1 :5:5       
 System.out: 打印结果 1 :6:6       
 System.out: 打印结果 1 :7:7       
 System.out: 打印结果 1 :1:1       
 System.out:                   
 System.out: 打印结果 2 :0:0       
 System.out: 打印结果 2 :2:2       
 System.out: 打印结果 2 :3:3       
 System.out: 打印结果 2 :4:4       
 System.out: 打印结果 2 :6:6       
 System.out: 打印结果 2 :7:7       
 System.out: 打印结果 2 :1:1       
 System.out: 打印结果 2 :5:5       
 System.out:                   
 System.out: 打印结果 3 :0:0       
 System.out: 打印结果 3 :2:2       
 System.out: 打印结果 3 :3:3       
 System.out: 打印结果 3 :4:4       
 System.out: 打印结果 3 :6:6       
 System.out: 打印结果 3 :7:7       
 System.out: 打印结果 3 :1:1       
 System.out: 打印结果 3 :5:5       
 System.out: 打印结果 3 :8:8       
 System.out: 打印结果 3 :9:9       
 System.out: 打印结果 3 :10:10     
```

accessOrder==false

```java
public void testLinkedHashMap1() {
    LinkedHashMap<Integer, Integer> map = new LinkedHashMap<>(0, 0.75f, false);
    map.put(0, 0);
    map.put(1, 1);
    map.put(2, 2);
    map.put(3, 3);
    map.put(4, 4);
    map.put(5, 5);
    map.put(6, 6);
    map.put(7, 7);
    map.get(1);
    printMapLog(map, "11");
    map.get(5);
    printMapLog(map, "12");
    map.put(8, 8);
    map.put(9, 9);
    map.put(10, 10);
    printMapLog(map, "13");
}
```

输出结果

```java
System.out: 打印结果 11 :0:0
System.out: 打印结果 11 :1:1
System.out: 打印结果 11 :2:2
System.out: 打印结果 11 :3:3
System.out: 打印结果 11 :4:4
System.out: 打印结果 11 :5:5
System.out: 打印结果 11 :6:6
System.out: 打印结果 11 :7:7
System.out:
System.out: 打印结果 12 :0:0
System.out: 打印结果 12 :1:1
System.out: 打印结果 12 :2:2
System.out: 打印结果 12 :3:3
System.out: 打印结果 12 :4:4
System.out: 打印结果 12 :5:5
System.out: 打印结果 12 :6:6
System.out: 打印结果 12 :7:7
System.out:
System.out: 打印结果 13 :0:0
System.out: 打印结果 13 :1:1
System.out: 打印结果 13 :2:2
System.out: 打印结果 13 :3:3
System.out: 打印结果 13 :4:4
System.out: 打印结果 13 :5:5
System.out: 打印结果 13 :6:6
System.out: 打印结果 13 :7:7
System.out: 打印结果 13 :8:8
System.out: 打印结果 13 :9:9
System.out: 打印结果 13 :10:10
```

 LruCache 的 put 和 get 方法。

## put 方法分析

put() 方法其实重点就在于 trimToSize() 方法里面，这个方法的作用就是判断加入元素后是否超过最大缓存数，如果超过就清除掉最少使用的元素。

```java
@Nullable
public final V put(@NonNull K key, @NonNull V value) {
     // 如果 key 或者 value 为 null，则抛出异常
    if (key == null || value == null) {
        throw new NullPointerException("key == null || value == null");
    }

    V previous;
    synchronized (this) {
        // 加入元素的数量，在 putCount() 用到
        putCount++;
        // 回调用 sizeOf(K key, V value) 方法，这个方法用户自己实现，默认返回 1
        size += safeSizeOf(key, value);
        // 返回之前关联过这个 key 的值，如果没有关联过则返回 null
        previous = map.put(key, value);
        if (previous != null) {
           // safeSizeOf() 默认返回 1
            size -= safeSizeOf(key, previous);
        }
    }

    if (previous != null) {
        // 该方法默认方法体为空
        entryRemoved(false, key, previous, value);
    }
	//重新计算缓存大小，并且删除超出缓存长时间不用的数据
    trimToSize(maxSize);
    return previous;
}
```

trimToSize  重新计算缓存大小，并且删除超出缓存长时间不用的数据

```java
public void trimToSize(int maxSize) {
    while (true) {
        K key;
        V value;
        synchronized (this) {
            if (size < 0 || (map.isEmpty() && size != 0)) {
                throw new IllegalStateException(getClass().getName()
                        + ".sizeOf() is reporting inconsistent results!");
            }
			// 直到缓存大小 size 小于或等于最大缓存大小 maxSize，则停止循环
            if (size <= maxSize || map.isEmpty()) {
                break;
            }
 			// 取出 map 中第一个元素
            Map.Entry<K, V> toEvict = map.entrySet().iterator().next();
            key = toEvict.getKey();
            value = toEvict.getValue();
            // 删除该元素
            map.remove(key);
            size -= safeSizeOf(key, value);
            // 统计计算删除的数量
            evictionCount++;
        }

        entryRemoved(true, key, value, null);
    }
}
```

## get 方法分析



```java

@Nullable
public final V get(@NonNull K key) {
    if (key == null) {
        throw new NullPointerException("key == null");
    }

    V mapValue;
    synchronized (this) {
        // 获取当前key对于value
        mapValue = map.get(key);
        if (mapValue != null) {
            // 当前数据被命中统计
            hitCount++;
            return mapValue;
        }
        // 获取数据失败的统计
        missCount++;
    }

    /*
     * Attempt to create a value. This may take a long time, and the map
     * may be different when create() returns. If a conflicting value was
     * added to the map while create() was working, we leave that value in
     * the map and release the created value.
     */

    V createdValue = create(key);
    if (createdValue == null) {
        return null;
    }

    synchronized (this) {
        createCount++;
        mapValue = map.put(key, createdValue);

        if (mapValue != null) {
            // There was a conflict so undo that last put
            map.put(key, mapValue);
        } else {
            size += safeSizeOf(key, createdValue);
        }
    }

    if (mapValue != null) {
        entryRemoved(false, key, createdValue, mapValue);
        return mapValue;
    } else {
        trimToSize(maxSize);
        return createdValue;
    }
}
```

LinkedHashMap 的get方法

```java
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    if (accessOrder)
        // 重要方法，访问后将访问的数据后移
        afterNodeAccess(e);
    return e.value;
}
```

LinkedHashMap 的afterNodeAccess方法，功能 将节点移到最后

```java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMapEntry<K,V> last;
    // 如果是末尾排序并且末尾数据不是当前对象
    // 并且将末尾数据赋值给last
    if (accessOrder && (last = tail) != e) {
        /**
        * 	p 将 e 转换成 LinkedHashMap.Entry赋值给P
        *	b e的上一个节点b
        *	a e的下一个节点a
        */
        LinkedHashMapEntry<K,V> p =
            (LinkedHashMapEntry<K,V>)e, b = p.before, a = p.after;
        // 将这个节点之后的节点置为 null
        p.after = null;
         // b 为 null，则代表这个节点是第一个节点，将它后面的节点置为第一个节点
        if (b == null)
            head = a;
          // 如果不是，则将 a 上前移动一位
        else
            b.after = a;
        // 如果 a 不为 null，则将 a 节点的元素变为 b
        if (a != null)
            a.before = b;
        // 如果a为空,将b赋值给last
        else
            last = b;
        // 如果last 为空说明没有数据，将p赋值给head
        if (last == null)
            head = p;
        else {
            // 如果最后不为空，将last赋值给当前对象的before节点
            // 将 p 赋值给last对象的after节点
            p.before = last;
            last.after = p;
        }
        // 将P赋值给链表的默认对象
        tail = p;
        ++modCount;
    }
}
```

# 2.LruCache 使用

```java
lruCacheX = new androidx.collection.LruCache<>(3);

setImage(image_view_1, "icon_1", R.drawable.icon_1);
printMapLog(lruCacheX.snapshot(), "图片1");
setImage(image_view_2, "icon_2", R.drawable.icon_2);
setImage(image_view_3, "icon_3", R.drawable.icon_3);
setImage(image_view_4, "icon_4", R.drawable.icon_4);
setImage(image_view_5, "icon_5", R.drawable.icon_5);
printMapLog(lruCacheX.snapshot(), "图片5");
setImage(image_view_6, "icon_1", R.drawable.icon_1);
printMapLog(lruCacheX.snapshot(), "图片6");
setImage(image_view_7, "icon_5", R.drawable.icon_5);
printMapLog(lruCacheX.snapshot(), "图片7");
```

结果显示

```java
System.out: 打印结果 图片1 :icon_1:android.graphics.Bitmap@8507e39 
System.out:                                                  
System.out: 打印结果 图片5 :icon_3:android.graphics.Bitmap@131b28a 
System.out: 打印结果 图片5 :icon_4:android.graphics.Bitmap@74b40fb 
System.out: 打印结果 图片5 :icon_5:android.graphics.Bitmap@5f95518 
System.out:                                                  
System.out: 打印结果 图片6 :icon_4:android.graphics.Bitmap@74b40fb 
System.out: 打印结果 图片6 :icon_5:android.graphics.Bitmap@5f95518 
System.out: 打印结果 图片6 :icon_1:android.graphics.Bitmap@d806d56 
System.out:                                                  
System.out: 打印结果 图片7 :icon_4:android.graphics.Bitmap@74b40fb 
System.out: 打印结果 图片7 :icon_1:android.graphics.Bitmap@d806d56 
System.out: 打印结果 图片7 :icon_5:android.graphics.Bitmap@5f95518 
```

使用总结

当我设置最大值为3的时候，

put的图片大小没有超过的时候会顺序添加到列表后面，

当超过最设置最大值的时候，会移除掉最开始添加的，

当获取缓存中已经存储的值，会将存储的值移动到链表的最后端

## 注意点 

这个方法要特别注意，跟我们实例化LruCache的maxSize要呼应， ，比如maxSize的大小为缓存的个数，这里就是return 1就ok，如果是内存的大小，如果5M，这个就不能是个数了，就需要覆盖这个方法，返回每个缓存 value的size大小，

如果是Bitmap，这应该是bitmap.getByteCount(); 

```java
/**
 * Returns the size of the entry for {@code key} and {@code value} in
 * user-defined units.  The default implementation returns 1 so that size
 * is the number of entries and max size is the maximum number of entries.
 *
 * <p>An entry's size must not change while it is in the cache.
 */
protected int sizeOf(@NonNull K key, @NonNull V value) {
    return 1;
}
```

# 3. 总结

- LruCache 其实使用了 LinkedHashMap 维护了强引用对象
- 总缓存的大小一般是可用内存的 1/8，当超过总缓存大小会删除最少使用的元素，也就是内部 LinkedHashMap 的头部元素
- 当使用 get() 访问元素后，会将该元素移动到 LinkedHashMap 的尾部