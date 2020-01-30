title: Scroller、ScrollTo、ScrollBy源码解析
date: 2019-2-2
tags: [Android,View事件体系,源码解析]
categories: Android开发艺术探索
description: 咦，移动方向怎么不对呀！
---
## 序言
 　　本着写了忘，忘了写。还是写篇博文分析了工作流程加深记忆，滑动方向每次都出错。
## ScrollTo与ScrollBy两兄弟
```java
/*Set scrollX,scrollY.inValidate 
   mSrollX:The offset, in pixels, by which the content of this view is scrolled horizontally.
   mSrollY:The offset, in pixels, by which the content of this view is scrolled
vertically.
*/
    public void scrollTo(int x, int y) {
        if (mScrollX != x || mScrollY != y) {
            int oldX = mScrollX;
            int oldY = mScrollY;
            mScrollX = x;
            mScrollY = y;
            invalidateParentCaches();
            onScrollChanged(mScrollX, mScrollY, oldX, oldY);
            if (!awakenScrollBars()) {
                postInvalidateOnAnimation();
            }
        }
    }
	    public void scrollBy(int x, int y) {
        scrollTo(mScrollX + x, mScrollY + y);
    }

```
可以看出来ScrollBy是调用scrollTo.这样只需要分析scrollTo源码即可。先设置mScrollX,mScrollY.而ScrollX，Y是View内容的滑动距离。所以这就解释为什么只能ViewGroup调用该方法有效。Scroll传入偏移距离加上起始滑动距离 = 绝对距离。进入onScrollChanged方法
```java
protected void onScrollChanged(int l, int t, int oldl, int oldt) {
        notifySubtreeAccessibilityStateChangedIfNeeded();

        if (AccessibilityManager.getInstance(mContext).isEnabled()) {
            postSendViewScrolledAccessibilityEventCallback();
        }

        mBackgroundSizeChanged = true;
        mDefaultFocusHighlightSizeChanged = true;
        if (mForegroundInfo != null) {
            mForegroundInfo.mBoundsChanged = true;
        }

        final AttachInfo ai = mAttachInfo;
        if (ai != null) {
            ai.mViewScrollChanged = true;
        }

        if (mListenerInfo != null && mListenerInfo.mOnScrollChangeListener != null) {
            mListenerInfo.mOnScrollChangeListener.onScrollChange(this, l, t, oldl, oldt);
        }
    }
```
当View注册了OnScrollChangeListener，onScrollChange才会被调用。在onScrollChanged()后调用了postInvalidateOnAnimation（）。
```java
    public void postInvalidateOnAnimation() {
        // We try only with the AttachInfo because there's no point in invalidating
        // if we are not attached to our window
        final AttachInfo attachInfo = mAttachInfo;
        if (attachInfo != null) {
            attachInfo.mViewRootImpl.dispatchInvalidateOnAnimation(this);
        }
    }
	 public void dispatchInvalidateOnAnimation(View view) {
        mInvalidateOnAnimationRunnable.addView(view);
    }
	    final class InvalidateOnAnimationRunnable implements Runnable {
        private boolean mPosted;
        private final ArrayList<View> mViews = new ArrayList<View>();
        private final ArrayList<AttachInfo.InvalidateInfo> mViewRects =
                new ArrayList<AttachInfo.InvalidateInfo>();
        private View[] mTempViews;
        private AttachInfo.InvalidateInfo[] mTempViewRects;

        public void addView(View view) {
            synchronized (this) {
                mViews.add(view);
                postIfNeededLocked();
            }
        }

        public void addViewRect(AttachInfo.InvalidateInfo info) {
            synchronized (this) {
                mViewRects.add(info);
                postIfNeededLocked();
            }
        }

        public void removeView(View view) {
            synchronized (this) {
                mViews.remove(view);

                for (int i = mViewRects.size(); i-- > 0; ) {
                    AttachInfo.InvalidateInfo info = mViewRects.get(i);
                    if (info.target == view) {
                        mViewRects.remove(i);
                        info.recycle();
                    }
                }

                if (mPosted && mViews.isEmpty() && mViewRects.isEmpty()) {
                    mChoreographer.removeCallbacks(Choreographer.CALLBACK_ANIMATION, this, null);
                    mPosted = false;
                }
            }
        }

        @Override
        public void run() {
            final int viewCount;
            final int viewRectCount;
            synchronized (this) {
                mPosted = false;

                viewCount = mViews.size();
                if (viewCount != 0) {
                    mTempViews = mViews.toArray(mTempViews != null
                            ? mTempViews : new View[viewCount]);
                    mViews.clear();
                }

                viewRectCount = mViewRects.size();
                if (viewRectCount != 0) {
                    mTempViewRects = mViewRects.toArray(mTempViewRects != null
                            ? mTempViewRects : new AttachInfo.InvalidateInfo[viewRectCount]);
                    mViewRects.clear();
                }
            }

            for (int i = 0; i < viewCount; i++) {
                mTempViews[i].invalidate();
                mTempViews[i] = null;
            }

            for (int i = 0; i < viewRectCount; i++) {
                final View.AttachInfo.InvalidateInfo info = mTempViewRects[i];
                info.target.invalidate(info.left, info.top, info.right, info.bottom);
                info.recycle();
            }
        }

        private void postIfNeededLocked() {
            if (!mPosted) {
                mChoreographer.postCallback(Choreographer.CALLBACK_ANIMATION, this, null);
                mPosted = true;
            }
        }
    }
```
在addView中postIfNeededLocked()会调用到run(),在run()里遍历所有添加进去的View，然后对每个View调用invalidate()，
```java
   public void invalidate() {
        invalidate(true);
    }
	    public void invalidate(boolean invalidateCache) {
        invalidateInternal(0, 0, mRight - mLeft, mBottom - mTop, invalidateCache, true);
    }
void invalidateInternal(int l, int t, int r, int b, boolean invalidateCache,
            boolean fullInvalidate) {
    ...
            // Propagate the damage rectangle to the parent view.
            final AttachInfo ai = mAttachInfo;
            final ViewParent p = mParent;
            if (p != null && ai != null && l < r && t < b) {
                final Rect damage = ai.mTmpInvalRect;
                damage.set(l, t, r, b);
                p.invalidateChild(this, damage);// Invalidate child
            }

            // Damage the entire projection receiver, if necessary.
            if (mBackground != null && mBackground.isProjected()) {
                final View receiver = getProjectionReceiver();
                if (receiver != null) {
                    receiver.damageInParent();
                }
            }
        }
    }
```
p.invalidateChild()这就是scrollTo移动的是View的内容的真相。再来揭示为何方向相反呢?进入ViewGroup的invalidateChild().
```java
@Deprecated
    @Override
    public final void invalidateChild(View child, final Rect dirty) {
    ···
                parent = parent.invalidateChildInParent(location, dirty);
                if (view != null) {
                    // Account for transform on current parent
                    Matrix m = view.getMatrix();
                    if (!m.isIdentity()) {
                        RectF boundingRect = attachInfo.mTmpTransformRect;
                        boundingRect.set(dirty);
                        m.mapRect(boundingRect);
                        dirty.set((int) Math.floor(boundingRect.left),
                                (int) Math.floor(boundingRect.top),
                                (int) Math.ceil(boundingRect.right),
                                (int) Math.ceil(boundingRect.bottom));
                    }
                }
            } while (parent != null);
        }
    }
```
里面有一行代码parent = parent.invalidateChildInParent(location, dirty);进去invalidateChildInParent（）
```java
  @Override
        public ViewParent invalidateChildInParent(int[] location, Rect dirty) {
            if (mHostView != null) {
                dirty.offset(location[0], location[1]);
                if (mHostView instanceof ViewGroup) {
                    location[0] = 0;
                    location[1] = 0;
                    super.invalidateChildInParent(location, dirty);
                    return ((ViewGroup) mHostView).invalidateChildInParent(location, dirty);
                } else {
                    invalidate(dirty);
                }
            }
            return null;
        }
    }
```
进入 invalidate(dirty);
```java
    public void invalidate(Rect dirty) {
        final int scrollX = mScrollX;
        final int scrollY = mScrollY;
        invalidateInternal(dirty.left - scrollX, dirty.top - scrollY,
                dirty.right - scrollX, dirty.bottom - scrollY, true, false);
    }
```
这就是为什么方向会相反的原因啦。整个scrollTo透露着一股观察者模式的气味，当调用scrollTo（）时，InvalidateOnAnimationRunnable中的run()会通知所有子View。让其重绘。

