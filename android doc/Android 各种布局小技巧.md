

ConstraintLayout 使用技巧

ConstraintLayout 下，第一个 TextView 第二个 TextView 一行排列
当第一个Textview 过长时候，第二个会被挤出去
设置属性
app:layout_constrainedWidth="true"



```xml
<androidx.constraintlayout.widget.ConstraintLayout
                android:layout_width="@dimen/margin_0dp"
                android:layout_height="match_parent"
                android:layout_marginStart="@dimen/margin_4dp"
                android:layout_weight="1">
                <!--一分钟标题-->
                <TextView
                    android:id="@+id/tv_one_title"
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:gravity="center_vertical"
                    android:text="涨跌幅 涨跌幅 涨跌幅 涨跌幅"
                    android:textColor="@color/color_6D778B"
                    android:textSize="@dimen/text_size_10sp"
                    app:layout_constrainedWidth="true"
                    app:layout_constraintBottom_toBottomOf="parent"
                    app:layout_constraintEnd_toStartOf="@+id/ll_one_sort"
                    app:layout_constraintHorizontal_bias="0"
                    app:layout_constraintHorizontal_chainStyle="packed"
                    app:layout_constraintStart_toStartOf="parent"
                    app:layout_constraintTop_toTopOf="parent" />
                <LinearLayout
                    android:id="@+id/ll_one_sort"
                    android:layout_width="@dimen/margin_6dp"
                    android:layout_height="match_parent"
                    android:layout_marginStart="@dimen/margin_4dp"
                    android:layout_marginEnd="@dimen/margin_4dp"
                    android:gravity="center"
                    android:orientation="vertical"
                    app:layout_constraintBottom_toBottomOf="parent"
                    app:layout_constraintEnd_toEndOf="parent"
                    app:layout_constraintStart_toEndOf="@+id/tv_one_title"
                    app:layout_constraintTop_toTopOf="parent">
                    <com.trade.eight.view.widget.TriangleView
                        android:id="@+id/tiv_one_sort_min"
                        android:layout_width="@dimen/margin_6dp"
                        android:layout_height="@dimen/margin_4dp"
                        android:layout_gravity="end"
                        app:tlv_color="@color/color_6D778B"
                        app:tlv_mode="regular" />
                    <com.trade.eight.view.widget.TriangleView
                        android:id="@+id/tiv_one_sort_max"
                        android:layout_width="@dimen/margin_6dp"
                        android:layout_height="@dimen/margin_4dp"
                        android:layout_gravity="end"
                        android:layout_marginTop="@dimen/margin_1dp"
                        app:tlv_color="@color/color_6D778B"
                        app:tlv_mode="inverted" />
                </LinearLayout>
            </androidx.constraintlayout.widget.ConstraintLayout>
```
