---
layout:     post
title:      VelocityTracker追踪手指滑动的速度
subtitle:   手指在View里面滑动是的速度
date:       2016-11-20
author:     陈再峰
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
    - VelocityTracker
---

如何在View中追踪手指的滑动速度呢？
>关键类：**VelocityTracker**
思路：**在VIew的onTouchEvent（）中，当MotionEvent.ACTION_DOWN时初始化VelocityTracker， 在MotionEvent.ACTION_MOVE进行追踪，当滑动停止之后(MotionEvent.ACTION_UP or MotionEvent.ACTION_CANCEL)不要忘记调用clear（）来回收内存.**


写了一个代码关于追踪手指滑动的速度。
用了两种方式来显示获取的值：一种是直接将追踪到的数值画在了view上；一种是写了一个回调的接口。

```

/**
 * Created by Administrator on 2016/11/16 0016.
 */
public class TestVelocityView extends View {

    //用于回调的接口
    GetVelocityListener listener;
    //追踪速度关键的类。没有这个这篇文章将毫无意义
    VelocityTracker velocityTracker;
    //要画文字或者任何东西都需要的paint
    Paint paint = new Paint();

    public GetVelocityListener getListener() {
        return listener;
    }

    public void setListener(GetVelocityListener listener) {
        this.listener = listener;
    }

    public TestVelocityView(Context context) {
        this(context,null);
    }

    public TestVelocityView(Context context, AttributeSet attrs) {
        this(context, attrs,0);
    }

    public TestVelocityView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        paint.setTextSize(50);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        //画文字的代码。
        canvas.save();
        paint.setColor(Color.BLACK);
        canvas.drawText("x = "+xVelocity+"y ="+yVelocity,getLeft(),getTop(),paint);
      //画完之后回收一下
        canvas.restore();
    }

    int xVelocity = 0;
    int yVelocity = 0;

    @Override
    public boolean onTouchEvent(MotionEvent event) {

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                //初始化
                velocityTracker = VelocityTracker.obtain();
                break;
            case MotionEvent.ACTION_MOVE:
              //追踪
                velocityTracker.addMovement(event);
                velocityTracker.computeCurrentVelocity(1000);
                xVelocity = (int) velocityTracker.getXVelocity();
                yVelocity = (int) velocityTracker.getYVelocity();

                if (listener != null) {
                    listener.get(xVelocity, yVelocity);
                    //强制刷新一下view，否则不会一直掉onDraw。
                    invalidate();
                }
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                //回收
                velocityTracker.clear();
                velocityTracker.recycle();
                break;
        }
        return true;
    }

    public interface GetVelocityListener {
        public void get(int x, int y);
    }
}

```
