Android 题目了解


# Android 核心框架

## Activity 的生命周期中，onStart() 和 onResume() 有什么区别？

要点：onStart 时界面可见但不可交互；onResume 时界面获取焦点，用户可操作。

##  说说 Activity 的 onNewIntent() 方法调用场景及其与生命周期的关系。

简要答案：调用场景： 当 Activity 的启动模式设置为 singleTop 或 singleTask 时，
如果目标 Activity 已经在栈顶或栈中存在（且符合启动模式要求），系统不会重新创建实例，而是直接调用其已存在的实例的 onNewIntent() 方法。
生命周期关系：
调用路径是：onPause() -> onNewIntent() -> onResume()（对于 singleTop 在栈顶）或 onNewIntent()（对于 singleTask/singleTop，且没有被覆盖）。它在 onCreate() 和 onStart() 之后，表明组件本身是存活且可见的，只是 Intent 更新了。

## 请简述 Service 的两种启动方式及其生命周期区别。
详细讲解： Service 主要有两种启动模式，它们的生命周期路径完全不同：
startService()：
生命周期：onCreate() -> onStartCommand() -> Service Running -> onDestroy()。
特点：启动后与调用者（如 Activity）失去关联。即使调用者被销毁，Service 依然会在后台运行，直到手动调用 stopService() 或自身调用 stopSelf()。

bindService()：
生命周期：onCreate() -> onBind() -> Service Running -> onUnbind() -> onDestroy()。
特点：启动后与调用者绑定。调用者可以通过 IBinder 接口与 Service 进行交互。如果所有绑定者都断开了连接（Context 销毁），Service 会自动销毁。

## Service 和 Thread 有什么区别？能在 Service 里直接做耗时操作吗？
本质区别：
Thread：是 CPU 执行的最小单元，是用来执行异步任务的。
Service：是 Android 的一种组件，它并不等同于线程。Service 默认运行在主线程（UI 线程）中。
耗时操作：
绝对不能在 Service 中直接执行网络请求或复杂的计算。如果在主线程 Service 中执行超过 20 秒的操作，会导致 ANR (Application Not Responding)。
正确做法：在 Service 内部开启新线程，或使用 IntentService / WorkManager。


## 如果我先 startService 又 bindService，最后只执行 unbindService，Service 会销毁吗？

答： 不会。这种情况下 Service 处于“既启动又绑定”状态。必须同时满足“没有绑定者”且“被手动停止（stopService/stopSelf）”这两个条件，onDestroy() 才会执行。


## 请解释 Context 的不同类型及其适用场景。

要点：Application Context（全局）与 Activity Context（UI 相关），注意内存泄漏风险。

## Fragment 生命周期与 Activity 生命周期的关系是什么？


## Fragment 的生命周期：

```
onAttach()：Fragment 与 Activity 关联时调用。
onCreate()：创建 Fragment 时调用。
onCreateView()：创建 Fragment 的视图时调用。
onViewCreated()：Fragment 的视图创建完成时调用。
onActivityCreated()：Activity 的 onCreate 方法执行完后调用。
onStart()：Fragment 可见时调用。
onResume()：Fragment 获得用户焦点时调用。
onPause()：Fragment 暂停时调用。
onStop()：Fragment 不可见时调用。
onDestroyView()：Fragment 的视图销毁时调用。
onDestroy()：Fragment 销毁时调用。
onDetach()：Fragment 与 Activity 分离时调用。
```

## BroadcastReceiver 的注册方式有哪几种？各自的优缺点是什么？

要点：静态注册（常驻、Manifest）与动态注册（跟随生命周期）。


## Handler 运行流程：一趟数据的“快递”之旅
发送：当你调用 handler.sendMessage(msg) 时，它最终会调用 queue.enqueueMessage()。此时，消息会根据 when（执行时间）插到队列的合适位置。
排队：MessageQueue 里的消息按时间先后顺序排列，队头的消息总是最先要执行的。
轮询：Looper.loop() 在一个死循环中不断调用 queue.next()。
如果队列没消息，线程会进入阻塞/休眠状态（利用 Linux 的 epoll 机制），不占用 CPU。
如果有消息且时间到了，就取出消息。
分发：Looper 拿到消息后，通过 msg.target.dispatchMessage(msg) 把消息交给发送它的那个 Handler 去处理。

