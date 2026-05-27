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



一、Java/Kotlin 基础（20题）
1. Java 中 == 和 equals() 的区别是什么？
答：
== 比较的是两个对象的引用是否相同（即内存地址是否一致）。
equals() 是方法，默认行为与 == 相同，但可以被重写以比较对象的内容（如 String 类重写了 equals() 来比较字符串内容）。
2. final 关键字的作用有哪些？
答：
修饰类：表示该类不能被继承。
修饰方法：表示该方法不能被子类重写。
修饰变量：表示该变量一旦赋值后不能被修改（常量）。
3. Java 中的四种引用类型是什么？
答：
强引用（StrongReference）：最常见的引用，只要强引用存在，GC 就不会回收对象。
软引用（SoftReference）：在内存不足时会被回收，常用于实现内存敏感的缓存。
弱引用（WeakReference）：无论内存是否充足，只要发生 GC 就会被回收。
虚引用（PhantomReference）：最弱的引用，无法通过它获取对象，主要用于跟踪对象被回收的状态。
4. String、StringBuilder 和 StringBuffer 的区别？
答：
String 是不可变的，每次操作都会生成新对象，效率低。
StringBuilder 是可变的，非线程安全，效率高。
StringBuffer 是可变的，线程安全（方法加了 synchronized），效率较低。
5. Java 中的异常体系是怎样的？
答：
所有异常都继承自 Throwable。
Error 表示 JVM 无法处理的严重错误（如 OutOfMemoryError）。
Exception 分为：
检查异常（Checked Exception）：编译期必须处理（如 IOException）。
非检查异常（Unchecked Exception）：运行时异常，继承自 RuntimeException（如 NullPointerException）。
6. Kotlin 中 val 和 var 的区别？
答：
val 声明只读变量（类似 Java 的 final），只能赋值一次。
var 声明可变变量，可以多次赋值。
7. Kotlin 中 ?.、!!、?: 操作符的作用？
答：
?.：安全调用操作符，如果对象为 null 则返回 null，不抛异常。
!!：强制非空断言，如果对象为 null 则抛出 NullPointerException。
?:：Elvis 操作符，如果左侧表达式为 null，则返回右侧默认值。
8. Kotlin 协程是什么？相比线程有什么优势？
答：
协程是轻量级的并发设计，基于挂起函数（suspend function）实现。
优势：
开销小：协程是用户态的，创建和切换成本远低于线程。
不阻塞线程：挂起时释放底层线程，可用于高并发场景。
代码简洁：避免回调地狱，用同步风格写异步代码。
9. HashMap 的工作原理？
答：
基于数组 + 链表/红黑树（JDK 8+）实现。
通过 key 的 hashCode() 计算哈希值，再通过扰动函数减少冲突。
当链表长度 ≥ 8 且数组长度 ≥ 64 时，链表转为红黑树。
扩容时容量翻倍，rehash 重新分配元素。
10. Java 中的泛型擦除是什么意思？
答：
泛型信息只在编译期存在，运行时会被擦除（类型变为 Object 或限定类型）。
目的是兼容 Java 5 之前的代码。
导致无法在运行时获取泛型的实际类型（可通过反射绕过）。
11. synchronized 和 ReentrantLock 的区别？
答：
synchronized 是 JVM 内置关键字，自动加锁/释放；ReentrantLock 是 API 级锁，需手动 lock/unlock。
ReentrantLock 支持公平锁、可中断、超时等待、多条件变量等高级特性。
synchronized 在 JDK 6 后经过优化（偏向锁、轻量级锁），性能已接近 ReentrantLock。
12. Kotlin 中的扩展函数是什么？
答：
允许在不修改类源码的情况下，为类添加新函数。
语法：fun ClassName.functionName() { ... }
编译后实际是静态工具方法，第一个参数是接收者对象。
13. Java 中的 volatile 关键字作用？
答：
保证变量的可见性：一个线程修改 volatile 变量后，其他线程能立即看到最新值。
禁止指令重排序（通过内存屏障实现）。
不保证原子性（如 i++ 仍需 synchronized 或 AtomicInteger）。
14. Kotlin 中 let、run、with、apply、also 的区别？
答：
let：obj.let { it -> ... }，返回 lambda 表达式结果，常用于空安全调用。
run：obj.run { ... }，返回 lambda 表达式结果，上下文是 obj 本身。
with：with(obj) { ... }，同 run，但不是扩展函数。
apply：obj.apply { ... }，返回 obj 本身，常用于对象配置。
also：obj.also { it -> ... }，返回 obj 本身，常用于日志或副作用。
15. Java 中的线程池核心参数有哪些？
答：
corePoolSize：核心线程数，即使空闲也不会被回收（除非 allowCoreThreadTimeOut）。
maximumPoolSize：最大线程数。
keepAliveTime：非核心线程空闲超时时间。
workQueue：任务队列（如 LinkedBlockingQueue）。
threadFactory：线程工厂。
handler：拒绝策略（如 AbortPolicy、CallerRunsPolicy）。
16. Kotlin 协程的三种启动模式（CoroutineStart）？
答：
DEFAULT：立即调度执行（可能挂起）。
LAZY：懒加载，需调用 start() 或 join() 才启动。
ATOMIC：类似 DEFAULT，但保证原子性（实验性）。
UNDISPATCHED：立即在当前线程执行，直到第一个挂起点。
17. Java 中的反射机制是什么？有什么优缺点？
答：
定义：在运行时动态获取类的信息并操作对象。
优点：灵活性高，实现通用框架（如 Retrofit、Gson）。
缺点：
性能开销大（比直接调用慢）。
破坏封装性，可能引发安全问题。
编译期无法检查，易出错。
18. Kotlin 中的密封类（Sealed Class）是什么？
答：
用于表示受限的类继承结构（子类数量固定且已知）。
所有子类必须嵌套在密封类内部或同一文件中。
常用于状态管理（如 Result.Success / Result.Error），配合 when 表达式实现 exhaustive check（穷尽检查）。
19. Java 中的 hashCode() 和 equals() 为什么需要一起重写？
答：
根据 Java 规范：如果两个对象 equals() 返回 true，则它们的 hashCode() 必须相同。
HashMap/HashSet 等集合依赖 hashCode() 定位桶，再用 equals() 比较元素。
若只重写 equals() 不重写 hashCode()，可能导致相同逻辑对象被存入不同桶，无法正确查找。
20. Kotlin 协程中的 Dispatchers 有哪些？
答：
Dispatchers.Main：Android 主线程，用于更新 UI。
Dispatchers.IO：专为 I/O 操作优化的线程池（如文件、网络）。
Dispatchers.Default：CPU 密集型任务的线程池（如计算、解析）。
Dispatchers.Unconfined：不指定线程，直接在调用者线程启动，直到第一个挂起点。
二、Android 基础组件（20题）
21. Activity 的生命周期有哪些？各方法的作用？
答：
onCreate()：初始化，加载布局。
onStart()：Activity 可见但不可交互。
onResume()：Activity 获得焦点，可交互。
onPause()：失去焦点（如弹出 Dialog），应暂停动画/传感器。
onStop()：完全不可见，应释放资源（如取消网络请求）。
onDestroy()：Activity 销毁，彻底清理。
onRestart()：从停止状态重新启动。
22. onStart() 和 onResume() 的区别？
答：
onStart() 表示 Activity 对用户可见（但可能被透明 Activity 覆盖）。
onResume() 表示 Activity 处于前台，可接收用户输入。
例如：弹出系统 Dialog 时，Activity 会 onPause() 但不会 onStop()。
23. Activity 的四种启动模式？
答：
standard：默认模式，每次启动都创建新实例。
singleTop：如果栈顶已是该 Activity，则复用（调用 onNewIntent()）。
singleTask：栈内唯一，启动时清除其上方所有 Activity。
singleInstance：独占一个任务栈，且栈中只有它自己。
24. Fragment 的生命周期与 Activity 的关系？
答：
Fragment 生命周期受宿主 Activity 控制。
关键回调：onAttach() → onCreate() → onCreateView() → onActivityCreated() → onStart() → onResume()。
当 Activity 进入后台，Fragment 也会依次调用 onPause()、onStop()。
25. 如何在 Fragment 中获取 Context？
答：
使用 requireContext()（推荐，非空）或 getActivity()（可能为 null）。
注意：在 onAttach() 之前或 onDetach() 之后调用会 crash。
26. Service 的两种启动方式及区别？
答：
startService()：启动服务，与调用者无关联，需主动 stopSelf() 或 stopService()。
bindService()：绑定服务，与调用者生命周期绑定，解绑时自动销毁。
可同时使用（既启动又绑定），此时需 unbind + stop 才销毁。
27. Intent 的显式和隐式调用区别？
答：
显式 Intent：指定目标组件（如 new Intent(this, TargetActivity.class)）。
隐式 Intent：通过 Action/Data/Category 描述意图，由系统匹配组件（如 ACTION_VIEW + URI）。
28. BroadcastReceiver 的注册方式及区别？
答：
静态注册：在 AndroidManifest.xml 中声明，应用未启动也能接收（如开机广播）。
动态注册：在代码中 registerReceiver()，随注册组件生命周期结束而失效。
Android 8.0+ 限制静态注册大部分隐式广播，需动态注册。
29. ContentProvider 的作用？
答：
提供跨应用数据共享的标准接口。
封装数据访问细节，通过 URI 区分数据类型（如 content://contacts/people）。
支持 CRUD 操作，底层可对接数据库、文件等。
30. Application 和 Activity 的 Context 区别？
答：
Application Context：生命周期与应用一致，适合全局操作（如启动 Service、Toast）。
Activity Context：生命周期与 Activity 一致，适合 UI 相关操作（如 inflate 布局、启动新 Activity）。
注意：避免在单例中持有 Activity Context，防止内存泄漏。
31. 如何保存 Activity 的临时状态？
答：
重写 onSaveInstanceState(Bundle outState)，将数据存入 Bundle。
在 onCreate() 或 onRestoreInstanceState() 中恢复。
适用于系统销毁重建（如横竖屏切换），不适用于主动 finish()。
32. Task 和 Back Stack 是什么？
答：
Task：一组相关 Activity 的集合，以栈（Back Stack）形式管理。
用户按 Back 键时，栈顶 Activity 出栈。
默认情况下，同一应用的 Activity 在同一 Task 中。
33. 如何退出整个应用？
答：
不推荐强行退出（违背 Android 设计理念）。
正确做法：finish 所有 Activity（可通过维护 Activity 栈或发送广播）。
极端情况：System.exit(0)（可能被系统杀死，不优雅）。
34. PendingIntent 是什么？和 Intent 有何不同？
答：
PendingIntent：封装 Intent 和目标组件，允许其他应用/系统以你的权限执行操作。
常用于 Notification、AlarmManager、Widget。
本质是“延迟的 Intent”，携带了授权令牌。
35. JobScheduler 的作用？
答：
在满足特定条件（如充电、联网）时执行后台任务。
替代早期的 AlarmManager + BroadcastReceiver 组合。
Android 5.0+ 推荐方案，更省电。
36. WorkManager 的优势？
答：
兼容 Android 5.0+（内部根据版本选择 JobScheduler/AlarmManager）。
保证任务最终执行（即使应用被杀）。
支持约束条件（网络、电量）、链式任务、唯一任务。
37. Android 中的进程优先级有哪些？
答（从高到低）：
前台进程（Foreground Process）：用户正在交互。
可见进程（Visible Process）：Activity 可见但不在前台。
服务进程（Service Process）：运行 startService() 的 Service。
后台进程（Background Process）：不可见 Activity。
空进程（Empty Process）：无任何组件，仅缓存。
38. 如何监听 Activity 的生命周期？
答：
实现 Application.ActivityLifecycleCallbacks 接口。
在 Application.onCreate() 中注册：registerActivityLifecycleCallbacks()。
可统一处理埋点、内存监控等。
39. Activity 之间如何传递数据？
答：
通过 Intent 的 extras（putExtra() / getExtra()）。
数据需实现 Serializable 或 Parcelable（后者效率更高）。
大数据（>1MB）避免传递，改用全局变量或数据库。
40. 如何启动一个没有在 Manifest 中注册的 Activity？
答：
不可能。Android 要求所有 Activity 必须在 Manifest 中声明，否则抛出 ActivityNotFoundException。
三、UI 与自定义 View（15题）
41. View 的绘制流程？
答：
Measure：确定 View 的宽高（onMeasure()）。
Layout：确定 View 在父容器中的位置（onLayout()）。
Draw：绘制内容（onDraw()），包括背景、子 View、前景等。
42. requestLayout()、invalidate()、postInvalidate() 的区别？
答：
requestLayout()：触发 Measure + Layout + Draw（整个流程）。
invalidate()：触发 Draw（仅重绘），必须在主线程调用。
postInvalidate()：同 invalidate()，但可在子线程调用（内部用 Handler 切回主线程）。
43. dp、sp、px 的区别？
答：
px：像素，绝对单位。
dp（density-independent pixel）：与屏幕密度无关的单位，1dp = 1px（在 160dpi 屏幕上）。
sp（scale-independent pixel）：类似 dp，但会随系统字体大小缩放，用于文字。
44. 如何优化 ListView/RecyclerView 的卡顿？
答：
ViewHolder 模式避免 findViewById()。
避免在 onBindViewHolder() 中做耗时操作。
图片加载使用 Glide/Picasso（带内存/磁盘缓存）。
布局层级扁平化（ConstraintLayout）。
Item 动画关闭或简化。
45. RecyclerView 的缓存机制？
答：
Scrap：屏幕内临时移除的 ViewHolder（如滑动时），可快速复用。
Cache：默认 2 个离屏 ViewHolder（mCachedViews），按 position 复用。
RecyclerPool：按 viewType 分组的共享池，跨 RecyclerView 复用。
ViewCacheExtension：开发者自定义缓存。
46. 自定义 View 时 onMeasure() 的注意事项？
答：
必须处理 MeasureSpec 的三种模式：
EXACTLY：精确值（如 match_parent、具体 dp）。
AT_MOST：最大值（如 wrap_content）。
UNSPECIFIED：无限制（如 ScrollView 中的子 View）。
调用 setMeasuredDimension() 设置最终尺寸。
47. onTouchEvent() 和 OnClickListener 的冲突如何解决？
答：
如果 onTouchEvent() 返回 true，会消费事件，OnClickListener 不触发。
解决方案：在 onTouchEvent() 中判断 ACTION_UP 时手动调用 performClick()。
48. SurfaceView 和 TextureView 的区别？
答：
SurfaceView：
独立 Surface，绘制在单独线程（不阻塞 UI 线程）。
有独立 Z 轴，不能做动画/截图。
适用于视频播放、游戏。
TextureView：
作为普通 View，可做动画、Transform。
绘制在 UI 线程，可能阻塞。
需硬件加速支持。
49. ConstraintLayout 的优势？
答：
扁平化布局，减少嵌套层级。
支持百分比、链（Chains）、Guideline 等高级约束。
性能优于 RelativeLayout。
50. Jetpack Compose 的核心思想？
答：
声明式 UI：描述“UI 应该是什么样”，而非“如何更新”。
基于 State 驱动：State 变化自动重组（Recomposition）。
无 XML，纯 Kotlin 代码构建 UI。
组合优于继承，函数即组件。
51. 如何实现一个圆形头像？
答：
方案1：自定义 View，重写 onDraw()，用 Canvas.clipPath() 裁剪。
方案2：使用 Glide 加载时 transform(CircleCrop())。
方案3：用 CardView + app:cardCornerRadius="50dp"。
52. View.post() 的作用？
答：
将 Runnable 投递到主线程消息队列尾部。
常用于在 View 初始化后获取宽高（因 onMeasure 未完成前 getWidth()=0）。
53. 如何检测 UI 卡顿？
答：
BlockCanary：监控主线程消息处理时间。
Systrace：分析帧渲染耗时。
Choreographer.FrameCallback：监听每一帧的绘制时间。
54. LayoutInflater 的 inflate() 方法中 attachToRoot 参数作用？
答：
true：将 root 作为 parent，并将 inflated View 添加到 parent。
false：仅解析 XML 生成 View，不添加到 parent（常用于 RecyclerView.ViewHolder）。
55. 如何实现 ViewPager 的无限轮播？
答：
虚拟法：Adapter.getCount() 返回 Integer.MAX_VALUE，position 取模映射真实数据。
首尾复制法：在数据首尾各加一个副本，滑动到副本时瞬间跳转到真实位置。
四、性能优化（15题）
56. 内存泄漏的常见原因及检测工具？
答：
原因：
静态变量持有 Activity/Context。
未注销广播/监听器。
单例持有 Context。
非静态内部类持有外部类引用（如 Handler）。
工具：
Android Studio Profiler。
LeakCanary（自动检测并提示）。
57. 如何避免 OOM？
答：
图片加载：使用 Glide（自动缩放、缓存），避免 BitmapFactory.decodeResource() 直接加载大图。
大对象复用：如 RecyclerView 的 ViewHolder。
内存缓存：LruCache 限制大小。
及时释放资源：Bitmap.recycle()（仅当确定不再使用时）。
58. ANR 的类型及原因？
答：
Input ANR：5 秒内未响应输入事件。
Broadcast ANR：前台广播 10 秒 / 后台广播 60 秒未处理完。
Service ANR：20 秒内未处理完 onStartCommand()。
ContentProvider ANR：10 秒内未响应查询。
根本原因：主线程被阻塞（如 I/O、复杂计算）。
59. 如何优化 APK 体积？
答：
代码：开启 ProGuard/R8 混淆、移除无用代码。
资源：压缩图片（WebP）、移除无用资源（shrinkResources true）。
So 库：只保留必要 ABI（如 arm64-v8a）。
动态交付：Play Feature Delivery 按需下载。
60. 启动优化的关键点？
答：
闪屏页：用 Theme 替代空白 Activity。
Application 初始化：
延迟初始化（非必要组件放到首页后）。
异步初始化（但注意线程安全）。
首页布局：简化 XML，避免过度绘制。
监控：通过 reportFullyDrawn() 测量启动时间。
61. 如何减少过度绘制（Overdraw）？
答：
移除不必要的背景（如 ViewGroup 和子 View 都设背景）。
使用 clipRect() 裁剪绘制区域。
开启 GPU Overdraw 调试（开发者选项）。
62. 冷启动 vs 热启动的区别？
答：
冷启动：应用进程不存在，需创建进程、Application、Activity。
热启动：应用进程存在，只需创建 Activity（走 onResume()）。
优化重点在冷启动。
63. StrictMode 的作用？
答：
开发阶段检测主线程 I/O、内存泄漏等违规操作。
通过 StrictMode.setThreadPolicy() / setVmPolicy() 配置。
注意：仅用于 Debug，Release 必须关闭。
64. 如何优化 RecyclerView 的滚动性能？
答：
setHasFixedSize(true)：Item 尺寸不变时启用。
setItemViewCacheSize()：增大离屏缓存。
DiffUtil：高效计算列表差异，避免全量刷新。
65. 网络请求优化策略？
答：
合并请求：减少 HTTP 请求数。
数据压缩：Gzip。
缓存策略：HTTP Cache-Control / 自定义内存/磁盘缓存。
协议升级：HTTP/2、QUIC。
66. 如何监控帧率（FPS）？
答：
Choreographer.FrameCallback：每帧回调，计算时间间隔。
GPU Rendering Profile：开发者选项中查看柱状图。
Perfetto：系统级性能分析工具。
67. 线程优化建议？
答：
避免频繁创建线程：使用线程池。
I/O 操作用 IO 线程池（如协程 Dispatchers.IO）。
CPU 密集型用 Default 线程池。
主线程只做 UI 更新。
68. SharedPreferences 的性能问题？
答：
apply() vs commit()：apply() 异步写入，commit() 同步阻塞。
ANR 风险：首次加载全量 XML 到内存，大数据量时卡顿。
替代方案：DataStore（基于 Kotlin 协程，异步、安全）。
69. 如何减少电池消耗？
答：
批量处理网络请求（JobScheduler/WorkManager）。
使用 FCM 替代轮询。
传感器使用后及时注销。
避免 WakeLock 长时间持有。
70. Memory Profiler 的使用场景？
答：
查看内存分配（Allocation Tracking）。
检测内存泄漏（Heap Dump + Analyzer）。
监控内存抖动（频繁 GC）。
五、架构与设计模式（15题）
71. MVC、MVP、MVVM 的区别？
答：
MVC：
Model：数据层。
View：UI 层（Activity/Fragment）。
Controller：Activity 兼任，耦合度高。
MVP：
Presenter：处理业务逻辑，隔离 View 和 Model。
View 通过接口与 Presenter 通信。
解决 Activity 胀肿问题。
MVVM：
ViewModel：持有 UI 相关数据，生命周期感知。
DataBinding/ViewBinding：自动同步数据到 UI。
更声明式，适合 Jetpack 生态。
72. Jetpack 组件有哪些？作用是什么？
答：
Lifecycle：管理组件生命周期。
LiveData：生命周期感知的数据持有者。
ViewModel：存储和管理 UI 相关数据。
Room：SQLite 对象映射库。
Navigation：单 Activity 多 Fragment 导航。
Paging：分页加载数据。
WorkManager：后台任务调度。
73. Repository 模式的优点？
答：
作为数据源的单一入口（本地 DB + 远程 API）。
解耦业务逻辑和数据获取细节。
便于单元测试（Mock Repository）。
74. 单例模式的双重检查锁定（DCL）实现？
答：
java

