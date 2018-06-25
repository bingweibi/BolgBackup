---
title: ListView工作机制
date: 2018-04-13 17:12:50
tags: 
	- ListView 
	- Android
---

# ListView的用途
当你希望展示的内容很大量的时候，这个时候你应该使用`ListView`，它能够使你展示的内容随着屏幕的滑动进而得到展示。

# 使用方法

- 在布局文件中添加`ListView`的控件

```
<ListView
      android:id="@+id/list_view"
      android:layout_width="match_parent"
      android:layout_height="match_parent" />
```

- 作为`ListView`本身是不知道要展示什么类型的数据，也不知道要展示数据的内容，这个时候你需要创建一个`ListAdapter`(提供数据来源和展示的规则)并与`ListView`进行关联。

```
package com.example.bibingwei.listview;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.ListAdapter;
import android.widget.ListView;
import android.widget.Toast;

import java.util.ArrayList;

/**
 * @author bibingwei
 */
public class MainActivity extends AppCompatActivity {

    private ListView mListView;
    ArrayList<String> list = new ArrayList<>();
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mListView = findViewById(R.id.listView);
    }

    @Override
    protected void onResume() {
        super.onResume();
        for (int i=0;i<20;i++){
            list.add(i+"条数据");
        }

        ListAdapter adapter = new ArrayAdapter<String>(MainActivity.this,android.R.layout.simple_list_item_1,list);
        mListView.setAdapter(adapter);
        mListView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int position, long l) {
                Toast.makeText(MainActivity.this,"点击"+position+"数据",Toast.LENGTH_LONG).show();
            }
        });
    }
}
```

# ListView加载大量数据为什么没有出现OOM

## RecycleBin机制

### 第一次Layout

1. 数据还在`Adapter`那里，这个时候不断循环从屏幕的顶端开始进行绘制子元素，但绘制完整个屏幕的时候也就是最后一个元素超出屏幕的时候，停止绘制，然后将这一屏的子布局进行填充数据，至此第一次`layout`就结束了。

### 第二次Layout

1. 这次将之前加载好的一屏的数据都清除掉，并重新进行加载。因为之前加载的数据已经被缓存起来了，这次加载不会消耗太多的时间

### 滑动加载更多数据

1. 经过之前的两次`Layout`过程，现在我们可以在`ListView`中看到已经加载的数据了。这个时候`ListView`只加载了第一屏的数据。当我们滑动屏幕的时候，系统会判断我们是向上滑动还是向下滑动
2. 如果向上滑动，那么就会将之前的View加入到废弃的缓存中去并计数加1，并进行`detach`操作。如果是向下滑动的话就继续加载`View`

<center>![](http://p2g00vblr.bkt.clouddn.com/ListView%E5%9B%BE%E8%A7%A3.png)</center>


# ListView优化

## Use a Background Thread

使用`Background`线程可以是主线程专心的绘制UI以便提高效率。常常会使用`AsyncTask`来进行耗时的操作。
```
// Using an AsyncTask to load the slow images in a background thread
new AsyncTask<ViewHolder, Void, Bitmap>() {
    private ViewHolder v;

    @Override
    protected Bitmap doInBackground(ViewHolder... params) {
        v = params[0];
        return mFakeImageLoader.getImage();
    }

    @Override
    protected void onPostExecute(Bitmap result) {
        super.onPostExecute(result);
        if (v.position == position) {
            // If this item hasn't been recycled already, hide the
            // progress and set and show the image
            v.progress.setVisibility(View.GONE);
            v.icon.setVisibility(View.VISIBLE);
            v.icon.setImageBitmap(result);
        }
    }
}.execute(holder);
```

## 使用ViewHolder
当你滑动`ListView`的时候你可能会不断的`findById`，这样就可能造成OOM
在`ViewHolder`中会存储布局中需要用来展示内容的控件，所以这个时候不用每次都去查询并新建

```
static class ViewHolder {
  TextView text;
  TextView timestamp;
  ImageView icon;
  ProgressBar progress;
  int position;
}
```

填充`ViewHolder`并存储在布局中
```
package com.example.bbw.listviewtest;

import android.content.Context;
import android.support.annotation.NonNull;
import android.support.annotation.Nullable;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.ImageView;
import android.widget.TextView;

import java.util.List;

/**
 * @author bbw
 * @date 2017/6/23
 */

public class FruitAdapter extends ArrayAdapter<Fruit> {

    private int resourceId;

    /**
     * @param context
     * @param textViewResourceId 子项布局的id
     * @param objects 数据源
     */
    FruitAdapter(@NonNull Context context, int textViewResourceId, List<Fruit> objects) {
        super(context, textViewResourceId,objects);
        resourceId = textViewResourceId;
    }

    /**
     * @param position 位置信息
     * @param convertView 缓存view
     * @param parent 父布局
     * @return
     */
    @NonNull
    @Override
    public View getView(int position, @Nullable View convertView, @NonNull ViewGroup parent) {

        Fruit fruit = getItem(position);
        View view ;
        ViewHolder viewHolder;
        if (convertView == null){
            //记住
            view = LayoutInflater.from(getContext()).inflate(resourceId,parent,false);
            //提高效率，不必每次获取控件的id
            viewHolder = new ViewHolder();
            viewHolder.fruitImage = view.findViewById(R.id.fruitImage);
            viewHolder.fruitName = view.findViewById(R.id.fruitName);
            view.setTag(viewHolder);
        }else{
            //是否有缓存，convertView就是用于将之前加载好的布局进行缓存以便重用
            view = convertView;
            viewHolder = (ViewHolder) view.getTag();
        }
        viewHolder.fruitImage.setImageResource(fruit.getImageId());
        viewHolder.fruitName.setText(fruit.getName());
        return view;
    }

    private class ViewHolder {
        ImageView fruitImage;
        TextView fruitName;
    }
}

```