##  Handler 机制中的 Looper.loop() 是如何实现消息阻塞和唤醒的？

简要答案：消息阻塞： MessageQueue 的 next() 方法是核心。
当消息队列中没有待处理的消息时，next() 方法会调用 Linux epoll/poll 机制，使当前线程（Looper 所在线程）进入休眠状态，释放 CPU 资源。
消息唤醒： 当有新消息（或延迟消息到期）需要入队时，MessageQueue 会通过 管道/文件描述符（通常是一个 wake 机制）写入数据，唤醒处于休眠状态的 Looper 线程，
使其继续执行 next() 并取出消息。


## 什么是 IdleHandler？它的核心作用是什么？
IdleHandler 是 MessageQueue 上的一个接口，其核心作用是允许开发者在消息队列空闲时（即没有需要立即处理的普通消息或 Runnable 时）执行一些低优先级、非紧急的任务。
这有助于利用 CPU 闲置时间进行优化，同时不影响 UI 性能和用户体验。

MessageQueue 接口, 队列空闲, 低优先级任务, 性能优化

## IdleHandler 是如何被触发执行的？它与 Looper 和 MessageQueue 有何关系？
当 Looper 通过 MessageQueue.next() 方法尝试获取下一个消息时，如果发现：
① 消息队列中没有任何消息；或者
② 下一个消息的执行时间在未来（尚未到达），MessageQueue 就会被判断为处于空闲状态。
此时，Looper 会调用已注册的 IdleHandler 的 queueIdle() 方法。

MessageQueue.next(), Looper, 队列空闲判断, 消息在未来

## IdleHandler 最典型的应用场景有哪些？
1. 应用启动优化： 在应用主界面首次绘制完成后，执行一些非必要的初始化工作（如日志系统、次要组件的预加载）。
2. 延迟加载 (Lazy Loading)： 延迟加载一些不影响首屏显示的 UI 资源或数据。
3. 数据预取/缓存： 在用户进行其他操作时，悄悄地预取和缓存数据。
启动优化, 延迟初始化, 首次渲染后, 资源预加载

## 必须实现的 queueIdle() 方法中的布尔返回值 (true 或 false) 有何意义？
- 返回 true：表示该 IdleHandler 是一次性的。执行完 queueIdle() 后，它将从 MessageQueue 中自动移除，不会再次被调用。
- 返回 false：表示该 IdleHandler 是常驻的。执行完后，它会保留在队列中，等待下一次消息队列空闲时再次被调用。
true: 自动移除 (一次性), false: 保留 (常驻), 需手动移除

## 为什么说 IdleHandler 的执行是“安全”的，对 UI 性能影响小？
因为它是在主线程（UI 线程）的消息队列真正空闲时才被调用。如果队列中有新的消息进来，IdleHandler 会立即停止或推迟执行，从而确保 UI 渲染和用户输入事件拥有最高的优先级。
主线程, 优先级低, 确保 UI 渲染优先, 队列繁忙时推迟

## 如何手动添加和移除 IdleHandler？
- 添加： 通过 Looper.myQueue().addIdleHandler(IdleHandler)。
- 移除： 通过 Looper.myQueue().removeIdleHandler(IdleHandler)。
通常建议在 queueIdle() 方法中返回 true 来实现一次性任务的自动移除。
addIdleHandler, removeIdleHandler, Looper.myQueue()

## 什么是同步屏障 (Sync Barrier)？它对 IdleHandler 的执行有何影响？
这是 Handler 机制中的一个“特殊关卡”，也是面试中的高阶考点。

1. 什么是同步屏障？
同步屏障是一种特殊的 Message，它的 target 为空 (null)。一旦这个特殊的“屏障”被插入到消息队列，它就会起到“拦截器”的作用。

2. 它的作用：开启异步消息优先通道
当 Looper 在 next() 中遇到一个同步屏障时，它会暂时忽略队列中所有的同步消息，而是直接向后寻找标记为 isAsynchronous = true 的“异步消息”。

同步消息：普通发送的消息，此时会被卡在屏障后面，得不到执行。

异步消息：具有更高优先级的消息，可以越过屏障优先执行。

