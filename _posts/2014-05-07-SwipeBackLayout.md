
#SwipeBackLayout 滑动返回关闭界面控件

--------------------------------------------

 _Edit by Fooyou Email : <foyou23@qq.com>_ 
    
    15:31 05/07/2014
    
----------------------------------------------

  因工作需要，在项目中需实现体验效果：在Android App 中，手势向右滑动，拖拽，关闭当前页面。具备飘逸的特效，阻泥效果。
  
  经过一番调研摸索。有一个了解决方案：基于Android自带的控件 `SlidingPaneLayout` 做修改，可以实现需求效果。
  
  [SlidingPaneLayout][1] 是一个侧滑动栏。手势向右拉，可以弹出侧滑菜单的，于是就想，如果把滑动菜单挖空，透明。并且菜单滑动最右边时候，触发关闭当前Activity呢？带这个设想，做一个了尝试，实践证明是可行的。
  
  先看看效果咯！
  
![enter image description here][2]

实现步骤：
--------

1. 拷贝`SlidingPaneLayout`源码，做适当修改。代码篇幅比较长，这里不做讨论。后面会附上`GitHub repo`地址，具体内容看代码注释。
2. `Activity`部分，我们需要重写一个Acitivity 中onPostCreate方法. 目的是：把`SlidingPaneLayout` 套入视图的顶层，即，让所有的视图控件都包含在这里面。这样随着`SlidingPaneLayout`滑动，整个界面也跟着滑动了。
3. 配置`Activity`主题，使其窗口变透明，以至于，当`SlidingPaneLayout` 向右边滑动时候，能够透过当前窗口，看到上一个`Acitivity`的窗口（如图中`MainActivity`），这样就产生了滑动推拉的效果了。

OK！ 上面是整体思路，下面我们看一下 关键点 代码片段。

>**文件：SwipeBackActivity.java**

```
public class SwipeBackActivity extends FragmentActivity implements PanelSlideListener{

	...
	//其他代码忽略
	...

    /**
	 *  onPostCreate 方法是在Acitivity start 完成之后执行，拦截这里，操控调整布局，把 SlidingPaneLayout 套入当前界面。
	 *  (after onStart and onRestoreInstanceState have been called)
	 */
	@Override
	protected void onPostCreate(Bundle savedInstanceState) {
		/*if (BuildConfig.DEBUG)Log.i(TAG, "onPostCreate >> swipeEnable >> "+swipeEnable);*/
		if (swipeEnable) {
			ViewGroup localViewGroup = (ViewGroup)getWindow().getDecorView();	//获取当前 窗口顶级DecorView
			View localView = localViewGroup.getChildAt(0);
			localViewGroup.removeView(localView);		//  把contentView 先移除
			SlidingPaneLayout slidingPaneLayout=new SlidingPaneLayout(this);
			slidingPaneLayout.setPanelSlideListener(this);
			slidingPaneLayout.setSliderFadeColor(getResources().getColor(android.R.color.transparent));
			slidingPaneLayout.setShadowResource(mShadowResource);
			if (mShadowDrawable!=null) {
				slidingPaneLayout.setShadowDrawable(mShadowDrawable);
			}
			slidingPaneLayout.setCoveredFadeColor(getResources().getColor(android.R.color.transparent));
			/*slidingPaneLayout.setBackgroundColor(Color.alpha(0));*/
			
			View localViewTemp=new TextView(this);
			/*localViewTemp.setTextColor(getResources().getColor(R.color.beta));
			localViewTemp.setBackgroundResource(android.R.color.black);*/
			ViewHelper.setAlpha(localViewTemp, 0.0f);
			ViewGroup.LayoutParams localLayoutParams = new ViewGroup.LayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT);
			slidingPaneLayout.addView(localViewTemp, localLayoutParams);
			slidingPaneLayout.addView(localView); // 构建完slidingPaneLayout 后，把上面 移除的 当前视图contentView ,在添加到slidingPaneLayout上去。
			localViewGroup.addView(slidingPaneLayout);// 最后 把slidingPaneLayout 当作窗口的contentView 。
			getWindow().setBackgroundDrawableResource(android.R.color.transparent);// 较为关键，把当前窗口设置为透明。
		}
		super.onPostCreate(savedInstanceState);
	}
	
	...
	//其他代码忽略
	...
```
这个方法在`Acitivity`中是核心，目的就是替换当前窗口的`contentView`，把`slidingPaneLayout` 嵌套其中。

是否有注意到，当前`Acitivity` 需实现`PanelSlideListener`接口。

