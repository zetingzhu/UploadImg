**Android Canvas**

# paint

# Paint常用设置

## Paint常用的设置函数：

 setAlpha()透明度
 setAntiAlias()抗锯齿
 setColor(),setARGB()设置颜色
 setStyle(Paint.Style style) 设置填充样式
 setStrokeCap(Paint.Cap cap) 画笔的样式（落笔，收笔时）
 setStrokeJoin(Paint.Join join)连接点的样式
 setStrokeWidth(float width)设置画笔宽度
 setShadowLayer(float radius, float dx, float dy, int shadowColor) 设置阴影
 setTextSize(float textSize) 字体大小
 setTextAlign(Paint.Align.RIGHT)设置字体对齐方式
 setColorFilter(ColorFilter filter) 设置颜色过滤
 setUnderlineText(true) 下划线
 setPathEffect() 设置路径效果
 setTypeface() 设置字体风格
 setFilterBitmap() 设置图片过滤
 setXfermode(Xfermode xfermode) xfermode设置图像混合模式
 setShader(Shader shader) 设置shader包括渐变shader，图片shader

### setStyle(Paint.Style style) 设置填充样式

```java
setStyle设置填充样式，所谓填充的样式指只绘制线或者绘制同时填充：
Paint.Style.FILL 填充内部，会把闭合区域填充颜色
Paint.Style.FILL_AND_STROKE 填充内部和描边
Paint.Style.STROKE 仅描边，仅仅绘制边界
默认FILL 填充内部，
         paint = new Paint(Paint.ANTI_ALIAS_FLAG);
        paint.setStrokeWidth(50);//设置画笔宽度 ，单位px
        paint.setColor(Color.BLUE);
 private void drawCircle(Canvas canvas) {
        paint.setColor(Color.BLUE);
        paint.setStyle(Paint.Style.FILL);
        paint.setAlpha(100);
        canvas.drawCircle(300, 300, 200, paint);

        paint.setStyle(Paint.Style.STROKE);
        paint.setAlpha(100);
        canvas.drawCircle(800, 300, 200, paint);

        paint.setStyle(Paint.Style.FILL_AND_STROKE);
        paint.setAlpha(100);
        canvas.drawCircle(300, 800, 200, paint);


        paint.setColor(Color.RED);
        paint.setStyle(Paint.Style.FILL);
        canvas.drawCircle(800, 800, 200, paint);

        paint.setColor(Color.BLUE);
        paint.setStyle(Paint.Style.STROKE);
        paint.setAlpha(100);
        canvas.drawCircle(800, 800, 200, paint);
    }
```

![image-20210802135808640](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210802135808640.png)

### `setTextAlign(Align align)` 设置对齐方式

```java
Paint.Align.LEFT` 左对齐`
Paint.Align.CENTER` 中心对齐，绘制从
Paint.Align.RIGHT`  右对齐
  private void drawText(Canvas canvas) {
        paint.setColor(Color.BLUE);
        paint.setStyle(Paint.Style.FILL);
        paint.setAlpha(100);
        canvas.drawCircle(500, 100, 20, paint);
        canvas.drawCircle(500, 300, 20, paint);
        canvas.drawCircle(500, 500, 20, paint);


        textPaint.setTextAlign(Paint.Align.LEFT);
        canvas.drawText("设置文字 left", 500, 100, textPaint);

        textPaint.setTextAlign(Paint.Align.CENTER);
        canvas.drawText("设置文字 CENTER", 500, 300, textPaint);

        textPaint.setTextAlign(Paint.Align.RIGHT);
        canvas.drawText("设置文字 RIGHT", 500, 500, textPaint);
    }
```

![image-20210802143554576](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210802143554576.png)

### setStrokeCap(Paint.Cap cap) 画笔的样式（落笔，收笔时）

```java
设置绘制起始点和结尾点的样式，
Paint.Cap.ROUND(圆形)
Paint.Cap.SQUARE(方形)
Paint.Cap.BUTT(无)
  private void drawCap(Canvas canvas) {
        paint.setStrokeWidth(100);
        paint.setColor(Color.BLUE);
        paint.setStyle(Paint.Style.STROKE);
        paint.setAlpha(100);

        paint.setStrokeCap(Paint.Cap.BUTT);
        canvas.drawLine(200, 300, 600, 300, paint);

        paint.setStrokeCap(Paint.Cap.ROUND);
        canvas.drawLine(200, 500, 600, 500, paint);

        paint.setStrokeCap(Paint.Cap.SQUARE);
        canvas.drawLine(200, 700, 600, 700, paint);
    }
```

![image-20210802150820442](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210802150820442.png)

### setStrokeJoin(Paint.Join join)连接点的样式

