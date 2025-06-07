**NCalendar 使用整理**



#### NestedScrollingParent 接口介绍

```java

/**
     * 有嵌套滑动到来了，判断父控件是否接受嵌套滑动
     *
     * @param child            嵌套滑动对应的父类的子类(因为嵌套滑动对于的父控件不一定是一级就能找到的，可能挑了两级父控件的父控件，child的辈分>=target)
     * @param target           具体嵌套滑动的那个子类
     * @param nestedScrollAxes 支持嵌套滚动轴。水平方向，垂直方向，或者不指定
     * @return 父控件是否接受嵌套滑动， 只有接受了才会执行剩下的嵌套滑动方法
     */
public boolean onStartNestedScroll(View child, View target, int nestedScrollAxes) { }

/**
     * 当onStartNestedScroll返回为true时，也就是父控件接受嵌套滑动时，该方法才会调用
     */
public void onNestedScrollAccepted(View child, View target, int axes) { }

/**
     * 在嵌套滑动的子控件未滑动之前，判断父控件是否优先与子控件处理(也就是父控件可以先消耗，然后给子控件消耗）
     *
     * @param target   具体嵌套滑动的那个子类
     * @param dx       水平方向嵌套滑动的子控件想要变化的距离 dx<0 向右滑动 dx>0 向左滑动
     * @param dy       垂直方向嵌套滑动的子控件想要变化的距离 dy<0 向下滑动 dy>0 向上滑动
     * @param consumed 这个参数要我们在实现这个函数的时候指定，回头告诉子控件当前父控件消耗的距离
     *                 consumed[0] 水平消耗的距离，consumed[1] 垂直消耗的距离 好让子控件做出相应的调整
     */
public void onNestedPreScroll(View target, int dx, int dy, int[] consumed) { }

/**
     * 嵌套滑动的子控件在滑动之后，判断父控件是否继续处理（也就是父消耗一定距离后，子再消耗，最后判断父消耗不）
     *
     * @param target       具体嵌套滑动的那个子类
     * @param dxConsumed   水平方向嵌套滑动的子控件滑动的距离(消耗的距离)
     * @param dyConsumed   垂直方向嵌套滑动的子控件滑动的距离(消耗的距离)
     * @param dxUnconsumed 水平方向嵌套滑动的子控件未滑动的距离(未消耗的距离)
     * @param dyUnconsumed 垂直方向嵌套滑动的子控件未滑动的距离(未消耗的距离)
     */
public void onNestedScroll(View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed) { }

/**
     * 嵌套滑动结束
     */
public void onStopNestedScroll(View child) { }

/**
     * 当子控件产生fling滑动时，判断父控件是否处拦截fling，如果父控件处理了fling，那子控件就没有办法处理fling了。
     * 
     * @param target    具体嵌套滑动的那个子类
     * @param velocityX 水平方向上的速度 velocityX > 0  向左滑动，反之向右滑动
     * @param velocityY 竖直方向上的速度 velocityY > 0  向上滑动，反之向下滑动
     * @return 父控件是否拦截该fling
     */
public boolean onNestedPreFling(View target, float velocityX, float velocityY) { }

/**
     * 当父控件不拦截该fling,那么子控件会将fling传入父控件
     * 
     * @param target    具体嵌套滑动的那个子类
     * @param velocityX 水平方向上的速度 velocityX > 0  向左滑动，反之向右滑动
     * @param velocityY 竖直方向上的速度 velocityY > 0  向上滑动，反之向下滑动
     * @param consumed  子控件是否可以消耗该fling，也可以说是子控件是否消耗掉了该fling
     * @return 父控件是否消耗了该fling
     */
public boolean onNestedFling(View target, float velocityX, float velocityY, boolean consumed) { }

/**
     * 返回当前父控件嵌套滑动的方向，分为水平方向与，垂直方法，或者不变
     */
public int getNestedScrollAxes() { }

```

#### NestedScrollingChild接口介绍


