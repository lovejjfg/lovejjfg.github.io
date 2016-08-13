---
layout:     post
title:      Android 自定义View 字母索引条
category: blog
description: 不管是在QQ上，还是在163的邮箱中，或者自己手机的通讯录中，右侧都会躺着一个这个玩意儿，我姑且不造官方有没有相关的东西，或者大家约定俗成的称呼这个玩意儿叫什么，反正我就叫它索引条-IndexBar了吧！
---

### 写在开头
这是自定义View的第三篇文章，第一篇是[Android drawPath实现QQ拖拽泡泡](http://www.jianshu.com/p/887b1ef3362d),主要实现的是题目说的东西，第二篇是[Android 自定义View 跳动的水果和文字](http://www.jianshu.com/p/887b1ef3362d),可能看这个题目不知道说的是撒，主要讲的是Android `drawTextOnPath()`的相关方法，以及属性动画相关的使用。当然个人觉得动画效果还是阔以的 嘻嘻。。
这篇主要还是说说在`onDraw() `中 `drawText()`相关的使用，实现的效果就是如图所示!

![index_静态.png](http://upload-images.jianshu.io/upload_images/2244299-c5bd8cee28941ca7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![index.gif](http://upload-images.jianshu.io/upload_images/2244299-e3688d4cd7b22841.gif?imageMogr2/auto-orient/strip)



一个View从出生到你能看到的话，肯定是会经历`onMeasure()`、`onLayout()`、`onDraw()`,这几方法的，而自定义View无外乎也要涉及到这几个相关的方法，这篇文章没有那么复杂，主要涉及的就是`onDraw()`方法！

### 开门见山-IndexBar
不管是在QQ上，还是在163的邮箱中，或者自己手机的通讯录中，右侧都会躺着一个这个玩意儿，我姑且不造官方有没有相关的东西，或者大家约定俗成的称呼这个玩意儿叫什么，反正我就叫它索引条-IndexBar了吧！

IndexBar从整体样式上（我观察的哈），分为两种，一种就是**不管三七二十一，26个字母糊糊的贴上去的那种，还有一种就是根据当前的具体内容，只展示相关的首字母的！**至于touch到IndexBar背景变为灰色，滑动时选中的字母呈现出选中的状态，这些都搜easy滴！！当然你可能要说还有开头是#号的，或者写着`热门`等等等的。。

### 实现思路
这个问题要一分为二来看，首先是怎么把26个字母画出来，然后才是怎么去识别触摸对应的是哪个字母！！
###### 画出对应的字母
这个不用多说，肯定是要调用 `drawText()`相关的方法，`drawText(@NonNull String text, float x, float y, @NonNull Paint paint)`,这里我们需要注意的就是这里的x和y是撒意思了！
它就是控制这个文字开始的左下定位的坐标。文字就是从这个点的开始向右上绘制出来的！**Demo中onDraw方法有对应的注释了的方法，打开可以直接看相关的效果。**

首先确定X轴的距离，就是（总的宽度-文字的宽度）/2,这样每个文字水平就是居中显示的了！！
然后确定Y轴的位置，就是（每个文字的总高度+文字的高度）/2，（文字是确定的左下方的坐标点，向下应该加起来！）这样每个文字在竖直方向单位高度中也是居中显示的了！！

那么问题来了，上面说的那些宽度高度等要怎么获取呢？
获取屏幕的高度，平分到26个字母，有'#  '或者 ‘热门’再把相关的东西加上！这个就是 **每个文字的总高度**！接下来就是涉及到这几个方法：
`onDraw()` 这个不用说，你不draw怎么能展示出来呢？
`onSizeChanged()`,如果屏幕尺寸发生了变化，不如说虚拟按键隐藏或者展示之后，还有就是屏幕旋转相关的。。
`setLetters()`,准备好了相关的字母之后，这里就需要去再去计算新的相关参数然后通知绘制。




        @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        if (letters != null) {
            for (int i = 0; i < letters.size(); i++) {
                String text = letters.get(i);
                float textWidth = mPaint.measureText(text);
                mPaint.getTextBounds(text, 0, text.length(), mRect);
                float textHeight = mRect.height();
                float x = mCellWidth * 0.5f - textWidth * 0.5f;
                float y = mCellHeight * 0.5f + textHeight * 0.5f + mCellHeight * i + beginY;
                mPaint.setColor(mIndex == i ? selecColor : normalColor);
                canvas.drawText(text, x, y, mPaint);

            }
        }
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mHeight = getMeasuredHeight()-getPaddingTop()-getPaddingBottom();
        mCellWidth = getMeasuredWidth();
        mCellHeight = mHeight * 1.0f / 26;
        if (letters != null) {
            beginY = (mHeight - mCellHeight * letters.size()) * 0.5f;
        }

    }


    public void setLetters(@Nullable List<String> letters) {

        if (letters == null) {
            setVisibility(GONE);
            return;
        }
        this.letters = letters;
        mHeight = getMeasuredHeight()-getPaddingTop()-getPaddingBottom();
        mCellWidth = getMeasuredWidth();
        mCellHeight = mHeight * 1.0f / 26;
        beginY = (mHeight - mCellHeight * letters.size()) * 0.5f;
        invalidate();
    }

`setLetters()` 和 `onSizeChanged()`里面的代码基本上是重复的，只是在`setLetters()`里面调用了 `invalidate()`去通知重新绘制。



###### 触摸的相关状态添加
首先是触摸到这个索引条，背景加深，这个肯定就是走touch事件了嘛，在`ACTION_DOWN`的时候修改相关状态，在`ACTION_UP`的时候，再次刷新相关状态咯。

这里要使用`refreshDrawableState()`和`onCreateDrawableState()`这两个方法，如果你知道了，就当我在这里瞎比比吧！哈哈。。如果不清楚，可以看看我之前写的一篇[自定义状态选择器](http://blog.csdn.net/lovejjfg/article/details/50623242)。
    
    //定义一个状态
    private static final int[] STATE_FOCUSED = new int[]{android.R.attr.state_focused};
    //DOWN 设为true UP 设为false
    private void refreshState(boolean state) {
        if (pressed != state) {
            pressed = state;
            refreshDrawableState();
        }
    }

    @Override
    public int[] onCreateDrawableState(int extraSpace) {
        int[] states = super.onCreateDrawableState(extraSpace + 1);
        if (pressed) {
            mergeDrawableStates(states, STATE_FOCUSED);
        }
        return states;
    }

背景选择基本欧克了！

然后是选中的字母的颜色，这个其实就是更换画笔的颜色就好了！！这个就放在下面的一块内容中。




###### 点击相关回调
用户看到的都是表象，触摸到的肯定是某一个坐标值，这个坐标应该对应这26个字母中的某一个字母的所在的坐标！比如说总高度是2600，然后每个字母Y轴所占的区域就是100，你触摸的坐标是（x,520）,那这个明显就是第六个字母了嘛，获取到了对应的position，这个问题就解决完了！

           @Override
    public boolean onTouchEvent(MotionEvent event) {
        float y;
        invalidate();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                Log.i("TAG", "onTouchEvent:Down ");
                getParent().requestDisallowInterceptTouchEvent(true);
                refreshState(true);
                y = event.getY();
                checkIndex(y);
                break;
            case MotionEvent.ACTION_MOVE:
                y = event.getY();
                checkIndex(y);
                break;
            case MotionEvent.ACTION_UP:
                refreshState(false);
                mIndex = -1;
                break;

            default:
                break;
        }
        return true;
    }



    private void checkIndex(float y) {
        int currentIndex;
        if (y < beginY+getPaddingTop()) {
            return;
        }
        currentIndex = (int) ((y - beginY-getPaddingTop()) / mCellHeight);
        if (currentIndex != mIndex) {
            if (mOnLetterChangeListener != null) {
                if (letters != null && currentIndex < letters.size()) {
                    mIndex = currentIndex;
                    mOnLetterChangeListener.onLetterChange(letters.get(currentIndex));
        //          Log.i(TAG, "checkIndex: "+letters.get(currentIndex));
                }
            }
           
        }
    }

然后就是上面遗留的那个问题，选中字母颜色的更改就是通过这个mIndex来实现的，在draw方法中的这行代码：

    mPaint.setColor(mIndex == i ? selecColor : normalColor);
                canvas.drawText(text, x, y, mPaint);
那到这里可以看到，那就是View的展现和相关逻辑的确是分开的，你看到的都是一些表面现象。

### 滚动到指定的位置

这个是最终的要求了，这里要区分实现机制了，如果你是使用了`ListView`，那么直接调用`setSelection()`就可以滚动到指定的位置了。
如果你是使用了`RecycleView`的话，那么就是使用LayoutManager的`manager.scrollToPositionWithOffset(pos,0)`。
**我在测验中发现直接使用`manager.scrollToPosition()`的话，的确可以滚动，但是不是出现在顶部位置！**

### 总结
本次`Indexbar`的话，绘制部分主要涉及到了`onDraw()`方法，`canvas.drawText()`。
细节的话，就是onSizeChanged() 和 setLetters()之后的通知重新绘制。
还有就是状态选择器，两个方法`refreshDrawableState()`和`onCreateDrawableState()`。
**需要注意的是，有两个偏移量 beginY 和 topPadding，beginY是用居中的一个偏移量，topPadding就不用多说了！**

回调部分，就是`onTouch`相关处理，根据`getY()`获取相关Y轴的值推算出对应的position,然后再回调到对应的`ListView`或者`RecycleView`

**Gif所示的Demo地址：[IndexDemo](https://github.com/lovejjfg/IndexBar)**

**自定义View的Demo相关地址[自定义View](https://github.com/lovejjfg/Circle)**
