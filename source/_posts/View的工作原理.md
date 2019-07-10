title: View的工作原理
date: 2019-2-21
tags: [Android,View的工作原理]
categories: Android开发艺术探索
description: 好久以前认为高深的技术。盘他
---
## ViewRoot和DecorView之间的交易
ViewRoot对应于ViewRootImpl类，它是连接Window和DecorView（FrameLayout）的纽带，View的三大流程是通过VeiwRoot来完成的。在ActivityThread中，当Activity对象被创建完毕后，会将DecorVeiw添加到Window中，同时会创建ViewRootImpl对象，并将ViewRootImpl对象和DecorView建立关联。

View的绘制流程是从ViewRoot的performTraversals方法开始的，它经过measure、layout和draw三个过程才能最终将一个View绘制出来。如图：
![enter description here](/images/performTraversals.png)
performTraversals会依次调用performMeasure、performLayout、performDraw三个方法，这个三个方法分别完成顶级View的measure、layout、draw这三大方法，performMeasure调用measure(),measure()调用onMeasure().在onMeasure方法中则会对所有的子元素进行measure过程，这个时候measure流程就从父容器传递到子元素中了，这样就完成一次measure过程，接着子元素会重复父容器的过程，如此反复就完成了整个View树的遍历。同理，其他两个步骤也是类似的过程。

measure过程决定了View的宽和高，Measure完成以后，可以通过getMeasureWidth和getMeasureHeight方法来获取到View的测量后的宽和高。
## MeasureSpec
MeasureSpec:测量规格，在很大程度上决定了一个View的尺寸规格，之所以说是很大程度上是因为这个过程还是受父容器的影响，因为父容器影响View的MeasureSpec的创建过程。在测量过程中，系统会将View的LayoutParams根据父容器所施加的规则转换成对应的MeasureSpec，然后再根据这个measureSpec来测量出View的宽和高。

MeasureSpec代表一个32位int值，高2位代表SpecMode，低30位代表SpecSize，SpecMode是指测量模式，而SpecSize是指在某种测量模式下的规格大小。MeasureSpec通过将SpecMode（int）和SpecSize(int)打包成一个int值来避免过多的对象内存分配，为了方便操作，其提供了打包和解包的方法。

### SpecMode 

- __Exactly__: 父容器已经检测出View所需要的精确大小，这个时候View的最终大小就是SpecSize所指定的值。它对应于LayoutParams中的match_parent和具体的数值这两种模式。
- __AT_MOST__:父容器指定了一个可用大小即SpecSize，View的大小不能大于这个值，具体是什么值要看不同的View的具体实现。它对应于LayoutParams中的wrap_content。
- __UNSPECIFIED__:不指定大小测量模式 View任意大小，通常在自定义View时会使用。

当View采用固定宽/高的时候，不管父容器的MeasureSpec是什么，View的MeasureSpec都是EXACTLY模式并且大小遵循LayoutParams中的大小

当View的宽/高是match_parent时，如果父容器的模式是EXACTLY，那么View也是精确模式并且其大小是父容器的剩余空间，如果父容器是最大模式，那么View也是最大模式并且其大小不会超过父容器的剩余空间

当View的宽/高是wrap_content时，不管父容器的模式是精确还是最大化，View的模式总是最大化并且大小不能超过父容器的剩余空间。

### MeasureSpec与layoutParams关系
- DecorView: MeasureSpec是由窗口的尺寸和其自身的LayoutParams确定
- 普通View:MeasureSpec是由父容器的MeasureSpec和自身的LayoutParams共同确定。

## View的工作流程
View的工作流程主要是指measure、layout、draw这三大流程，即测量、布局、绘制，其中measure确定View的测量宽高，layout确定View的最终宽高和四个顶点的位置，而draw则将View绘制到屏幕上。

### measure过程
一个原始的View，那么通过measure方法就完成了其测量过程，如果是一个ViewGroup，除了完成自己的测量过程外，还会遍历去调用所有子元素的measure方法，各个子元素再递归去执行这个流程。
#### View的measure()
View的measure过程由其measure()完成，measure()是一个final方法（不能被子类重写）。measure()中会调用onMeasure()
```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }
```

```java
public static int getDefaultSize(int size, int measureSpec) {
        int result = size;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result;
    }
```
getDefaultSize方法返回的大小就是MeasureSpec中的specSize，而这个specSize就是View测量后的大小，但View的最终大小是在layout阶段确定的，几乎所有情况下的View的测量大小和最终大小是相等的。

同时，直接继承View的自定义控件需要重写onMeasure方法并设置wrap_content时的自身大小，否则在布局中使用wrap_content就相当于使用match_parent。如果View在布局中使用wrap_content，那么它的specMode是AT_MOST模式，在这种模式下，它的宽/高等于specSize，也就是说，这种情况下的View的specSize是parentSize，而parentSize是父容器中目前当前剩余使用的大小，也就是父容器当前剩余的空间大小。
TextView、ImageView等解决方法是重写onMeasure()
```java
   @Override protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);
        if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(200, 200);//Default width and height
        } else if (widthSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(200, heightSpecSize);
        } else if (heightSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(widthSpecSize, 200);
        }
    }
```
#### ViewGroup的measure
ViewGroup是一个抽象类，因此它没有重写View的onMeasure方法，但是它提供了一个叫measureChildren的方法。

