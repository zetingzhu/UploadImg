Android Anim
# 属性动画
## ValueAnimator
ValueAnimator.ofInt: 传入一个int类型的开始值和结束值，构建动画对象。

valueAnimator.setDuration：设置动画时间

valueAnimator.setRepeatCount： 设置动画重复次数。

valueAnimator.setRepeatMode：设置动画重复模式， 正序重复或者是逆序重复。

valueAnimator.setStartDelay：设置动画开始后的延迟执行时间。

valueAnimator.addUpdateListener： 设置动画监听器，监听数值的变化，获取到当前的数值，然后去设置view的属性。这里设置的是view的x位置。效果如上面的gif图。

valueAnimator.start：启动动画




##  ObjectAnimator

ObjectAnimator 动画方法

```
ofArgb(Object target, String propertyName, int... values)
ofArgb(T target, Property<T, Integer> property, int... values)
ofFloat(Object target, String xPropertyName, String yPropertyName, Path path)
ofFloat(T target, Property<T, Float> property, float... values)
ofFloat(T target, Property<T, Float> xProperty, Property<T, Float> yProperty, Path path)
ofFloat(Object target, String propertyName, float... values)

ofInt(T target, Property<T, Integer> xProperty, Property<T, Integer> yProperty, Path path)
ofInt(T target, Property<T, Integer> property, int... values)
ofInt(Object target, String propertyName, int... values)
ofInt(Object target, String xPropertyName, String yPropertyName, Path path)

ofMultiFloat(Object target, String propertyName, float[][] values)
ofMultiFloat(Object target, String propertyName, Path path)
ofMultiFloat(Object target, String propertyName, TypeConverter<T, float[]> converter, TypeEvaluator<T> evaluator, T... values)
ofMultiInt(Object target, String propertyName, int[][] values)
ofMultiInt(Object target, String propertyName, Path path)
ofMultiInt(Object target, String propertyName, TypeConverter<T, int[]> converter, TypeEvaluator<T> evaluator, T... values)

ofObject(T target, Property<T, V> property, TypeEvaluator<V> evaluator, V... values)
ofObject(Object target, String propertyName, TypeConverter<PointF, ?> converter, Path path)
ofObject(T target, Property<T, V> property, TypeConverter<PointF, V> converter, Path path)
ofObject(T target, Property<T, P> property, TypeConverter<V, P> converter, TypeEvaluator<V> evaluator, V... values)
ofObject(Object target, String propertyName, TypeEvaluator evaluator, Object... values)

ofPropertyValuesHolder(Object target, PropertyValuesHolder... values)
```


## AnimatorSet
```
AnimatorSet.playTogether：将动画组合一起执行
AnimatorSet.playSequentially： 将动画组合有序执行
AnimatorSet.play：播放当前动画
AnimatorSet.after：将现有动画延迟x毫秒后执行
AnimatorSet.with(Animator：将现有动画和传入的动画同时执行
AnimatorSet.after：将现有动画插入到传入的动画之后执行
AnimatorSet.before：将现有动画插入到传入的动画之前执行
```

## TimeInterpolator 内置插值器
Android内置了 9 种内置的插值器实现：

|作用|资源ID|对应的Java类|
|:--------|:--------|:--------|
|动画加速进行|@android:anim/accelerate_interpolator|AccelerateInterpolator|
|快速完成动画，超出再回到结束样式|@android:anim/overshoot_interpolator|OvershootInterpolator|
|先加速再减速|@android:anim/accelerate_decelerate_interpolator|AccelerateDecelerateInterpolator|
|先退后再加速前进|@android:anim/anticipate_interpolator|AnticipateInterpolator|
|先退后再加速前进，超出终点后再回终点|@android:anim/anticipate_overshoot_interpolator|AnticipateOvershootInterpolator|
|最后阶段弹球效果|@android:anim/bounce_interpolator|BounceInterpolator|
|周期运动|@android:anim/cycle_interpolator|CycleInterpolator|
|减速|@android:anim/decelerate_interpolator|DecelerateInterpolator|
|匀速|@android:anim/linear_interpolator|LinearInterpolator|
