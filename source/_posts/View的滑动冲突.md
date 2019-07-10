title:  View的滑动冲突
date: 2019-2-18
tags: [Android,View事件分发]
categories: Android开发艺术探索
description: 　　
---
## 外部滑动与内部滑动不一致
如在HorizontalScrollView中嵌套RecycleView.
### 外部拦截法
外部拦截就是点击事件先经过父容器的拦截处理，如果父容器需要此事件就拦截，如果不需要就不拦截。
 外部拦截法需要重写父容器的onInterceptTouchEvent（）方法。
 伪代码：
```java
public boolean onInterceptTouchEvent(MotionEvent event) {
boolean intercepted=false;
int x= (int) event.getX();
int y= (int) event.getY();
switch (event.getAction()){
case MotionEvent.ACTION_DOWN:
intercepted=false;//必须不能拦截，否则后续的ACTION_MOME和ACTION_UP事件都会拦截。
break;
case MotionEvent.ACTION_MOVE:
if (父容器需要当前点击事件){
intercepted=true;
            }else {
intercepted=false;
            }
break;
case MotionEvent.ACTION_UP:
intercepted=false;
break;
default:
break;
    }
mLastXIntercept=x;
mLastXIntercept=y;
return intercepted;
}
```
实例：重写HorizontalScrollView
```java
import android.content.Context;
import android.util.AttributeSet;
import android.util.Log;
import android.view.MotionEvent;
import android.view.VelocityTracker;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Scroller;

/**
 * Created by night on 2019/2/14.
 */
public class HorizontalScrollViewEx extends ViewGroup {
  private static final String TAG = "HorizontalScrollViewEx";

  private int mChildrenSize;
  private int mChildWidth;
  private int mChildIndex;

  // 分别记录上次滑动的坐标
  private int mLastX = 0;
  private int mLastY = 0;
  // 分别记录上次滑动的坐标(onInterceptTouchEvent)
  private int mLastXIntercept = 0;
  private int mLastYIntercept = 0;

  private Scroller mScroller;
  private VelocityTracker mVelocityTracker;

  public HorizontalScrollViewEx(Context context) {
    super(context);
    init();
  }

  public HorizontalScrollViewEx(Context context, AttributeSet attrs) {
    super(context, attrs);
    init();
  }

  public HorizontalScrollViewEx(Context context, AttributeSet attrs,
                                int defStyle) {
    super(context, attrs, defStyle);
    init();
  }

  private void init() {
    mScroller = new Scroller(getContext());
    mVelocityTracker = VelocityTracker.obtain();
  }

  @Override
  public boolean onInterceptTouchEvent(MotionEvent event) {
    boolean intercepted = false;
    int x = (int) event.getX();
    int y = (int) event.getY();

    switch (event.getAction()) {
      case MotionEvent.ACTION_DOWN: {
        intercepted = false;
        if (!mScroller.isFinished()) {
          mScroller.abortAnimation();
          intercepted = true;
        }
        break;
      }
      case MotionEvent.ACTION_MOVE: {
        int deltaX = x - mLastXIntercept;
        int deltaY = y - mLastYIntercept;
        if (Math.abs(deltaX) > Math.abs(deltaY)) {
          intercepted = true;
        } else {
          intercepted = false;
        }
        break;
      }
      case MotionEvent.ACTION_UP: {
        intercepted = false;
        break;
      }
      default:
        break;
    }

    Log.d(TAG, "intercepted=" + intercepted);
    mLastX = x;
    mLastY = y;
    mLastXIntercept = x;
    mLastYIntercept = y;

    return intercepted;
  }

  @Override
  public boolean onTouchEvent(MotionEvent event) {
    mVelocityTracker.addMovement(event);
    int x = (int) event.getX();
    int y = (int) event.getY();
    switch (event.getAction()) {
      case MotionEvent.ACTION_DOWN: {
        if (!mScroller.isFinished()) {
          mScroller.abortAnimation();
        }
        break;
      }
      case MotionEvent.ACTION_MOVE: {
        int deltaX = x - mLastX;
        int deltaY = y - mLastY;
        scrollBy(-deltaX, 0);
        break;
      }
      case MotionEvent.ACTION_UP: {
        int scrollX = getScrollX();
        int scrollToChildIndex = scrollX / mChildWidth;
        mVelocityTracker.computeCurrentVelocity(1000);
        float xVelocity = mVelocityTracker.getXVelocity();
        if (Math.abs(xVelocity) >= 50) {
          mChildIndex = xVelocity > 0 ? mChildIndex - 1 : mChildIndex + 1;
        } else {
          mChildIndex = (scrollX + mChildWidth / 2) / mChildWidth;
        }
        mChildIndex = Math.max(0, Math.min(mChildIndex, mChildrenSize - 1));
        int dx = mChildIndex * mChildWidth - scrollX;
        smoothScrollBy(dx, 0);
        mVelocityTracker.clear();
        break;
      }
      default:
        break;
    }

    mLastX = x;
    mLastY = y;
    return true;
  }

  @Override
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    int measuredWidth = 0;
    int measuredHeight = 0;
    final int childCount = getChildCount();
    measureChildren(widthMeasureSpec, heightMeasureSpec);

    int widthSpaceSize = MeasureSpec.getSize(widthMeasureSpec);
    int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
    int heightSpaceSize = MeasureSpec.getSize(heightMeasureSpec);
    int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
    if (childCount == 0) {
      setMeasuredDimension(0, 0);
    } else if (heightSpecMode == MeasureSpec.AT_MOST) {
      final View childView = getChildAt(0);
      measuredHeight = childView.getMeasuredHeight();
      setMeasuredDimension(widthSpaceSize, childView.getMeasuredHeight());
    } else if (widthSpecMode == MeasureSpec.AT_MOST) {
      final View childView = getChildAt(0);
      measuredWidth = childView.getMeasuredWidth() * childCount;
      setMeasuredDimension(measuredWidth, heightSpaceSize);
    } else {
      final View childView = getChildAt(0);
      measuredWidth = childView.getMeasuredWidth() * childCount;
      measuredHeight = childView.getMeasuredHeight();
      setMeasuredDimension(measuredWidth, measuredHeight);
    }
  }

  @Override
  protected void onLayout(boolean changed, int l, int t, int r, int b) {
    int childLeft = 0;
    final int childCount = getChildCount();
    mChildrenSize = childCount;

    for (int i = 0; i < childCount; i++) {
      final View childView = getChildAt(i);
      if (childView.getVisibility() != View.GONE) {
        final int childWidth = childView.getMeasuredWidth();
        mChildWidth = childWidth;
        childView.layout(childLeft, 0, childLeft + childWidth,
            childView.getMeasuredHeight());
        childLeft += childWidth;
      }
    }
  }

  private void smoothScrollBy(int dx, int dy) {
    mScroller.startScroll(getScrollX(), 0, dx, 0, 500);
    invalidate();
  }

  @Override
  public void computeScroll() {
    if (mScroller.computeScrollOffset()) {
      scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
      postInvalidate();
    }
  }

  @Override
  protected void onDetachedFromWindow() {
    mVelocityTracker.recycle();
    super.onDetachedFromWindow();
  }
}

```
```xml
<com.hdu.night.scrollconflicts.HorizontalScrollViewEx
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal"
    tools:context="com.hdu.night.scrollconflicts.MainActivity">

        <android.support.v7.widget.RecyclerView android:id="@+id/rv_test1"
                                                android:layout_width="match_parent"
                                                android:layout_height="match_parent"
        />
        <android.support.v7.widget.RecyclerView android:id="@+id/rv_test"
                                                android:layout_width="match_parent"
                                                android:layout_height="match_parent"
        />

        <android.support.v7.widget.RecyclerView android:id="@+id/rv_test2"
                                                android:layout_width="match_parent"
                                                android:layout_height="match_parent"
        />

</com.hdu.night.scrollconflicts.HorizontalScrollViewEx>

```
### 内部拦截法
在子view中拦截事件，父viewGroup默认是不拦截任何事件的，所以，当事件传递到子view时。根据偏移方向和速度，如果该事件是需要子view来处理的，那么子view就自己消耗处理，如果该事件不需要由子view来处理，那么就调用getParent().requestDisallowInterceptTouchEvent()方法来通知父viewgroup来拦截 。
伪代码：
```java
//重写子View
@Override
  public boolean dispatchTouchEvent(MotionEvent event) {
    int x = (int) event.getX();
    int y = (int) event.getY();

    switch (event.getAction()) {
      case MotionEvent.ACTION_DOWN: {
        parent.requestDisallowInterceptTouchEvent(true);
        break;
      }
      case MotionEvent.ACTION_MOVE: {
        int deltaX = x - mLastX;
        int deltaY = y - mLastY;
        if (父容器需要此类点击事件) {
          parent.requestDisallowInterceptTouchEvent(false);
        }
        break;
      }
      case MotionEvent.ACTION_UP: {
        break;
      }
      default:
        break;
    }

    mLastX = x;
    mLastY = y;
    return super.dispatchTouchEvent(event);
  }
  //重写父View
  @Override
public boolean onInterceptTouchEvent(MotionEvent event) {

        int action = event.getAction();
        if (action == MotionEvent.ACTION_DOWN) {
            return false;
        } else {
            return true;
        }
     }
```
实例：
```java
//HorizontalEx2
  @Override public boolean onInterceptTouchEvent(MotionEvent ev) {
    boolean intercepted = false;
    int x = (int) ev.getX();
    int y = (int) ev.getY();
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
      mLastX = x;
      mLastY = y;
      if (!mScroller.isFinished()) {
        mScroller.abortAnimation();
        return true;
      }
      return false;
    } else {
      return true;
    }
  }
public class RecyclerViewEv extends RecyclerView {

  private int mLastX = 0;
  private int mLastY = 0;

...
  @Override public boolean dispatchTouchEvent(MotionEvent ev) {
    int x = (int) ev.getX();
    int y = (int) ev.getY();
    switch (ev.getAction()) {
      case MotionEvent.ACTION_DOWN:
        getParent().requestDisallowInterceptTouchEvent(true);
        break;
      case MotionEvent.ACTION_MOVE:
        int dx = x - mLastX;
        int dy = y - mLastY;
        if (Math.abs(dx) > Math.abs(dy)) {
          getParent().requestDisallowInterceptTouchEvent(false);
        }
        break;
      case MotionEvent.ACTION_UP:
        break;
      default:
        break;
    }
    mLastX = x;
    mLastY = y;
    return super.dispatchTouchEvent(ev);
  }
}

```

