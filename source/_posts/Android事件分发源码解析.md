title: Android事件分发源码解析
date: 2017-08-20 00:00:00
tags: [Android]
categories: 进阶之光
description: 事件分发机制
---
## View的事件分发机制

### 事件分发机制图解

 ![该图示自上而下有些许错误，请参考底部图解](/images/事件分发机制图解.png)

 由上图可看出所涉及到了三个重要的方法

 * dispatchTouchEvent(MotionEvent ev):用于事件的分发

 * onIerveptTouchEvent(MotionEvent ev):用来进行事件的拦截，
   在dispatchTouchEvent()中进行调用，且View没有该方法，只有ViewGroup有。

 * onTouchEvent(MotionEvent ev):用来处理点击事件，
   在dispatchTouchEvent()中进行调用。


  `事件分发`：当我们点击屏幕时，就产生了点击事件，该事件被封装成了一个MotionEvent类。系统会将MotionEvent传递给View的层级，MotionEvent在View中的层级传递过程就是事件分发。

### View的事件分发机制

  当点击事件产生后，事件会首先传递给Activity，Activity会调用dispatchEvent()，将事件分发给phoneWindow,phoneWindow再分发给DecorView，最后由DecorView分发给根ViewGroup.
  
  进入ViewGroup的dispatchTouchEvent

```java
@Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
 ...
            if (actionMasked == MotionEvent.ACTION_DOWN) {//判断是否是Down事件，是则进行初始化mFirstTouchEvent
                cancelAndClearTouchTargets(ev);
                resetTouchState();
            }

            final boolean intercepted;
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {//mFirstTouchEvent表示当前ViewGroup是否拦截了事件，mFirstTouchEvent！=null表示不拦截
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;//FLAG_DISALLOW_INTERCEPT：禁止ViewGroup拦截除了Down之外的事件，一般可通过子View的requestDisallowInterceptTouchEvent()来设置
                if (!disallowIntercept) {//进行拦截
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); // restore action in case it was changed
                } else {
                    intercepted = false;
                }
            } else {//既不是touch目标，也不是down事件，则进行拦截，如ACTION_UP,ACTION_MOVE
                         intercepted = true;
            }
              ...
                        final View[] children = mChildren;
                        for (int i = childrenCount - 1; i >= 0; i--) {
       //倒序遍历子View（即从最外层开始遍历），判断其是否能接收到点击事件，如果能，则交由子View来进行处理，
                            final int childIndex = getAndVerifyPreorderedIndex(
                                    childrenCount, i, customOrder);
                            final View child = getAndVerifyPreorderedView(
                                    preorderedList, children, childIndex);

                        
                            if (childWithAccessibilityFocus != null) {
                                if (childWithAccessibilityFocus != child) {
                                    continue;
                                }
                                childWithAccessibilityFocus = null;
                                i = childrenCount - 1;
                            }

                            if (!canViewReceivePointerEvents(child)
                                    || !isTransformedTouchPointInView(x, y, child, null)) {
     //判断子View是否在播放动画或者触摸点位置是否在子View的范围里，均不满足则continue
                                ev.setTargetAccessibilityFocus(false);
                                continue;
                            }

                            newTouchTarget = getTouchTarget(child);
                            if (newTouchTarget != null) {
                                // Child is already receiving touch within its bounds.
                                // Give it the new pointer in addition to the ones it is handling.
                                newTouchTarget.pointerIdBits |= idBitsToAssign;
                                break;
                            }

                            resetCancelNextUpFlag(child);
                            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {//转换触控分发事件
                                // Child wants to receive touch within its bounds.
                                mLastTouchDownTime = ev.getDownTime();
                                if (preorderedList != null) {
                                    // childIndex points into presorted list, find original index
                                    for (int j = 0; j < childrenCount; j++) {
                                        if (children[childIndex] == mChildren[j]) {
                                            mLastTouchDownIndex = j;
                                            break;
                                        }
                                    }
                                } else {
                                    mLastTouchDownIndex = childIndex;
                                }
                                mLastTouchDownX = ev.getX();
                                mLastTouchDownY = ev.getY();
                                newTouchTarget = addTouchTarget(child, idBitsToAssign);
                                alreadyDispatchedToNewTouchTarget = true;
                                break;
                            }
                            ev.setTargetAccessibilityFocus(false);
                        }
                       
```

  当ViewGroup要拦截事件的时候，后续事件全部交由它处理，而不用再调用onInterceptTouchEvent()进行判断，因此onInterceptTouchEvent()并不是每次事件都会执行。

```java
  public boolean onInterceptTouchEvent(MotionEvent ev) {
        if (ev.isFromSource(InputDevice.SOURCE_MOUSE)
                && ev.getAction() == MotionEvent.ACTION_DOWN
                && ev.isButtonPressed(MotionEvent.BUTTON_PRIMARY)
                && isOnScrollbarThumb(ev.getX(), ev.getY())) {
            return true;
        }
        return false;
    }
```

  onInterceptTouchEvent()默认返回false，不进行拦截，可在ViewGroup中重写该方法进行拦截。

  查看上述转换触控分发事件dispatchTransformedTouchEvent()：

