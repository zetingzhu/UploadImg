

# Activity
## Android设置背景图延伸到状态栏
1、在Activity onCreate方法中设置以下代码：

```

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
    //5.0 全透明实现
    //getWindow.setStatusBarColor(Color.TRANSPARENT)
    Window window = getWindow();
    window.clearFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
    window.getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN | View.SYSTEM_UI_FLAG_LAYOUT_STABLE);
    window.addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);
    window.setStatusBarColor(Color.TRANSPARENT);
} else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
    //4.4 全透明状态栏
    getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
}

```
​
2、记得layout布局文件的android:fitsSystemWindows="true"这个设置一定要去掉，要不没有效果



# Dialog

## dialog 横屏时候，软键盘会压缩布局，导致输入内容看不见
> 设置软键盘模式为SOFT_INPUT_ADJUST_PAN，但是这个只能在常规activity下正常，dialog默认还是会顶上去的，可以强制设置dialog的window高度为屏幕高度



# EditText

## 自定义背景色
```
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:bottom="0dp"
        android:left="-2dp"
        android:right="-2dp"
        android:top="-2dp">
        <shape>
            <solid android:color="@android:color/transparent" />
            <!--设置未选中输入框的底色-->
            <stroke
                android:width="1dp"
                android:color="#017DDF" />
            <padding android:bottom="4dp" />
        </shape>
    </item>

</layer-list>
```


# Resources
## 根据名字查找资源
``` java 
res = getResources().getIdentifier(name, "drawable", getPackageName());
```



# Flow
## constraintlayout 流布局学习

androidx.constraintlayout.core.widgets.Flow
