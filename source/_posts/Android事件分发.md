---
title: Android事件分发
date: 2021-04-25 14:32:58
tags: Android
---
一个事件产生后，事件先传递到Activity，再传到ViewGroup，最终传到View.

其主要通过一下三个方法完成事件传递：

|方法|作用|调用时刻|
|----|----|----|
|dispatchTouchEvent()|传递点击事件|当事件能传递到当前View时调用|
|onTouchEvent()|处理点击事件|在dispatchTouchEvent()内部调用|
|onInterceptTouchEvent()|判断是否拦截某个事件（只存在于ViewGroup，普通View没有）|在ViewGroup的dispatchTouchEvent()内调用|

<https://blog.csdn.net/carson_ho/article/details/54136311>
<!--more-->
# Activity事件分发
事件最先传到Activity的dispatchTouchEvent()进行事件分发

```
    //Activity的dispatchTouchEvent()
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }

    public void onUserInteraction() {
    }

    //PhoneWindow类
    public boolean superDispatchTouchEvent(MotionEvent event) {
        return mDecor.superDispatchTouchEvent(event);
    }

    //DecorView类
    public boolean superDispatchTouchEvent(MotionEvent event) {
        return super.dispatchTouchEvent(event);
    }

    //Activity
    public boolean onTouchEvent(MotionEvent event) {
        if (mWindow.shouldCloseOnTouch(this, event)) {
            finish();
            return true;
        }
        return false;
    }

    public boolean shouldCloseOnTouch(Context context, MotionEvent event) {
        // 主要是对于处理边界外点击事件的判断：是否是DOWN事件，event的坐标是否在边界内等
        if (mCloseOnTouchOutside && event.getAction() == MotionEvent.ACTION_DOWN
            && isOutOfBounds(context, event) && peekDecorView() != null) {
            return true;
        }
        return false;
        // 返回true：说明事件在边界外，即 消费事件
        // 返回false：未消费（默认）
    }
```
- 一般事件都是以ACTION_DOWN开始，所以必定调用onUserInteraction()；
- onUserInteraction()为空方法；当此activity在栈顶时，触屏点击按home，back，menu键等都会触发此方法
- Window类是抽象类，其唯一实现类 = PhoneWindow类
- DecorView类是PhoneWindow类的一个内部类，其调用父类的方法 = ViewGroup的dispatchTouchEvent()，即将事件传递到ViewGroup去处理
- 当一个点击事件未被Activity下任何一个View接收 / 处理时，调用Activity的onTouchEvent()
- 其中先通过shouldCloseOnTouch()方法来处理边界外点击事件的判断

```flow
st=>start: Start
op1=>operation: 调用Activity.dispatchTouchEvent()
op2=>operation: getWindow().superDispatchTouchEvent()
op3=>operation: mDecor.superDispatchTouchEvent(),即ViewGroup.dispatchTouchEvent()
cond=>condition: ViewGroup是否处理事件
op4=>operation: Activity.dispatchTouchEvent()返回true
op5=>operation: Activity.onTouchEvent(),事件在边界范围内，返回false
e=>end
st->op1->op2->cond
cond(yes)->op4->e
cond(no)->op5->e
```

