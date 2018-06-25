---
title: Android艺术开发探索---View的事件体系
date: 2018-05-03 11:10:29
tags: Android
---

# View基础知识

## MotionEvent 和 TouchSlop

1. MotionEvent
    - ACTION_DOWN:手指刚接触屏幕
    - ACTION_MOVE:手指在屏幕上移动
    - ACTION_UP:手指在屏幕上松开的一瞬间
2. TouchSlop
TouchSlop:系统所能识别出来的被认为是滑动的最小距离。换句话说，当手指在屏幕上滑动时，如果两次滑动之间的距离小于这个常量，那么系统就不认为你是在进行滑动操作。

<!--more-->

# View的滑动

实现View滑动的三种方式：使用scrollTo/scrollBy方法、通过动画、通过改变View的LayoutParams

## 使用scrollTo/scrollBy方法

```
/**
     * Set the scrolled position of your view. This will cause a call to
     * {@link #onScrollChanged(int, int, int, int)} and the view will be
     * invalidated.
     * @param x the x position to scroll to
     * @param y the y position to scroll to
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
```

```
/**
     * Move the scrolled position of your view. This will cause a call to
     * {@link #onScrollChanged(int, int, int, int)} and the view will be
     * invalidated.
     * @param x the amount of pixels to scroll by horizontally
     * @param y the amount of pixels to scroll by vertically
     */
    public void scrollBy(int x, int y) {
        scrollTo(mScrollX + x, mScrollY + y);
    }
```
scrollBy是基于当前位置的相对滑动，而scrollTo则是基于所传递参数的绝对滑动。
在滑动过程中，mScrollX的值总是等于View左边缘和View内容左边缘在水平方向的距离，而mScrollY的值总是等于View上边缘和View内容上边缘在竖直方向的距离。

<center>![](http://p2g00vblr.bkt.clouddn.com/mScrollX%E5%92%8CmScrollY%E7%9A%84%E5%8F%98%E5%8C%96%E8%A7%84%E5%BE%8B%E7%A4%BA%E6%84%8F%E5%9B%BE.png)</center>

## 使用动画

此动画可以再100ms内讲一个View从原始位置向右下角移动100个

### 使用View动画

```
<translate
    android:duration="100"
    android:fromXDelta = "0"
    android:fromYDelya = "0"
    andorid:interpolator = "@android:anim/linear_interpolator"
    android:toXDelta="100"
    android:toYDelta="100">
```

新的位置只是原来的一个影像，只能在新的位置在重新建一个与原来一样的View

### 使用属性动画

```
ObjectAnimator.ofFloat(targetView,"translationX",0,100).setDuration(100).start();
```

## 改变布局参数

改变布局参数即改变LayoutParams

```
MarginLayoutParams params = (MarginLayoutParams)mButton1.getLayoutParams();
params.width += 100;
params.leftMargin += 100;
mbutton1.requestLayout();
```

## 结论

1. scrollTo/scrollBy:操作简单，适合对View内容的滑动
2. 动画：操作简单，主要适用于没有交互的View和实现复杂的动画效果
3. 改变布局参数：操作稍微复杂，适合于有交互的View

# 弹性滑动

将一次大的滑动分成若干次小的滑动并在一个时间段内完成

## 使用Scroller
## 通过动画
## 使用延时策略

# View的事件分发机制

之前有过介绍。

# View的滑动冲突

滑动冲突的三种方式
1. 外部滑动方向和内部滑动方向不一致
2. 外部滑动方向和内部滑动方向一致
3. 上面两种情况的嵌套

<center>![](http://p2g00vblr.bkt.clouddn.com/%E6%BB%91%E5%8A%A8%E5%86%B2%E7%AA%81.png)</center>

## 解决冲突的两种方式

### 外部拦截法

点击事件都是先经过父容器的拦截处理，如果父容器需要此事件就拦截，否则就不拦截

实现这个方法主要是重写父容器的`onInterceptTouchEvent`方法
```
@Override
    public boolean onInterceptTouchEvent(MotionEvent event) {
        boolean intercepted=false;
        int x=(int)event.getX();
        int y=(int)event.getY();
        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                intercepted=false;
                break;
            case MotionEvent.ACTION_MOVE:
                if(父容器需要当前点击事件){
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
        mLastYIntercept=y;
        return intercepted;
    }
```

### 内部拦截法

父容器不拦截任何事件，所有的事件都传递给子容器，如果子容器需要那就去消耗掉，否则就交给父容器进行处理.需要配合`requestDisallowInterceptTouchEvent`方法才可以正常工作，并且同时需要重写子容器的`dispatchTouchEvent`方法

子容器
```
@Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        int x=(int)event.getX();
        int y=(int)event.getY();
        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                getParent().requestDisallowInterceptTouchEvent(true);
                break;
            case MotionEvent.ACTION_MOVE:
                int deltaX=x-mLastX;
                int deltaY=y-mLastY;
                if(父容器需要当前点击事件){
                    getParent().requestDisallowInterceptTouchEvent(false);
                }
                break;
            case MotionEvent.ACTION_UP:
                break;
            default:
                break;
        }
        mLastX=x;
        mLastY=y;
        return super.dispatchTouchEvent(event);
    }
```

父容器
```
@Override
    public boolean onInterceptTouchEvent(MotionEvent event) {
        int action=event.getAction();
        if (action==MotionEvent.ACTION_DOWN){
            return false;
        }else{
            return true;
        }
    }
```

上面的三种场景都可以采用这两种处理方法来解决，按需索取。