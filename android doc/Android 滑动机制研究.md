Android 滑动机制研究

## NestedScrollingParent3
```
/**
 * 子视图触发滑动时会回调该方法，父容器在该方法中根据子view、滑动方向、触摸类型等判断自己是否支持接收，
 * 若接收返回true，否则返回false。（可由NestedScrollingChild2的startNestedScroll方法触发）
 */
boolean onStartNestedScroll(@NonNull View child, @NonNull View target, @ScrollAxis int axes, @NestedScrollType int type);

/**
 * onStartNestedScroll返回true后会回调该方法，可在此方法中做一些初始配置操作。
 */
void onNestedScrollAccepted(@NonNull View child, @NonNull View target, @ScrollAxis int axes, @NestedScrollType int type);

/**
 * 开始滑动时，子视图会优先回调该方法。父容器可以处理自己的滚动操作，之后将剩余的滚动偏移量 传回给子视图。（可由NestedScrollingChild2的dispatchNestedPreScroll方法触发）
 */
void onNestedPreScroll(@NonNull View target, int dx, int dy, @NonNull int[] consumed, @NestedScrollType int type);

/**
 * 子视图处理完剩余的滚动偏移量后，若还有剩余，则将剩余的滚动偏移量再通过该回调传给父容器处理。（可由NestedScrollingChild2的dispatchNestedScroll方法触发）
 */
void onNestedScroll(@NonNull View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed, @NestedScrollType int type);

/**
 * 子视图滑动结束时，回调该方法。（可由NestedScrollingChild2的stopNestedScroll方法触发）
 */
void onStopNestedScroll(@NonNull View target, @NestedScrollType int type);

//是纵向滑动还是横向滑动
  int getNestedScrollAxes();

```

## NestedScrollingChild3
```
/**
 * 通知开始滑动，会回调父容器的onStartNestedScroll方法。
 */
boolean startNestedScroll(@ScrollAxis int axes, @NestedScrollType int type);

/**
 * 通知停止滑动，会回调父容器的onStopNestedScroll方法。
 */
void stopNestedScroll(@NestedScrollType int type);

/**
 * 查询是否有父容器支持指定类型的嵌套滑动。
 */
boolean hasNestedScrollingParent(@NestedScrollType int type);

/**
 * 在子视图处理滑动前，先将滚动偏移量传递给父容器。
 */
boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed, @Nullable int[] offsetInWindow, @NestedScrollType int type);

/**
 * 子视图处理滑动后，再将剩余的滚动偏移量传递给父容器。
 */
boolean dispatchNestedScroll(int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow, @NestedScrollType int type);
```

---

## 部分参数含义说明
- child：表示包含target的当前容器的直接子view。
- target：表示调用startNestedScroll触发onStartNestedScroll回调的那个子view。
- axes：表示即将滑动的坐标轴方向，通过位运算求出方向。
- type：表示触摸类型，有TYPE_TOUCH（用户触摸）、TYPE_NON_TOUCH（惯性滑动）两种类型。

- dx：水平滑动偏移量。<0表示手指向右划，>0则相反。

- dy：垂直滑动偏移量。<0表示手指向下划，>0则相反。

- consumed：保存父容器滑动消耗的偏移量（索引0存x轴偏移，1存y轴偏移）。在父容器滑动后，子view会将原偏移量减去consumed中的值得到剩余偏移量，再进行自身的滚动处理。

- dxConsumed：子view消耗的水平偏移量。

- dyConsumed：子view消耗的垂直偏移量。

- dxUnconsumed：子view滑动后还剩下的水平偏移量。
- dyUnconsumed：子view滑动后还剩下的垂直偏移量。
注意：若有用户触摸滑动到惯性滑动，会走两遍方法执行流程，即不同type各触发一次流程。


---


## NestedScrollingChild 与 NestedScrollingParent 方法的对应关系

|子View|父View|方法描述|
|:--------|:---|:------------------ |
startNestedScroll | onStartNestedScroll , onNestedScrollAccepted | Scrolling Child 开始滑动的时候，通知 Scrolling Parent 要开始滑动了，通常是在 Action_down 动作 的时候调用这个方法
dispatchNestedPreScroll | onNestedPreScroll | 在 Scrolling Child 要开始滑动的时候，询问 Scrolling Parent 是否先于 Scrolling Child 进行相应的处理，同时是在 Action_move 的时候调用
dispatchNestedScroll | onNestedScroll | 在 Scrolling Child 滑动后会询问 Scrolling Parent 是否需要继续滑动
dispatchNestedPreFling | onNestedPreFling | 在 Scrolling Child 开始处理 Fling 动作的时候，询问 Scrolling Parent  是否需要先处理 Fling 动作
dispatchNestedFling | onNestedFling | 在 Scrolling Child  处理 Fling 动作完毕的时候，询问 Scrolling Parent 是都还需要进行相应的处理
stopNestedScroll | onStopNestedScroll | 在 Scrolling Child 停止滑动的时候，会调用 Scrolling Parent 的这个方法。通常是在 Action_up  或者 Action_cancel 或者被别的 View 消费 Touch 事件的时候调用的


---


## NestedScrollView源码分析