编辑



public class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
volatile 防止指令重排序导致未初始化完成就被访问。
75. LiveData 为什么不会内存泄漏？
答：
LiveData 与 LifecycleOwner（如 Activity）绑定。
当 LifecycleOwner 销毁时，自动移除 Observer。
内部通过 WeakReference + Lifecycle 状态判断实现。
76. ViewModel 的原理？
答：
通过 HolderFragment（已废弃）或 ViewModelStore（Activity/Fragment 内部）持有。
Configuration Change（如旋转）时，ViewModelStore 不会被销毁。
依赖 SavedStateHandle 保存临时状态。
77. 如何设计一个网络请求框架？
答：
分层：
接口层：Retrofit 定义 API。
数据层：Repository 整合本地/远程数据。
业务层：UseCase 封装逻辑。
功能：
统一错误处理。
缓存策略。
加载状态管理（Loading/Success/Error）。
78. 依赖注入（DI）的好处？
答：
解耦：对象创建与使用分离。
可测试：轻松替换依赖（Mock）。
灵活性：运行时切换实现。
Android 推荐：Hilt（基于 Dagger）。
79. 如何实现 EventBus？
答：
核心：观察者模式 + 注解处理器。
步骤：
注册：扫描 @Subscribe 注解的方法，存入 Map<EventType, List>。
发送：post(event) 时，根据 event 类型找到所有 Subscriber 并反射调用。
解注册：移除对应 Subscriber。
注意：线程切换、粘性事件、内存泄漏。
80. Clean Architecture 的分层？
答：
Presentation Layer：UI 相关（Activity/Fragment + ViewModel）。
Domain Layer：业务逻辑（UseCase + Entities）。
Data Layer：数据获取（Repository + DataSource）。
依赖方向：外层依赖内层，内层不知道外层。
81. 如何避免 Activity 胀肿？
答：
逻辑拆分到 ViewModel/Presenter。
使用 UseCase 封装业务规则。
工具类提取通用代码。
采用 MVVM/MVP 架构。
82. DataBinding 和 ViewBinding 的区别？
答：
DataBinding：
支持双向绑定、表达式。
编译期生成 Binding 类。
性能略低（反射）。
ViewBinding：
仅替代 findViewById()。
无额外性能开销。
更轻量，Google 推荐。
83. 如何设计一个图片加载框架？
答：
三级缓存：内存（LruCache）→ 磁盘（DiskLruCache）→ 网络。
生命周期感知：自动暂停/恢复加载。
变换：圆角、模糊等。
内存优化：Bitmap 复用（inBitmap）。
84. 什么是响应式编程？RxJava 的核心概念？
答：
响应式：数据流 + 变化传播。
RxJava 核心：
Observable：可观察数据流。
Observer：观察者。
Operator：操作符（map、filter、flatMap）。
Scheduler：线程调度。
85. 如何实现组件化？
答：
模块划分：业务模块（app、user、shop）+ 基础模块（base、network）。
通信：
Router：ARouter 实现页面跳转。
接口下沉：公共接口放在 base 模块。
事件总线：跨模块通信。
依赖：implementation project(':module')。
六、高级与系统原理（15题）
86. Handler 的工作原理？
答：
MessageQueue：存储 Message 的单链表（按时间排序）。
Looper：循环从 MQ 取 Message 并分发。
Handler：发送 Message 到 MQ，并处理 callback。
ThreadLocal：每个线程的 Looper 存储在线程局部变量中。
87. Looper.loop() 为什么不会阻塞主线程？
答：
主线程的 Looper.loop() 是应用的主循环，负责处理 UI 事件、输入等。
“阻塞”是指等待消息，而非死循环占用 CPU。
当无消息时，epoll_wait() 会休眠，不消耗 CPU。
88. Binder 机制的作用？
答：
Android IPC（进程间通信）的核心。
基于 mmap 实现高效内存共享。
通过 Binder 驱动，Client 调用 Server 服务（如 ActivityManagerService）。
89. AMS（ActivityManagerService）的作用？
答：
管理四大组件的生命周期。
负责 Task/Process 调度。
处理应用启动、切换、销毁。
90. Zygote 进程的作用？
答：
Android 应用进程的“孵化器”。
预加载常用类（如 Activity、View），通过 fork() 创建新进程，节省启动时间。
91. APK 安装过程？
答：
PackageManagerService 验证 APK 签名、完整性。
解析 AndroidManifest.xml，注册组件信息。
拷贝 APK 到 /data/app/。
生成 dex 优化文件（OAT）。
更新 Launcher 桌面图标。
92. Dalvik vs ART 的区别？
答：
Dalvik：
JIT（Just-In-Time）编译：运行时编译热点代码。
每次启动需解释字节码，性能较低。
ART：
AOT（Ahead-Of-Time）编译：安装时编译为机器码。
启动快、性能高，但占用更多存储空间。
Android 5.0+ 默认使用 ART。
93. 如何实现热修复？
答：
ClassLoader 方案（如 Tinker）：
修复 dex 插入到 DexPathList 前端。
重启生效。
Native Hook 方案（如 AndFix）：
替换方法指针（ArtMethod 结构体）。
无需重启，但兼容性差。
Instant Run 方案（已废弃）：基于 Split APK。
94. 插件化原理？
答：
核心：动态加载未安装的 APK。
关键技术：
Hook AMS：拦截 startActivity()，替换为 Stub Activity。
资源加载：AssetManager.addAssetPath()。
ClassLoader：DexClassLoader 加载插件类。
代表框架：RePlugin、VirtualAPK。
95. Android 的安全机制？
答：
沙箱：每个应用独立 UID，文件隔离。
权限模型：运行时权限（Android 6.0+）。
签名机制：APK 必须签名，更新需相同签名。
SELinux：内核级访问控制。
96. 如何调试 Native Crash？
答：
获取 tombstone 日志（/data/tombstones/）。
使用 addr2line + 符号表定位崩溃代码。
NDK 提供 crash handler（如 Breakpad）。
97. Systrace 的作用？
答：
系统级性能分析工具。
可视化 CPU、I/O、渲染、Binder 调用等。
通过 Trace.beginSection() 自定义标记。
98. 如何实现无侵入埋点？
答：
ASM 字节码插桩：编译期修改字节码，在 onClick() 等方法插入埋点代码。
AOP：AspectJ 切面编程。
Hook：反射替换 View.OnClickListener。
99. Android 的编译打包流程？
答：
AAPT2：编译资源 → R.java + resources.arsc。
Java Compiler：编译 Java → .class。
Kotlin Compiler：编译 Kotlin → .class。
D8/R8：.class → .dex（混淆、优化）。
打包：.dex + resources.arsc + so → APK。
签名：jarsigner 或 apksigner。
100. 如何保持技术成长？
答（开放题）：
阅读官方文档（Android Developers Blog）。
源码学习（AOSP、Jetpack）。
参与开源项目。
技术分享与总结。
关注新技术（Compose、Kotlin Multiplatform）。