```java
设置绘制path连接点的样式
Paint.Join.MITER（结合处为锐角）
Paint.Join.Round(结合处为圆弧)
Paint.Join.BEVEL(结合处为直线)
private void drawJoin(Canvas canvas) {
        paint.setStrokeWidth(50);
        paint.setColor(Color.BLUE);
        paint.setStyle(Paint.Style.STROKE);
        paint.setAlpha(100);

        paint.setStrokeJoin(Paint.Join.BEVEL);
        canvas.drawRect(100, 100, 400, 400, paint);

        paint.setStrokeJoin(Paint.Join.MITER);
        canvas.drawRect(500, 100, 900, 400, paint);

        paint.setStrokeJoin(Paint.Join.ROUND);
        canvas.drawRect(100, 500, 400, 800, paint);

        paint.setStrokeJoin(Paint.Join.BEVEL);
        Path path = new Path();
        path.moveTo(500, 500);
        path.lineTo(900, 500);
        path.lineTo(500, 800);
        canvas.drawPath(path, paint);

        path.reset();
        paint.setStrokeJoin(Paint.Join.MITER);
        path.moveTo(100, 900);
        path.lineTo(400, 900);
        path.lineTo(100, 1300);
        canvas.drawPath(path, paint);

        path.reset();
        paint.setStrokeJoin(Paint.Join.ROUND);
        path.moveTo(500, 900);
        path.lineTo(900, 900);
        path.lineTo(500, 1300);
        canvas.drawPath(path, paint);
    }
```

![image-20210802150406839](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210802150406839.png)

### setPathEffect() 设置路径效果

ComposePathEffect

CornerPathEffect

DashPathEffect

DiscretePathEffect

PathDashPathEffect

SumPathEffect

```java
CornerPathEffect 设置拐角路据， CornerPathEffect用来绘制手绘效果，可以使线变得圆滑
 private void drawPathEffect(Canvas canvas) {
        paint.setStrokeWidth(20);
        paint.setColor(Color.BLUE);
        paint.setStyle(Paint.Style.STROKE);
        paint.setAlpha(100);


        Path path = new Path();
        path.moveTo(100, 100);
        path.lineTo(300, 400);
        path.lineTo(500, 100);
        path.lineTo(700, 400);
        path.lineTo(900, 100);
        path.lineTo(1100, 400);
        canvas.drawPath(path, paint);

        PathEffect pathEffect = new CornerPathEffect(20);
        paint.setPathEffect(pathEffect);
        path.reset();
        path.moveTo(100, 500);
        path.lineTo(300, 800);
        path.lineTo(500, 500);
        path.lineTo(700, 800);
        path.lineTo(900, 500);
        path.lineTo(1100, 800);

        canvas.drawPath(path, paint);

        PathEffect pathEffect1 = new CornerPathEffect(100);
        paint.setPathEffect(pathEffect1);
        path.reset();
        path.moveTo(0, 900);
        path.lineTo(200, 1200);
        path.lineTo(400, 900);
        path.lineTo(600, 1200);
        path.lineTo(800, 900);
        path.lineTo(1000, 1200);
        canvas.drawPath(path, paint);

        paint.setColor(Color.RED);
        paint.setAlpha(100);
        PathEffect pathEffect2 = new CornerPathEffect(50);
        paint.setPathEffect(pathEffect2);
        path.reset();
        path.moveTo(100, 900);
        path.lineTo(300, 1200);
        path.lineTo(500, 900);
        path.lineTo(700, 1200);
        path.lineTo(900, 900);
        path.lineTo(1100, 1200);
        canvas.drawPath(path, paint);

        paint.setColor(Color.YELLOW);
        paint.setAlpha(100);
        PathEffect pathEffect3 = new CornerPathEffect(20);
        paint.setPathEffect(pathEffect3);
        path.reset();
        path.moveTo(200, 900);
        path.lineTo(400, 1200);
        path.lineTo(600, 900);
        path.lineTo(800, 1200);
        path.lineTo(1000, 900);
        path.lineTo(1200, 1200);
        canvas.drawPath(path, paint);
    }
```

![image-20210802153047635](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210802153047635.png)

```
DashPathEffect 虚线效果
private void drawDashPathEffect(Canvas canvas) {
        paint.setStrokeWidth(20);
        paint.setColor(Color.BLUE);
        paint.setStyle(Paint.Style.STROKE);
        paint.setAlpha(100);

        PathEffect pathEffectOne = new DashPathEffect(new float[]{40, 20}, 0);
        paint.setPathEffect(pathEffectOne);
        Path pathOne = new Path();
        pathOne.moveTo(100, 100);
        pathOne.lineTo(1000, 100);
        canvas.drawPath(pathOne, paint);

        Path pathTwo = new Path();
        PathEffect pathEffectTwo = new DashPathEffect(new float[]{100, 20}, 50);  //偏移量 添加 50
        paint.setPathEffect(pathEffectTwo);
        pathTwo.moveTo(100, 300);
        pathTwo.lineTo(1000, 300);
        canvas.drawPath(pathTwo, paint);

        paint.setPathEffect(new DashPathEffect(new float[]{40, 20}, 0));
        canvas.drawCircle(500, 700, 300, paint);
        // drawLine 画不出虚线效果
        canvas.drawLine(50, 1100, 1100, 1100, paint);
    }
```

