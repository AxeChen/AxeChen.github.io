我记得有个在工作中有个需求是关于判断双击。当时我一个同事给我一个方案：用计时的方法去做，在view的onTouchEvent中。当ACTION_DOWN时记录当时的时间。然后第二次ACTION_DOWN时再记录一次，通过这两次点击的时间间隔来判断是不是双击。 当时我就采纳了这个建议，效果也做出来了，还有点小高兴。
某天下午，偶然看到GestureDetector这个东西然后就大概的了解了一下。
突然觉得在onTouch里面处理双击有点过于麻烦。感觉我和同事都是：

![哈哈](http://upload-images.jianshu.io/upload_images/1930161-b5d69e5404205bf8.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我们来看下源代码（注释太多我全部干掉了 只剩下几个方法）。
```
public class GestureDetector {
    public interface OnGestureListener {
        //down事件 
        boolean onDown(MotionEvent e);
        //触摸屏幕，尚未松开或者拖动 
        void onShowPress(MotionEvent e);
        //触摸屏幕后松开，单击事件 
        boolean onSingleTapUp(MotionEvent e);
        //触摸屏幕后拖动，滑动事件
        boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY);
        //长按事件
        void onLongPress(MotionEvent e);
        //快速滑动
        boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY);
    }

    public interface OnDoubleTapListener {
        //严格的单击行为，不是双击中的一次单击
        boolean onSingleTapConfirmed(MotionEvent e);
        //双击事件
        boolean onDoubleTap(MotionEvent e);
        //表示发生了双击行为
        boolean onDoubleTapEvent(MotionEvent e);
    }

    public interface OnContextClickListener {
        boolean onContextClick(MotionEvent e);
    }
}
```
从以上源码可以看到GestureDetector 已经包括了大部分的手势事件。但是源码的用意我感觉有点奇怪如果我要监听滑动又要监听双击那我不是要继承两个接口？如果我只监听其中一个事件，难道我要重写所有方法？于是又有下面SimpleOnGestureListener 这个类，它是属于GestureDetector的静态内部类
SimpleOnGestureListener实现了OnGestureListener 和 OnDoubleTapListener 。实现了两个接口中的所有方法。如此一来你写个类继承SimpleOnGestureListener然后根据自己的需求重写需要的方法即可。
看源码：
```
public class GestureDetector {
    //前面三个接口就是上面提到的接口
    public interface OnGestureListener {
        boolean onDown(MotionEvent e);
        void onShowPress(MotionEvent e);
        boolean onSingleTapUp(MotionEvent e);
        boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY);
        void onLongPress(MotionEvent e);
        boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY);
    }

    public interface OnDoubleTapListener {
        boolean onSingleTapConfirmed(MotionEvent e);
        boolean onDoubleTap(MotionEvent e);
        boolean onDoubleTapEvent(MotionEvent e);
    }
  
    public interface OnContextClickListener {
        boolean onContextClick(MotionEvent e);
    }

  //警察叔叔，就是这个类
    public static class SimpleOnGestureListener implements OnGestureListener, OnDoubleTapListener,
            OnContextClickListener {

        public boolean onSingleTapUp(MotionEvent e) {
            return false;
        }

        public void onLongPress(MotionEvent e) {
        }

        public boolean onScroll(MotionEvent e1, MotionEvent e2,
                float distanceX, float distanceY) {
            return false;
        }

        public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX,
                float velocityY) {
            return false;
        }

        public void onShowPress(MotionEvent e) {
        }

        public boolean onDown(MotionEvent e) {
            return false;
        }

        public boolean onDoubleTap(MotionEvent e) {
            return false;
        }

        public boolean onDoubleTapEvent(MotionEvent e) {
            return false;
        }

        public boolean onSingleTapConfirmed(MotionEvent e) {
            return false;
        }

        public boolean onContextClick(MotionEvent e) {
            return false;
        }
    }
```
我写了个方法去测试这些方法。（至于怎么去测试，这个确实简单，想怎么写就怎么写
不过还是把几句至关重要的代码指出来，我写的是自定义View。直接写在Activity中也可以，测试方法很多。但是实现方式是一样的）
以下是关键代码：
```
 GestureDetector gestureDetector;

   public TestGestureDetectorView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        //初始化（注意：GestureDetector的构造函数有一堆，我这里传入了一个继承SimpleOnGestureListener 的自定义类）
        gestureDetector = new GestureDetector(context, new MySimpleOnGestureListener());
    }


 @Override
 public boolean onTouchEvent(MotionEvent event) {
        // 在onTouchEvent调用GestureDetector的onTouchEvent方法
        gestureDetector.onTouchEvent(event);
        return true;
    }
```
以下为源码：
```
public class TestGestureDetectorView extends View  {

    GestureDetector gestureDetector;

    public TestGestureDetectorView(Context context) {
        this(context, null);
    }

    public TestGestureDetectorView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public TestGestureDetectorView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        gestureDetector = new GestureDetector(context, new MySimpleOnGestureListener());
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        gestureDetector.onTouchEvent(event);
        return true;
    }

    private class MySimpleOnGestureListener extends SimpleOnGestureListener {

        public MySimpleOnGestureListener() {
            super();
        }

        @Override
        public boolean onDown(MotionEvent e) {
            Log.i("TestGestureDetectorView", "onDown");
            return false;
        }

        @Override
        public void onShowPress(MotionEvent e) {
            Log.i("TestGestureDetectorView", "onShowPress");
        }

        @Override
        public boolean onSingleTapUp(MotionEvent e) {
            Log.i("TestGestureDetectorView", "onSingleTapUp");
            return false;
        }

        @Override
        public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
            Log.i("TestGestureDetectorView", "onScroll");
            return true;
        }

        @Override
        public void onLongPress(MotionEvent e) {
            Log.i("TestGestureDetectorView", "onLongPress");
        }

        @Override
        public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
            Log.i("TestGestureDetectorView", "onFling");
            return false;
        }

        @Override
        public boolean onSingleTapConfirmed(MotionEvent e) {
            Log.i("TestGestureDetectorView", "onSingleTapConfirmed");
            return false;
        }

        @Override
        public boolean onDoubleTap(MotionEvent e) {
            Log.i("TestGestureDetectorView", "onDoubleTap");
            return false;
        }

        @Override
        public boolean onDoubleTapEvent(MotionEvent e) {
            Log.i("TestGestureDetectorView", "onDoubleTapEvent");
            return false;
        }

        @Override
        public boolean onContextClick(MotionEvent e) {
            return super.onContextClick(e);
        }
    }
}

```
推荐的博客：
http://www.jianshu.com/p/bac3875908d7
http://blog.sina.com.cn/s/blog_77c6324101017hs8.html
http://blog.csdn.net/harvic880925/article/details/39520901
我翻了下Android开发艺术探索，里面也提到了，不过没有我写的这么详细。不过书里面有句话我加在最后：
**如果只是监听滑动相关的，建议在onTouchEvent中实现，如果要监听双击的这种行为，那么就使用GestureDetector。**