```java
/**
     * 开启一个嵌套滑动
     *
     * @param axes 支持的嵌套滑动方法，分为水平方向，竖直方向，或不指定
     * @return 如果返回true, 表示当前子控件已经找了一起嵌套滑动的view
     */
public boolean startNestedScroll(int axes) {}

/**
     * 在子控件滑动前，将事件分发给父控件，由父控件判断消耗多少
     *
     * @param dx             水平方向嵌套滑动的子控件想要变化的距离 dx<0 向右滑动 dx>0 向左滑动
     * @param dy             垂直方向嵌套滑动的子控件想要变化的距离 dy<0 向下滑动 dy>0 向上滑动
     * @param consumed       子控件传给父控件数组，用于存储父控件水平与竖直方向上消耗的距离，consumed[0] 水平消耗的距离，consumed[1] 垂直消耗的距离
     * @param offsetInWindow 子控件在当前window的偏移量
     * @return 如果返回true, 表示父控件已经消耗了
     */
public boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed, @Nullable int[] offsetInWindow) {}


/**
     * 当父控件消耗事件后，子控件处理后，又继续将事件分发给父控件,由父控件判断是否消耗剩下的距离。
     *
     * @param dxConsumed     水平方向嵌套滑动的子控件滑动的距离(消耗的距离)
     * @param dyConsumed     垂直方向嵌套滑动的子控件滑动的距离(消耗的距离)
     * @param dxUnconsumed   水平方向嵌套滑动的子控件未滑动的距离(未消耗的距离)
     * @param dyUnconsumed   垂直方向嵌套滑动的子控件未滑动的距离(未消耗的距离)
     * @param offsetInWindow 子控件在当前window的偏移量
     * @return 如果返回true, 表示父控件又继续消耗了
     */
public boolean dispatchNestedScroll(int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow) {}

/**
     * 子控件停止嵌套滑动
     */
public void stopNestedScroll() {}


/**
     * 当子控件产生fling滑动时，判断父控件是否处拦截fling，如果父控件处理了fling，那子控件就没有办法处理fling了。
     *
     * @param velocityX 水平方向上的速度 velocityX > 0  向左滑动，反之向右滑动
     * @param velocityY 竖直方向上的速度 velocityY > 0  向上滑动，反之向下滑动
     * @return 如果返回true, 表示父控件拦截了fling
     */
public boolean dispatchNestedPreFling(float velocityX, float velocityY) {}

/**
     * 当父控件不拦截子控件的fling,那么子控件会调用该方法将fling，传给父控件进行处理
     *
     * @param velocityX 水平方向上的速度 velocityX > 0  向左滑动，反之向右滑动
     * @param velocityY 竖直方向上的速度 velocityY > 0  向上滑动，反之向下滑动
     * @param consumed  子控件是否可以消耗该fling，也可以说是子控件是否消耗掉了该fling
     * @return 父控件是否消耗了该fling
     */
public boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed) {}

/**
     * 设置当前子控件是否支持嵌套滑动，如果不支持，那么父控件是不能够响应嵌套滑动的
     *
     * @param enabled true 支持
     */
public void setNestedScrollingEnabled(boolean enabled) {}

/**
     * 当前子控件是否支持嵌套滑动
     */
public boolean isNestedScrollingEnabled() {}

/**
     * 判断当前子控件是否拥有嵌套滑动的父控件
     */
public boolean hasNestedScrollingParent() {}

```


### 谷歌嵌套滑动的方法调用设计

通过上文，我相信大家大概基本了解了NestedScrollingParent与NestedScrollingChild两个接口方法的作用，但是我们并不知道这些方法之间对应的关系与调用的时机。那么现在我们一起来分析谷歌对整个嵌套滑动过程的实现与设计。为了处理嵌套滑动，谷歌将整个过程分为了以下几个步骤：

- 1.当父控件不拦截事件，子控件收到滑动事件后，会先询问父控件是否支持嵌套滑动。
- 2.如果父控件支持嵌套滑动，那么父控件进行预先滑动。然后将处理剩余的距离交由给子控件处理。
- 3.子控件收到父控件剩余的滑动距离并滑动结束后，如果滑动距离还有剩余，又会再问一下父控件是否需要再继续消耗剩下的距离。
- 4.如果子控件产生了fling，会先询问父控件是否`预先拦截`fling。如果父控件预先拦截。则交由给父控件处理。`子控件则不处理fling`。
- 5.如果父控件不预先拦截fling, 那么会将fling传给父控件处理。同时子控件也会处理fling。
- 6.当整个嵌套滑动结束时，子控件通知父控件嵌套滑动结束。



### NestedScrollingChild 与 NestedScrollingParent 方法的对应关系

