# 为什么重写订阅逻辑
- 本地持仓计算因为行情来异常就轮询计算一次，导致内存抖动严重，优化内存抖动，放缓计算次数
- 之前行情订阅功能不太支持太快的行情发送，会导致 EventBus 内部堵塞
- 之前行情订阅会定期丢数据，导致部分订阅品种得不到最新行情


# 现在长连接订阅发送逻辑
1. 得到长连接订阅信息
2. 先把所有订阅消息统一放到缓存里面，然后 400ms 时间内保存这段时间的所有 code 去重复，
3. 统一发送所有 code 到订阅界面，
4. 订阅界面在轮询得到的 code 到缓存中取到最新的行情展示


# 之前订阅逻辑
1. 得到长连接订阅信息
2. 判断上次发送时间，超过100ms就发送当前得到产品，小号掉这次100内发送3次机会中的1次，将当前 code保存到队列中
3. 不在100ms 之内就，比较当前发送code是否在队列中，如果在队列中丢弃这次数据，
  不在队列中发送数据，消耗一次机会，后面类似，100ms 最多发送3次
4. 发送到页面，页面直接得到发送数据，修改最新行情


# 新发送逻辑和之前发送逻辑优势
1. 所有得到的code被统一发送，减少观察者轮次次数，可以控制一定时间的发送量，来控制发送速度
2. 所有行情最新数据都被保存在缓存中，其他地方如果有需要自取更方便

# 被改变的习惯
1. 之前多个品种同一批订阅模式，是一个一个单独修改，现在统一时间被改成了统一修改


新发送代码
```
public void writeOption(Optional optData) {
       if (optData == null) { return; }
       if (refreshUtilOpt != null) { refreshUtilOpt.stop();  }
       // 订阅持仓code
       String holdCodes = NettyClient.getInstance().getHoldCodes();
       // 订阅其他code
       String otherCodes = NettyClient.getInstance().getOtherCodes();
       // 产品code
       String productCode = optData.getProductCode();
       // 保存如缓存
       put(productCode, optData);
       // 当前时间
       long nowTime = System.currentTimeMillis();
       // 当前品种code保存起来
       nextCodeSet.add(productCode);
       // 400ms 发一次
       if (nowTime - lastPostTime > 400) {
           /**
            * 这里处理发送多条行情数据，分开处理订阅行情的只发送行情的，订阅持仓的只发送持仓的
            */
           mHoldMap = new CopyOnWriteArrayList<>();
           mSubsMap = new CopyOnWriteArrayList<>();
           for (String code : nextCodeSet) {
               Optional optional = get(code);
               if (optional != null) {
                   String pCode = optional.getProductCode();
                   if (StringUtil.isNotNull(holdCodes) && holdCodes.contains(pCode)) {
                       // 持仓
                       mHoldMap.add(pCode);
                   }

                   // 这两个可能出现相同的，同时塞入
                   if (StringUtil.isNotNull(otherCodes) && otherCodes.contains(pCode)) {
                       // 订阅
                       mSubsMap.add(pCode);
                   }
               }
           }
           // 清空发送的行情
           nextCodeSet.clear();
           if (Utils.isNotListEmpty(mHoldMap) || Utils.isNotListEmpty(mSubsMap)) {
               // 发送订阅行情code
               NettyResponse<Optional> response = new NettyResponse<>();
               response.setSuccess(true);
               response.setType(NettyUtil.TYPE_QP);
               response.setHoldList(mHoldMap);
               response.setSubsList(mSubsMap);
               notifyChangedMarket(new OptionalEvent(response));
               lastPostTime = nowTime;
           }
       }
   }
```
接收代码
```
WsOptionalObserver wsObserver = new WsOptionalLifecycleObserver("") {
  @Override public void sendHoldEvent(OptionalEvent optional) {
    madeOptionEvent(optional);

  }
};
WSCodeLruCacheUtil.getInstance().registerObserver(wsObserver)
```

老发送代码
```
// 先进先出队列
private static volatile CircularFifoQueue<String> eventPostQueue = new CircularFifoQueue<>(10);
// 额外比较发送数量,不可大于2，经过测试 100ms 内最多连发3条eventbus 超过等于4条就会导致延迟
private AtomicInteger atomicInt = new AtomicInteger(2);


public void postEvent(NettyResponse<Optional> response) {
       if (response == null) {
           return;
       }
       Optional optData = response.getData();
       if (optData == null) {
           return;
       }

       // 得到当前code
       String productCode = response.getData().getProductCode();
       long nowTime = System.currentTimeMillis();
       // 100 毫秒发一次，数据太多了 eventbus 收到都是好几秒之前的了
       if (nowTime - lastPostTime > 100) {
           EventBus.getDefault().post(new OptionalEvent(response));
           lastPostTime = nowTime;
           if (atomicInt != null) {
               atomicInt.set(2);
           }
           try {
               boolean offer = eventPostQueue.offer(productCode);
           } catch (Exception e) {
               e.printStackTrace();
           }
 } else {
           // 查看可修改数量
           if (atomicInt != null) {
              // 得到当前剩余数量比自减
              int andDecrement = atomicInt.getAndDecrement();
              // 100 ms 额外在发送一个
              if (andDecrement > 0) {
                   boolean contains = eventPostQueue.contains(productCode);
                   if (!contains) {
                       EventBus.getDefault().post(new OptionalEvent(response));
                       try {
                           eventPostQueue.offer(productCode);
                       } catch (Exception e) {
                           e.printStackTrace();
                       }
                   }
               }
           }
       }
   }
```


# 下面从日志，内存，演示效果
1. 新老版本订阅功能
2. 新老版本内存情况


# 比较订阅发送进化版，有没有更好的方案

# 项目遗留问题
1. 应用可用内存只有512M,本地计算引起的内存抖动还是存在，集思广益有没有更好方案


|||
|:|:|
<10       |  
<100      |   100
100-200   |   200
300       |   500



20以下的  实时刷新
20-50的   50ms
50-100   100ms
100-500  200ms
500+      300ms
