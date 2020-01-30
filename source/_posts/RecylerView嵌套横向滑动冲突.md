title: RecylerView中嵌套横向滑动冲突
date: 2017-05-09 00:00:00
tags: [Android]
categories: bug修复
description: 耗时两天的修复...
---
## Bug描述

  在RecylerView中的item存在横向滑动，(PS:item的横向滑动通过属性动画实现)可是参照网上已存的教程要么是item不能垂直滑动，要么item不能水平滑动，这就相当恼人。结果在和朋友交流中发现，不能简单的通过覆写父类进行外部拦截，应该要从子View进行内部拦截。

## 解决代码

```java
public class TopAreaContainer extends RelativeLayout {

    private AvatarsContainers mAvatarsContainers;


    private ViewGroup parent;
    private int mLastX;
    private int mLastY;
    private boolean state;
    private int touchSlop;

    public PostTopAreaContainer(Context context) {
        super(context);
    }

    public PostTopAreaContainer(Context context, AttributeSet attrs) {
        super(context, attrs);
        touchSlop = ViewConfiguration.get(context).getScaledEdgeSlop();
    }


    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        if (mAvatarsContainers.getChildCount() == 1) {
            return true;
        }
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                parent.requestDisallowInterceptTouchEvent(true);//默认按下时不让父View进行拦截
                break;
            case MotionEvent.ACTION_MOVE:
                int dx = x - mLastX;
                int dy = y - mLastY;

                if ((Math.abs(dx) < Math.abs(dy)) && (Math.abs(dy) > touchSlop)) {
                    state = true;
                } else {
                    state = false;
                }

                if (state) {//垂直滑动则让父View拦截本次事件
                    parent.requestDisallowInterceptTouchEvent(false);
                } else {//横向滑动
                    if (deltaX < 0) {
                        mAvatarsContainers.shrink();
                    } else {
                        mAvatarsContainers.expand();
                    }
                }
                break;
            case MotionEvent.ACTION_UP:
                break;
        }

        mLastX = x;
        mLastY = y;
        return true;
    }
}

```

  里面还有个坑就是之前和朋友讨论用int值记录状态，就会出现迷之滑动，在上下滑的同时也在左右滑，但是左右滑时却是正常的。Bug代码：

```java
   public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                parent.requestDisallowInterceptTouchEvent(true);//请求父View(RecylerView)不进行拦截
                state = 0;
                break;
            case MotionEvent.ACTION_MOVE:
                int deltaX = x - mLastX;
                int deltaY = y - mLastY;

                if (state == 0) {
                    if (Math.abs(deltaX) < Math.abs(deltaY)) {
                        state = 1;
                    } else {
                        state = 2;
                    }
                }

                if (state == 1) {
                    parent.requestDisallowInterceptTouchEvent(false);
                } else {
                    if (deltaX < 0) {
                        Log.i(TAG, "左滑");
                    } else {
                        Log.i(TAG, "右滑");
                    }
                }
                break;
            case MotionEvent.ACTION_UP :
                break;
        }

        mLastX = x;
        mLastY = y;
        return true;
    }
```

  调试后发现问题出在if(state==0)这里，因为在state=0情况只会出现在第一次按下的情况，后续滑动state都是不等于0的，所以后续滑动的state都没有再变化过了，只会重复第一次的state，所以采取直线救国,直接将state的改为boolean类型，并取消之前的判断就阔以啦！！！嗨呀，算是交代了这个Bug了.....

  那问题是出自哪尼？为什么横向滑动会惨遭recylerview拦截呢？欢迎走入recylerviewonInterceptTouchEvent源码解析。

## recylerviewonInterceptTouchEvent源码

```java
@Override
    public boolean onInterceptTouchEvent(MotionEvent e) {
       ...
        if (dispatchOnItemTouchIntercept(e)) {
            cancelTouch();
            return true;
        }

        if (mLayout == null) {
            return false;
        }

        final boolean canScrollHorizontally = mLayout.canScrollHorizontally();
        final boolean canScrollVertically = mLayout.canScrollVertically();

        if (mVelocityTracker == null) {
            mVelocityTracker = VelocityTracker.obtain();
        }
        mVelocityTracker.addMovement(e);

        final int action = e.getActionMasked();
        final int actionIndex = e.getActionIndex();

        switch (action) {
            case MotionEvent.ACTION_DOWN:
                if (mIgnoreMotionEventTillDown) {
                    mIgnoreMotionEventTillDown = false;
                }
                mScrollPointerId = e.getPointerId(0);
                mInitialTouchX = mLastTouchX = (int) (e.getX() + 0.5f);
                mInitialTouchY = mLastTouchY = (int) (e.getY() + 0.5f);

                if (mScrollState == SCROLL_STATE_SETTLING) {
                    getParent().requestDisallowInterceptTouchEvent(true);
                    setScrollState(SCROLL_STATE_DRAGGING);
                }

                // Clear the nested offsets
                mNestedOffsets[0] = mNestedOffsets[1] = 0;

                int nestedScrollAxis = ViewCompat.SCROLL_AXIS_NONE;
                if (canScrollHorizontally) {
                    nestedScrollAxis |= ViewCompat.SCROLL_AXIS_HORIZONTAL;
                }
                if (canScrollVertically) {
                    nestedScrollAxis |= ViewCompat.SCROLL_AXIS_VERTICAL;
                }
                startNestedScroll(nestedScrollAxis, TYPE_TOUCH);
                break;

            case MotionEvent.ACTION_POINTER_DOWN:
                mScrollPointerId = e.getPointerId(actionIndex);
                mInitialTouchX = mLastTouchX = (int) (e.getX(actionIndex) + 0.5f);
                mInitialTouchY = mLastTouchY = (int) (e.getY(actionIndex) + 0.5f);
                break;

            case MotionEvent.ACTION_MOVE: {
                final int index = e.findPointerIndex(mScrollPointerId);
                if (index < 0) {
                    Log.e(TAG, "Error processing scroll; pointer index for id "
                            + mScrollPointerId + " not found. Did any MotionEvents get skipped?");
                    return false;
                }

                final int x = (int) (e.getX(index) + 0.5f);
                final int y = (int) (e.getY(index) + 0.5f);
                if (mScrollState != SCROLL_STATE_DRAGGING) {
                    final int dx = x - mInitialTouchX;
                    final int dy = y - mInitialTouchY;
                    boolean startScroll = false;
                    if (canScrollHorizontally && Math.abs(dx) > mTouchSlop) {//罪魁祸首
                        mLastTouchX = x;
                        startScroll = true;
                    }
                    if (canScrollVertically && Math.abs(dy) > mTouchSlop) {
                        mLastTouchY = y;
                        startScroll = true;
                    }
                    if (startScroll) {
                        setScrollState(SCROLL_STATE_DRAGGING);
                    }
                }
            } break;          
        }
        return mScrollState == SCROLL_STATE_DRAGGING;
    }
```  

  罪魁祸首不难看出是其进入了recylerview中的if (canScrollHorizontally && Math.abs(dx) > mTouchSlop)然后描述性地设置了startScroll为true，startScroll为true设置了mScrollState = SCROLL_STATE_DRAGGING,所以最后返回true拦截了子View的事件。

  之前的事件分发顺序有遗漏，在从上自下中，通过requestDisallowInterceptTouchEvent()会先询问子View是否允许父View拦截事件，如果允许则进入onIntercept()，否则将事件分发给子View的dispatchEvent().

  本文参考至[http://blog.csdn.net/qq_36523667/article/details/79223576](http://blog.csdn.net/qq_36523667/article/details/79223576)