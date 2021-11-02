# SparseArray 知识点总结


## 优质文章

[这一次，彻底搞懂SparseArray实现原理 - zhangpan's blog 👍](https://zhpanvip.gitee.io/2021/06/13/38-sparsearray/)

[SparseArray 的使用及实现原理 - 掘金](https://juejin.cn/post/6844903442901630983)

[SparseArray详解及源码简析 - 掘金](https://juejin.cn/post/6844903760548855816)

[Android轻量级数据SparseArray详解_楠枫的博客-CSDN博客](https://blog.csdn.net/weixin_40299948/article/details/99889024)

## 常见问题

{{< admonition type=tip title="提示" open=true >}}
下面是一些常见的问题，可以先尝试的先自我回答一下，点击题目即可查看答案。
{{< /admonition >}}



{{< admonition type=question title="什么是SparseArray？" open=false >}}
1. 以键值对形式进行存储，基于分查找,因此查找的时间复杂度为`O(LogN)`;
2. 由于SparseArray中Key存储的是数组形式,因此可以直接以`int作为Key`。避免了HashMap的装箱拆箱操作,性能更高且int的存储开销远远小于Integer;
3. 采用了`延迟删除的机制`(重要）
{{< /admonition >}}





{{< admonition type=question title="SparseArray都是在时候删除数据" open=false >}}
* put、size、index、append
1. put的时候，记录总大小和实际数组大小不一致时
{{< /admonition >}}





{{< admonition type=question title="SparseArray默认长度是多数？" open=false >}}
SparseArray默认的无参构造方法的`初始容量为10`，但是经过内部处理后变为11。初始化后mKeys和mValues都是长度为11的未赋值数组
```java
public SparseArray() {
        this(10);
}

public SparseArray(int initialCapacity) {
    if (initialCapacity == 0) {
        mKeys = EmptyArray.INT;
        mValues = EmptyArray.OBJECT;
    } else {
        mValues = ArrayUtils.newUnpaddedObjectArray(initialCapacity);
        mKeys = new int[mValues.length];
    }
    mSize = 0;
}
```
{{< /admonition >}}




{{< admonition type=question title="SparseArray最大容量？每次扩容多少？" open=false >}}
SparseArray并不像HashMap一样定义有最大容量是多少，最大可以达到Integer.MAX_VALUE，可能会报oom。
每次扩容时如果当前容量`小于5则扩容是8`，否则原容量的`2倍`，看put原理时会知道为什么。
{{< /admonition >}}






{{< admonition type=question title="SparseArray get的原理是？" open=false >}}
因为SparseArray采用的是`两个一维数组`分别存储键和值，所以根据key找到下标，就可以使用该下标取出对应的值。其中找到`key对应的下标采用的是二分法`。
```java
public E get(int key) {
        return get(key, null);
}

public E get(int key, E valueIfKeyNotFound) {
    //通过二分法找在mkeys数组中找到匹配的key的下标
    int i = ContainerHelpers.binarySearch(mKeys, mSize, key);

    if (i < 0 || mValues[i] == DELETED) {
        //如果没找到对应的值，则返回默认值null
        return valueIfKeyNotFound;
    } else {
        //返回找到匹配的值
        return (E) mValues[i];
    }
}
```
{{< /admonition >}}






{{< admonition type=question title="SparseArray put的原理是？" open=false >}}
1. 通过二分法找出key的下标，判断是否已经存在要添加的key，如果找到直接替换对应的value
2. 如果不存在要添加的key，且容量充足，直接添加
3. 如果不存在要添加的key，则分别对存储键和值的两个数组进行扩容并添加元素

透过put的操作可以得到的信息有：
1. SparseArray是允许插入value为null的情况，因为key是int类型，所以不存在key为null的情况
2. SparseArray添加元素时key是确保是有序的，是按key的大小进行排序的，因为其采用的二分法现在找到对应key对应的下标
3. SpareseArray默认每次扩容是原容量的2倍（特殊情况如果当前容量小5则扩容到8）

```java
public void put(int key, E value) {
    //通过二分法找到key所在的下标
    int i = ContainerHelpers.binarySearch(mKeys, mSize, key);

    if (i >= 0) {
        //如果i >= 0 代表当前以及存在key及其对应的值，则直接替换value即可
        mValues[i] = value;
    } else {
        //如果i < 0 代表当前并不存在key，意思就是添加新的键值对
        // i 取反操作，得到要添加的下标，为什么取反？后面进行分析会进行讲解
        i = ~i;

        if (i < mSize && mValues[i] == DELETED) {
            //当前容量充足，直接添加即可
            mKeys[i] = key;
            mValues[i] = value;
            return;
        }

        if (mGarbage && mSize >= mKeys.length) {
            //当前容量不足，进行回收操作
            gc();

            // Search again because indices may have changed.
            //因为进行回收操作，需要重新使用获取要添加的下标
            i = ~ContainerHelpers.binarySearch(mKeys, mSize, key);
        }
        //在存储按键数组中下标为i，添加新键为key
        mKeys = GrowingArrayUtils.insert(mKeys, mSize, i, key);
        //在存储值数组中下标为i，添加新值为value
        mValues = GrowingArrayUtils.insert(mValues, mSize, i, value);
        //真实数据个数加1
        mSize++;
    }
}


//GrowingArrayUtils.java
public static <T> T[] insert(T[] array, int currentSize, int index, T element) {
    assert currentSize <= array.length;

    if (currentSize + 1 <= array.length) {
        //如果当前数组容量充足，先将当前下标index往后移动
        System.arraycopy(array, index, array, index + 1, currentSize - index);
        //在将要添加的元素放到下标为index的地方
        array[index] = element;
        return array;
    }

    //如果容量不足，先进行扩容生成新的数组newArray
    @SuppressWarnings("unchecked")
    T[] newArray = ArrayUtils.newUnpaddedArray((Class<T>)array.getClass().getComponentType(),
            growSize(currentSize));
    //将原数组中index个元素拷贝到新数组中
    System.arraycopy(array, 0, newArray, 0, index);
    //将要添加的元素添加到index位置
    newArray[index] = element;
    //将原数组中index+1之后的元素拷贝到新数组中
    System.arraycopy(array, index, newArray, index + 1, array.length - index);
    return newArray;
}

public static int growSize(int currentSize) {
    //扩容计算规则，当前容量小于5返回8；否则返回2倍的容量
    return currentSize <= 4 ? 8 : currentSize * 2;
}
```
{{< /admonition >}}






{{< admonition type=question title="SparseArray的扩容机制？" open=false >}}
1. SparseArray占用的内存比HashMap更少
2. SparseArray的速度比HashMap更慢，SparseArray是O(LogN)(二分查找)，HashMap是O(1)
3. SparseArray的Key只能是Int，HashMap的Key可以为Object
4. SparseArray删除是懒删除，HashMap是直接删除
{{< /admonition >}}







{{< admonition type=question title="SparseArray和HashMap的区别？" open=false >}}
1. SparseArray占用的内存比HashMap更少
2. SparseArray的速度比HashMap更慢，SparseArray是O(LogN)(二分查找)，HashMap是O(1)
3. SparseArray的Key只能是Int，HashMap的Key可以为Object
4. SparseArray删除是懒删除，HashMap是直接删除
{{< /admonition >}}