| 子View                  | 父View                                      | 方法描述                                                     | 步骤 |
| :---------------------- | :------------------------------------------ | :----------------------------------------------------------- | ---- |
| startNestedScroll       | onStartNestedScroll、onNestedScrollAccepted | Scrolling Child 开始滑动的时候，通知 Scrolling Parent 要开始滑动了，通常是在 Action_down 动作 的时候调用这个方法 | 1    |
| dispatchNestedPreScroll | onNestedPreScroll                           | 在 Scrolling Child 要开始滑动的时候，询问 Scrolling Parent 是否先于 Scrolling Child 进行相应的处理，同时是在 Action_move 的时候调用 | 2    |
| dispatchNestedScroll    | onNestedScroll                              | 在 Scrolling Child 滑动后会询问 Scrolling Parent 是否需要继续滑动 | 3    |
| dispatchNestedPreFling  | onNestedPreFling                            | 在 Scrolling Child 开始处理 Fling 动作的时候，询问 Scrolling Parent  是否需要先处理 Fling 动作 | 4    |
| dispatchNestedFling     | onNestedFling                               | 在 Scrolling Child  处理 Fling 动作完毕的时候，询问 Scrolling Parent 是都还需要进行相应的处理 | 5    |
| stopNestedScroll        | onStopNestedScroll                          | 在 Scrolling Child 停止滑动的时候，会调用 Scrolling Parent 的这个方法。通常是在 Action_up  或者 Action_cancel 或者被别的 View 消费 Touch 事件的时候调用的 | 6    |



## NestedScrollingParent2 接口介绍

```java
/**
    * 即将开始嵌套滑动，此时嵌套滑动尚未开始，由子控件的 startNestedScroll 方法调用
    *
    * @param child  嵌套滑动对应的父类的子类(因为嵌套滑动对于的父控件不一定是一级就能找到的，可能挑了两级父控件的父控件，child的辈分>=target)
    * @param target 具体嵌套滑动的那个子类
    * @param axes   嵌套滑动支持的滚动方向
    * @param type   嵌套滑动的类型，有两种ViewCompat.TYPE_NON_TOUCH fling效果,ViewCompat.TYPE_TOUCH 手势滑动
    * @return true 表示此父类开始接受嵌套滑动，只有true时候，才会执行下面的 onNestedScrollAccepted 等操作
    */
boolean onStartNestedScroll(@NonNull View child, @NonNull View target, @ScrollAxis int axes,  @NestedScrollType int type);

/**
    * 当onStartNestedScroll返回为true时，也就是父控件接受嵌套滑动时，该方法才会调用
    *
    * @param child
    * @param target
    * @param axes
    * @param type
    */
void onNestedScrollAccepted(@NonNull View child, @NonNull View target, @ScrollAxis int axes, @NestedScrollType int type);

/**
    * 在子控件开始滑动之前，会先调用父控件的此方法，由父控件先消耗一部分滑动距离，并且将消耗的距离存在consumed中，传递给子控件
    * 在嵌套滑动的子View未滑动之前
    * ，判断父view是否优先与子view处理(也就是父view可以先消耗，然后给子view消耗）
    *
    * @param target   具体嵌套滑动的那个子类
    * @param dx       水平方向嵌套滑动的子View想要变化的距离
    * @param dy       垂直方向嵌套滑动的子View想要变化的距离 dy<0向下滑动 dy>0 向上滑动
    * @param consumed 这个参数要我们在实现这个函数的时候指定，回头告诉子View当前父View消耗的距离
    *                 consumed[0] 水平消耗的距离，consumed[1] 垂直消耗的距离 好让子view做出相应的调整
    * @param type     滑动类型，ViewCompat.TYPE_NON_TOUCH fling效果,ViewCompat.TYPE_TOUCH 手势滑动
    */
void onNestedPreScroll(@NonNull View target, int dx, int dy, @NonNull int[] consumed,
                       @NestedScrollType int type);

/**
    * 在 onNestedPreScroll 中，父控件消耗一部分距离之后，剩余的再次给子控件，
    * 子控件消耗之后，如果还有剩余，则把剩余的再次还给父控件
    *
    * @param target       具体嵌套滑动的那个子类
    * @param dxConsumed   水平方向嵌套滑动的子控件滑动的距离(消耗的距离)
    * @param dyConsumed   垂直方向嵌套滑动的子控件滑动的距离(消耗的距离)
    * @param dxUnconsumed 水平方向嵌套滑动的子控件未滑动的距离(未消耗的距离)
    * @param dyUnconsumed 垂直方向嵌套滑动的子控件未滑动的距离(未消耗的距离)
    * @param type     滑动类型，ViewCompat.TYPE_NON_TOUCH fling效果,ViewCompat.TYPE_TOUCH 手势滑动
    */
void onNestedScroll(@NonNull View target, int dxConsumed, int dyConsumed,
                    int dxUnconsumed, int dyUnconsumed, @NestedScrollType int type);

/**
    * 停止滑动
    *
    * @param target
    * @param type     滑动类型，ViewCompat.TYPE_NON_TOUCH fling效果,ViewCompat.TYPE_TOUCH 手势滑动
    */
void onStopNestedScroll(@NonNull View target, @NestedScrollType int type);

```

