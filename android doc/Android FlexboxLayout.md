# Android FlexboxLayout 属性介绍

## 所有可用属性汇总
1. 尺寸相关属性

属性|	作用|	取值|	默认值|	备注
--|:--|:--:|:--:|:--
flexBasis|	初始尺寸|	≥0 的整数(px)|	AUTO|	优先级低于 flexBasisPercent
flexBasisPercent|	初始尺寸百分比|	0.0-1.0	-1.0|	-1| 表示不设置
minWidth/minHeight|	最小尺寸|	≥0 的整数(px)|	0|	不会被压缩到小于此值
maxWidth/maxHeight|	最大尺寸|	≥0 的整数(px)|	MAX_VALUE|	不会放大到超过此值


2. 伸缩相关属性

属性|	作用|	取值| 默认值|	公式影响
--|:--|:--:|:--:|:--
flexGrow|	放大比例|	≥0 的浮点数|	0.0|	剩余空间分配比例
flexShrink|	缩小比例	≥0 的浮点数|	1.0	|不足空间分配比例
flex|	简写属性|	无|	|-|	flexGrow, flexShrink, flexBasis

3. 对齐相关属性

属性|	作用|	取值|	默认值|	影响
--|:--|:--:|:--:|:--
alignSelf|	自身对齐|	AUTO, START, END, CENTER, BASELINE, STRETCH	|AUTO|	覆盖容器对齐
order|	显示顺序|	整数|	0|	值越小越靠前
wrapBefore|	强制换行|	true/false|	false|	在此项目前换行

4. 容器相关属性
在 FlexboxLayoutManager 中设置

属性|	作用|	取值|	默认值
--|:--|:--:|:--
flexDirection|	主轴方向|	ROW, ROW_REVERSE, COLUMN, COLUMN_REVERSE|	ROW
flexWrap|	换行方式|	NOWRAP, WRAP, WRAP_REVERSE|	NOWRAP
justifyContent|	主轴对齐|	FLEX_START, FLEX_END, CENTER, SPACE_BETWEEN, SPACE_AROUND, SPACE_EVENLY|	FLEX_START
alignItems|	交叉轴对齐|	STRETCH, FLEX_START, FLEX_END, CENTER, BASELINE|	STRETCH
alignContent|	多行对齐|	FLEX_START, FLEX_END, CENTER, SPACE_BETWEEN, SPACE_AROUND, STRETCH|	STRETCH