```java
   protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
        final int size = mChildrenCount;
        final View[] children = mChildren;
        for (int i = 0; i < size; ++i) {
            final View child = children[i];
            if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
                measureChild(child, widthMeasureSpec, heightMeasureSpec);
            }
        }
    }
	protected void measureChild(View child, int parentWidthMeasureSpec,int parentHeightMeasureSpec) {
        final LayoutParams lp = child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,mPaddingLeft + mPaddingRight, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,mPaddingTop + mPaddingBottom, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```
measureChild的思想就是取出子元素的LayoutParams，结合父容器的MeasureSpec共同确定子View的MeasureSpec.ViewGroup并未定义onMeasure(),因为不同ViewGroup如LinearLayout与FrameLayout有不同的布局特性。不同布局需要实现自己的onMeasure()具体测量方法。

- 在View中正确的获得View的测量宽高：onLayout()


```java
  @Override
  protected void onLayout(boolean changed, int l, int t, int r, int b) {
    view.getMeasuredWidth();
  }

```
- Activity中获取View的宽和高：


```java
  @Override public void onWindowFocusChanged(boolean hasFocus) {
    super.onWindowFocusChanged(hasFocus);
    if (hasFocus) {
      int width = view.getMeasuredWidth();
      int height = view.getMeasuredHeight();
    }
  }
```
### layout过程
Layout的作用是ViewGroup用来确定子元素的位置，当ViewGroup的位置被确定后，它在onLayout中会遍历所有的子元素并调用其layout方法，在layout方法中onLayout方法又会被调用。先看下View中的layout方法

```java
    public void layout(int l, int t, int r, int b) {
        if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
            onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
            mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
        }

        int oldL = mLeft;
        int oldT = mTop;
        int oldB = mBottom;
        int oldR = mRight;

        boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            onLayout(changed, l, t, r, b);

            if (shouldDrawRoundScrollbar()) {
                if(mRoundScrollbarRenderer == null) {
                    mRoundScrollbarRenderer = new RoundScrollbarRenderer(this);
                }
            } else {
                mRoundScrollbarRenderer = null;
            }

            mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnLayoutChangeListeners != null) {
                ArrayList<OnLayoutChangeListener> listenersCopy =
                        (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
                int numListeners = listenersCopy.size();
                for (int i = 0; i < numListeners; ++i) {
                    listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
                }
            }
        }

        mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
        mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;

        if ((mPrivateFlags3 & PFLAG3_NOTIFY_AUTOFILL_ENTER_ON_LAYOUT) != 0) {
            mPrivateFlags3 &= ~PFLAG3_NOTIFY_AUTOFILL_ENTER_ON_LAYOUT;
            notifyEnterOrExitForAutoFillIfNeeded(true);
        }
    }

```
layout方法的大致流程：首先会通过setFrame方法来设定View的四个顶点的位置，即初始化mLeft、mRight、mTop和mBottom这四个值，View的四个顶点一旦确定，那么View在父容器中的位置也就确定了，接着就会调用onLayout方法，这个方法用途是父容器确定子元素的位置，和onMeasure方法类似，onLayout的具体实现同样和具体的布局有关，所以View和ViewGroup均没有真正实现onLayout方法。

关于getMeasuredWIdth()和getWidth()差异：在最终情况即在测量完毕后，是相同的。但在测量过程中可能不同。

### draw过程
1. 绘制背景background.draw(canvas)
2. 绘制自己(onDraw)
3. 绘制children(dispatchDraw)
4. 绘制装饰(onDrawScrollBars)
```java
public void draw(Canvas canvas) {
        if (mClipBounds != null) {
            canvas.clipRect(mClipBounds);
        }
        final int privateFlags = mPrivateFlags;
        final boolean dirtyOpaque = (privateFlags & PFLAG_DIRTY_MASK) == PFLAG_DIRTY_OPAQUE &&
                (mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);
        mPrivateFlags = (privateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;

        /*
         * Draw traversal performs several drawing steps which must be executed
         * in the appropriate order:
         *
         *      1. Draw the background
         *      2. If necessary, save the canvas' layers to prepare for fading
         *      3. Draw view's content
         *      4. Draw children
         *      5. If necessary, draw the fading edges and restore layers
         *      6. Draw decorations (scrollbars for instance)
         */

        // Step 1, draw the background, if needed
        int saveCount;

        if (!dirtyOpaque) {
         ...
                if ((scrollX | scrollY) == 0) {
                    background.draw(canvas);
                } else {
                    canvas.translate(scrollX, scrollY);
                    background.draw(canvas);
                    canvas.translate(-scrollX, -scrollY);
                }
            }
        }

        // skip step 2 & 5 if possible (common case)
        final int viewFlags = mViewFlags;
        boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
        boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
        if (!verticalEdges && !horizontalEdges) {
            // Step 3, draw the content
            if (!dirtyOpaque) onDraw(canvas);

            // Step 4, draw the children
            dispatchDraw(canvas);

            // Step 6, draw decorations (scrollbars)
            onDrawScrollBars(canvas);

            if (mOverlay != null && !mOverlay.isEmpty()) {
                mOverlay.getOverlayView().dispatchDraw(canvas);
            }

            // we're done...
            return;
        }
...
}
```