3. 为什么要设计它？
为了 UI 绘制的绝对优先。 当系统准备好绘制屏幕（VSYNC 信号到达）时，为了保证画面不卡顿，系统会向主线程 MessageQueue 插入一个同步屏障，并发送一个异步的“绘制消息”。这样即使队列里有一堆普通的点击事件、网络回调，它们也必须让路，让 UI 渲染先执行。




# 自定义 View
## 解释 View 的绘制流程，并说明 invalidate() 和 requestLayout() 的区别。

简要答案：绘制流程： 分为三个核心步骤：Measure (测量) > Layout (布局) > Draw (绘制)。
这个过程通常由 ViewRootImpl 协调。
invalidate()：
只影响 Draw 阶段。标记 View 需要重绘，但不会影响其大小和位置。系统会调用 onDraw()。用于颜色、文本、背景等视觉变化的更新。
requestLayout()：
影响 Measure 和 Layout 阶段，并最终导致 Draw。标记 View 及其父容器需要重新测量和布局。用于 View 的大小、位置发生根本性变化时。


## 自定义 View 时，处理复杂触摸事件（如滑动冲突）有哪些常用方法？
简要答案：滑动冲突产生原因： 多个 View（通常是嵌套的 ScrollView、RecyclerView 等）同时拦截了触摸事件，导致行为混乱。
解决方法（两种）：外部拦截法： 由父 View 在其 onInterceptTouchEvent() 方法中，判断是否需要拦截事件。
如果拦截，则由父 View 处理；否则，交给子 View 处理。
内部拦截法： 由父 View 不拦截任何事件，而是子 View 通过调用父 View 的 requestDisallowInterceptTouchEvent(boolean disallow) 方法，
主动告诉父 View 是否允许在后续的事件中进行拦截。



## 简述 Android 事件分发机制中的三个核心方法及其作用。
1. dispatchTouchEvent(): 负责事件的分发，是事件传递的入口。
2. onInterceptTouchEvent(): 在 ViewGroup 中，询问是否拦截当前事件。
3. onTouchEvent(): 负责事件的处理，由最终接收事件的 View 调用。
分发 (dispatch), 拦截 (intercept), 处理 (touch)


## Android中onTouch，onTouchEvent，onClick优先级
在 Android 中，onTouch、onTouchEvent和onClick的优先级从高到低依次是：onTouch、onTouchEvent、onClick。
onTouch方法会在触摸事件发生时首先被调用，如果在onTouch方法中返回true，表示该事件已经被处理完毕，不会再传递给onTouchEvent方法。
onTouchEvent在onTouch未处理或处理后返回false时被调用。
onClick是在触摸操作满足点击条件时触发，前提是前面的触摸处理没有消耗掉该事件。


## 事件分发的优先级顺序是怎样的？如何判断一个 ViewGroup 是否拦截事件？
优先级：Activity -> ViewGroup -> View。
在 ViewGroup 中，事件会先到达 dispatchTouchEvent，然后询问 onInterceptTouchEvent。
如果 onInterceptTouchEvent 返回 true，则表示拦截，事件将交给自己的 onTouchEvent 处理；返回 false 则将事件继续分发给子 View。
Activity > ViewGroup > View, onInterceptTouchEvent 返回值

## View 绘制流程的三个关键阶段是什么？分别负责什么？
1. Measure (测量): 确定 View 及其内容需要占据的尺寸 (onMeasure)。
2. Layout (布局): 确定 View 在父容器中的位置 (onLayout)。
3. Draw (绘制): 将 View 的内容（背景、内容、子 View 等）绘制到 Canvas 上 (onDraw)。
Measure (尺寸), Layout (位置), Draw (内容)


## View 和 ViewGroup 在绘制阶段的根本区别是什么？
View (叶子节点) 的主要职责是绘制自身的内容（如文本、图片、形状），通过重写 onDraw(Canvas) 实现。
ViewGroup (容器) 的主要职责是绘制它的所有子 View，通过调用 dispatchDraw(Canvas) 实现，它通常不绘制自身内容（除非设置了背景）。
View (绘制自身内容), ViewGroup (绘制子 View)

