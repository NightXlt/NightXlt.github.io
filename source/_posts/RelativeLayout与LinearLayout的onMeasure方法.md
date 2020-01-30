title: RelativeLayout与LinearLayout的onMeasure方法
date: 2019-4-5
tags: [Android]
categories: Android开发艺术探索
description: 两次测绘与一次测绘是怎么实现的？
---
转自： [Android中RelativeLayout和LinearLayout性能分析](https://www.jianshu.com/p/8a7d059da746)
## RelativeLayout的onMeasure()方法
```java
View[] views = mSortedHorizontalChildren;
    int count = views.length;

    for (int i = 0; i < count; i++) {
      View child = views[i];
      if (child.getVisibility() != GONE) {
        LayoutParams params = (LayoutParams) child.getLayoutParams();
        int[] rules = params.getRules(layoutDirection);

        applyHorizontalSizeRules(params, myWidth, rules);
        measureChildHorizontal(child, params, myWidth, myHeight); // Horizontal measure
    ...
    for (int i = 0; i < count; i++) {
      View child = views[i];
      if (child.getVisibility() != GONE) {
        LayoutParams params = (LayoutParams) child.getLayoutParams();
       
        applyVerticalSizeRules(params, myHeight);
        measureChild(child, params, myWidth, myHeight); // Vertical measure
        if (positionChildVertical(child, params, myHeight, isWrapContentHeight)) {
          offsetVerticalAxis = true;
        }
...
    }
```
RelativeLayout 会对子View进行两次measure，因为Relative中的子View的布局是依赖于子View的相互关系，这个顺序可能与layout文件中的子View编写不一致，故在确定子View位置时，先要给子View排一下序。
> 又因为RelativeLayout允许A，B 2个子View，横向上B依赖A，纵向上A依赖B。所以需要横向纵向分别进行一次排序测量。

## LinearLayout的onMeasure()方法
```java
 @Override
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    if (mOrientation == VERTICAL) {
      measureVertical(widthMeasureSpec, heightMeasureSpec);
    } else {
      measureHorizontal(widthMeasureSpec, heightMeasureSpec);
    }
  }
```
进入measureVertical中
```java
for (int i = 0; i < count; ++i) {
      final View child = getVirtualChildAt(i);

      if (child == null) {
        mTotalLength += measureNullChild(i);
        continue;
      }

      if (child.getVisibility() == View.GONE) {
       i += getChildrenSkipCount(child, i);
       continue;
      }

      if (hasDividerBeforeChildAt(i)) {
        mTotalLength += mDividerHeight;
      }

      LinearLayout.LayoutParams lp = (LinearLayout.LayoutParams) child.getLayoutParams();

      totalWeight += lp.weight;
     
      if (heightMode == MeasureSpec.EXACTLY && lp.height == 0 && lp.weight > 0) {
        // Optimization: don't bother measuring children who are going to use
        // leftover space. These views will get measured again down below if
        // there is any leftover space.
        final int totalLength = mTotalLength;
        mTotalLength = Math.max(totalLength, totalLength + lp.topMargin + lp.bottomMargin);
      } else {
        int oldHeight = Integer.MIN_VALUE;

        if (lp.height == 0 && lp.weight > 0) {
          // heightMode is either UNSPECIFIED or AT_MOST, and this
          // child wanted to stretch to fill available space.
          // Translate that to WRAP_CONTENT so that it does not end up
          // with a height of 0
          oldHeight = 0;
          lp.height = LayoutParams.WRAP_CONTENT;
        }

        // Determine how big this child would like to be. If this or
        // previous children have given a weight, then we allow it to
        // use all available space (and we will shrink things later
        // if needed).
        measureChildBeforeLayout(
           child, i, widthMeasureSpec, 0, heightMeasureSpec,
           totalWeight == 0 ? mTotalLength : 0);// Measure child 

        if (oldHeight != Integer.MIN_VALUE) {
         lp.height = oldHeight;
        }

        final int childHeight = child.getMeasuredHeight();
        final int totalLength = mTotalLength;
        mTotalLength = Math.max(totalLength, totalLength + childHeight + lp.topMargin +
           lp.bottomMargin + getNextLocationOffset(child));

        if (useLargestChild) {
          largestChildHeight = Math.max(childHeight, largestChildHeight);
        }
      }
```
父视图在对子视图进行measure操作的过程中，使用变量mTotalLength保存已经measure过的child所占用的高度，该变量刚开始时是0。在for循环中调用measureChildBeforeLayout（）对每一个child进行测量;  每次for循环对child测量完毕后，调用child.getMeasuredHeight()获取该子视图最终的高度，并将这个高度添加到mTotalLength中。在本步骤中，暂时避开了lp.weight>0的子视图，即暂时先不测量这些子视图，因为后面将把父视图剩余的高度按照weight值的大小平均分配给相应的子视图。
源码中使用了一个局部变量totalWeight累计所有子视图的weight值。处理lp.weight>0的情况需要注意，如果变量`heightMode是EXACTLY`，那么，当其他子视图占满父视图的高度后，weight>0的子视图`分配不到布局空间`，从而不被显示，只有当heightMode是AT_MOST或者UNSPECIFIED时，weight>0的视图才能优先获得布局高度。最后我们的结论是：如果`不使用weight属性`，LinearLayout会在当前方向上进行`一次measure`的过程，`如果使用weight属性`，LinearLayout会避开设置过weight属性的view做第一次measure，完了再对设置过weight属性的view做`第二次measure`。由此可见，weight属性对性能是有影响的，而且本身有大坑，请注意避让。

## 结论
1. RelativeLayout慢于LinearLayout是因为它会让子View调用2次measure过程，而后者只需一次，但是有weight属性存在时，后者同样会进行两次measure。
2. 无嵌套布局的情况使用LinearLayout,否则使用RelativeLayout
3. DecorView的层级深度已知且固定的，上面一个标题栏，下面一个内容栏，采用RelativeLayout并不会降低层级深度，因此这种情况下使用LinearLayout效率更高。而为开发者默认新建RelativeLayout是希望开发者能采用尽量少的View层级，很多效果是需要多层LinearLayout的嵌套，这必然不如一层的RelativeLayout性能更好。因此我们应该尽量减少布局嵌套，减少层级结构。

