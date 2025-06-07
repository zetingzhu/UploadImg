**ViewPager2**

官网地址

https://developer.android.google.cn/training/animation/screen-slide-2

https://developer.android.google.cn/training/animation/vp2-migration

ViewPager2 比较 ViewPager Api 改变

1.    ViewPager2 自定义了RecycleView

2.    ViewPager2被声明成了final。

3.    FragmentStatePagerAdapter被FragmentStateAdapter 替代

4.    PagerAdapter被RecyclerView.Adapter替代

5.    addPageChangeListener被registerOnPageChangeCallback 替代

6.    并且 OnPageChangeCallback的抽象类

## ViewPager2 使用

准备工作

```java
dependencies {     
  implementation  'androidx.viewpager2:viewpager2:1.0.0-beta05'   
}  
```

布局

```java
<androidx.viewpager2.widget.ViewPager2       
  android:id="@+id/viewpager1"       
  android:layout_width="match_parent"       
  android:layout_height="0dp"     
  android:layout_weight="1"  />  
```

# ViewPager2显示View 使用RecyclerView.Adapter

```java
private void initviewViewpager2() {
    viewPager2 = findViewById(R.id.viewpager2);
    RecyclerView.Adapter adapter2 = new RecyclerView.Adapter<MyVH>() {
        @NonNull
        @Override
        public MyVH onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
View view = LayoutInflater.from(ActivityViewPager.this).inflate(R.layout.item_viewpager, parent, false);
            return new MyVH(view);;
        }
        @Override
        public void onBindViewHolder(@NonNull MyVH holder, int position) {}
        @Override
        public int getItemCount() { return 0;}
    };
    // 设置滚动方向
    viewPager2.setOrientation(ViewPager2.ORIENTATION_HORIZONTAL);
    viewPager2.setAdapter(adapter2);
}
```

onCreateViewHolder 中创建view方法

LayoutInflater.from(ActivityViewPager.this).inflate(R.layout.item_viewpager, parent, false);

错误一

LayoutInflater.from(ActivityViewPager.this).inflate(R.layout.item_viewpager, null);

<font color=red>java.lang.IllegalStateException: Pages must fill the whole ViewPager2 (use match_parent)</font>

错误二

LayoutInflater.from(ActivityViewPager.this).inflate(R.layout.item_viewpager, parent);

<font color=red>java.lang.IllegalStateException: ViewHolder views must not be attached when created.
 \* Ensure that you are not passing 'true' to the attachToRoot parameter of LayoutInflater.inflate(..., boolean attachToRoot)</font>

# 设置ViewPager2方向 ,默认为横向