1. OkHttp 的核心优势是什么？
答：
高效连接管理：支持连接池复用 TCP 连接，减少握手开销。
透明 GZIP 压缩：自动处理请求/响应的压缩与解压。
强大的拦截器机制：可自定义网络层行为（如日志、重试、认证）。
同步/异步 API：简洁的调用方式，异步基于回调或协程。
默认支持 HTTP/2：多路复用，减少延迟。
失败重试与重定向：自动处理常见网络问题。
2. OkHttp 的拦截器（Interceptor）有哪几种？区别是什么？
答：
Application Interceptor（应用拦截器）：
通过 addInterceptor() 添加。
位于整个拦截链的最外层，只调用一次。
可看到原始请求和最终响应（包括重定向、重试后的结果）。
适合添加公共参数、日志记录。
Network Interceptor（网络拦截器）：
通过 addNetworkInterceptor() 添加。
位于 ConnectInterceptor 和 CallServerInterceptor 之间。
每次网络请求都会调用（包括重试、重定向）。
可访问连接信息（如 IP、TLS 版本）。
适合监控网络质量、调试底层协议。
✅ 关键区别：应用拦截器关注“逻辑请求”，网络拦截器关注“物理请求”。
3. 请描述 OkHttp 的拦截器链工作流程。
答：
OkHttp 使用责任链模式，拦截器按顺序执行：
RetryAndFollowUpInterceptor：处理失败重试和重定向。
BridgeInterceptor：补充请求头（如 Content-Length、Host）、处理 GZIP。
CacheInterceptor：根据缓存策略返回缓存或发起网络请求。
ConnectInterceptor：建立连接（从连接池获取或新建）。
CallServerInterceptor：真正发送请求并读取响应。