![image-20210802154619485](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210802154619485.png)

```
 PathDashPathEffect 绘制路径虚线
 private void drawPathDashPathEffect(Canvas canvas) {
        paint.setStrokeWidth(20);
        paint.setColor(Color.BLUE);
        paint.setStyle(Paint.Style.STROKE);
        paint.setAlpha(100);

        Path dashPath1 = new Path();
        Path pp1 = new Path();
        pp1.moveTo(20, 0);
        pp1.lineTo(0, 30);
        pp1.lineTo(40, 30);
        pp1.close();
        dashPath1.addPath(pp1);
        PathEffect pathEffect1 = new PathDashPathEffect(dashPath1, 40, 0, PathDashPathEffect.Style.TRANSLATE);
        paint.setPathEffect(pathEffect1);
        canvas.drawRect(100, 100, 900, 300, paint);

        Path dashPath2 = new Path();
        Path pp2 = new Path();
        pp2.moveTo(20, 0);
        pp2.lineTo(0, 30);
        pp2.lineTo(40, 30);
        pp2.close();
        dashPath2.addPath(pp2);
        PathEffect pathEffect2 = new PathDashPathEffect(dashPath2, 40, 0, PathDashPathEffect.Style.MORPH);
        paint.setPathEffect(pathEffect2);
        canvas.drawRect(100, 500, 900, 700, paint);


        Path dashPath3 = new Path();
        Path pp3 = new Path();
        pp3.moveTo(20, 0);
        pp3.lineTo(0, 30);
        pp3.lineTo(40, 30);
        pp3.close();
        dashPath3.addPath(pp3);
        PathEffect pathEffect3 = new PathDashPathEffect(dashPath3, 40, 0, PathDashPathEffect.Style.ROTATE);
        paint.setPathEffect(pathEffect3);
        canvas.drawRect(100, 900, 900, 1300, paint);

    }
```

![image-20210802161244227](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210802161244227.png)

### setShader(Shader shader) 设置shader包括渐变shader，图片shader

```java
线性渐变：LinearGradient
径向渐变：RadialGradient
扫描渐变：SweepGradient
位图渐变：BitmapShader
混合渐变：ComposeShader

Shader.TileMode.CLAMP：如果着色器超出原始边界范围，会复制边缘颜色。
Shader.TileMode.MIRROR：横向和纵向的重复着色器的图像，交替镜像图像是相邻的图像总是接合。这个官方的说明可能不太好理解，说白了，就是图像不停翻转来平铺，直到平铺完毕。
Shader.TileMode.REPEAT： 横向和纵向的重复着色器的图像。
```

```java
  线性渐变：LinearGradient
  private void drawLinearGradient(Canvas canvas) {
        paint.setStrokeWidth(50);
        paint.setColor(Color.BLACK);
        paint.setStyle(Paint.Style.FILL);

        LinearGradient linearGradient = new LinearGradient(100, 100, 300, 100, Color.RED, Color.BLUE, Shader.TileMode.MIRROR);
        paint.setShader(linearGradient);
        canvas.drawLine(100, 100, 1000, 100, paint);

        linearGradient = new LinearGradient(100, 200, 800, 800, Color.RED, Color.BLUE, Shader.TileMode.CLAMP);
        paint.setShader(linearGradient);
        canvas.drawRect(100, 200, 800, 800, paint);


        linearGradient = new LinearGradient(100, 100, 800, 100,
                new int[]{Color.RED, Color.YELLOW, Color.BLUE}, new float[]{0, 0.5f, 1}, Shader.TileMode.CLAMP);  //设置渐变区域 属性
        paint.setShader(linearGradient);
        canvas.drawLine(100, 900, 800, 900, paint);
    }
```

![image-20210802171057485](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210802171057485.png)

```java
 径向渐变：RadialGradient
 private void drawRadialGradient(Canvas canvas) {
        paint.setStrokeWidth(50);
        paint.setColor(Color.BLACK);
        paint.setStyle(Paint.Style.FILL);
        paint.setAlpha(200);

        //两个颜色
        RadialGradient radialGradient = new RadialGradient(400, 400, 300, Color.RED, Color.BLUE, Shader.TileMode.CLAMP);
        paint.setShader(radialGradient);
        canvas.drawCircle(400, 400, 300, paint);

        //多种颜色
        radialGradient = new RadialGradient(400, 1200, 300,
                new int[]{Color.RED, Color.YELLOW, Color.GRAY, Color.BLUE}, new float[]{0, 0.4f, 0.8f, 1}, Shader.TileMode.CLAMP);
        paint.setShader(radialGradient);
        canvas.drawCircle(400, 1200, 300, paint);
    }
```

![image-20210802171904276](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210802171904276.png)

