**为了学习过程中不断的加强记忆，把自己实现的功能记录下来。**

### ViewPager
#### 说明

ViewPager的用途是首次登录时候的导航图片的轮播，或者多个图片或Fragment的轮播等。

ViewPager的主要用法有以下几种。
1. ViewPager+PagerAdapter -> 传入List<View>
2. ViewPager+FragmentPagerAdapter -> 传入List<Fragment>和FragmentManager
3. ViewPager+FragmentPagerAdapter+Tablayout -> 实现在首页上各标签页面的切换

**如果只是实现轮播功能，谷歌更推荐第2种，即用Fragment实现。
原因是可以更加自定义，且可以容易控制各个Fragment的生命周期。**


#### ViewPager中常用的方法

* **setAdapter(PagerAdapter pagerAdapter)**
  为ViewPager设置Adapter

* **ViewPager.getCurrentItem()**
  返回当前的索引，即当前item的位置
  
* **ViewPager.setCurrentItem(int position)**
  为当前的item设置索引

#### 实现方式

##### ViewPager+PagerAdapter
1. 引入Support库。

2. 创建布局
```java
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.hyejeanmoon.test.main">
    <android.support.v4.view.ViewPager
        android:id="@+id/viewpager"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</RelativeLayout>
```

3. 创建Activity
```java

public class MainActivity extends AppCompatActivity {
    private Context context = MyApplication.getContext();
    private ViewPager viewPager;
    private ArrayList<Integer> datas = new ArrayList<>();
    private ArrayList<ImageView> views = new ArrayList<>();
    private ViewAdapter viewAdapter;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initView();
    }
    private void initView(){
    
        datas.add(R.drawble.pic1);
        datas.add(R.drawble.pic2);
        datas.add(R.drawble.pic3);
        datas.add(R.drawble.pic4);
       

        for(int i=0;i<mGoodsList.size();i++){
           ImageView imageView = new ImageView(this);
           imageView.setImageResource(datas.get(i));
           views.add(imageView);
        }
        viewAdapter = new ViewAdapter(this,datas);
        viewPager = (ViewPager) findViewById(R.id.viewpager);
        viewPager.setAdapter(viewAdapter);
    }
    
}

```


4. 创建Adapter(继承PagerAdapter)
这里需要重载的方法是:

* **getCount()**, 获取需要绘制的view的数量。

* **destroyItem(ViewGroup container,int position,Object object)**,当前Item离开屏幕时为了减少内存损耗需要移除，移除功能在该方法中实现。

* **instantiateItem(ViewGroup container, int position)**, 获取当前的View，把当前的View加入到container中，让其显示。

* **isViewFromObject(View view,Object obj)**, 判断View和Object是否一致，这里默认写成 **return view==obj;** 就可以了。

```java
    class ViewAdapter extends PagerAdapter {
        Context context;
        ArrayList<ImageView> views;
        public GoodsAdapter(Context context, ArrayList<ImageView> views){
            this.context=context;
            this.views=views;
        }

        @Override
        public int getCount() {
            return views==null? 0:views.size();
        }

        @Override
        public boolean isViewFromObject(View view, Object object) {
            return view==object;
        }

        @Override
        public ImageView instantiateItem(ViewGroup container, int position) {
            ImageView imageView=views.get(position);
            container.addView(imageView);
            return imageView;
        }

        @Override
        public void destroyItem(ViewGroup container, int position, Object object) {
            ImageView imageView=(ImageView)object;
            container.removeView(imageView);
        }
    }
```



