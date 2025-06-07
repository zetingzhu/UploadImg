FlexboxLayout 使用详解

# FlexboxLayout 常用属性
## flexDirection
> flexDirection 属性决定主轴项目排列方向。类似 LinearLayout 的 vertical 和 horizontal，但是 FlexboxLayout 更加强大，不仅支持横向和纵向还可以设置不同的排列的起点。

- row（默认值）：主轴为水平方向，起点在左端。
- row_reverse：主轴为水平方向，起点在右端。
- column：主轴为垂直方向，起点在上沿。
- column_reverse：主轴为垂直方向，起点在下沿。

## flexWrap

>默认 FlexboxLayout 和 LinearLayout 一样是不带换行属性的，但是 flexWrap 属性可以支持换行排列。这就是 FlexboxLayout 方便的地方了。换行方式有两种，一种是按项目排列方向换行，一种是反方向换行。

- nowrap （默认）：不换行。
- wrap：按正常方向换行。
- wrap_reverse：按反方向换行。


## justifyContent
> justifyContent 属性定义了项目在主轴上的对齐方式。

- flex_start（默认值）：左对齐。
- flex_end ：右对齐。
- center ： 居中。
- space_between ：左右靠边，间隔相同，
- space_around ：左右留个边，间隔二倍往中间靠

## alignItems
> alignItems 属性定义项目在副轴轴上如何对齐，我们通过一张图来了解这个属性比较直观一点。

- flex_start：交叉轴的起点对齐。
- flex_end：交叉轴的终点对齐。
- center：交叉轴的中点对齐。
- baseline: 项目的第一行文字的基线对齐。
- stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。


## alignContent
> alignContent 属性定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。

- flex_start：与交叉轴的起点对齐。
- flex_end：与交叉轴的终点对齐。
- center：与交叉轴的中点对齐。
- space_between：与交叉轴两端对齐，轴线之间的间隔平均分布。
- space_around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
- stretch（默认值）：轴线占满整个交叉轴。

> alignContent 和 justifyContent 其实里面的属性值都是一样的 ，一个是设置主轴的对齐方式，一个是设置多个轴的对齐方式，通俗的讲可以理解为比如是项目是水平换行，justifyContent就是设置垂直方向的对齐方式，如果项目是垂直方向对齐方式，那么justifyContent就是设置水平方向的对齐方式。现在我们想让每个项目距离上右下左的距离是一样的，需要把alignContent和justifyContent都设置为space_around就可以了。


## showDividerHorizontal
> showDividerHorizontal 控制显示水平方向的分割线，值为none | beginning | middle | end其中的一个或者多个。

## dividerDrawableHorizontal
> dividerDrawableHorizontal 设置Flex 轴线之间水平方向的分割线。

## showDividerVertical
> showDividerVertical 控制显示垂直方向的分割线，值为none | beginning | middle | end其中的一个或者多个。

## dividerDrawableVertical
> dividerDrawableVertical 设置子元素垂直方向的分割线。

## showDivider
> showDivider 控制显示水平和垂直方向的分割线，值为none | beginning | middle | end其中的一个或者多个。

## dividerDrawable
> dividerDrawable 设置水平和垂直方向的分割线，但是注意,如果同时和其他属性使用，比如为 Flex 轴、子元素设置了justifyContent="space_around" 、alignContent="space_between" 等等。可能会看到意料不到的空间，因此应该避免和这些值同时使用。

# 子元素属性

除以上之外，FlexboxLayout 不仅有自身的属性，还可以设置子元素的属性。这也是 FlexboxLayout 能完成聪明布局的原因之一。

## layout_order
> 默认情况下子元素的排列方式按照文档流的顺序依次排序，而 order 属性可以控制排列的顺序，负值在前，正值在后，按照从小到大的顺序依次排列。简而言之就是你可以定义子元素的排列顺序。

我们给子元素加上 order 属性并且自定义他们的顺序。

## layout_flexGrow
> layout_flexGrow 属性定义项目的放大比例，默认为0，即如果存在剩余空间，也不放大。其实就是 LinearLayout 中的weight属性，如果所有项目的layout_flexGrow 属性都为1，则它们将等分剩余空间。如果一个项目的layout_flexGrow 属性为2，其他项目都为1，则前者占据的剩余空间将比其他项多一倍。

## layout_flexShrink
> layout_flexShrink 属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小（设置了换行则无效）。如果所有项目的 layout_flexShrink 属性都为1，当空间不足时，都将等比例缩小。如果一个项目的 flex-shrink 属性为0，其他项目都为1，则空间不足时，前者不缩小。负值对该属性无效。

## layout_alignSelf
> layout_alignSelf 属性允许单个子元素有与其他子元素不一样的对齐方式，可覆盖 alignItems 属性。默认值为auto，表示继承父元素的 alignItems 属性，如果没有父元素，则等同于stretch。

- auto (default)
- flex_start
- flex_end
- center
- baseline
- stretch

该属性可能取6个值，除了auto，其他都与align_items属性完全一致，我们设置alignItems为flex_start属性，其中一个子元素设置layout_alignSelf属性为baseline。

baseline的基线是第一个元素的 baseline基线。

## layout_flexBasisPercent
> layout_flexBasisPercent 属性定义了在分配多余空间之前，子元素占据的主轴空间的百分比。它的默认值为auto，即子元素的本来大小。

我们设置第一个和第三个都占据的主轴空间的80%，给子元素添加属性
