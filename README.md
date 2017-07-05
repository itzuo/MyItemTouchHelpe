# RecyclerView实现条目Item长按拖拽排序与滑动删除 

## 效果图
![Mou icon](http://upload-images.jianshu.io/upload_images/2918620-eb4210464ce2c4b4.gif?imageMogr2/auto-orient/strip)

### 需求和技术分析
   1. RecyclerView Item拖拽排序：长按RecyclerView的Item或者触摸Item的某个按钮。
   2. RecyclerView Item滑动删除：RecyclerView Item滑动删除：RecyclerView的Item滑动删除。

### 实现方案与技术
利用ItemTouchHelper绑定RecyclerView、ItemTouchHelper.Callback来实现UI更新，并且实现动态控制是否开启拖拽功能和滑动删除功能。
### 实现步骤
1. 继承抽象类ItemTouchHelper，并在构造方法传入实现的ItemTouchHelper.Callback。
2. recyclerView绑定ItemTouchHelper：itemTouchHelper.attachToRecyclerView(recyclerView)。
3. 自定义ItemTouchHelper.Callback的实现接口OnItemTouchCallbackListener，由外部更新RecyclerView的Item。
###几个主要的布局

####实现ItemTouchHelperCallback并继承自ItemTouchHelper.Callback
```java
public class ItemTouchHelperCallback extends ItemTouchHelper.Callback {

    private final ItemTouchHelperAdapter mAdapter;
    /**
     * 是否可以拖拽
     */
    private boolean isCanDrag = false;
    /**
     * 是否可以被滑动
     */
    private boolean isCanSwipe = false;

    public ItemTouchHelperCallback(ItemTouchHelperAdapter adapter) {
        mAdapter = adapter;
    }

    public ItemTouchHelperCallback(ItemTouchHelperAdapter adapter,boolean canDrag,boolean canSwipe) {
        mAdapter = adapter;
        this.isCanDrag = canDrag;
        this.isCanSwipe = canSwipe;
    }

    /**
     * 设置是否可以被拖拽
     *
     * @param canDrag 是true，否false
     */
    public void setDragEnable(boolean canDrag) {
        isCanDrag = canDrag;
    }

    /**
     * 设置是否可以被滑动
     *
     * @param canSwipe 是true，否false
     */
    public void setSwipeEnable(boolean canSwipe) {
        isCanSwipe = canSwipe;
    }

    /**
     * 当Item被长按的时候是否可以被拖拽
     * @return
     */
    @Override
    public boolean isLongPressDragEnabled() {
        return isCanDrag;
    }

    /**
     * Item是否可以被滑动(H：左右滑动，V：上下滑动)
     * @return
     */
    @Override
    public boolean isItemViewSwipeEnabled() {
        return isCanSwipe;
    }

    /**
     * 当用户拖拽或者滑动Item的时候需要我们告诉系统滑动或者拖拽的方向
     * @param recyclerView
     * @param viewHolder
     * @return
     */
    @Override
    public int getMovementFlags(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
        RecyclerView.LayoutManager layoutManager = recyclerView.getLayoutManager();
        if (layoutManager instanceof GridLayoutManager) {// GridLayoutManager
            // flag如果值是0，相当于这个功能被关闭
            int dragFlag = ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT | ItemTouchHelper.UP | ItemTouchHelper.DOWN;
            int swipeFlag = 0;
            // create make
            return makeMovementFlags(dragFlag, swipeFlag);
        } else if (layoutManager instanceof LinearLayoutManager) {// linearLayoutManager
            LinearLayoutManager linearLayoutManager = (LinearLayoutManager) layoutManager;
            int orientation = linearLayoutManager.getOrientation();

            int dragFlag = 0;
            int swipeFlag = 0;

            // 为了方便理解，相当于分为横着的ListView和竖着的ListView
            if (orientation == LinearLayoutManager.HORIZONTAL) {// 如果是横向的布局
                swipeFlag = ItemTouchHelper.UP | ItemTouchHelper.DOWN;
                dragFlag = ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT;
            } else if (orientation == LinearLayoutManager.VERTICAL) {// 如果是竖向的布局，相当于ListView
                dragFlag = ItemTouchHelper.UP | ItemTouchHelper.DOWN;
                swipeFlag = ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT;
            }
            return makeMovementFlags(dragFlag, swipeFlag);
        }
        return 0;
    }

    /**
     * 当Item被拖拽的时候被回调
     * @param recyclerView
     * @param viewHolder
     * @param target
     * @return
     */
    @Override
    public boolean onMove(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, RecyclerView.ViewHolder target) {
        /**
         * 回调
         */
        mAdapter.onMove(viewHolder.getAdapterPosition(), target.getAdapterPosition());
        return true;
    }

    @Override
    public void onSwiped(RecyclerView.ViewHolder viewHolder, int i) {
        /**
         * 回调
         */
        mAdapter.onSwipe(viewHolder.getAdapterPosition());
    }
}
```
上面主要就五个方法比较重要
```java
/**
 * 是否可以长按拖拽排序。
 */
@Override
public boolean isLongPressDragEnabled() {}
/**
 * Item是否可以被滑动(H：左右滑动，V：上下滑动)
 */
@Override
public boolean isItemViewSwipeEnabled() {}
/**
 * 当用户拖拽或者滑动Item的时候需要我们告诉系统滑动或者拖拽的方向
 */
@Override
public int getMovementFlags(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {}
/**
 * 当Item被拖拽的时候被回调
 */
@Override
public boolean onMove(RecyclerView r, ViewHolder rholer, ViewHolder tholder) {}

/**
 * 当View被滑动删除的时候
 */
@Override
public void onSwiped(RecyclerView.ViewHolder viewHolder, int direction) {}
```
`isItemViewSwipeEnabled()`返回值是否可以拖拽排序，true可以，false不可以，`isItemViewSwipeEnabled()`是否可以滑动删除，true可以，false不可以；这两个方法都是配置是否可以操作的。我们上面的代码中返回了一个成员变量值，并且这个值通过外部可以修改，所以提供了外部控制的方法。

`onMove()`当Item被拖拽排序移动到另一个Item的位置的时候被回调，`onSwiped()`当Item被滑动删除到不见；这两个方法是当用户操作了，来回调我们，我们就该去更新UI了。这里我们提供了一个Listener去通知外部，并且返回出去了必要的值，来降低代码耦合度。

`getMovementFlags()`说明一：是当用户拖拽或者滑动Item的时候需要我们告诉系统滑动或者拖拽的方向，那我们又知道支持拖拽和滑动删除的无非就是`LinearLayoutManager`和`GridLayoutManager`了，相当于我们老早的时候用的`ListView`和`GridView`了。所以我们根据布局管理器的不同做了响应的区分。

`getMovementFlags()`说明二：其他都好理解，就是这里的`return makeMovementFlags(dragFlag, swipeFlag)`;这句话是最终的返回值，也就是它决定了我们的拖拽或者滑动的方法。第一个参数是拖拽flag，第二个是滑动的flag。
####实现ItemTouchHelperAdapter接口
用于监听长按拖拽或滑动删除时的事件
```java
public interface ItemTouchHelperAdapter {

    /**
     * @param fromPosition 起始位置
     * @param toPosition 移动的位置
     */
    void onMove(int fromPosition, int toPosition);
    void onSwipe(int position);
}
```
####定义RecyclerViewAdapter并实现ItemTouchHelperAdapter 接口
```java
public class RecyclerViewAdapter extends RecyclerView.Adapter<RecyclerViewAdapter.ViewHolder> implements ItemTouchHelperAdapter{
    private List<String> mDataList = new ArrayList<>();

    /**
     * 当加载更多的时候可以使用
     *
     * @param dataList
     */
    public void addData(List<String> dataList) {
        if (dataList != null) {
            mDataList.addAll(dataList);
        }
        notifyDataSetChanged();
    }

    /**
     * 更新Adapter
     *
     * @param dataList
     */
    public void replaceData(List<String> dataList) {
        if (dataList != null) {
            mDataList.clear();
            addData(dataList);
        }
    }

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_main, parent, false);
        ViewHolder holder = new ViewHolder(view);
        return holder;
    }

    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        String data = mDataList.get(position);
        holder.tv.setText(data);
    }

    @Override
    public int getItemCount() {
        return mDataList.size();
    }

    @Override
    public void onMove(int fromPosition, int toPosition) {
        /**
         * 在这里进行给原数组数据的移动
         */
        Collections.swap(mDataList, fromPosition, toPosition);
        /**
         * 通知数据移动
         */
        notifyItemMoved(fromPosition, toPosition);
    }

    @Override
    public void onSwipe(int position) {
        /**
         * 原数据移除数据
         */
        mDataList.remove(position);
        /**
         * 通知移除
         */
        notifyItemRemoved(position);
    }

    class ViewHolder extends RecyclerView.ViewHolder{
        private TextView tv;
        private ImageView iv;
        public ViewHolder(View itemView) {
            super(itemView);
            tv = (TextView) itemView.findViewById(R.id.text);
            iv = (ImageView) itemView.findViewById(R.id.handle);
        }
    }
}
```
####最后让RecyclerView绑定ItemTouchHelper
```java
public class MainActivity extends AppCompatActivity {
    private RecyclerViewAdapter mAdapter;
    private RecyclerView mRecyclerView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mRecyclerView = (RecyclerView) findViewById(R.id.rv);
        mRecyclerView.setLayoutManager(new LinearLayoutManager(this,LinearLayoutManager.VERTICAL,false));
//        mRecyclerView.setLayoutManager(new GridLayoutManager(this,3));
        mAdapter = new RecyclerViewAdapter();
        mRecyclerView.setAdapter(mAdapter);
        initData();
    }

    private void initData() {
        List<String> list = new ArrayList<>();
        for(int i=0;i<100;i++){
            list.add("第"+i+"个");
        }
        mAdapter.addData(list);

        ItemTouchHelperCallback helperCallback = new ItemTouchHelperCallback(mAdapter);
        helperCallback.setSwipeEnable(true);
        helperCallback.setDragEnable(true);
        ItemTouchHelper helper = new ItemTouchHelper(helperCallback);
        helper.attachToRecyclerView(mRecyclerView);
    }
}
```
到此结束