{@link #ORIENTATION_HORIZONTAL} or {@link #ORIENTATION_VERTICAL}

public void setOrientation(@Orientation int orientation)

# TabLayout 和ViewPager2 关联
```java
dependencies {
   implementation "com.google.android.material:material:1.9.0"
}

TabLayoutMediator tabLayoutMediator = new TabLayoutMediator(tab_layout, viewPager2, new TabLayoutMediator.TabConfigurationStrategy() {
    @Override
    public void onConfigureTab(@NonNull TabLayout.Tab tab, int position) {
        tab.setText("Tab " + marray[position]);
    }
});
tabLayoutMediator.attach();
```

# ViewPager2 滑动监听
```
viewpager2.registerOnPageChangeCallback(new ViewPager2.OnPageChangeCallback() {
    @Override
    public void onPageSelected(int position) {
        super.onPageSelected(position);
        Log.d(TAG, ">1> viewpager2 onPageSelected position:" + position);
    }
});
```


# Viewpager2动态添加删除view

```java
switch (v.getId()) {
    case R.id.btn_add:
        int item = viewPager2.getCurrentItem();
        itemList.add(item, "添加：" + itemList.size());
        viewPager2.getAdapter().notifyItemInserted(item);
        break;
    case R.id.btn_del:
        int item1 = viewPager2.getCurrentItem();
        itemList.remove(item1);
        viewPager2.getAdapter().notifyItemRangeRemoved(item1, 1);
        break;
}
```

# ViewPager显示Fragment ，使用 FragmentStateAdapter

```java
private void initviewViewpager2() {
        viewPager2 = findViewById(R.id.viewpager2);
       FragmentStateAdapter adapter2 = new FragmentStateAdapter(ActivityViewPagerFragment.this){
            @NonNull
            @Override
            public Fragment createFragment(int position) {
                return fragmentList2.get(position);
            }
            @Override
            public int getItemCount() {  return 0;  }
        };
        viewPager2.setAdapter(adapter2);
    }
```

# ViewPager2 + Fragment 生命周期

ViewPager2 首次加载

![](https://gitee.com/ZeTing/UploadImg/raw/main/img/v1.png)

切换到ViewPager2 第3个下标

 ![](https://gitee.com/ZeTing/UploadImg/raw/main/img/v2.png)

切换到ViewPager2 第4个下标

 ![](https://gitee.com/ZeTing/UploadImg/raw/main/img/v3.png)

Viewpager2 默认会缓存2条item，

# Viewpager2 跳转 setCurrentItem

**public void setCurrentItem(int item)**

调用时候,Viewpager2 在滑动的时候会将中间跳过的所有item都预加载，超过2条的预加载又会再次释放掉, 预加载的时候

**RecyclerView.Adapter**

**public final void onBindViewHolder(final @NonNull FragmentViewHolder holder, int position)**

布局已经全部加载完成了，只是没有走onResume

# Viewpager2 支持离屏加载

**viewPager2.setOffscreenPageLimit(1)**

默认情况下，ViewPager2会缓存两条数据，所以滑动到第4页，第1页的Fragment才开始移除，

设置offscreenPageLimit=1时，ViewPager2在第1页会加载两条数据，会把下一页View提前加载进来；以后每滑一页，会加载下一页数组，直到第5页，会移除第1页的Fragment；第6页会移除第2页的Fragment



# ViewPager2 默认支持懒加载的，懒加载方式需要使用到

FragmentTransaction.setMaxLifecycle()

# ViewPager2动态添加Fragment

```java
switch (v.getId()) {
    case R.id.btn_add:
        int item = viewPager2.getCurrentItem();
itemList.add(item, new ItemData("添加Item:" + nextValue, nextValue));
nextValue++;
        viewPager2.getAdapter().notifyDataSetChanged();
        viewPager2.setCurrentItem(item, false);
        break;
    case R.id.btn_del:
        int item1 = viewPager2.getCurrentItem();
        if (itemList.size() > 0) {
            itemList.remove(item1);
            viewPager2.getAdapter().notifyDataSetChanged();
        }
        break;
}
FragmentStateAdapter adapter2 = new FragmentStateAdapter(ActivityViewPagerAdd.this) {
            ……

            @Override
            public long getItemId(int position) {
                /**这个地方，需要重写*/
                return itemList.get(position).getPage();
            }

……
        };
```

 这里说一下重写getItemid(int position) 遇到的问题

![](https://gitee.com/ZeTing/UploadImg/raw/main/img/v5.png)

 ![](https://gitee.com/ZeTing/UploadImg/raw/main/img/v6.png)

 ![](https://gitee.com/ZeTing/UploadImg/raw/main/img/v7.png)

 ![](https://gitee.com/ZeTing/UploadImg/raw/main/img/v8.png)

# Viewpager2 显示前后多个view

```java
/**设置显示viewpager 显示位置大小*/
int padding = (int) dpTopx(50);
RecyclerView recyclerView = (RecyclerView) viewPager2.getChildAt(0);
recyclerView.setPadding(padding, 0, padding, 0);
/**设置裁剪，并且有其他子类填充*/
recyclerView.setClipToPadding(false);

RecyclerView.Adapter adapter2 = new RecyclerView.Adapter<MyVH>() {
。。。。。。
    @Override
    public void onBindViewHolder(@NonNull MyVH holder, int position) {

        /**设置布局间距*/
        int padding = (int) dpTopx(10);
        RecyclerView.LayoutParams layoutParams = (RecyclerView.LayoutParams) holder.item_pager_bg.getLayoutParams();
        layoutParams.setMargins(padding, 0, padding, 0);
        holder.item_pager_bg.setLayoutParams(layoutParams);
    }
。。。。。。
};
```

#  Viewpager 禁止滑动

```java
viewPager2.setUserInputEnabled(false); //true:滑动，false：禁止滑动
```


# ViewPager2 给两个item设置间距
```
viewpager2.setPageTransformer(new MarginPageTransformer(30));
```

# 模拟拖拽
在使用fakeDragBy前需要先beginFakeDrag方法来开启模拟拖拽。fakeDragBy会返回一个boolean值，true表示有fake drag正在执行，而返回false表示当前没有fake drag在执行
```
向右拖拽
viewPager2.beginFakeDrag();
if (viewPager2.fakeDragBy(-200)) {
    viewPager2.endFakeDrag();
}
```
# 设置缓存数量
```
((RecyclerView) viewPager2.getChildAt(0)).setItemViewCacheSize(0);
```


# 添加动画
实现 ViewPager2.PageTransformer

```java
viewPager2.setPageTransformer(new DepthPageTransformer());
public class DepthPageTransformer implements ViewPager2.PageTransformer {
    private final static String TAG = DepthPageTransformer.class.getSimpleName();
    private static final float MIN_SCALE = 0.75f;
    public void transformPage(View view, float position) {
        Log.d(TAG, "动画 position:" + position);
        int pageWidth = view.getWidth();
        if (position < -1) { // [-Infinity,-1)
            // This page is way off-screen to the left.
            view.setAlpha(0f);
        } else if (position <= 0) { // [-1,0]
// Use the default slide transition when moving to the left page
            view.setAlpha(1f);
            view.setTranslationX(0f);
            view.setScaleX(1f);
            view.setScaleY(1f);
        } else if (position <= 1) { // (0,1]
            // Fade the page out.
            view.setAlpha(1 - position);
            // Counteract the default slide transition
            view.setTranslationX(pageWidth * -position);
            // Scale the page down (between MIN_SCALE and 1)
            float scaleFactor = MIN_SCALE
                    + (1 - MIN_SCALE) * (1 - Math.abs(position));
            view.setScaleX(scaleFactor);
            view.setScaleY(scaleFactor);
        } else { // (1,+Infinity]
            // This page is way off-screen to the right.
            view.setAlpha(0f);
        }
    }
}
```


#  ViewPager2问题

问题：Recycleview 如何实现 一次只滑动一下，并且没有惯性滑动？

问题：Recycleview 如何实现Fragment的生命周期管理？