# ViewGroup事件的分发机制
```
public boolean dispatchTouchEvent(MotionEvent ev) { 

    ... // 仅贴出关键代码

    // 重点分析1：ViewGroup每次事件分发时，都需调用onInterceptTouchEvent()询问是否拦截事件
    if (disallowIntercept || !onInterceptTouchEvent(ev)) {  

    // 判断值1：disallowIntercept = 是否禁用事件拦截的功能(默认是false)，
    // 可通过调requestDisallowInterceptTouchEvent（）修改
    // 判断值2： !onInterceptTouchEvent(ev) = 对onInterceptTouchEvent()返回值取反
        // a. 若在onInterceptTouchEvent()中返回false（即不拦截事件），就会让第二个值为true，从而进入到条件判断的内部
        // b. 若在onInterceptTouchEvent()中返回true（即拦截事件），就会让第二个值为false，从而跳出了这个条件判断
        // c. 关于onInterceptTouchEvent() ->>分析1

        ev.setAction(MotionEvent.ACTION_DOWN);  
        final int scrolledXInt = (int) scrolledXFloat;  
        final int scrolledYInt = (int) scrolledYFloat;  
        final View[] children = mChildren;  
        final int count = mChildrenCount;  

        // 重点分析2
        // 通过for循环，遍历了当前ViewGroup下的所有子View
        for (int i = count - 1; i >= 0; i--) {  
            final View child = children[i];  
            if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE  
                        || child.getAnimation() != null) {  
                 child.getHitRect(frame);  

                // 判断当前遍历的View是不是正在点击的View，从而找到当前被点击的View
                // 若是，则进入条件判断内部
                if (frame.contains(scrolledXInt, scrolledYInt)) {  
                    final float xc = scrolledXFloat - child.mLeft;  
                    final float yc = scrolledYFloat - child.mTop;  
                    ev.setLocation(xc, yc);  
                    child.mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;  

                    // 条件判断的内部调用了该View的dispatchTouchEvent()
                    // 即 实现了点击事件从ViewGroup到子View的传递（具体请看下面的View事件分发机制）
                    if (child.dispatchTouchEvent(ev))  { 
                        mMotionTarget = child;  
                        return true; 
                        // 调用子View的dispatchTouchEvent后是有返回值的
                        // 若该控件可点击，那么点击时，dispatchTouchEvent的返回值必定是true，因此会导致条件判断成立
                        // 于是给ViewGroup的dispatchTouchEvent（）直接返回了true，即直接跳出
                        // 即把ViewGroup的点击事件拦截掉
                    }
                } 
            }
        }
        boolean isUpOrCancel = (action == MotionEvent.ACTION_UP) ||  
                (action == MotionEvent.ACTION_CANCEL);  
        if (isUpOrCancel) {  
            mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;  
        }  
        final View target = mMotionTarget;  

        // 重点分析3
        // 若点击的是空白处（即无任何View接收事件） / 拦截事件（手动复写onInterceptTouchEvent（），从而让其返回true）
        if (target == null) {  
            ev.setLocation(xf, yf);  
            if ((mPrivateFlags & CANCEL_NEXT_UP_EVENT) != 0) {  
                ev.setAction(MotionEvent.ACTION_CANCEL);  
                mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;  
            }  
            
            return super.dispatchTouchEvent(ev);
            // 调用ViewGroup父类的dispatchTouchEvent()，即View.dispatchTouchEvent()
            // 因此会执行ViewGroup的onTouch() ->> onTouchEvent() ->> performClick（） ->> onClick()，即自己处理该事件，事件不会往下传递（具体请参考View事件的分发机制中的View.dispatchTouchEvent（））
            // 此处需与上面区别：子View的dispatchTouchEvent（）
        } 
        ... 
}
/**
  * 分析1：ViewGroup.onInterceptTouchEvent()
  * 作用：是否拦截事件
  * 说明：
  *     a. 返回true = 拦截，即事件停止往下传递（需手动设置，即复写onInterceptTouchEvent（），从而让其返回true）
  *     b. 返回false = 不拦截（默认）
  */
  public boolean onInterceptTouchEvent(MotionEvent ev) {  
    return false;
  }
```

```flow
st=>start: Start
op1=>operation: 调用该控件所在布局的ViewGroup.dispatchTouchEvent()
op2=>operation: ViewGroup.onIntercepTouchEvent()
cond=>condition: 是否拦截,默认不拦截

op3=>operation: 允许事件向View传递
op4=>operation: 遍历ViewGroup，找到被点击的响应View
op5=>operation: View.dispatchTouchEvent()

op6=>operation: 不允许事件向View传递
op7=>operation: 调用ViewGroup父类的dispatchTouchEvent(),即View.dispatchTouchEvent()
op8=>operation: 自己处理点击,因此会执行ViewGroup的onTouch() ->> onTouchEvent() ->> performClick（） ->> onClick()

e=>end
st->op1->op2->cond
cond(yes)->op3->op4->op5->e
cond(no)->op6->op7->op8->e
```

# View事件的分发机制
注：onTouch()的执行 先于 onClick()
```
/**
  * 源码分析：View.dispatchTouchEvent（）
  */
  public boolean dispatchTouchEvent(MotionEvent event) {  

        if (mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED &&  
                mOnTouchListener.onTouch(this, event)) {  
            return true;  
        } 
        return onTouchEvent(event);  
  }
  // 说明：只有以下3个条件都为真，dispatchTouchEvent()才返回true；否则执行onTouchEvent()
  //     1. mOnTouchListener != null
  //     2. (mViewFlags & ENABLED_MASK) == ENABLED
  //     3. mOnTouchListener.onTouch(this, event)
  // 下面对这3个条件逐个分析


/**
  * 条件1：mOnTouchListener != null
  * 说明：mOnTouchListener变量在View.setOnTouchListener（）方法里赋值
  */
  public void setOnTouchListener(OnTouchListener l) { 
    mOnTouchListener = l;  
    // 即只要我们给控件注册了Touch事件，mOnTouchListener就一定被赋值（不为空）
} 

/**
  * 条件2：(mViewFlags & ENABLED_MASK) == ENABLED
  * 说明：
  *     a. 该条件是判断当前点击的控件是否enable
  *     b. 由于很多View默认enable，故该条件恒定为true
  */

/**
  * 条件3：mOnTouchListener.onTouch(this, event)
  * 说明：即 回调控件注册Touch事件时的onTouch（）；需手动复写设置，具体如下（以按钮Button为例）
  */
    button.setOnTouchListener(new OnTouchListener() {  
        @Override  
        public boolean onTouch(View v, MotionEvent event) {  
            return false;  
        }  
    });
    // 若在onTouch（）返回true，就会让上述三个条件全部成立，从而使得View.dispatchTouchEvent（）直接返回true，事件分发结束
    // 若在onTouch（）返回false，就会使得上述三个条件不全部成立，从而使得View.dispatchTouchEvent（）中跳出If，执行onTouchEvent(event)
```