2. 数据处理与渲染
高频刷新挑战： 健康数据（心率、血氧）通常是高频采集的。你是如何设计缓存策略和数据解析逻辑，以保证 UI 渲染（尤其是多形态图表）不掉帧、不卡顿的？

图表实现： 在高帧率渲染交互中，你使用的是原生 Canvas 绘制、第三方库（如 MPAndroidChart）还是自研引擎？针对大量历史数据点的渲染，做了哪些内存和计算优化？

3. 启动优化
Splash 冷启动优化： 你提到进行了“UI 渲染层级压扁”，具体操作是什么？（例如：Merge 标签、ConstraintLayout 优化、还是移除了冗余背景？）

协程并发： 在梳理三方库初始化时，哪些库是必须在主线程初始化的，哪些可以延迟或异步？如何处理三方库之间的依赖关系（例如 A 库初始化必须在 B 库之后）？

1. IM 核心技术
长连接与协议： 该政务级 IM 使用的是什么通讯协议（WebSocket, MQTT, 还是自定义 TCP）？如何处理弱网环境下的消息重发和心跳机制？

本地数据库同步： 在高频读写同步逻辑中，如何保证消息的顺序性和唯一性？Room 或 SQLite 在高并发写入时你是如何防止数据库锁定（Database is locked）的？

