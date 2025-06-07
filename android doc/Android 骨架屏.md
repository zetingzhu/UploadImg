
通过View或者Adapter的替换来实现骨架屏是最普遍的方案，该方案需要单独为骨架屏页面进行布局，如果页面过多或者比较复杂，
写起来就还是蛮繁琐的。具体实现有
ShimmerRecyclerView{https://github.com/sharish/ShimmerRecyclerView}、
Skeleton{https://github.com/ethanhua/Skeleton}及
spruce-android{https://github.com/willowtreeapps/spruce-android}等开源库。
自定义一个View来对布局中的每个View进行一层包裹，当加载数据时则根据View来绘制骨架，否则显示正常UI。

由于该方案需要将每个View包裹一层，所以会增加额外的布局层次。具体实现有
Skeleton Android{https://github.com/rasoulmiri/Skeleton}等开源库。
