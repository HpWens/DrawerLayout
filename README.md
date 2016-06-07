# DrawerLayout

乐观是一首激昂优美的进行曲，时刻鼓舞着你向事业的大路勇猛前进。——大仲马

##一、概述

`Drag`拖拽；`ViewDrag`拖拽视图，拖拽控件；`ViewDragHelper`拖拽视图助手，拖拽操作类。利用`ViewDragHelper`类可以实现很多绚丽的效果，比如：拖拽删除，拖拽排序，侧滑栏等。本篇主要讲解简易侧滑栏的实现。

**注意**：`ViewDragHelper`是作用在一个`ViewGroup`上，也就是说他不能直接作用到被拖拽的控件`view`上， 因为控件的位置是由父控件决定的。

最终效果效果图：

![drag](http://img.blog.csdn.net/20160607112334986)

##二、相关概念

###1、create

`ViewDragHelper`的实例是通过静态工厂方法创建的。

方法预览：

```
 public static ViewDragHelper create(ViewGroup forParent, float sensitivity, Callback cb)
```

参数 `forParent`   当前`ViewGroup`
参数 `sensitivity`   敏感度参数，主要用于设置`touchSlop`  `helper.mTouchSlop = (int) (helper.mTouchSlop * (1 / sensitivity));`，可见`sensitivity`值越大，`touchSlop`值就越小。
参数 `cb`   `Callback`是连接`ViewDragHelper`与`view`之间的桥梁

###2、setEdgeTrackingEnabled（拖动的方向）

`setEdgeTrackingEnabled`源码：

```
    /**
     * Enable edge tracking for the selected edges of the parent view.
     * The callback's {@link Callback#onEdgeTouched(int, int)} and
     * {@link Callback#onEdgeDragStarted(int, int)} methods will only be invoked
     * for edges for which edge tracking has been enabled.
     *
     * @param edgeFlags Combination of edge flags describing the edges to watch
     * @see #EDGE_LEFT
     * @see #EDGE_TOP
     * @see #EDGE_RIGHT
     * @see #EDGE_BOTTOM
     */
    public void setEdgeTrackingEnabled(int edgeFlags) {
        mTrackingEdges = edgeFlags;
    }
```

可见`edgeFlags`参数是枚举类型，可以从左边，上边，右边，下边拖动。如果我想实现左右拖动怎么设置：

```
mDragHelper.setEdgeTrackingEnabled(ViewDragHelper.EDGE_LEFT | ViewDragHelper.EDGE_RIGHT);
```

###3、setMinVelocity（最小拖动速度）

```
public void setMinVelocity(float minVel)
```

###4、触摸相关方法

```
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        return mDragHelper.shouldInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        mDragHelper.processTouchEvent(event);
        return true;
    }
```

我们在拖拽侧滑栏的时候，禁止主界面的事件响应。那么就需要重写`onInterceptTouchEvent`方法拦截当前事件，通过`mDragHelper.shouldInterceptTouchEvent(event)`来决定我们是否应该拦截当前的事件。`onTouchEvent`触摸方法返回`true`，能够接收到手指`down`以后的操作，通过`mDragHelper.processTouchEvent(event)`来处理事件。

###5、ViewDragHelper.CallCack相关方法

`ViewDragHelper.CallCack`相关方法：

```
        mDragHelper=ViewDragHelper.create(this, 1.0f, new ViewDragHelper.Callback() {
            @Override
            public boolean tryCaptureView(View child, int pointerId) {
                return false;
            }
            @Override
            public int clampViewPositionHorizontal(View child, int left, int dx)
            {
                return left;
            }

            @Override
            public int clampViewPositionVertical(View child, int top, int dy)
            {
                return top;
            }
        });
```

`ViewDragHelper`拦截和处理事件时，通过`CallBack`回调方法来处理那些子视图`View`可以拖拽，边界控制等。

####1、tryCaptureView
 
  `tryCaptureView`捕获子视图`View`，如果返回`true`，则表示该`View`被捕获。试想一个`ViewGroup`里面有许多子`View`，如果我想拖动`View0`，就可以这么处理：

```
   return child == view0;
```

#### 2、 clampViewPositionHorizontal（水平方向固定被捕获View的位置）

我们一起来看一看下面这张图：
 
 ![drag](http://img.blog.csdn.net/20160607131240281)

上图可以看出蓝色`View`可以移动的水平区域为灰色区域，假设移动区域为m，则paddingleft<=m<=viewgroup.getWidth()-paddingright-view.getwidth。编写成代码：

```
    @Override
    public int clampViewPositionHorizontal(View child, int left, int dx) {
        int leftBound = getPaddingLeft();
        int rightBound = getWidth() - child.getWidth() - leftBound;
        int newLeft = Math.min(Math.max(left, leftBound), rightBound);
        return newLeft;
    }
```

这样就可以实现水平拖拽，上效果图：

![drag](http://img.blog.csdn.net/20160607133315649)

####3、 clampViewPositionVertical（垂直方向固定被捕获View的位置）

原理同上。

```
   @Override
   public int clampViewPositionVertical(View child, int top, int dy) {
       int topBound = getPaddingTop();
       int bottomBound = getHeight() - child.getHeight() - topBound;
       int newTop = Math.min(Math.max(top, topBound), bottomBound);
       return newTop;
   }
```

实现水平+垂直拖拽：

![drag](http://img.blog.csdn.net/20160607134345715)

####4、 onEdgeDragStarted（边界拖动时回调）

```
 @Override
 public void onEdgeDragStarted(int edgeFlags, int pointerId) {
     if(edgeFlags==ViewDragHelper.EDGE_LEFT){
         mDragHelper.captureChildView(mDragView, pointerId);
     }
 }
```

在`onEdgeDragStarted`回调方法中，通过`mDragHelper`对子`View`进行捕获，该方法可以绕过`tryCaptureView`方法，不管`tryCaptureView`返回真假。能够在边界拖动还要加上：

```
mDragHelper.setEdgeTrackingEnabled(ViewDragHelper.EDGE_LEFT);
```

效果图：

![drag](http://img.blog.csdn.net/20160607140106911)

####5、onViewReleased（手指释放的时候回调）

```
 //手指释放的时候回调
 @Override
 public void onViewReleased(View releasedChild, float xvel, float yvel) {
     //mAutoBackView手指释放时可以自动回去
     if (releasedChild == mDragView) {
         mDragHelper.settleCapturedViewAt(200, 200);
         invalidate();
     }
 }
```

`settleCapturedViewAt`方法设置释放后`releasedChild`回到的位置。从你手机抬起到回到（200,200）是个过程，视图在不断的变化，所以会调用刷新视图的方法`invalidate()`。注意`invalidate()`结合`computeScroll`方法一起使用：

```
    @Override
    public void computeScroll() {
        if (mDragHelper.continueSettling(true)) {
            invalidate();
        }
    }
```

效果图：

![drag](http://img.blog.csdn.net/20160607141334640)

**注意：**如果你拖动`View`添加了`clickable = true` 或者为`Button`，你会发现拖不动了，尼玛怎么回事？
原因是拖动的时候`onInterceptTouchEvent`方法，判断是否可以捕获，而在判断的过程中会去判断另外两个回调的方法`getViewHorizontalDragRange`和`getViewVerticalDragRange`，只有这两个方法返回大于0的值才能正常的捕获。如果未能正常捕获就会导致手势`down`后面的`move`以及`up`都没有进入到`onTouchEvent`。

处理方案：

```
@Override public int getViewHorizontalDragRange(View child) {
    return getMeasuredWidth() - child.getMeasuredWidth();
}

@Override public int getViewVerticalDragRange(View child) {
    return getMeasuredHeight() - child.getMeasuredHeight();
}
```

##三、侧滑栏

###1、xml布局

```
<?xml version="1.0" encoding="utf-8"?>
<com.ws.viewdragdemo.app.VDHLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="com.ws.viewdragdemo.MainActivity">


    <!--content-->
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#44ff0000"
        android:clickable="true">

        <TextView
            android:id="@+id/id_content_tv"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerInParent="true"
            android:text="我是侧滑栏"
            android:textSize="40sp"/>
    </RelativeLayout>

    <!--menu-->
    <FrameLayout
        android:id="@+id/id_container_menu"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        >
    </FrameLayout>


</com.ws.viewdragdemo.app.VDHLayout>

```

###2、VDHLayout文件

```
package com.ws.viewdragdemo.app;

import android.content.Context;
import android.support.v4.widget.ViewDragHelper;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.view.View;
import android.view.ViewGroup;

/**
 * Created by Administrator on 6/6 0006.
 * <p/>
 */
public class VDHLayout extends ViewGroup {

    private static final int MIN_DRAWER_MARGIN = 80; // dp
    /**
     * Minimum velocity that will be detected as a fling
     */
    private static final int MIN_FLING_VELOCITY = 400; // dips per second

    /**
     * drawer离父容器右边的最小外边距
     */
    private int mMinDrawerMargin;

    private View mLeftMenuView;
    private View mContentView;

    private ViewDragHelper mDragHelper;
    /**
     * drawer显示出来的占自身的百分比
     */
    private float mLeftMenuOnScreen;


    public VDHLayout(Context context) {
        this(context, null);
    }

    public VDHLayout(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public VDHLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        float density = getResources().getDisplayMetrics().density;
        float minVel = MIN_FLING_VELOCITY * density;  //1200
        mMinDrawerMargin = (int) (MIN_DRAWER_MARGIN * density + 0.5f);

        mDragHelper = ViewDragHelper.create(this, 1.0f, new ViewDragHelper.Callback() {
            @Override
            public boolean tryCaptureView(View child, int pointerId) {
                //捕获该view
                return child == mLeftMenuView;
            }

            @Override
            public int clampViewPositionHorizontal(View child, int left, int dx) {
                int newLeft = Math.max(-child.getWidth(), Math.min(left, 0));
                //始终都是取left的值，初始值为-child.getWidth()，当向右拖动的时候left值增大，当left大于0的时候取0
                return newLeft;
            }
            
            //手指释放的时候回调
            @Override
            public void onViewReleased(View releasedChild, float xvel, float yvel) {
                int childWidth = releasedChild.getWidth();
                //0~1f
                float offset = (childWidth + releasedChild.getLeft()) * 1.0f / childWidth;

                mDragHelper.settleCapturedViewAt(xvel > 0 || xvel == 0 && offset > 0.5f ? 0 : -childWidth,
                        releasedChild.getTop());
                //由于offset 取值为0~1，所以settleCapturedViewAt初始值为 -childWidth，滑动小于0.5取值也为-childWidth， 
                //大于0.5取值为0
                invalidate();
            }

            //在边界拖动时回调
            @Override
            public void onEdgeDragStarted(int edgeFlags, int pointerId) {
                mDragHelper.captureChildView(mLeftMenuView, pointerId);
            }

            @Override
            public void onViewPositionChanged(View changedView, int left, int top, int dx, int dy) {
                int childWidth = changedView.getWidth();
                float offset = (float) (childWidth + left) / childWidth;
                mLeftMenuOnScreen = offset;
                changedView.setVisibility(offset == 0 ? View.INVISIBLE : View.VISIBLE);
                //offset 为0 的时候隐藏 ， 不为0显示
                invalidate();
            }

            @Override
            public int getViewHorizontalDragRange(View child) {
                //始终取值为child.getWidth()
                return mLeftMenuView == child ? child.getWidth() : 0;
            }

        });

        mDragHelper.setEdgeTrackingEnabled(ViewDragHelper.EDGE_LEFT);
        mDragHelper.setMinVelocity(minVel);
    }

    @Override
    public void computeScroll() {
        if (mDragHelper.continueSettling(true)) {
            invalidate();
        }
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightSize = MeasureSpec.getSize(heightMeasureSpec);

        setMeasuredDimension(widthSize, heightSize);

        View leftMenuView = getChildAt(1);
        MarginLayoutParams lp = (MarginLayoutParams)
                leftMenuView.getLayoutParams();

        final int drawerWidthSpec = getChildMeasureSpec(widthMeasureSpec,
                mMinDrawerMargin + lp.leftMargin + lp.rightMargin,
                lp.width);
        final int drawerHeightSpec = getChildMeasureSpec(heightMeasureSpec,
                lp.topMargin + lp.bottomMargin,
                lp.height);
        //确定侧滑栏尺寸
        leftMenuView.measure(drawerWidthSpec, drawerHeightSpec);


        View contentView = getChildAt(0);
        lp = (MarginLayoutParams) contentView.getLayoutParams();
        final int contentWidthSpec = MeasureSpec.makeMeasureSpec(
                widthSize - lp.leftMargin - lp.rightMargin, MeasureSpec.EXACTLY);
        final int contentHeightSpec = MeasureSpec.makeMeasureSpec(
                heightSize - lp.topMargin - lp.bottomMargin, MeasureSpec.EXACTLY);
        //确定主界面尺寸
        contentView.measure(contentWidthSpec, contentHeightSpec);

        mLeftMenuView = leftMenuView;
        mContentView = contentView;

    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        View menuView = mLeftMenuView;
        View contentView = mContentView;

        MarginLayoutParams lp = (MarginLayoutParams) contentView.getLayoutParams();     
        contentView.layout(lp.leftMargin, lp.topMargin,
                lp.leftMargin + contentView.getMeasuredWidth(),
                lp.topMargin + contentView.getMeasuredHeight());

        lp = (MarginLayoutParams) menuView.getLayoutParams();

        final int menuWidth = menuView.getMeasuredWidth();
        int childLeft = -menuWidth + (int) (menuWidth * mLeftMenuOnScreen);
        //确定侧滑栏尺寸
        menuView.layout(childLeft, lp.topMargin, childLeft + menuWidth,
                lp.topMargin + menuView.getMeasuredHeight());


    }
    
    public void closeDrawer() {
        View menuView = mLeftMenuView;
        mLeftMenuOnScreen = 0.f;
        mDragHelper.smoothSlideViewTo(menuView, -menuView.getWidth(), menuView.getTop());
    }

    public void openDrawer() {
        View menuView = mLeftMenuView;
        mLeftMenuOnScreen = 1.0f;
        mDragHelper.smoothSlideViewTo(menuView, 0, menuView.getTop());
    }
    
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        return mDragHelper.shouldInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        mDragHelper.processTouchEvent(event);
        return true;
    }


    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        mContentView = getChildAt(0);
        mLeftMenuView = getChildAt(1);

    }

    @Override
    protected LayoutParams generateDefaultLayoutParams() {
        return new MarginLayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT);
    }

    @Override
    protected LayoutParams generateLayoutParams(LayoutParams p) {
        return new MarginLayoutParams(p);
    }

    @Override
    public LayoutParams generateLayoutParams(AttributeSet attrs) {
        return new MarginLayoutParams(getContext(), attrs);
    }
}

```

需要注意的地方我都在程序中注释了。

###3、LeftMenuFragment文件

```
package com.ws.viewdragdemo;

import android.content.Context;
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.ListView;
import android.widget.TextView;

/**
 * Created by Administrator on 6/7 0007.
 */
public class LeftMenuFragment extends Fragment {

    private ListView lv;
    private Context mContext;

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.left_menu_frgment, container, false);
        lv = (ListView) view.findViewById(R.id.lv);
        lv.setAdapter(new BaseAdapter() {
            @Override
            public int getCount() {
                return 10;
            }

            @Override
            public Object getItem(int i) {
                return i;
            }

            @Override
            public long getItemId(int i) {
                return i;
            }

            @Override
            public View getView(int i, View view, ViewGroup viewGroup) {
                TextView tv = new TextView(mContext);
                tv.setPadding(16, 16, 16, 16);
                tv.setText("我是侧滑栏条目" + i);
                return tv;
            }
        });
        return view;
    }

    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        this.mContext = context;
    }
}

```

###4、MainActivity文件

```
package com.ws.viewdragdemo;

import android.os.Bundle;
import android.support.v4.app.FragmentManager;
import android.support.v7.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    private LeftMenuFragment mLeftMenuFragment;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        FragmentManager fm = getSupportFragmentManager();
        mLeftMenuFragment = (LeftMenuFragment) fm.findFragmentById(R.id.id_container_menu);
        if (mLeftMenuFragment == null) {
            fm.beginTransaction().add(R.id.id_container_menu, mLeftMenuFragment = new LeftMenuFragment())
                    .commit();
        }
    }
}

```

敬请关注：http://blog.csdn.net/u012551350/article/details/51601985

