**Android setSystemUiVisibility详解**

-  SYSTEM_UI_FLAG_LOW_PROFILE 

- SYSTEM_UI_FLAG_HIDE_NAVIGATION 

- SYSTEM_UI_FLAG_FULLSCREEN 

- SYSTEM_UI_FLAG_LAYOUT_STABLE 
- SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION 

- SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN 
- SYSTEM_UI_FLAG_IMMERSIVE 
- SYSTEM_UI_FLAG_IMMERSIVE_STICKY

```java
SYSTEM_UI_FLAG_LOW_PROFILE  弱化状态栏和导航栏的图标
SYSTEM_UI_FLAG_HIDE_NAVIGATION 隐藏导航栏，用户点击屏幕会显示导航栏
SYSTEM_UI_FLAG_FULLSCREEN  隐藏状态栏
SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION  拓展布局到导航栏后面
SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN   拓展布局到状态栏后面
SYSTEM_UI_FLAG_LAYOUT_STABLE   稳定的布局，不会随系统栏的隐藏、显示而变化
SYSTEM_UI_FLAG_IMMERSIVE   沉浸模式，用户可以交互的界面
SYSTEM_UI_FLAG_IMMERSIVE_STICKY    沉浸模式，用户可以交互的界面。同时，用户上下拉系统栏时，会自动隐藏系统栏
```

```
View.SYSTEM_UI_FLAG_FULLSCREEN 隐藏状态栏，点击屏幕区域不会出现，需要从状态栏位置下拉才会出现。
```

```

 View.SYSTEM_UI_FLAG_HIDE_NAVIGATION 隐藏导航栏，点击屏幕任意区域，导航栏将重新出现，并且不会自动消失。

```

```
 View.SYSTEM_UI_FLAG_IMMERSIVE 使状态栏和导航栏真正的进入沉浸模式,即全屏模式，
 如果没有设置这个标志，设置全屏时，我们点击屏幕的任意位置，就会恢复为正常模式。
 所以，View.SYSTEM_UI_FLAG_IMMERSIVE 都是配合View.SYSTEM_UI_FLAG_FULLSCREEN和View.SYSTEM_UI_FLAG_HIDE_NAVIGATION一起使用的。
```

```
 View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY
它的效果跟View.SYSTEM_UI_FLAG_IMMERSIVE一样。但是，它在全屏模式下，
 用户上下拉状态栏或者导航栏时，这些系统栏只是以半透明的状态显示出来，并且在一定时间后会自动消息。
```