```java
 private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
            View child, int desiredPointerIdBits) {
        final boolean handled;

        // Canceling motions is a special case.  We don't need to perform any transformations
        // or filtering.  The important part is the action, not the contents.
        final int oldAction = event.getAction();
        if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
            event.setAction(MotionEvent.ACTION_CANCEL);
            if (child == null) {
                handled = super.dispatchTouchEvent(event);
            } else {
                handled = child.dispatchTouchEvent(event);
            }
            event.setAction(oldAction);
            return handled;
        }
        
    }
```

   如果viewGroup有子View，则调用子View的dispatchTouchEvent(event),没有则调用super.dispatchTouchEvent(event)，因为ViewGroup是继承自View，进入View的dispatchTouchEvent(event)

```java
public boolean dispatchTouchEvent(MotionEvent event) {
        if (event.isTargetAccessibilityFocus()) {
            // We don't have focus or no virtual descendant has it, do not handle the event.
            if (!isAccessibilityFocusedViewOrHost()) {
                return false;
            }
            // We have focus and got the event, then use normal event dispatch.
            event.setTargetAccessibilityFocus(false);
        }

        boolean result = false;

        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(event, 0);
        }

        final int actionMasked = event.getActionMasked();
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // Defensive cleanup for new gesture
            stopNestedScroll();
        }

        if (onFilterTouchEventForSecurity(event)) {
            if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
                result = true;
            }
            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }

            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }
        ...
        return result;
    }
```
  
  如果OnTouchListener不为null且onTouch()返回为true，则表示事件被消费，不会执行onTouchEvent(event),否则就会执行onTouchEvent(event).根据执行顺序可看出OnTouchListener.onTouch()优先级>onTouchEvent(event).下面进入omTouchEvent()。

```java
public boolean onTouchEvent(MotionEvent event) {
        final float x = event.getX();
        final float y = event.getY();
        final int viewFlags = mViewFlags;
        final int action = event.getAction();

        final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
                || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;

        if ((viewFlags & ENABLED_MASK) == DISABLED) {
            if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
                setPressed(false);
            }
            mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
            // A disabled view that is clickable still consumes the touch
            // events, it just doesn't respond to them.
            return clickable;
        }
        if (mTouchDelegate != null) {
            if (mTouchDelegate.onTouchEvent(event)) {
                return true;
            }
        }

        if (clickable || (viewFlags & TOOLTIP) == TOOLTIP) {
            switch (action) {
                case MotionEvent.ACTION_UP:
                    mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
                    if ((viewFlags & TOOLTIP) == TOOLTIP) {
                        handleTooltipUp();
                    }
                    if (!clickable) {
                        removeTapCallback();
                        removeLongPressCallback();
                        mInContextButtonPress = false;
                        mHasPerformedLongPress = false;
                        mIgnoreNextUpEvent = false;
                        break;
                    }
                    boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
                    if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                        // take focus if we don't have it already and we should in
                        // touch mode.
                        boolean focusTaken = false;
                        if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                            focusTaken = requestFocus();
                        }

                        if (prepressed) {
                            // The button is being released before we actually
                            // showed it as pressed.  Make it show the pressed
                            // state now (before scheduling the click) to ensure
                            // the user sees it.
                            setPressed(true, x, y);
                        }

                        if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                            // This is a tap, so remove the longpress check
                            removeLongPressCallback();

                            // Only perform take click actions if we were in the pressed state
                            if (!focusTaken) {
                                // Use a Runnable and post this rather than calling
                                // performClick directly. This lets other visual state
                                // of the view update before click actions start.
                                if (mPerformClick == null) {
                                    mPerformClick = new PerformClick();
                                }
                                if (!post(mPerformClick)) {
                                    performClick();
                                }
                            }
                        }

                        ...
            }

            return true;
        }

        return false;
    }
``` 

  只要View的CLICKABLE和LONG_CLICKABLE有一个为true，那么onTouchEvent()就会返回true消费这个事件。只要设置View设置了对应的监听事件setOnClickListener和setOnLongClickListener来设置，其会自动将对应属性设置为true。进入至performClick()。

```java
 public boolean performClick() {
        final boolean result;
        final ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnClickListener != null) {//如果view设置了onClickListener，则就会执行其onClick()，
            playSoundEffect(SoundEffectConstants.CLICK);
            li.mOnClickListener.onClick(this);
            result = true;
        } else {
            result = false;
        }

        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);

        notifyEnterOrExitForAutoFillIfNeeded(true);

        return result;
    }
```  

## 点击事件分发的传递原则
   
   ![事件分发自上而下分发](/images/事件分发.png)

  根ViewGroupdispatchTouchEvent()遍历子View询问是否允许拦截，不允许则将事件分发给子View的dispatchTouchEvent()，允许则进入onInterceptTouchEvent(event)判断自己是否要拦截这个事件，不拦截则将事件分发给子View的dispatchTouchEvent()，拦截则进入自己的onTouchEvent()进行处理。当传递到子View，如果其onTouchEvent返回true表明其处理了该事件，返回false，会将事件传递给父View进行处理。
