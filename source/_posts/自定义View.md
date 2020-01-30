title: 自定义View
date: 2019-2-21
tags: [Android,View的工作原理]
categories: Android开发艺术探索
description: 兜兜转转，还是回到了这
---
## 自定义View的分类
1. 继承View重写onDraw方法。这种方法主要用于实现一些不规则的效果，即这种效果不方便通过布局的组合方式来达到，往往需要静态或者动态地显示一些不规则的图形。这种方式需要重写onDraw方法，同时需要自己支持wrap_content，并且padding也需要自己处理。
2. 继承ViewGroup派生特殊的Layout。这种方法主要用于实现自定义的布局。需要处理ViewGroup和子View的测量和布局。
3. 继承特定的View(比如TextView)。这种方法比较常见，一般是用于扩展某种已有的View的功能，比如TextView。
4. 继承特定的ViewGroup(比如LinearLayout)。这种效果看起来很像几种View组合在一起的时候，可以采用这种方法实现，不用处理ViewGroup的测量和布局。

## 注意事项
1. 让View支持wrap_content.在onMeasure()中对MeasureSpec.AT_MOST进行处理
2. 如果有必要，让你的View支持padding。在onDraw（）中考虑padding.
3. 尽量不要在View中使用Handler，View.post（）可替代
4. View中如果有线程或者动画，需要及时在View#onDetachedFromWindow停止
5. View带有滑动嵌套情形时，需要处理好滑动冲突

## 继承View重写onDraw方法
实例. CircleView:wrap_content与padding适配
```java
public class CircleView extends View {

  private int mColor = Color.RED;
  private Paint mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);

  public CircleView(Context context) {
    super(context);
  }

  public CircleView(Context context, @Nullable AttributeSet attrs) {
    this(context, attrs, 0);
  }

  public CircleView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
    super(context, attrs, defStyleAttr);
    TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.CircleView);
    mColor = a.getColor(R.styleable.CircleView_circle_color, Color.RED);
    a.recycle();
    init();
  }

  public CircleView(Context context, @Nullable AttributeSet attrs, int defStyleAttr, int defStyleRes) {
    super(context, attrs, defStyleAttr, defStyleRes);
    init();
  }

  private void init() {
    mPaint.setColor(mColor);
  }

  @Override protected void onDraw(Canvas canvas) {

    super.onDraw(canvas);

    final int paddingLeft = getPaddingLeft();
    final int paddingRight = getPaddingRight();
    final int paddingTop = getPaddingTop();
    final int paddingBottom = getPaddingBottom();

    int width = getWidth() - paddingLeft - paddingRight;
    int height = getHeight() - paddingBottom - paddingTop;
    int radius = Math.min(width, height) / 2;

    canvas.drawCircle(paddingLeft + width / 2, paddingTop + height / 2, radius, mPaint);
  }

  @Override protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
    int widthSize = MeasureSpec.getSize(widthMeasureSpec);
    int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
    int heightSize = MeasureSpec.getSize(heightMeasureSpec);
    if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST) {
      setMeasuredDimension(200, 200);//Default width and height
    } else if (widthSpecMode == MeasureSpec.AT_MOST) {
      setMeasuredDimension(200, heightSize);
    } else if (heightSpecMode == MeasureSpec.AT_MOST) {
      setMeasuredDimension(widthSize, 200);
    }
  }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.hdu.night.scrollconflicts.CircleView android:layout_width="wrap_content"
                                              android:layout_height="100dp"
                                              android:layout_margin="20dp"
                                              android:padding="20dp"
                                              android:background="#000000"
                                              app:circle_color="@color/colorPrimary"
                                              app:layout_constraintLeft_toLeftOf="parent"
                                              app:layout_constraintRight_toRightOf="parent"
                                              app:layout_constraintTop_toTopOf="parent"/>

</android.support.constraint.ConstraintLayout>
```
```xml
//attrs.xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="CircleView">
        <attr name="circle_color" format="color"/>
    </declare-styleable>
</resources>
```
## 继承特定的ViewGroup派生特殊的Layout
实例2. HorizontalViewEx再回顾：onMeasure() + onDraw()
先假设所有子View都是等宽高的。
### onMeasure()
```java
@Override
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    int measuredWidth = 0;
    int measuredHeight = 0;
    final int childCount = getChildCount();
    measureChildren(widthMeasureSpec, heightMeasureSpec);////measure all children view width and height.

    int widthSpaceSize = MeasureSpec.getSize(widthMeasureSpec);
    int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
    int heightSpaceSize = MeasureSpec.getSize(heightMeasureSpec);
    int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
    if (childCount == 0) {// If ViewGroup has not  child, the size of viewGroup will be 0  
      setMeasuredDimension(0, 0);
    } else if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST) {//Handle wrap_content
      final View childView = getChildAt(0);
      measuredWidth = childView.getMeasuredWidth() * childCount;
      measuredHeight = childView.getMeasuredHeight();
      setMeasuredDimension(measuredWidth, measuredHeight);
    } else if (widthSpecMode == MeasureSpec.AT_MOST) {
      final View childView = getChildAt(0);
      measuredWidth = childView.getMeasuredWidth() * childCount;
      setMeasuredDimension(measuredWidth, heightSpaceSize);
    } else if(heightSpecMode == MeasureSpec.AT_MOST){
      final View childView = getChildAt(0);
      measuredHeight = childView.getMeasuredHeight();
      setMeasuredDimension(widthSpaceSize, measuredHeight);
    }
  }
```

### onLayout()
```java
@Override
  protected void onLayout(boolean changed, int l, int t, int r, int b) {
    int childLeft = 0;
    final int childCount = getChildCount();
    mChildrenSize = childCount;

    for (int i = 0; i < childCount; i++) {//Traverse all child and put them at a right place
      final View childView = getChildAt(i);
      if (childView.getVisibility() != View.GONE) {
        final int childWidth = childView.getMeasuredWidth();
        mChildWidth = childWidth;
        childView.layout(childLeft, 0, childLeft + childWidth,
            childView.getMeasuredHeight());//(l,t,r,b)
        childLeft += childWidth;
      }
    }
  }
```