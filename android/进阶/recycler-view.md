### RecyclerView

引入：

```
implementation 'com.android.support:recyclerview-v7:28.0.0'
```

改为：

```xml
    implementation 'androidx.recyclerview:recyclerview:1.0.0-alpha1'
```



#### 概述

##### 四大组成

- Layout Manager(必选)：ItemView布局
- Adapter(必选)：提供数据
- Item Decoration(可选，默认为空)：ItemView的修饰
- Item Animator(可选，默认为DefaultItemAnimator)：ItemView增加/删除动画

 <img src="F:\Typora\Nodes\Android\进阶\RecyclerView\b848e2a3ffd8d61d685e0f782ed53d08.webp" alt="b848e2a3ffd8d61d685e0f782ed53d08" style="zoom:50%;" />



##### ItemAnimation：子视图动画

```java
recyclerView.setItemAnimator(new DefaultItemAnimator());
```



##### ItemDecoration：分割线

```java
rv.addItemDecoration(new DividerItemDecoration(this,DividerItemDecoration.VERTICAL_LIST));
```

```java
class MyItemDecoration extends RecyclerView.ItemDecoration {
    /**
     *
     * @param outRect 边界
     * @param view recyclerView ItemView
     * @param parent recyclerView
     * @param state recycler 内部数据管理
     */
    @Override
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent, 	RecyclerView.State state) {
        outRect.set(0, 0, 0, 1);
    }
}

// 使用
mRecyclerView.addItemDecoration(new MyItemDecoration());
```

- 使用 getItemOffsets设置padding

  ```java
  public class SimplePaddingDecoration extends RecyclerView.ItemDecoration {
  
      private int dividerHeight;
  
      public SimplePaddingDecoration(Context context) {
          dividerHeight = context.getResources().getDimensionPixelSize(R.dimen.divider_height);
      }
  
      @Override
      public void getItemOffsets(Rect outRect, View view, RecyclerView parent, RecyclerView.State state) {
          super.getItemOffsets(outRect, view, parent, state);
          outRect.bottom = dividerHeight;//类似加了一个bottom padding outRect.set(0, 0, 0, 1); 这里的1单位为
      }
  }
  ```

红色分割线：

```java
public class SimpleDividerDecoration extends RecyclerView.ItemDecoration {

    private int dividerHeight;
    private Paint dividerPaint;


    public SimpleDividerDecoration(Context context) {
        dividerPaint = new Paint();
        dividerPaint.setColor(context.getResources().getColor(R.color.colorAccent));
        dividerHeight = context.getResources().getDimensionPixelSize(R.dimen.divider_height);
    }


    @Override
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent, RecyclerView.State state) {
        super.getItemOffsets(outRect, view, parent, state);
        outRect.bottom = dividerHeight;
    }


    @Override
    public void onDraw(Canvas c, RecyclerView parent, RecyclerView.State state) {
        int childCount = parent.getChildCount();
        int left = parent.getPaddingLeft();
         int right = parent.getWidth() - parent.getPaddingRight();


        for (int i = 0; i < childCount - 1; i++) {
            View view = parent.getChildAt(i);
            float top = view.getBottom();
            float bottom = view.getBottom() + dividerHeight;
            c.drawRect(left, top, right, bottom, dividerPaint);
        }
    }
}
```

效果图：

 <img src="F:\Typora\Nodes\Android\进阶\RecyclerView\186157-11a6344281993a69.webp" alt="186157-11a6344281993a69" style="zoom:50%;" />

带标签分割线：

```java
@Override
public void onDrawOver(@NonNull Canvas c, @NonNull RecyclerView parent,
                       @NonNull RecyclerView.State state) {
    super.onDrawOver(c, parent, state);

    int count = parent.getChildCount();
    for (int i = 0; i < count; i++) {
        View curView = parent.getChildAt(i);
        int pos = parent.getChildAdapterPosition(curView);
        if (pos % 2 == 0) { // left
            tagPaint.setColor(leftColor);
            float left = curView.getLeft();
            float right = left + tagWidth;
            float top = curView.getTop();
            float bottom = (top + curView.getBottom()) / 2;
            c.drawRect(left, top, right, bottom, tagPaint);
        } else { // right
            tagPaint.setColor(rightColor);
            float right = curView.getRight();
            float left = right - tagWidth;
            float top = curView.getTop();
            float bottom = (top + curView.getBottom()) / 2;
            c.drawRect(left, top, right, bottom, tagPaint);
        }
    }
}
```

效果图：

 <img src="F:\Typora\Nodes\Android\进阶\RecyclerView\Image-1615992507963.png" alt="Image" style="zoom: 25%;" />



##### Recycler

负责ViewHolder的引用与销毁

- RecyclerView.Recycler.getViewForPosition(int position, boolean dryRun)：

  根据位置获取itemView先后从scrapped（将要在RecyclerView中删除的ItemView），cached（一级缓存），exCached（二级缓存，默认没有）中获取;



##### 刷新

ListView中使用notifyDataSetchanged();

RecyclerView支持局部数据更新:

- notifyItemChanged(position);
- notifyItemInsert(position);
- notifyItemRemove(position);

定位：

- findFirstVisibleItemPosition()        返回当前第一个可见 Item 的 
- positionfindFirstCompletelyVisibleItemPosition()  返回当前第一个完全可见 Item 的 
- positionfindLastVisibleItemPosition()       返回当前最后一个可见 Item 的 
- positionfindLastCompletelyVisibleItemPosition()  返回当前最后一个完全可见 Item 的 position.scrollBy()   





##### 刷新细节

- notifyItemRangeChanged() --> requestLayout() (重新布局) 

将ViewHolder放入scrap中（attachScrap 和 changedScrap）：