## 如果一个 ViewGroup 需要在所有子 View 都绘制完成后再绘制一些内容（例如子 View 之间的分割线），应该如何实现？
可以在重写 dispatchDraw(Canvas) 方法时，在调用 super.dispatchDraw(canvas) 之后（即子 View 已经绘制完毕）再执行自己的绘制逻辑（如绘制分割线）。	dispatchDraw 之后, 分割线

## Activity、ViewGroup、View事件分发的调用顺序是怎样的？
调用顺序：Activity.dispatchTouchEvent() →
ViewGroup.dispatchTouchEvent() → ViewGroup.onInterceptTouchEvent()（决定是否拦截）→
子 View 的 dispatchTouchEvent() → 子 View 的 onTouchEvent() → 若子 View 不消费，
回传父 View 的 onTouchEvent() → 最终回传 Activity.onTouchEvent()。

##  ViewGroup 如何拦截事件？如果想让 ViewGroup 拦截某个事件（比如长按），需要注意什么？
简要回答：
拦截逻辑：在 onInterceptTouchEvent() 中返回 true，则事件不再向下传递，转而触发自身的 onTouchEvent()。
注意事项：
若拦截 DOWN 事件，后续 MOVE/UP 事件会直接交给该 ViewGroup 处理（子 View 无法接收）；
若仅想拦截特定事件（如长按），需先放行 DOWN 事件，在 MOVE 或后续事件中判断条件后返回 true（避免误拦截初始事件）；
可通过 requestDisallowInterceptTouchEvent(true) 让子 View 禁止父 ViewGroup 拦截（如 ScrollView 与子 View 滑动冲突场景）。

##   View 的 onTouch()、onTouchEvent()、onClick() 优先级顺序是什么？为什么？
简要回答：
优先级：onTouch()（View.setOnTouchListener）> onTouchEvent() > onClick()。
原因：
dispatchTouchEvent() 中会先调用 onTouch()，若 onTouch() 返回 true（消费事件），则 onTouchEvent() 不执行；
onClick() 是在 onTouchEvent() 中 UP 事件触发时通过 PerformClick.run() 间接调用的，属于 onTouchEvent() 的后续逻辑。

## 自定义 View 时，为什么有时需要重写 onMeasure()？如果不重写，可能会出现什么问题？
简要回答：
重写原因：系统默认的 onMeasure() 仅支持 MATCH_PARENT 和 WRAP_CONTENT 等同于 MATCH_PARENT（对于 View 而言），若自定义 View 需支持 WRAP_CONTENT 并设置自身默认宽高，必须重写 onMeasure()。
不重写问题：当 View 的布局参数设为 WRAP_CONTENT 时，宽高会充满父容器（与 MATCH_PARENT 效果一致），无法实现 “包裹内容” 的预期效果。

## 如何获取 View 的正确宽高？
（因为测量是异步的）	可以在 onGlobalLayoutListener 中获取，
或者在 onMeasure 结束后通过 view.getMeasuredWidth() 获取。
也可以使用 post(Runnable) 延后获取。核心是确保在测量过程完成后才去读取。
onGlobalLayoutListener, post(Runnable), onMeasure 后


# RecycleView
## 如何实现一个高性能的自定义布局管理器（Custom LayoutManager）用于 RecyclerView？

简要答案：核心： 继承 RecyclerView.LayoutManager，并重写 onLayoutChildren()。
高性能关键点：复用机制： 使用 Recycler 接口管理 View 的回收和复用，这是性能的基石。
布局计算： 只计算屏幕可见区域内的 View，对于滑出屏幕的 View 要及时回收。
滑动： 重写 scrollVerticallyBy() 或 scrollHorizontallyBy()，计算位移量并调用 offsetChildrenVertical()/offsetChildrenHorizontal() 来移动已布局的 View，而不是重新布局 (Measure/Layout)。
性能工具： 尽量使用 Rect 和 SparseArray 等高效的数据结构进行计算和缓存。

## RecyclerView 的缓存机制是如何工作的？
RecyclerView 具有 四级缓存：
1. Scrap/Mian Cache (一级缓存)： 用于布局阶段，包含正在屏幕上或即将离开屏幕的 Item，无需重新 bind，速度最快。
2. Cache (二级缓存)： 默认 5 个，离开屏幕但未被回收的 Item，无需重新 bind，可直接复用。
3. ViewCacheExtension (三级缓存)： 开发者自定义缓存。
4. RecycledPool (四级缓存/缓存池)： 跨多个 RecyclerView 实例共享的缓存，View 是“脏”的，必须重新执行 onBindViewHolder() 才能复用。