## 外部滑动与内部滑动一致
可以采取嵌套滑动机制解决。Android 的事件分发机制中，只要有一个控件消费了事件，其他控件就没办法在接收到这个事件了。因此，当有嵌套滑动场景时，我们都需要自己手动解决事件冲突。而在 Android 5.0 Lollipop 之后，Google 官方通过 嵌套滑动机制 解决了传统 Android 事件分发无法共享事件这个问题。

嵌套滑动机制 的基本原理可以认为是事件共享，即当子控件接收到滑动事件，准备要滑动时，会先通知父控件(startNestedScroll）；然后在滑动之前，会先询问父控件是否要滑动（dispatchNestedPreScroll)；如果父控件响应该事件进行了滑动，那么就会通知子控件它具体消耗了多少滑动距离；然后交由子控件处理剩余的滑动距离；最后子控件滑动结束后，如果滑动距离还有剩余，就会再问一下父控件是否需要在继续滑动剩下的距离（dispatchNestedScroll)...

如在ScrollVIew中嵌套RecyclerView。直接ScrollView进行拦截的话会出现RecyclerView内容显示不全。
可以将ScrollView换为NestScrollView，并将RecyclerView的nestedScrollingEnabled设为false。即RecyclerView不参加嵌套滑动，滑动交由NestScrollView负责。享受丝滑般的触感。
