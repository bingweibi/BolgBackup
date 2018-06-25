---
title: Android基础---RecyclerView的简单使用
date: 2018-04-17 19:38:35
tags: 
	- Android
---

# 用法

## 添加依赖

```
implementation 'com.android.support:recyclerview-v7:27.0.2'
```

## 布局文件

```
<android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
```

<!--more-->

## 适配器

```
public class FruitRecyclerViewAdapter extends RecyclerView.Adapter<FruitRecyclerViewAdapter.ViewHolder> {

    private ArrayList<Fruit> fruitList;
    private Context mContext;

    static class ViewHolder extends RecyclerView.ViewHolder{

        private ImageView mImageView;
        private TextView mTextView;

        public ViewHolder(View itemView) {
            super(itemView);
            mImageView = itemView.findViewById(R.id.fruitImage);
            mTextView = itemView.findViewById(R.id.fruitId);
        }
    }

    public FruitRecyclerViewAdapter(ArrayList<Fruit> fruitList) {
        this.fruitList = fruitList;
    }

    //创建子项item的布局
    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view  = LayoutInflater.from(parent.getContext()).inflate(R.layout.fruititem,parent,false);
        mContext = parent.getContext();
        return new ViewHolder(view);
    }

    //子项布局设置数据
    @Override
    public void onBindViewHolder(ViewHolder holder, final int position) {
        holder.mTextView.setText(fruitList.get(position).getFruitName());
        holder.mImageView.setImageResource(fruitList.get(position).getImageViewId());
        //点击事件，也可以点击删除，
        holder.itemView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(mContext,"click" + fruitList.get(position).getFruitName(), Toast.LENGTH_SHORT).show();
            }
        });
    }

    @Override
    public int getItemCount() {
        return fruitList.size();
    }
}
```

## 常见和不常见用法

```
        mRecyclerView = findViewById(R.id.recyclerView);
//        线性布局上下滑动
//        mRecyclerView.setLayoutManager(new LinearLayoutManager(this));
//        线性布局左右滑动
//        LinearLayoutManager manager = new LinearLayoutManager(this);
//        manager.setOrientation(LinearLayoutManager.HORIZONTAL);
//        mRecyclerView.setLayoutManager(manager);
//        grid布局上下滑动,左右布局与上面类似
//        mRecyclerView.setLayoutManager(new GridLayoutManager(this,2));
//        瀑布流上下滑动,左右滑动类似
        mRecyclerView.setLayoutManager(new StaggeredGridLayoutManager(2,StaggeredGridLayoutManager.VERTICAL));
//        添加分割线,也可以自定义
        mRecyclerView.addItemDecoration(new DividerItemDecoration(this,DividerItemDecoration.VERTICAL));
//        添加动画效果(默认)
        mRecyclerView.setItemAnimator(new DefaultItemAnimator());
//        ItemTouchHelper进行拖拽排序和侧滑删除
        mRecyclerView.setAdapter(new FruitRecyclerViewAdapter(fruitList));
```

## RecyclerView的优势
1. 默认实现View的复用，不需要添加`if(convertView == null )`语句
2. 默认支持局部刷新
3. 容易实现添加item，删除item的动画效果
4. 容易实现拖拽、侧滑删除的功能

待续

参考：
https://blog.csdn.net/fkq_2016/article/details/78397174
https://www.cnblogs.com/bugly/p/6264751.html