```java
扫描渐变：SweepGradient
private void drawSweepGradient(Canvas canvas) {
        paint.setStrokeWidth(50);//设置画笔宽度 ，单位px
        paint.setColor(Color.BLACK);
        paint.setStyle(Paint.Style.FILL);
        //两个颜色
        SweepGradient sweepGradient = new SweepGradient(400, 400, Color.RED, Color.BLUE);
        paint.setShader(sweepGradient);
        canvas.drawCircle(400, 400, 300, paint);

        //多种颜色
        sweepGradient = new SweepGradient(400, 1200,
                new int[]{Color.RED, Color.YELLOW, Color.GRAY, Color.BLUE}, null);
        paint.setShader(sweepGradient);
        canvas.drawCircle(400, 1200, 300, paint);
    }
```

![image-20210802172301300](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210802172301300.png)

```java
混合渐变：ComposeShader
private void drawComposeShader(Canvas canvas) {
        paint.setStrokeWidth(50);//设置画笔宽度 ，单位px
        paint.setColor(Color.BLACK);
        paint.setStyle(Paint.Style.FILL);

        Shader shader1 = new BitmapShader(bitmap, Shader.TileMode.REPEAT, Shader.TileMode.MIRROR);
        LinearGradient shader2 = new LinearGradient(0, 0, 500, 0, Color.RED, Color.BLUE, Shader.TileMode.MIRROR);
        // ComposeShader：结合两个 Shader
        Shader shader = new ComposeShader(shader1, shader2, PorterDuff.Mode.SCREEN);
        paint.setShader(shader);

        canvas.drawRect(0, 0, getRight(), getBottom(), paint);
    }
```

![image-20210802174013113](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210802174013113.png)

### setColorFilter(ColorFilter filter) 设置颜色过滤

### setShadowLayer 阴影

```java
 private void drawShadowLayer(Canvas canvas) {
        /**
         * float radius 为模糊半径，入宫为0移除阴影
         */
        textPaint.setShadowLayer(2, 20, 30, Color.GRAY);
        textPaint.setTextAlign(Paint.Align.CENTER);
        canvas.drawText("android 阴影绘制", getWidth() / 2, 450, textPaint);
    }
```

![image-20210802190205215](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210802190205215.png)

### setXfermode(Xfermode xfermode) xfermode设置图像混合模式

```java
    /**
     * 绘制目标图像圆
     */
    Bitmap makeDst(int w, int h) {
        Bitmap bm = Bitmap.createBitmap(w, h, Bitmap.Config.ARGB_8888);
        Canvas c = new Canvas(bm);
        Paint p = new Paint(Paint.ANTI_ALIAS_FLAG);
        p.setColor(Color.RED);
        c.drawOval(new RectF(0, 0, w * 3 / 4, h * 3 / 4), p);
        return bm;
    }

    /**
     * 绘制源图像方块
     */
    Bitmap makeSrc(int w, int h) {
        Bitmap bm = Bitmap.createBitmap(w, h, Bitmap.Config.ARGB_8888);
        Canvas c = new Canvas(bm);
        Paint p = new Paint(Paint.ANTI_ALIAS_FLAG);
        p.setColor(Color.BLUE);
        c.drawRect(w / 3, h / 3, w * 19 / 20, h * 19 / 20, p);
        return bm;
    }
private static int W = 250;
private static int H = 250;
private static final int ROW_MAX = 4;   // number of samples per row
private Bitmap mDstB;// destination 目标像素
private Bitmap mSrcB;// source 源像素
 private static final Xfermode[] sModes = {
            new PorterDuffXfermode(PorterDuff.Mode.CLEAR),
            new PorterDuffXfermode(PorterDuff.Mode.SRC),
            new PorterDuffXfermode(PorterDuff.Mode.DST),
            new PorterDuffXfermode(PorterDuff.Mode.SRC_OVER),
            new PorterDuffXfermode(PorterDuff.Mode.DST_OVER),
            new PorterDuffXfermode(PorterDuff.Mode.SRC_IN),
            new PorterDuffXfermode(PorterDuff.Mode.DST_IN),
            new PorterDuffXfermode(PorterDuff.Mode.SRC_OUT),
            new PorterDuffXfermode(PorterDuff.Mode.DST_OUT),
            new PorterDuffXfermode(PorterDuff.Mode.SRC_ATOP),
            new PorterDuffXfermode(PorterDuff.Mode.DST_ATOP),
            new PorterDuffXfermode(PorterDuff.Mode.XOR),
            new PorterDuffXfermode(PorterDuff.Mode.DARKEN),
            new PorterDuffXfermode(PorterDuff.Mode.LIGHTEN),
            new PorterDuffXfermode(PorterDuff.Mode.MULTIPLY),
            new PorterDuffXfermode(PorterDuff.Mode.SCREEN)
    };
    private static final String[] sLabels = {
            "Clear", "Src", "Dst", "SrcOver",
            "DstOver", "SrcIn", "DstIn", "SrcOut",
            "DstOut", "SrcATop", "DstATop", "Xor",
            "Darken", "Lighten", "Multiply", "Screen"
    };

private void initView(Context context) {     
   bitmap = BitmapFactory.decodeResource(getResources(), R.mipmap.ic_cursor);
   mSrcB = makeSrc(W, H);
   mDstB = makeDst(W, H);
}
/**
* PorterDuff.Mode 混合模式
*/
private void drawPorterDuff(Canvas canvas) {
    canvas.drawColor(Color.WHITE);
    Paint labelP = new Paint(Paint.ANTI_ALIAS_FLAG);
    labelP.setTextAlign(Paint.Align.CENTER);
    labelP.setTextSize(30);
    Paint paint = new Paint();
    paint.setFilterBitmap(false);
    canvas.translate(30, 70);
    int x = 0;
    int y = 0;
    for (int i = 0; i < sModes.length; i++) {
        //画背景
        paint.setStyle(Paint.Style.FILL);
        paint.setColor(Color.parseColor("#BBB4ED9F"));
        canvas.drawRect(x, y, x + W, y + H, paint);

        int sc = canvas.saveLayer(x, y, x + W, y + H, null);

        canvas.translate(x, y);
        canvas.drawBitmap(mDstB, 0, 0, paint);

        paint.setXfermode(sModes[i]);
        canvas.drawBitmap(mSrcB, 0, 0, paint);
        paint.setXfermode(null);
        canvas.restoreToCount(sc);

        canvas.drawText(sLabels[i], x + W / 2, y - labelP.getTextSize() / 2, labelP);
        x += W + 10;
        if ((i % ROW_MAX) == ROW_MAX - 1) {
            x = 0;
            y += H + 70;
        }
    }
}
```