##	如何提高 RecyclerView 的性能？
1. 设置 setHasFixedSize(true)： 如果 Item 大小固定，可避免不必要的布局计算。
2. 局部刷新： 使用 notifyItemChanged(position, payload) 进行 局部刷新，而不是使用 notifyDataSetChanged()。
3. DiffUtil： 使用 DiffUtil 计算最小变更集，提高列表更新效率和动画平滑度。
4. RecycledPool 优化： 如果有多个相同 ItemType 的列表，可以共享 RecycledPool。
5. 避免在 onBindViewHolder() 中创建对象或执行复杂计算。

## 什么时候使用 notifyItemChanged()？如何实现数据局部更新？
当 Item 的 内容 发生变化，但 Item 本身（id 或 type）不变时使用。实现局部更新：
1. 无 Payload： 调用 notifyItemChanged(pos)，会调用完整的 onBindViewHolder()。
2. 带 Payload (推荐)： 调用 notifyItemChanged(pos, payload)。这会触发 onBindViewHolder(holder, position, payloads)，您可以在这个方法中判断 payloads，只更新 Item 中变化的 部分视图，避免重绘整个 Item，达到真正的局部刷新。



# 性能优化与内存管理
## 在处理内存泄漏时，除了工具检测，你还会重点关注哪些常见的泄漏点？

简要答案：静态引用： 避免将 Activity、Context 或 View 实例赋值给静态变量。
非静态内部类/匿名内部类： 在 Activity 中定义的非静态 Handler 或匿名 AsyncTask 会隐式持有外部 Activity 的引用。
应使用 静态内部类 + WeakReference 解决。
未注销的监听器/回调： 如 BroadcastReceiver、EventBus 注册、SensorManager、LocationManager 等，需要在 onDestroy() 或 onStop() 中及时注销。
大的 Bitmap 对象： 未正确释放 Bitmap 占用的内存。


# 架构与设计模式
## 谈谈您对 Google 推荐的 MVVM + AAC 架构模式的理解，以及它如何解决传统 MVP 的痛点？

简要答案：MVVM 核心： 通过 数据驱动 (Data Binding/LiveData) 实现 View (Activity/Fragment) 和 ViewModel 的双向解耦。
View 只负责显示，ViewModel 负责业务逻辑和数据准备。
解决 MVP 痛点：View 与 Presenter 紧密耦合：

MVP 中 View 需要持有 Presenter 引用，导致双向依赖。MVVM 通过数据绑定，ViewModel 不直接持有 View 引用，降低耦合。
生命周期感知： MVP 需要手动管理 Presenter 的生命周期。AAC 的 ViewModel 和 LiveData 自动感知生命周期，避免了内存泄漏和手动销毁的复杂性。
测试难度： ViewModel 独立于 Android 框架，更容易进行单元测试。

# kotlin
## Kotlin 协程 (Coroutines) 相较于传统线程或 AsyncTask 有哪些主要优势？

简要答案：
轻量级 (Lightweight)： 协程在底层是基于线程池调度，但一个线程可以挂起和恢复成千上万个协程，极大地节省了系统资源（不需要为每个任务创建新线程）。
结构化并发 (Structured Concurrency)： 协程通过 CoroutineScope 和 Job 来管理生命周期。
当 Scope 被取消时，其下所有子协程都会被自动取消，避免了资源泄露和“僵尸”任务。避免回调地狱 (Callback Hell)：
使用 suspend 函数，可以像同步代码一样编写异步逻辑，极大地提高了代码的可读性和维护性。调度简单：
通过 Dispatchers 可以轻松地在不同线程之间切换（如 Main, IO, Default）。



