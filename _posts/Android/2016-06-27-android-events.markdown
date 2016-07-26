---
layout: post
title:  "Android触摸事件分发机制"
date:   2016-06-27 23:27:43
categories: Android
comments: true
---

Android触摸事件分发机制其实不复杂，但一直没有弄得很清楚，今天开发中遇到了复杂的ViewGroup中处理事件的问题，上网查找资料发现很多文章对事件分发机制描述都不全面，要想理解Android事件分发机制，还是要从源码中去找。

首先，为了更好理解事件分发机制，先列出几个类用于分析：

| 触摸事件相关的方法       | View | ViewGroup | Activity  |
| -------------------------|:----:| ---------:| ---------:|
| dispatchTouchEvent       | 有   |  有       |有         |
| onInterceptTouchEvent    | 有   |  有       |无         |
| onTouch、onTouchEvent    | 有   |  无       |没有onTouch|

ViewGroup继承于View所以当然兼具三种方法，但因为ViewGroup的响应复杂些，所以分开列出了。首先说明一下这三个方法的功能：dispatchTouchEvent用于事件分发，用于事件拦截，onTouch则用于响应事件。这些方法的返回值都是boolean类型，所以有些人就只根据返回类型来判断它们的调用机制了，仔细想来这样显然是错误的。就拿含有多层View的布局举例，事件的分发、调用不同View的相关方法，根本就不是顺序调用，而是有一定的层次顺序。

当一个触摸事件发生时，这个事件经历的过程，可以概括为“先向下分发，再向上冒泡”。在这个过程中每个（层）所属类都是dispatchTouchEvent先被调用。这个向下向上的顺序，如果只有一层控件不必在意，但布局比较复杂的时候，就不得不详细考虑了。

先从View开始讲——现在用户点击了屏幕上的某个按钮（Button也是View）。点击事件发生之后，会进入这个Button的dispatchTouchEvent方法，由这个方法来分发事件，下面是View.dispatchTouchEvent的源码：


    public boolean dispatchTouchEvent(MotionEvent event) {
        // If the event should be handled by accessibility focus first.
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

    if (!result && mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
    }

    // Clean up after nested scrolls if this is the end of a gesture;
    // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
    // of the gesture.
    if (actionMasked == MotionEvent.ACTION_UP ||
            actionMasked == MotionEvent.ACTION_CANCEL ||
            (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
        stopNestedScroll();
    }

    return result;
}


源码都比较多，为了更快，直接找关键部分，看到后两个if判断，它们分别调用了onTouch和onTouchEvent，我们知道了两个方法谁先谁后了。对if的条件进行分析，在第一个if中，所有条件都为true才能进去。既然发生了触摸事件，那第一个肯定true了，第二个是有没有监听，第三个条件，控件是否开启，在这里也是true，第四个条件是onTouch的返回值，熟悉监听机制的就明白了——onTouch是设置监听器时我们要重载的方法，通常在其中写上要进行地方操作，那么既然有这样的判断，如果我们在onTouch中返回true，就能进这个if，result会变成true，注意接下来这个if的写法：

`if (!result && onTouchEvent(event))`

result判断是写在前面的，所以，由于&&具有短路的功能，第一个条件不满足时，第二个条件就不执行了——也就是如果我们onTouch中返回true，将导致onTouchEvent不能得到执行。从onTouchEvent的源码可以得知，点击事件是在那里面处理的，所以按钮其实是先响应touch再响应click。对于ListView这样自带滑动的控件，如果我们在onTouch里返回了True，自带的滑动会就不会执行了。所以onTouch方法默认返回的是false，如果返回true则表示这个事件被消费了，被消费的事件就不会再被处理了。

然而，对于可点击的控件onTouchEvent都会返回true，这将导致dispatchTouchEvent返回true，所以只要是可点击的控件，View.dispatchTouchEvent都会最终返回true，表示这个事件被消费。

对于ViewGroup的事件响应首先明确一点，当点击事件发生时程序会首先找到最顶层View的dispatchTouchEvent开始调用，这个顶层的判断方法是，如果A中有B，那么A就在B的顶层。接下来看在ViewGroup中的dispatchTouchEvent方法，了解ViewGroup是怎样响应事件的。ViewGroup的dispatchTouchEvent方法的第42行调用了onInterceptTouchEvent来获取一个bool值以判断是否被事件拦截，如果结果为true，那么第57行的if就进不去了。

这个if中有很多代码，很复杂，但结合这个if的注释可以知道，它的工作是将事件继续向下分发给这个View的被点中的子View，这就是事件向下分发的过程了。在121行有dispatchTransformedTouchEvent方法的调用，在该方法中有这么一段：


    if (child == null) {
        handled = super.dispatchTouchEvent(event);
    } else {
        handled = child.dispatchTouchEvent(event);
    }


可以看出该方法会寻找相应的子控件的dispatchTouchEvent，如果没有子控件了，就说明这个控件已经是最底层控件，于是用父类的dispatchTouchEvent，也就是View.dispatchTouchEvent，然后全按照前面说的View的响应机制来。所以如果子控件中还有子控件，那么子控件的子控件的dispatchTouchEvent也会被调用，直到最下面一层的控件为止。dispatchTouchEvent是栈式的调用，此时父控件的dispatchTouchEvent并没有返回。

这个最底一层的控件通常也是我们想在ViewGroup中实际操作的控件，如果这个时候我们已经为它重载了onTouch，这时候返回值就不能随意。从View.dispatchTouchEvent的源码可以知道如果控件是可点击的这个方法一定返回true，所以，在ViewGroup中的ViewGroup.dispatchTransformedTouchEvent中返回值为true，表示这个事件已经被消费掉了，之后事件随着dispatchTouchEvent一路向上返回父View时事件就不会再被处理。