综合性通用问题（针对你的资历）
选型对比： 在第一个项目中你用了 Kotlin + 协程，第二个项目用了 RxJava。在现在的开发视角看，你会如何评价这两者的优劣？在什么场景下你仍会推荐使用 RxJava？

性能监控： 除了 LeakCanary，你平时如何监控线上 App 的 ANR 和 Crash 率？如果出现了一个偶发性的 OOM，你的排查思路是怎样的？

## 1. 设计理念：冷流 (Cold Stream) 与 热流 (Hot Stream)
面试题： 既然 RxJava 和 Flow 都可以处理异步数据流，它们在“冷/热流”的设计上有何异同？

参考答案：

共同点： 默认情况下，RxJava 的 Observable 和 Kotlin 的 Flow 都是冷流。即：如果没有订阅者（Subscriber / Collector），它们不会开始发射数据。

不同点：

RxJava 的热流通过 Subject 或 Processor 实现，或者通过 publish().connect() 转换。

Flow 的热流通过 SharedFlow 和 StateFlow 实现。StateFlow 特别适合 UI 状态管理，因为它天生自带“粘性”和“去重（distinctUntilChanged）”特性，且必须有初始值。

## 2. 线程调度与上下文切换
面试题： RxJava 使用 subscribeOn 和 observeOn，Flow 使用 flowOn。它们在切换线程时的底层机制有什么区别？