## 为什么ArrayMap比HashMap更适合Android开发
### HashMap
HashMap的数据结构为数组加链表的结构，jdk1.8之后改为数组加链表加红黑树的结构
put的时候，会先计算key的hashcode,然后去数组中寻找这个hashcode的下标，如果数据为空就先resize,然后检查对应下标值(下标值=(数组长度-1)&hashcode)里面是否为空，空则生成一个entry插入，否就判断hascode与key值是否分别都相等，如果相等则覆盖，如果不等就发生哈希冲突，生成一个新的entry插入到链表后面，如果此时链表长度已经大于8且数组长度大于64，则先转成树，将entry添加到树里面
get的时候，也是先去查找数组对应下标值里面是否为空，如果不为空且key与hascode都相等，直接返回value,否就判断该节点是否为一个树节点，是就在树里面返回对应entry,否就去遍历整个链表，找出key值相等的entry并返回

### ArrayMap
内部维护两个数组，一个是int类型的数组(mHashes)保存key的hashcode,另一个是Object的数组(mArray)，用来保存与mHashes对应的key-value
put数据的时候，首先用二分查找法找出mHashes里面的下标index来存放hashcode,在mArray对应下标index<<1与(index<<1)+1的位置存放key与value
get数据的时候，同样也是用二分查找法找出与key值对应的下标index,接着再从mArray的(index<<1)+1位置将value取出

### 对比
HashMap在存放数据的时候，无论存放的量是多少，首先是会生成一个Entry对象，这个就比较浪费内存空间，而ArrayMap只是把数据插入到数组中，不用生成新的对象
存放大量数据的时候，ArrayMap性能上就不如HashMap,因为ArrayMap使用的是二分查找法找的下标，当数据多了下标值找起来时间就花的久，此外还需要将所有数据往后移再插入数据，而HashMap只要插入到链表或者树后面即可
所以这就是为什么，在没有那么大的数据量需求下，Android在性能角度上比较适合用ArrayMap

## SparseArray,HashMap , HashTable , ConcurrentHashMap , ArrayMap , LongSparseArray

线程安全性：HashTable 是线程安全的，方法都被 synchronized 修饰。HashMap 是非线程安全的。
空值处理：HashMap 允许键和值为 null 。HashTable 不允许键和值为 null 。
性能：由于 HashTable 的线程同步，在单线程环境下，HashMap 的性能通常优于 HashTable 。

##  为什么在 Android 中推荐使用 SparseArray 替代 HashMap<Integer, Object>?

内存占用：HashMap 存储每个 Entry 都需要一个额外的对象实例。而 SparseArray 内部使用原生的 int[] 和 Object[]，省去了 Entry 对象的开销。
避免自动装箱：HashMap<Integer, V> 在操作时会将 int 包装成 Integer 对象。在大量数据下，这会产生大量临时对象，触发频繁的 GC。SparseArray 直接处理 int。


# 解释依赖注入 (DI) 的原理和 Hilt 在 Android 中的优势。
简要答案：DI 原理： 依赖注入是一种设计模式，其核心思想是反转控制 (IoC)：不是由组件自己创建其依赖，而是由外部容器（DI 框架）在运行时提供所需的依赖。这提升了模块的解耦性、可测试性和可维护性。
Hilt 优势： Hilt 是基于 Dagger 的 Android 官方推荐库。简化配置： 自动集成了 Android 框架类（Application、Activity、Fragment 等），减少了手动编写样板代码。生命周期感知： 提供了内置的作用域 (Scope)，如 @ActivityScoped，确保依赖的生命周期与对应的 Android 组件生命周期一致。编译期安全： 继承了 Dagger 的优点，在编译期检查依赖图的完整性和正确性。


# JetPack 全家桶核心组件（Lifecycle、ViewModel、LiveData、Room、Navigation、WorkManager、DataBinding 等）

## 面试题：JetPack 的核心定位和主要作用是什么？它解决了 Android 开发中的哪些痛点？
简要回答：
定位：Google 推出的 Android 官方组件库，旨在帮助开发者快速构建稳定、高效、易维护的 Android 应用，遵循 “最佳实践”，兼容低版本系统。
核心作用：统一架构规范、简化重复工作、解决生命周期管理、数据持久化、组件通信等痛点；
解决痛点：① 生命周期管理混乱（如 Activity 销毁前数据泄露）；② 组件间通信复杂；③ 数据持久化繁琐；④ 导航逻辑分散；⑤ 后台任务管理复杂（如适配 Doze 模式）。