```java
PorterDuff.Mode.CLEAR:所绘制不会提交到画布上，清除当前绘制区域,正常是没东西
PorterDuff.Mode.SRC:只绘制源图
PorterDuff.Mode.DST:只保留目标图。
PorterDuff.Mode.SRC_OVER:把原图绘制在上方
PorterDuff.Mode.DST_OVER: 目标图绘制在上方
PorterDuff.Mode.SRC_IN:两者相交的地方绘制原图。
PorterDuff.Mode.DST_IN: 相交的地方绘制目标图，绘制效果会受到原图处的透明度影响
PorterDuff.Mode.SRC_OUT:不相交的地方绘制原图
PorterDuff.Mode.DST_OUT: 不相交的地方绘制目标图
PorterDuff.Mode.SRC_ATOP: 原图和目标图相交处绘制原图，不相交绘制目标图
PorterDuff.Mode.DST_ATOP： 源图和目标图相交处绘制目标图，不相交的地方绘制源图。
PorterDuff.Mode.XOR:不相交的地方按原样绘制原图和目标图
PorterDuff.Mode.DARKEN: 取两图层全部区域，交集部分 ，颜色加深
PorterDuff.Mode.LIGHTEN: 取两图层全部区域，点亮交集部分颜色
PorterDuff.Mode.MULTIPLY: 取两图层交集部分叠加后颜色
PorterDuff.Mode.SCREEN:取两图层全部区域，交集部分透明
```

可以看到下方两种图片，在canvas 和canvas.saveLayer 新建一个层上绘制效果不一样

![image-20210803120405684](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210803120405684.png)

![image-20210803120644701](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210803120644701.png)

# Canvas

### 画直线drawLine

```
drawLine(float startX, float startY, float stopX, float stopY, @NonNull Paint paint)
drawLines(@Size(min=4,multiple=2) @NonNull float[] pts, int offset, int count, Paint paint)
drawLines(@Size(min=4,multiple=2) @NonNull float[] pts, @NonNull Paint paint)

 private void drawCanvas(Canvas canvas) {
        paint.setStrokeWidth(30);
        paint.setColor(Color.BLUE);
        paint.setStyle(Paint.Style.FILL);
        paint.setAlpha(100);
        canvas.drawLine(100, 100, 1000, 100, paint);

        canvas.drawLines(new float[]{100, 200, 500, 200, 400, 300, 900, 300}, paint);
        //new float[]{  500, 400, 400, 500, 900, 500, 50, 700 }
        canvas.drawLines(new float[]{100, 400, 500, 400, 400, 500, 900, 500, 50, 700, 750, 800}, 2, 8, paint);
    }
```

![image-20210803132618866](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210803132618866.png)

### 点及多个点drawPoint、drawPoints

```
drawPoint(float x, float y, @NonNull Paint paint)
drawPoints(@Size(multiple=2) @NonNull float[] pts, @NonNull Paint paint)
drawPoints(@Size(multiple=2) @NonNull float[] pts, int offset, int count, @NonNull Paint paint)
```

### 矩形drawRect、drawRoundRect