## NestedScrollingChild2  接口介绍

```java
/**
    * 开始滑动前调用，在惯性滑动和触摸滑动前都会进行调用，此方法一般在 onInterceptTouchEvent或者onTouch中，通知父类方法开始滑动
    * 会调用父类方法的 onStartNestedScroll onNestedScrollAccepted 两个方法
    *
    * @param axes 滑动方向
    * @param type 开始滑动的类型 the type of input which cause this scroll event
    * @return 有父视图并且开始滑动，则返回true 实际上就是看parent的 onStartNestedScroll 方法
    */
boolean startNestedScroll(@ScrollAxis int axes, @NestedScrollType int type);

/**
    * 子控件停止滑动，例如手指抬起，惯性滑动结束
    *
    * @param type 停止滑动的类型 TYPE_TOUCH，TYPE_NON_TOUCH
    */
void stopNestedScroll(@NestedScrollType int type);

/**
    * 判断是否有父View 支持嵌套滑动
    */
boolean hasNestedScrollingParent(@NestedScrollType int type);

/**
    * 在dispatchNestedPreScroll 之后进行调用
    * 当滑动的距离父控件消耗后，父控件将剩余的距离再次交个子控件，
    * 子控件再次消耗部分距离后，又继续将剩余的距离分发给父控件,由父控件判断是否消耗剩下的距离。
    * 如果四个消耗的距离都是0，则表示没有神可以消耗的了，会直接返回false，否则会调用父控件的
    * onNestedScroll 方法，父控件继续消耗剩余的距离
    * 会调用父控件的
    *
    * @param dxConsumed     水平方向嵌套滑动的子控件滑动的距离(消耗的距离)    dx<0 向右滑动 dx>0 向左滑动 （保持和 RecycleView 一致）
    * @param dyConsumed     垂直方向嵌套滑动的子控件滑动的距离(消耗的距离)    dy<0 向下滑动 dy>0 向上滑动 （保持和 RecycleView 一致）
    * @param dxUnconsumed   水平方向嵌套滑动的子控件未滑动的距离(未消耗的距离)dx<0 向右滑动 dx>0 向左滑动 （保持和 RecycleView 一致）
    * @param dyUnconsumed   垂直方向嵌套滑动的子控件未滑动的距离(未消耗的距离)dy<0 向下滑动 dy>0 向上滑动 （保持和 RecycleView 一致）
    * @param offsetInWindow 子控件在当前window的偏移量
    * @return 如果返回true, 表示父控件又继续消耗了
    */
boolean dispatchNestedScroll(int dxConsumed, int dyConsumed,int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow, @NestedScrollType int type);

/**
    * 子控件在开始滑动前，通知父控件开始滑动，同时由父控件先消耗滑动时间
    * 在子View的onInterceptTouchEvent或者onTouch中，调用该方法通知父View滑动的距离
    * 最终会调用父view的 onNestedPreScroll 方法
    *
    * @param dx             水平方向嵌套滑动的子控件想要变化的距离 dx<0 向右滑动 dx>0 向左滑动 （保持和 RecycleView 一致）
    * @param dy             垂直方向嵌套滑动的子控件想要变化的距离 dy<0 向下滑动 dy>0 向上滑动 （保持和 RecycleView 一致）
    * @param consumed       父控件消耗的距离，父控件消耗完成之后，剩余的才会给子控件，子控件需要使用consumed来进行实际滑动距离的处理
    * @param offsetInWindow 子控件在当前window的偏移量
    * @param type           滑动类型，ViewCompat.TYPE_NON_TOUCH fling效果,ViewCompat.TYPE_TOUCH 手势滑动
    * @return true    表示父控件进行了滑动消耗，需要处理 consumed 的值，false表示父控件不对滑动距离进行消耗，可以不考虑consumed数据的处理，此时consumed中两个数据都应该为0
    */
boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed,
     @Nullable int[] offsetInWindow, @NestedScrollType int type);

```