参考答案：

RxJava： 采用的是显式调度器 (Schedulers)。subscribeOn 影响上游，observeOn 影响下游。其底层是基于装饰器模式，每切换一次线程都会包装一层新的 Observable。

Flow： 采用的是协作式上下文 (Context Preservation)。flowOn 只影响其上游。

核心区别： Flow 遵循“上下文保存”原则，不允许在 flow { ... } 内部通过 withContext 切换线程，否则会抛出异常。必须使用 flowOn。这保证了 Flow 的执行环境是安全且可预测的。

## 3. 操作符与性能开销
面试题： 从性能和包体积角度看，为什么 Google 现在更推荐在 Android 中使用 Flow 替代 RxJava？

参考答案：

包体积： RxJava 是一个庞大的库，拥有数百个操作符；Flow 是 Kotlin 协程库的一部分，利用了 扩展函数 (Extension Functions)。这意味着你只会在编译后包含你用到的部分，且无需引入复杂的 RxJava 依赖。

内存开销： * RxJava 的每一个操作符都会创建一个新的对象，链路越长，内存抖动风险越大。

Flow 利用了 Kotlin 的 suspend 关键字。Flow 的操作符大多是 inline 或者是简单的 suspend 函数，避免了大量的类加载和对象创建。

## 4. 背压 (Backpressure) 处理机制
面试题： RxJava 专门有 Flowable 来处理背压，而 Flow 似乎没有类似类，它是如何处理生产速度大于消费速度的情况的？

参考答案：

RxJava： Flowable 使用响应式拉取（Reactive Pull）策略，通过 request(n) 来告诉上游下游能接收多少数据，实现较为复杂。

Flow： 天生支持背压。因为 Flow 的采集（collect）是一个 suspend 函数。

如果下游处理太慢，上游的 emit 会被挂起 (Suspend)，直到下游处理完并准备好接收下一个。

此外，Flow 提供了专门的缓冲操作符，如 .buffer()（异步缓冲）、.conflate()（丢弃旧数据只留最新）或 .collectLatest()（取消旧任务执行新任务）。

## 5. 生命周期感知 (Lifecycle Awareness)
面试题： 在 Android 中，Flow 如何比 RxJava 更安全地处理生命周期？

参考答案：

RxJava： 需要手动管理 Disposable，在 onCleared() 或 onDestroy() 中调用 dispose()，否则容易内存泄漏。

Flow： 配合 lifecycleScope 或 repeatOnLifecycle 使用。

尤其是 repeatOnLifecycle(Lifecycle.State.STARTED)，它能确保协程只在 UI 可见时运行，并在 UI 进入后台时自动销毁并重启，这比 RxJava 的手动管理要安全且简洁得多。