```
drawRect(@NonNull RectF rect, @NonNull Paint paint)
drawRect(@NonNull Rect r, @NonNull Paint paint)
drawRect(float left, float top, float right, float bottom, @NonNull Paint paint)
drawRoundRect(@NonNull RectF rect, float rx, float ry, @NonNull Paint paint)
drawRoundRect(float left, float top, float right, float bottom, float rx, float ry, @NonNull Paint paint)


private void canvasDrawRect(Canvas canvas) {
        paint.setStrokeWidth(30);
        paint.setColor(Color.BLUE);
        paint.setStyle(Paint.Style.FILL);
        paint.setAlpha(100);
        canvas.drawRect(100, 100, 1000, 200, paint);

        paint.setStyle(Paint.Style.STROKE);
        canvas.drawRect(100, 300, 1000, 500, paint);

        RectF rectF = new RectF(100, 600, 500, 700);
        canvas.drawRect(rectF, paint);

        Rect rect = new Rect(600, 600, 1000, 700);
        canvas.drawRect(rect, paint);

        // RectF ： 绘制的矩形
        //rx ： 生成圆角的椭圆X轴半径
        //ry ： 生成圆角的椭圆Y轴的半径
        RectF rectF1 = new RectF(100, 800, 500, 900);
        canvas.drawRoundRect(rectF1, 30, 30, paint);
        canvas.drawRoundRect(600, 800, 1000, 900, 10, 10, paint);
    }
```

![image-20210803133642330](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210803133642330.png)

### 圆形drawCircle

```
drawCircle(float cx, float cy, float radius, @NonNull Paint paint)
cx ： 圆心X坐标
cy ： 圆心Y坐标
radius ： 半径
private void canvasDrawCircle(Canvas canvas) {
    paint.setStrokeWidth(30);
    paint.setColor(Color.BLUE);
    paint.setStyle(Paint.Style.FILL_AND_STROKE);
    paint.setAlpha(100);
    canvas.drawCircle(300, 300, 100, paint);
}
```

### 椭圆drawOval

```
drawOval(@NonNull RectF oval, @NonNull Paint paint)
drawOval(float left, float top, float right, float bottom, @NonNull Paint paint)

 private void canvasDrawOval(Canvas canvas) {
        paint.setStrokeWidth(30);
        paint.setColor(Color.BLUE);
        paint.setStyle(Paint.Style.FILL_AND_STROKE);
        paint.setAlpha(100);
        canvas.drawOval(200, 200, 800, 400, paint);

        RectF rectF = new RectF(100, 600, 500, 700);
        canvas.drawOval(rectF, paint);
    }
```

### 圆弧drawArc

```
/**
 * drawArc(@NonNull RectF oval, float startAngle, float sweepAngle, boolean useCenter, @NonNull Paint paint)
 * drawArc(float left, float top, float right, float bottom, float startAngle, float sweepAngle, boolean useCenter, @NonNull Paint paint)
 *
 * oval ： 生成椭圆的矩形
 * startAngle ： 弧开始的角度 （X轴正方向为0度，顺时针弧度增大）
 * sweepAngle ： 绘制多少弧度 （注意不是结束弧度）
 * useCenter ： 是否有弧的两边 true有两边 false无两边
 */
private void canvasDrawArc(Canvas canvas) {
    paint.setStrokeWidth(30);
    paint.setColor(Color.BLUE);
    paint.setStyle(Paint.Style.FILL);
    paint.setAlpha(100);
    RectF rectF = new RectF(100, 200, 600, 700);
    canvas.drawArc(rectF, 0, 150, false, paint);
}
```

### 绘制文字canvas.drawText

```java
 private void canvasDrawText(Canvas canvas) {
        paint.setColor(Color.BLUE);
        paint.setStyle(Paint.Style.FILL);
        paint.setAlpha(100);
        canvas.drawCircle(500, 300, 20, paint);

        textPaint.setTextAlign(Paint.Align.CENTER);
        String text = "这个是测试文案";
        canvas.drawText(text, 500, 300, textPaint);

        Paint.FontMetrics fontMetrics = textPaint.getFontMetrics();
        Log.d(TAG, "Paint.FontMetrics top:" + fontMetrics.top);
        Log.d(TAG, "Paint.FontMetrics ascent:" + fontMetrics.ascent);
        Log.d(TAG, "Paint.FontMetrics descent:" + fontMetrics.descent);
        Log.d(TAG, "Paint.FontMetrics bottom:" + fontMetrics.bottom);
        Log.d(TAG, "Paint.FontMetrics leading:" + fontMetrics.leading);

        Log.e(TAG, "W:" + textPaint.measureText(text) + " H:" + (fontMetrics.bottom - fontMetrics.top));

        Rect rect = new Rect();
        textPaint.getTextBounds(text, 0, text.length(), rect);
        Log.w(TAG, "text Rect:" + rect.toString());
        Log.e(TAG, "W:" + (rect.right - rect.left) + " H:" + (rect.bottom - rect.top));

        RectF rectF = new RectF(200, 500, 700, 700);
        canvas.drawRect(rectF, paint);

        //计算baseline
//        float distance = (fontMetrics.bottom - fontMetrics.top) / 2 - fontMetrics.bottom;
//        float baseline = rectF.centerY() + distance;
        float baseline = rectF.centerY() - fontMetrics.top / 2 - fontMetrics.bottom / 2;
        canvas.drawText(text, rectF.centerX(), baseline, textPaint);

        canvas.drawCircle(500, 900, 20, paint);
        String showText = "折戟沉沙铁未销，自将磨洗认前朝。东风不与周郎便，铜雀春深锁二乔。";
        canvas.drawTextRun(showText, 0, showText.length(), 0, showText.length(), 500, 900, true, textPaint);


        Path path = new Path();
        rectF = new RectF(200, 1000, 900, 1200);
        path.addRoundRect(rectF, 10, 10, Path.Direction.CCW);
        canvas.drawTextOnPath(showText, path, 0, 0, textPaint);
        paint.setStrokeWidth(10);
        paint.setStyle(Paint.Style.STROKE);
        canvas.drawRect(rectF, paint);
    }
```