## Scroller源码
Scroller典型用法：
```java
@Override
public void computeScroll() {
    super.computeScroll();
    // 判断Scroller是否执行完毕
    if (mScroller.computeScrollOffset()) {
        ((View) getParent()).scrollTo(
                mScroller.getCurrX(),//获取当前的滑动坐标
                mScroller.getCurrY());
        // 通过重绘来不断调用computeScroll
        postInvalidate();
    }
}

@Override
public boolean onTouchEvent(MotionEvent event) {
    int x = (int) event.getX();
    int y = (int) event.getY();
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            lastX = (int) event.getX();
            lastY = (int) event.getY();
            break;
        case MotionEvent.ACTION_MOVE:
            int offsetX = x - lastX;
            int offsetY = y - lastY;
            ((View) getParent()).scrollBy(-offsetX, -offsetY);
            break;
        case MotionEvent.ACTION_UP:
            // 手指离开时，执行滑动过程
            View viewGroup = ((View) getParent());
            mScroller.startScroll(
                    viewGroup.getScrollX(),
                    viewGroup.getScrollY(),
                    -viewGroup.getScrollX(),
                    -viewGroup.getScrollY());
            invalidate();
            break;
    }
    return true;
}
```
与scrollTo（）、scrollBy（）同样，滑动的是View的内容。首先进入startScroll（）