```
/**
  * 源码分析：View.onTouchEvent（）
  */
  public boolean onTouchEvent(MotionEvent event) {  
    final int viewFlags = mViewFlags;  
    if ((viewFlags & ENABLED_MASK) == DISABLED) {  
        return (((viewFlags & CLICKABLE) == CLICKABLE ||  
                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));  
    }  
    if (mTouchDelegate != null) {  
        if (mTouchDelegate.onTouchEvent(event)) {  
            return true;  
        }  
    }  
    // 若该控件可点击，则进入switch判断中
    if (((viewFlags & CLICKABLE) == CLICKABLE ||  
            (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {  
                switch (event.getAction()) { 
                    // a. 若当前的事件 = 抬起View（主要分析）
                    case MotionEvent.ACTION_UP:  
                        boolean prepressed = (mPrivateFlags & PREPRESSED) != 0;  
                            ...// 经过种种判断，此处省略
                            // 执行performClick() ->>分析1
                            performClick();  
                            break;  

                    // b. 若当前的事件 = 按下View
                    case MotionEvent.ACTION_DOWN:  
                        if (mPendingCheckForTap == null) {  
                            mPendingCheckForTap = new CheckForTap();  
                        }  
                        mPrivateFlags |= PREPRESSED;  
                        mHasPerformedLongPress = false;  
                        postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());  
                        break;  

                    // c. 若当前的事件 = 结束事件（非人为原因）
                    case MotionEvent.ACTION_CANCEL:  
                        mPrivateFlags &= ~PRESSED;  
                        refreshDrawableState();  
                        removeTapCallback();  
                        break;

                    // d. 若当前的事件 = 滑动View
                    case MotionEvent.ACTION_MOVE:  
                        final int x = (int) event.getX();  
                        final int y = (int) event.getY();  
        
                        int slop = mTouchSlop;  
                        if ((x < 0 - slop) || (x >= getWidth() + slop) ||  
                                (y < 0 - slop) || (y >= getHeight() + slop)) {  
                            // Outside button  
                            removeTapCallback();  
                            if ((mPrivateFlags & PRESSED) != 0) {  
                                // Remove any future long press/tap checks  
                                removeLongPressCallback();  
                                // Need to switch from pressed to not pressed  
                                mPrivateFlags &= ~PRESSED;  
                                refreshDrawableState();  
                            }  
                        }  
                        break;  
                }  
                // 若该控件可点击，就一定返回true
                return true;  
            }  
             // 若该控件不可点击，就一定返回false
            return false;  
        }

/**
  * 分析1：performClick（）
  */  
    public boolean performClick() {  
        if (mOnClickListener != null) {  
            playSoundEffect(SoundEffectConstants.CLICK);  
            mOnClickListener.onClick(this);  
            return true;  
            // 只要我们通过setOnClickListener（）为控件View注册1个点击事件
            // 那么就会给mOnClickListener变量赋值（即不为空）
            // 则会往下回调onClick（） & performClick（）返回true
        }  
        return false;
    }
```

```flow
st=>start: Start
op1=>operation: 调用View.dispatchTouchEvent()
op2=>operation: mOnTouchListener.onTouch()
cond=>condition: 返回onTouch()返回值

op3=>operation: 事件被消费，不再往下传递
op4=>operation: dispatchTouchEvent()返回true，不再调用onClick()

op5=>operation: 事件未被消费，继续往下传递
op6=>operation: View.onTouchEvent()
op7=>operation: View.performClick()
op8=>operation: mOnClickListener.onClick()

e=>end
st->op1->op2->cond
cond(yes)->op3->op4->e
cond(no)->op5->op6->op7->op8->e
```