![image-20210803153444227](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20210803153444227.png)

# Path

| 方法                                   | 作用     | 备注                                                 |
| ------------------------------------ |:------ | -------------------------------------------------- |
| **moveTo**                           | 移动起点   | 移动下一次操作的起点位置                                       |
| **lineTo**                           | 连接直线   | 连接上一个点到当前点之间的直线                                    |
| **setLastPoint**                     | 设置终点   | 重置最后一个点的位置                                         |
| **close**                            | 闭合路劲   | 从最后一个点连接最初的一个点，形成一个闭合区域                            |
| **addRect**                          | 添加矩形   | 添加矩形到当前Path                                        |
| **addRoundRect**                     | 添加圆角矩形 | 添加圆角矩形到当前Path                                      |
| **addOval**                          | 添加椭圆   | 添加椭圆到当前Path                                        |
| **addCircle**                        | 添加圆    | 添加圆到当前Path                                         |
| **addPah**                           | 添加路劲   | 添加路劲到当前Path                                        |
| **addArc**                           | 添加圆弧   | 添加圆弧到当前Path                                        |
| **arcTo**                            | 圆弧     | 绘制圆弧，注意和addArc的区别                                  |
| **isEmpty**                          | 是否为空   | 判定Path是否为空                                         |
| **isRect**                           | 是否为矩形  | 判定Path是否是一个矩形                                      |
| **set**                              | 替换路劲   | 用新的路劲替换当前路劲的所有内容                                   |
| **offset**                           | 偏移路劲   | 对当前的路劲进行偏移                                         |
| **quadTo**                           | 贝塞尔曲线  | 二次贝塞尔曲线的方法                                         |
| **cubicTo**                          | 贝塞尔曲线  | 三次贝塞尔曲线的方法                                         |
| **rMoveTo rlineTo rQuadTo rCubicTo** | rXxx方法 | 不带r的方法是基于原点坐标系（偏移量），带r的基于当前点坐标系（偏移量）               |
| **op**                               | 布尔操作   | 对两个Path进行布尔运算（交集，并集）等操作                            |
| **setFillType**                      | 填充模式   | 设置Path的填充模式                                        |
| **getFillType**                      | 填充模式   | 获取Path的填充                                          |
| **isInverseFillType**                | 是否逆填充  | 判断是否是逆填充模式                                         |
| **toggleInverseFillType**            | 相反模式   | 切换相反的填充模式                                          |
| **getFillType**                      | 填充模式   | 获取Path的填充                                          |
| **incReserve**                       | 提示方法   | 提示Path还有多少个点等待加入                                   |
| **computeBounds**                    | 计算边界   | 计算Path的路劲                                          |
| **reset，rewind**                     | 重置路劲   | 清除Path中的内容（reset相当于new Path , rewind 会保留Path的数据结构） |
| **transform**                        | 矩阵操作   | 矩阵变换                                               |

```
private void canvasPath(Canvas canvas) {
    paint.setStrokeWidth(30);
    paint.setColor(Color.BLUE);
    paint.setStyle(Paint.Style.STROKE);
    paint.setAlpha(100);

    Path path = new Path();
    path.moveTo(100, 100);
    path.lineTo(300, 200);
    path.lineTo(500, 200);
    path.setLastPoint(700, 800);
    path.close();
    canvas.drawPath(path, paint);


    /**
     * Direction的意思是方向，指导，趋势。点进去跟一下你会发现Direction是一个枚举类型（Enum）
     * 分别有CW（顺时针），CCW（逆时针）两个常量
     */
    path.addRect(100, 600, 400, 800, Path.Direction.CCW );
    path.setLastPoint(200,400);
    canvas.drawPath(path, paint);


    path.addArc(500, 600, 900, 800, 30, 100);
    canvas.drawPath(path, paint);

    path.arcTo(500, 600, 900, 800, 190, 100, false);
    canvas.drawPath(path, paint);

    path.moveTo(600, 400);
    path.quadTo(800, 200, 1000, 300);

    canvas.drawPath(path, paint);

}
```