```java
    public void startScroll(int startX, int startY, int dx, int dy, int duration) {
        mMode = SCROLL_MODE;
        mFinished = false;
        mDuration = duration;
        mStartTime = AnimationUtils.currentAnimationTimeMillis();
        mStartX = startX;
        mStartY = startY;
        mFinalX = startX + dx;
        mFinalY = startY + dy;
        mDeltaX = dx;
        mDeltaY = dy;
        mDurationReciprocal = 1.0f / (float) mDuration;
    }
```
里面除了设值什么也没做，那Scroller是怎么让View滑起来的呢？秘密就在startScroll（）的下一行invalidate()重绘中调用View中的draw()
```java
    boolean draw(Canvas canvas, ViewGroup parent, long drawingTime) {
     ...
	 if (!drawingWithRenderNode) {
            computeScroll();
            sx = mScrollX;
            sy = mScrollY;
        }

  ```
  里面会调用我们重写的computeScroll(),而在我们重写的computeScroll()里我们利用scrollTo完成了滑动操作。并同时启动了postInvalidate();开启了第二次重绘。反复直至滑动结束。最后再看看computeScrollOffset（）
 ```java
     public boolean computeScrollOffset() {
        if (mFinished) {
            return false;
        }

        int timePassed = (int)(AnimationUtils.currentAnimationTimeMillis() - mStartTime);
    
        if (timePassed < mDuration) {
            switch (mMode) {
            case SCROLL_MODE:
                final float x = mInterpolator.getInterpolation(timePassed * mDurationReciprocal);
                mCurrX = mStartX + Math.round(x * mDeltaX);
                mCurrY = mStartY + Math.round(x * mDeltaY);
                break;
  ...
    }
        return true;

 ```
根据时间流逝的百分比算出scrollX和scrollY。返回为true意味着滑动还未结束，false：滑动已结束