- 修改（type | data）的放入changedScrap中

  changedScrap 中的ViewHolder 会放入RecycledViewPool，然后从pool中取出：

  （如果从RecycledViewPool中获取，需要重新onBindViewHolder；如果从cache中获取ViewHolder，则直接显示即可）

- 其余放入attachScrap中；

![Image](F:\Typora\Nodes\Android\进阶\RecyclerView\Image.png)

当cache中存放ViewHolder（默认大小2；setItemViewCachedSize()设置）超出时，会将其放入RecycledViewPool中；

因此：滑动列表时，超出屏幕的item会存放入cache中，cache超出后存放入pool中；pool超出则丢弃；

##### 场景

1. 向上滑动，屏幕下方出现新的item（经历onCreate -> onBind），屏幕上方遮挡的item --> cache 中；
2. 随着继续滑动，cache空间不足则将先放入的放入到pool中；继续向下滑动，如果有需要显示的itemView，那么从pool中获取（如果有的话，然后再onBindViewHolder）；
3. 此时如果向下滑动（显示上面的），屏幕下方遮挡的会进入cache，然后从cache中获取显示ItemView，直接显示（不用create 和 bind）；

简单再现：

显示下方item时候，上方遮挡的进入cache，甚至进入pool；那么下方显示的可以从pool中获取，onBind；

显示上方item时候，下方遮挡的进入cache，甚至进入pool；重新显示的可以从cache中获取，没有则从pool中获取重新bind；



#### Adapter

将数据转换为视图；

```java
recyclerView.setAdapter(adapter);
```

##### 常见封装

```java
public abstract class QuickAdapter<T> extends RecyclerView.Adapter<QuickAdapter.VH>{
    private List<T> mDatas;
    public QuickAdapter(List<T> datas){
        this.mDatas = datas;
    }

    public abstract int getLayoutId(int viewType);

    @Override
    public VH onCreateViewHolder(ViewGroup parent, int viewType) {
        return VH.get(parent,getLayoutId(viewType));
    }

    @Override
    public void onBindViewHolder(VH holder, int position) {
        convert(holder, mDatas.get(position), position);
    }

    @Override
    public int getItemCount() {
        return mDatas.size();
    }

    public abstract void convert(VH holder, T data, int position);
    
    static class VH extends RecyclerView.ViewHolder{
       private SparseArray<View> mViews; // 保存ItemView中的子view，比如TextView
       private View mConvertView;
    
        private VH(View v){
            super(v);
            mConvertView = v;
            mViews = new SparseArray<>();
        }
    
        public static VH get(ViewGroup parent, int layoutId){
          	View convertView = LayoutInflater.from(parent.getContext())
                							 .inflate(layoutId, parent, false);
         	return new VH(convertView);
        }
    
        public <T extends View> T getView(int id){
            View v = mViews.get(id);
            if(v == null){
                v = mConvertView.findViewById(id);
                mViews.put(id, v);
            }
            return (T)v;
        }
    
        public void setText(int id, String value){
            TextView view = getView(id);
            view.setText(value);
        }
    }
}
```

具体实现：

```java
public class RegularAccountAdapter extends RecyclerView.Adapter<RegularAccountAdapter.ViewHolder> {
    private List<AccountOverview> accOvList;

    public RegularAccountAdapter(List<AccountOverview> accountOverviewList){
        this.accOvList = accountOverviewList;
    }

    @NonNull
    @Override
    public ViewHolder onCreateViewHolder(@NonNull ViewGroup viewGroup, int i) {
        View view = LayoutInflater
        		.from(viewGroup.getContext())                                                       .inflate(R.layout.regular_acc_item,viewGroup,false);
        return new ViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull ViewHolder viewHolder, int i) {
        AccountOverview accOv = accOvList.get(i);
        viewHolder.tvAcc.setText(accOv.getCreditcard());
        viewHolder.tvType.setText(accOv.getType());
    }

    @Override
    public int getItemCount() {
        return accOvList.size();
    }


    static class ViewHolder extends RecyclerView.ViewHolder{
        TextView tvAcc,tvType;
        ViewHolder(@NonNull View itemView) {
            super(itemView);
            tvAcc = itemView.findViewById(R.id.reg_item_tv_acc);
            tvType = itemView.findViewById(R.id.reg_item_tv_type);
           
        }
    }
}
```





setAdapter()具体实现：

```java
public void setAdapter(@Nullable Adapter adapter) {
    // bail out if layout is frozen
    setLayoutFrozen(false);
    setAdapterInternal(adapter, false, true);
    processDataSetCompletelyChanged(false);
    requestLayout();
}

setAdapterInternal(adapter, false, true) // 主要是下面这两行
// mAdapter = adapter
// adapter.registerAdapterDataObserver(mObserver);

// requestLayout() RecyclerView 会调用 
    void onLayout(boolean changed, int l, int t, int r, int b) {
        TraceCompat.beginSection(TRACE_ON_LAYOUT_TAG); 
        dispatchLayout(); 
        TraceCompat.endSection();
        mFirstLayoutComplete = true;
}
```

```java
void dispatchLayout() {
       ......
        if (mState.mLayoutStep == State.STEP_START) {
            dispatchLayoutStep1(); // 做一下准备工作：决定哪一个动画被执行，保存一些目前view的相关信息
            mLayout.setExactMeasureSpecsFrom(this);
              //分发第二步
            dispatchLayoutStep2(); // 找到实际的view和最终的状态后运行layout。
        }
        ......
         //分发第三步
        dispatchLayoutStep3(); // 保存动画和一些其他的信息
        ......
    }
}
```

 ![1658859ccd64da54](F:\Typora\Nodes\Android\进阶\RecyclerView\1658859ccd64da54.webp)