# AndroidView绘制流程知识点总结


## 优质文章

[Android View的绘制流程知识点总结 - 知乎](https://zhuanlan.zhihu.com/p/44976896)

[Android View的绘制流程 - 简书](https://www.jianshu.com/p/5a71014e7b1b)

[Android View绘制三大流程探索及常见问题_梦工厂-CSDN博客](https://blog.csdn.net/zhuwentao2150/article/details/53494760)

[Android View知识点面试题_lizhong-CSDN博客](https://blog.csdn.net/generallizhong/article/details/103964206)

[Android 面试总结之布局常见问题 - 程序员大本营](https://www.pianshen.com/article/520885170/)

## 常见问题

{{< admonition type=tip title="提示" open=true >}}
下面是一些常见的问题，可以先尝试的先自我回答一下，点击题目即可查看答案。
{{< /admonition >}}


{{< admonition type=question title="什么是View？" open=false >}}
简单来说，View是Android系统在屏幕上的视觉呈现，也就是说你在手机屏幕上看到的东西都是View。
{{< /admonition >}}


{{< admonition type=question title="View的绘制流程分几步，从哪开始？哪个过程结束以后能看到view？" open=false >}}
从ViewRoot的performTraversals开始，经过measure，layout,draw 三个流程。draw流程结束以后就可以在屏幕上看到view了。
{{< /admonition >}}


{{< admonition type=question title="View的测量宽高和实际宽高有区别吗？" open=false >}}
基本上百分之99的情况下都是可以认为没有区别的。

有两种情况，有区别。

第一种 就是有的时候会因为某些原因 view会多次测量，那第一次测量的宽高 肯定和最后实际的宽高 是不一定相等的，但是在这种情况下最后一次测量的宽高和实际宽高是一致的。

此外，实际宽高是在layout流程里确定的，我们可以在layout流程里 将实际宽高写死 写成硬编码，这样测量的宽高和实际宽高就肯定不一样了，虽然这么做没有意义 而且也不好。
{{< /admonition >}}

{{< admonition type=question title="View的measureSpec 由谁决定?顶级view呢？" open=false >}}
由view自己的layoutparams和父容器  一起决定自己的measureSpec。

一旦确定了spec，onMeasure中就可以确定view的宽高了。

顶级view就稍微特殊一点，对于decorView的测量在ViewRootImpl的源码里。

```java
//desire的这2个参数就代表屏幕的宽高，
childWidthMeasureSpec = getRootMeasureSpec(desiredWindowWidth, lp.width);
childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height);
performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);

//decorView的measureSpec就是在这里确定的，其实比普通view的measurespec要简单的多
//代码就不分析了 一目了然的东西
private static int getRootMeasureSpec(int windowSize, int rootDimension) {
    int measureSpec;
    switch (rootDimension) {

    case ViewGroup.LayoutParams.MATCH_PARENT:
        // Window can't resize. Force root view to be windowSize.
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
        break;
    case ViewGroup.LayoutParams.WRAP_CONTENT:
        // Window can resize. Set max size for root view.
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
        break;
    default:
        // Window wants to be an exact size. Force root view to be that size.
        measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
        break;
    }
    return measureSpec;
}
```
{{< /admonition >}}


{{< admonition type=question title="ViewGroup有onMeasure方法吗？为什么?" open=false >}}
没有，这个方法是交给子类自己实现的。不同的viewgroup子类 肯定布局都不一样，那onMeasure索性就全部交给他们自己实现好了。
{{< /admonition >}}


{{< admonition type=question title="layout和onLayout方法有什么区别？" open=false >}}
layout是确定本身view的位置 而onLayout是确定所有子元素的位置。layout里面 就是通过serFrame方法设设定本身view的 四个顶点的位置。这4个位置以确定 自己view的位置就固定了，然后就调用onLayout来确定子元素的位置。view和viewgroup的onlayout方法都没有写。都留给我们自己给子元素布局
{{< /admonition >}}


{{< admonition type=question title="draw方法 大概有几个步骤？" open=false >}}
一共是4个步骤， 绘制背景---------绘制自己--------绘制chrildren----绘制装饰。
{{< /admonition >}}


{{< admonition type=question title="View和ViewGroup什么区别？ " open=false >}}
Android的UI界面都是由View和ViewGroup及其派生类组合而成的。其中，View是所有UI组件的基类，而ViewGroup是容纳这些组件的容器，其本身也是从View派生出来的。AndroidUI界面的一般结构：

{{< mermaid >}}
graph TD;
    B(RViewGroup) --> C(ViewGroup)
    B --> D(View)
    D --> E(View)
    D --> F(View)
{{< /mermaid >}}


{{< /admonition >}}



## 视频资料

{{< bilibili BV1V5411b7qt >}}