```
@Override
	public void onPanelSlide(View panel, float slideOffset) {
	}

	@Override
	public void onPanelOpened(View panel) {
		super.finish();
	}
```

>**文件：SlidingPaneLayout.java**

```
    ...
	//其他代码忽略
	...
	
    // 这个片段，是把子控件中包含 左右滑动的控件 挑出来。
    //对于他们的触控onTouch事件，需要单独处理。
    
    private void findIgnoredView(View v) {
		/*if(BuildConfig.DEBUG)Log.d(TAG, "findIgnoredView");*/
		if (v instanceof ViewGroup) {
			ViewGroup vg = (ViewGroup) v;
			for (int i = 0; i < vg.getChildCount(); i++) {
				final View child = vg.getChildAt(i);
				if (child instanceof ViewPager
						|| child instanceof ScrollView
						|| child instanceof Gallery
						|| child instanceof HorizontalScrollView) {
					mIgnoredViews.add(child);
				} else {
					findIgnoredView(child);
				}
			}
		} else {
			if (v instanceof ViewPager
					|| v instanceof ScrollView
					|| v instanceof Gallery
					|| v instanceof HorizontalScrollView) {
				mIgnoredViews.add(v);
			}
		}
	}
    
    
     @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        final int action = MotionEventCompat.getActionMasked(ev);

        // Preserve the open state based on the last view that was touched.
        if (!mCanSlide && action == MotionEvent.ACTION_DOWN && getChildCount() > 1) {
            // After the first things will be slideable.
            final View secondChild = getChildAt(1);
            if (secondChild != null) {
                mPreservedOpenState = !mDragHelper.isViewUnder(secondChild,
                        (int) ev.getX(), (int) ev.getY());
            }
        }

        if (!mCanSlide || (mIsUnableToDrag && action != MotionEvent.ACTION_DOWN)) {
            mDragHelper.cancel();
            return super.onInterceptTouchEvent(ev);
        }

        if (action == MotionEvent.ACTION_CANCEL || action == MotionEvent.ACTION_UP) {
            mDragHelper.cancel();
            return false;
        }
        
        //关键点，当遇到ScrollView，ViewPager 诸如此类的控件，他们本身有左右滑动事件，避免冲突，再次做拦截。
        if (isIgnoredViewUnder(ev)) {//hack by Fooyou
        	mDragHelper.cancel();
			return false;
		}
    
    ...
	//其他代码忽略
	...
```

> **文件：styles.xml**

```
    // 这里我们定义了2个主题，带左右推拉动画，透明属性。
     <style name="CustomTheme.Animation" parent="AppBaseTheme">
        <item name="android:windowIsTranslucent">true</item>
        <item name="android:windowBackground">@color/alpa</item>
        <item name="android:windowAnimationStyle">@style/AnimationActivity</item>
    </style>

    <style name="AnimationActivity" parent="@android:style/Animation.Activity">
        <item name="android:activityOpenEnterAnimation">@anim/in_from_right</item>
        <item name="android:activityOpenExitAnimation">@anim/out_to_left</item>
        <item name="android:activityCloseEnterAnimation">@anim/in_from_left</item>
        <item name="android:activityCloseExitAnimation">@anim/out_to_right</item>
        <!-- <item name="android:activityCloseExitAnimation">@anim/fade_out</item> -->
    </style>
```

> **文件：AndroidManifest.xml**
```
// 这里是主配置文件，注意到，部分Acitivity 设置我们上面定义的主题。
 <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name" >
        <activity
            android:name="name.teze.test.MainActivityB"
            android:label="@string/title_activity_b"
            android:theme="@style/CustomTheme.Animation" >
        </activity>
        <activity
            android:name="name.teze.test.MainActivityA"
            android:label="@string/title_activity_main"
            android:theme="@style/AppTheme" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity
            android:name="name.teze.test.MainActivityC"
            android:label="@string/title_activity_c"
            android:theme="@style/CustomTheme.Animation" >
        </activity>
        <activity
            android:name="name.teze.layout.lib.SwipeBackActivity"
            android:label="@string/title_activity_swipe_back" >
        </activity>
    </application>
```
关键点代码，主要是这些。

代码下载
-------
  程序这东西，看代码比较容易理解。
  
  [GitHub](https://github.com/teze/swipebacklayout)项目地址。

























  [1]: http://developer.android.com/reference/android/support/v4/widget/SlidingPaneLayout.html
  [2]: https://drive.google.com/uc?id=0By_KX7XayXSldDJnc0hvMzdGRzA