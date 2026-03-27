Adnroid 屏幕适配

今日头条屏幕适配方案

```
private static float aNoncompatDensity;
private static float aNoncompatScaledDensity;

private static void setCustomDensity(Application application, Activity activity) {
    final DisplayMetrics appDisplayMetrics = application.getResources().getDisplayMetrics();
    if (aNoncompatDensity == 0) {
        aNoncompatDensity = appDisplayMetrics.density;
        aNoncompatScaledDensity = appDisplayMetrics.scaledDensity;
        application.registerComponentCallbacks(new ComponentCallbacks() {
            @Override
            public void onConfigurationChanged(@NonNull Configuration newConfig) {
                if (newConfig.fontScale > 0) {
                    aNoncompatScaledDensity = application.getResources().getDisplayMetrics().scaledDensity;
                }
            }

            @Override
            public void onLowMemory() {

            }
        });
    }

    final float targetDensity = appDisplayMetrics.widthPixels / 360.0;
    final float targetScaledDensity = targetDensity * (aNoncompatScaledDensity/ aNoncompatDensity);
    final int targetDensityDpi = (int)(targetDensity * 160);

    //设置application的Density
    appDisplayMetrics.density = targetDensity;
    appDisplayMetrics.scaledDensity = targetScaledDensity;
    appDisplayMetrics.densityDpi = targetDensityDpi;

    //设置activity的Density
    final DisplayMetrics activityDisplayMetrics = activity.getResources().getDisplayMetrics();
    activityDisplayMetrics.density = targetDensity;
    activityDisplayMetrics.scaledDensity = targetScaledDensity;
    activityDisplayMetrics.densityDpi = targetDensityDpi;
}
```