## clipRect  裁剪画布，后面补充
Region.Op 参数解读

* Op.DIFFERENCE
实际上就是求得的A和B的差集范围，即A－B，只有在此范围内的绘制内容才会被显
* Op.REVERSE_DIFFERENCE
实际上就是求得的B和A的差集范围，即B－A，只有在此范围内的绘制内容才会被显示
* Op.INTERSECT
即A和B的交集范围，只有在此范围内的绘制内容才会被显示
* Op.REPLACE
不论A和B的集合状况，B的范围将全部进行显示，如果和A有交集，则将覆盖A的交集范围
* Op.UNION
即A和B的并集范围，即两者所包括的范围的绘制内容都会被显示；
* Op.XOR
A和B的补集范围，此例中即A除去B以外的范围，只有在此范围内的绘制内容才会被显示

首先是背景
```
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    canvas.save();
    mPaint.setColor(Color.RED);
    canvas.drawRect(0, 0, 200, 200, mPaint);
    mPaint.setColor(Color.BLUE);
    canvas.drawCircle(450, 400, 400, mPaint);
    mPaint.setColor(Color.YELLOW);
    canvas.drawRect(300, 0, 700, 400, mPaint);
    canvas.restore();
}
```
![默认背景图片](https://gitee.com/ZeTing/UploadImg/raw/main/img/20230612164304.png)

1. Region.Op.DIFFERENCE
实际上就是求得的A和B的差集范围，即A－B，只有在此范围内的绘制内容才会被显
```
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    canvas.save();
    mPaint.setColor(Color.RED);
    canvas.drawRect(0, 0, 200, 200, mPaint);
    mPaint.setColor(Color.BLUE);
    canvas.drawCircle(450, 400, 400, mPaint);
    mPaint.setColor(Color.YELLOW);
    canvas.drawRect(300, 0, 700, 400, mPaint);

    // 裁剪形状 A
    canvas.clipRect(new RectF(0, 0, 300, 300));
    // 画布裁剪一个圆形
    Path mPath = new Path();
    mPath.addCircle(280, 190, 150, Path.Direction.CCW);
    // 裁剪形状 B
    canvas.clipPath(mPath, Region.Op.DIFFERENCE);
//        canvas.clipPath(mPath, Region.Op.REVERSE_DIFFERENCE);
//        canvas.clipPath(mPath, Region.Op.INTERSECT);
//        canvas.clipPath(mPath, Region.Op.REPLACE);
//        canvas.clipPath(mPath, Region.Op.UNION);
//        canvas.clipPath(mPath, Region.Op.XOR);

    mPaint.setColor(Color.parseColor("#FF4081"));
    canvas.drawRect(new RectF(0, 0, Integer.MAX_VALUE, Integer.MAX_VALUE), mPaint);

    canvas.restore();
}
```
![DIFFERENCE](https://gitee.com/ZeTing/UploadImg/raw/main/img/20230612164625.png)

2. Op.REVERSE_DIFFERENCE
实际上就是求得的B和A的差集范围，即B－A，只有在此范围内的绘制内容才会被显示
```
代码都是一样，只不过设置参数不一致，后面直接贴图
```
![](https://gitee.com/ZeTing/UploadImg/raw/main/img/20230612164824.png)

3. Op.INTERSECT
即A和B的交集范围，只有在此范围内的绘制内容才会被显示
![](https://gitee.com/ZeTing/UploadImg/raw/main/img/20230612164901.png)

4. Op.REPLACE
不论A和B的集合状况，B的范围将全部进行显示，如果和A有交集，则将覆盖A的交集范围
![](https://gitee.com/ZeTing/UploadImg/raw/main/img/20230612164938.png)

5. Op.UNION
即A和B的并集范围，即两者所包括的范围的绘制内容都会被显示
![](https://gitee.com/ZeTing/UploadImg/raw/main/img/20230612165014.png)

6. Op.XOR
A和B的补集范围，A和B的并集，除去A和B交集以外的范围
![](https://gitee.com/ZeTing/UploadImg/raw/main/img/20230612165051.png)


## 参数默认
默认指的是设置裁切区域和当前屏幕最大的交集范围内，默认 Op.INTERSECT
```
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    canvas.save();

    // 默认裁剪和屏幕最大的交集部分
    canvas.clipRect(new Rect(50, 100, 600, 500));

    mPaint.setColor(Color.RED);
    canvas.drawRect(0, 0, 200, 200, mPaint);
    mPaint.setColor(Color.BLUE);
    canvas.drawCircle(450, 400, 400, mPaint);
    mPaint.setColor(Color.YELLOW);
    canvas.drawRect(300, 0, 700, 400, mPaint);
    canvas.restore();
}
```