## 面试题：DataStore 的核心作用是什么？它和 SharedPreferences 相比有哪些改进？
简要回答：
核心作用：轻量级数据存储（替代 SharedPreferences），用于存储少量键值对（如用户配置、token）或 protobuf 结构化数据。
相比 SharedPreferences 的改进：① 异步操作（避免主线程阻塞，SharedPreferences 同步操作易 ANR）；② 类型安全（支持 Kotlin 协程、Flow，无需手动转换数据类型）；③ 无编辑器提交问题（SharedPreferences.Editor 需 commit ()/apply ()，易出现数据不一致）；④ 支持观察数据变化（Flow 监听）。


##  ViewModel 数据粘连？它通常发生在哪些场景下？
常见场景：
单 Activity 多 Fragment 场景：多个 Fragment 共享同一 Activity 宿主的 ViewModel，切换 Fragment 时旧数据未清空；
页面复用场景：如 ViewPager2 预加载 Fragment、导航组件（Navigation）跳转后返回，ViewModel 未重置数据；
配置变化异常场景：如非配置变化（如手动 finish 后重建 Activity）本应销毁 ViewModel，却因错误用法导致数据残留。
二、核心原因：为什么会出现


## 解释 Android 中启动优化（冷启动）的主要思路和措施。

简要答案：冷启动流程： 启动 App $\rightarrow$
创建 Process $\rightarrow$ Application.onCreate() $\rightarrow$
创建 Main Activity $\rightarrow$ 布局绘制 $\rightarrow$ 首次渲染。优化思路（三板斧）：延迟初始化： 将非核心、非必要的初始化工作延迟到首次渲染后或子线程进行（主线程任务分解）。
异步初始化： 使用 Thread 或 协程 在子线程并行初始化任务。多进程预加载： 使用一个独立进程提前进行耗时初始化或数据预加载（较少使用，成本较高）。
具体措施： 优化 Application.onCreate() 耗时、减少布局层级、使用 ViewStub 延迟加载非必需 View。



## 如何优化Android内存使用？
优化数据结构：使用更高效的数据结构，减少内存使用。
避免内存抖动：减少短时间内大量对象的创建和销毁，避免频繁的垃圾回收。
优化图片加载：使用Glide或Coil等库进行图片加载和缓存。
避免内存泄漏：确保及时释放不再使用的对象和资源，使用LeakCanary等工具检测内存泄漏，并修复。
使用内存缓存：如 LRUCache ，合理使用内存缓存来提高性能。
减少内存分配：避免在主线程进行大量的内存分配。
对象复用：对于频繁创建和销毁的对象，使用对象池进行复用。
使用ProGuard或R8：移除无用的代码和资源，减少应用体积。
合理使用线程：避免创建过多的线程，使用线程池来管理线程。


## LRUCache
### 什么是 LRU 算法？ LRU 算法的核心思想
核心思想： LRU 是一种内存管理算法。它的逻辑是：如果数据最近被访问过，那么将来被访问的几率也更高。当缓存空间满时，优先淘汰最长时间未被访问的数据。


## 如何检测和优化布局嵌套带来的过度绘制（Overdraw）？

## 常见的内存泄漏场景有哪些？如何使用 LeakCanary 定位问题？

要点：匿名内部类、静态变量、未关闭的资源、单例持有 Context。

## 如何进行 App 启动优化（冷启动与热启动）？

## 图片加载框架（如 Glide）是如何做三级缓存和内存管理的？

## ANR（Application Not Responding）产生的原因及排查思路？
关键：主线程耗时操作、I/O 等待、锁竞争。

## 什么是 App Startup 库？它解决了什么问题？


## kotlin的特点还有看法
1. 空安全 (Null Safety) —— 核心杀手锏
这是 Kotlin 最著名的特性。它在编译期就区分了“可为空引用”和“不可为空引用”，从而在源头上极大地减少了 NullPointerException (NPE)。

2. 简洁性与表现力 (Conciseness)
Kotlin 能用更少的代码实现更多的逻辑，减少了 Java 中常见的“样板代码”（Boilerplate Code）：
Data Classes：一行代码搞定 equals(), hashCode(), toString() 和 copy()。
扩展函数 (Extension Functions)：无需继承类就能给现有类增加新方法（如给 View 增加一个 show() 方法）。
Lambda 表达式 & 高阶函数：让集合操作（filter, map, flatMap）像写英语一样顺滑。

