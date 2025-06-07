Android Lifecycle 几种用法

# 1、LifecycleObserver
OnLifecycleEvent 注解已经被官方标注过时

Deprecated
  This annotation required the usage of code generation or reflection, which should be avoided.
  Use DefaultLifecycleObserver or LifecycleEventObserver instead.
```
public class MLifecycleV1 implements LifecycleObserver {
    private static final String TAG = "Lifecycle  MLifecycleV1";

    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    public void create() {
        Log.d(TAG, "观察者调用了 Lifecycle.Event.ON_CREATE 方法");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    public void start() {
        Log.d(TAG, "观察者调用了 Lifecycle.Event.ON_START 方法");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void resume() {
        Log.d(TAG, "观察者调用了 Lifecycle.Event.ON_RESUME 方法");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void pause() {
        Log.d(TAG, "观察者调用了 Lifecycle.Event.ON_PAUSE 方法");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    public void stop() {
        Log.d(TAG, "观察者调用了 Lifecycle.Event.ON_STOP 方法");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    public void destry() {
        Log.d(TAG, "观察者调用了 Lifecycle.Event.ON_DESTROY 方法");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_ANY)
    public void anyMethod() {
        Log.d(TAG, "观察者调用了 Lifecycle.Event.ON_ANY 方法");
    }
}

// 1. 添加自己定义的监听
       getLifecycle().addObserver(new MLifecycleV1());
```

# 2、LifecycleEventObserver
```
public class MLifecycleV2 implements DefaultLifecycleObserver {
    private static final String TAG = "Lifecycle  MLifecycleV1";

    @Override
    public void onCreate(@NonNull LifecycleOwner owner) {
        Log.w(TAG, ">>>>>>> onCreate owner:" + owner);
    }

    @Override
    public void onStart(@NonNull LifecycleOwner owner) {
        Log.w(TAG, ">>>>>>> onStart owner:" + owner);

    }

    @Override
    public void onResume(@NonNull LifecycleOwner owner) {

        Log.w(TAG, ">>>>>>> onResume owner:" + owner);
    }

    @Override
    public void onPause(@NonNull LifecycleOwner owner) {

        Log.w(TAG, ">>>>>>> onPause owner:" + owner);
    }

    @Override
    public void onStop(@NonNull LifecycleOwner owner) {

        Log.w(TAG, ">>>>>>> onStop owner:" + owner);
    }

    @Override
    public void onDestroy(@NonNull LifecycleOwner owner) {

        Log.w(TAG, ">>>>>>> onDestroy owner:" + owner);
    }
}
// 2. DefaultLifecycleObserver
getLifecycle().addObserver(new MLifecycleV2());
```
# 3、LifecycleEventObserver

```
public class MLifecycleV3 implements LifecycleEventObserver {
    private static final String TAG = "Lifecycle  MLifecycleV3";

    @Override
    public void onStateChanged(@NonNull LifecycleOwner source, @NonNull Lifecycle.Event event) {
        Log.i(TAG, ">>>>>>> onStateChanged source:" + source + " event:" + event);
    }
}

// 3. LifecycleEventObserver
getLifecycle().addObserver(new MLifecycleV3());

```


# Fragment 的 setMaxLifecycle
setMaxLifecycle 可以直接控制 fragment 生命周期


setMaxLifecycl Lifecycle.State.CREATED
```
    public void setFragmentV1(FullscreenFragment cardFragment) {
        FragmentTransaction fragmentTransaction = getSupportFragmentManager().beginTransaction();
        fragmentTransaction.add(R.id.fl_content, cardFragment);
        fragmentTransaction.setMaxLifecycle(cardFragment, Lifecycle.State.CREATED);
        fragmentTransaction.commit();
    }
```
>  D/Lifecycle Fullscreen: >>>>>   onAttach  <br>
   D/Lifecycle Fullscreen: >>>>>   onCreate

setMaxLifecycl Lifecycle.State.CREATED
```
    public void setFragmentV2(FullscreenFragment cardFragment) {
        FragmentTransaction fragmentTransaction = getSupportFragmentManager().beginTransaction();
        fragmentTransaction.add(R.id.fl_content, cardFragment);
        fragmentTransaction.setMaxLifecycle(cardFragment, Lifecycle.State.STARTED);
        fragmentTransaction.commit();
    }
```
> D/Lifecycle  Fullscreen: >>>>>   onAttach  <br>
  D/Lifecycle  Fullscreen: >>>>>   onCreate  <br>
  D/Lifecycle  Fullscreen: >>>>>   onCreateView  <br>
  D/Lifecycle  Fullscreen: >>>>>   onViewCreated  <br>
  D/Lifecycle  Fullscreen: >>>>>   onStart


```
    public void setFragmentV3(FullscreenFragment cardFragment) {
        FragmentTransaction fragmentTransaction = getSupportFragmentManager().beginTransaction();
        fragmentTransaction.add(R.id.fl_content, cardFragment);
        fragmentTransaction.setMaxLifecycle(cardFragment, Lifecycle.State.RESUMED);
        fragmentTransaction.commit();
    }
```
>  D/Lifecycle  Fullscreen: >>>>>   onAttach  <br>
  D/Lifecycle  Fullscreen: >>>>>   onCreate      <br>
  D/Lifecycle  Fullscreen: >>>>>   onCreateView  <br>
  D/Lifecycle  Fullscreen: >>>>>   onViewCreated  <br>
  D/Lifecycle  Fullscreen: >>>>>   onStart  <br>
  D/Lifecycle  Fullscreen: >>>>>   onResume  <br>


# 全局生命周期监听
