---
title: "LruCache 知识点总结"
date: 2021-11-02T15:25:07+08:00
draft: false
author: "Sollyu"
authorLink: "https://github.com/sollyu"
resources:
- name: "featured-image"
  src: "featured-image.jpg"

tags: ["安卓", "LruCache", "缓存"]
categories: ["安卓"]
---

## 优质文章

[从 LRU Cache 带你看面试的本质 - 码农田小齐 - 博客园 👍](https://www.cnblogs.com/nycsde/p/13722270.html)

[Glide都在用的LruCache，你学会了吗？ - 掘金](https://juejin.cn/post/6844904073221799944)

## 常见问题

{{< admonition type=tip title="提示" open=true >}}
下面是一些常见的问题，可以先尝试的先自我回答一下，点击题目即可查看答案。
{{< /admonition >}}



{{< admonition type=question title="谈谈你对LruCache的理解" open=false >}}
1. 最近使用的元素会移动到顶部
2. 最老使用的元素会在最后面
3. 当增加元素时，如果长度已满，将移除最后的元素
4. LruCache内部主要使用LinkedHashMap实现元素移动（设置accessOrder=true)
5. LruCache是线程安全的，关键函数里使用的synchronized进行保护
6. LruCache在使用时候必须提供容量的大小
{{< /admonition >}}





{{< admonition type=question title="LruCache是线程安全的吗？" open=false >}}
答案：是，举个例子
```java
public void resize(int maxSize) {
    if (maxSize <= 0) {
        throw new IllegalArgumentException("maxSize <= 0");
    }

    synchronized (this) {
        this.maxSize = maxSize;
    }
    trimToSize(maxSize);
}
```
{{< /admonition >}}





{{< admonition type=question title="LruCache检索速度" open=false >}}
速度和HashMap的检索速度一直，因为内部主要是LinkedHashMap在驱动
在最好的情况下是`O(1)`，最坏的情况下是O(LogN)
{{< /admonition >}}






{{< admonition type=question title="LruCache的Key和Value有什么要注意的吗？" open=false >}}
LruCache put时不允许key和value都不能为null
```java
public final V put(K key, V value) {
    if (key == null || value == null) {
        throw new NullPointerException("key == null || value == null");
    }
    ...
}
```
{{< /admonition >}}







{{< admonition type=question title="LruCache如何处理默认值？" open=false >}}
由于LruCache是对LinkHashMap的封装，因此，当key对应的value不存在时`会返回null`，如果你需要不返回null，可以通过覆盖`create()`方法创建默认的返回值。
```java
LruCache<String, String> strCache = new LruCache<String, String>(20) {
   // 重写 `create`方法
    protected String create(String key) {
        return "默认值";
    }
}
```
{{< /admonition >}}






{{< admonition type=question title="LruCache对象被移除能否感知到？" open=false >}}
可以。可以重写LruCache的`entryRemoved`函数来实现。
```java
int cacheSize = 4 * 1024 * 1024; // 4MiB
LruCache<String, Bitmap> bitmapCache = new LruCache<String, Bitmap>(cacheSize) {
   // 重写 `sizeOf`方法
    protected int sizeOf(String key, Bitmap value) {
        return value.getByteCount();
    }

   /**
    * @param evicted 只有当容量超过maxsize，自动释放条目时为true，put\get\remove时为false
    * @param newValue 如果是主动释放时或remove时，该值为null；如果是put，该值为put的值；如果是get时，并且
    *  key对应的value不存在时，并且`create`不为空，old 和 new 都为`create`创建的默认值
    */
    protected void entryRemoved(boolean evicted, String key, Bitmap oldValue, Bitmap newValue){
        oldValue.recycle();
    }
}
```
{{< /admonition >}}




## 视频资料

{{< bilibili BV13i4y1b7Qy >}}