3. 互操作性 (Interoperability)
Kotlin 与 Java 是 100% 互操作的。
你可以在同一个项目中混合编写 Java 和 Kotlin。
Kotlin 可以无缝调用所有现有的 Java 库，反之亦然。这让存量项目的迁移变得毫无压力。

4. 协程 (Coroutines) —— 轻量级并发
这是 Kotlin 处理异步任务的终极方案。
非阻塞：用同步的写法写异步代码，避免了“回调地狱”。
资源占用极低：一个线程可以运行数千个协程，远比传统的线程切换高效。


## 协程的启动方法
1. launch：开启任务，不关心结果
launch 是最常用的启动方式，属于 “发射后不管”（Fire and Forget） 模式。
返回值：返回一个 Job 对象。你可以用它来取消协程。
特性：非阻塞。它会立即启动协程并继续执行后面的代码。

2. async：开启任务，并等待结果
如果你需要协程执行完后给回一个结果，就要用 async。
返回值：返回一个 Deferred<T>（类似于 Java 的 Future）。
特性：非阻塞。但你可以通过调用 .await() 来挂起当前协程，直到拿到结果。
3. runBlocking：阻塞当前线程（慎用）
runBlocking 会连接“非协程世界”和“协程世界”。
特性：阻塞。它会卡住当前线程，直到内部所有的协程逻辑执行完毕。

## 内存泄漏的排查优化

## 线上anr如何排查及处理


## 自定义view

## view的绘制流程及几个方法




## RxJava
### 1: 什么是 RxJava？它与普通的观察者模式有什么区别？

答案要点： RxJava 是响应式扩展（Reactive Extensions）的 Java 实现。它不仅是观察者模式，还融合了迭代器模式和函数式编程思想。

区别： 普通观察者模式通常是同步的，且没有便捷的方式处理序列结束或错误。RxJava 提供了丰富的操作符、线程切换控制以及对**事件序列完成（onComplete）和异常（onError）**的显式支持。

### 2: 请简述 RxJava 的观察者模式流程。

答案要点： 1. Observable (被观察者)：产生事件。 2. Observer (观察者)：接收事件并做出响应。 3. Subscribe (订阅)：连接起被观察者和观察者。 4. Disposable (可丢弃资源)：用于取消订阅，防止内存泄漏。

### 3: map 和 flatMap 操作符有什么区别？
答案要点：
map：一对一转换。将事件序列中的元素 T 转换为元素 R。
flatMap：一对多转换。将元素 T 转换成一个 Observable<R>，然后将这些 Observables “拍扁”合并成一个单一的序列。
关键点：flatMap 不保证元素的原始顺序，如果需要保证顺序，应使用 concatMap。

### 4: RxJava 如何实现线程切换？常用的 Scheduler 有哪些？
答案要点： 通过 subscribeOn() 和 observeOn()。
subscribeOn()：指定 Observable 创建和执行耗时操作的线程（通常只生效第一次调用）。
observeOn()：指定下游 Observer 接收回调的线程（可多次切换）。

常见调度器：
Schedulers.io()：用于非 CPU 密集型操作（网络请求、读写文件）。
Schedulers.computation()：用于计算密集型逻辑。
AndroidSchedulers.mainThread()：Android 主线程。

### 5: 什么是背压（Backpressure）？RxJava 2/3 是如何解决的？

答案要点： 当被观察者发送事件的速度远大于观察者处理事件的速度时，会导致内存溢出（OOM）。
解决方案： 引入了 Flowable 类。
Flowable 支持背压策略（如 BackpressureStrategy.DROP 丢弃、BUFFER 缓冲、LATEST 只留最新的）。

它通过 Subscription.request(n) 机制，让观察者主动告知被观察者“我还能处理多少数据”。

### 6: 如何防止 RxJava 导致的内存泄漏？
答案要点：
在生命周期结束时（如 Activity 的 onDestroy）调用 Disposable.dispose()。
使用 CompositeDisposable 统一管理多个订阅。
在 Android 中可以使用封装好的库（如 AutoDispose）。

### RxJava 和协程（Coroutines）的区别，
RxJava 强在流式处理和极其丰富的操作符，而协程在处理简单的串行异步逻辑时更轻量、代码更像同步代码。
