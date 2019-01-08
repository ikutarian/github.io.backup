---
title: 一个易维护的RecyclerView.Adapter的实现
date: 2018-09-17 10:55:30
tags:
  - RecyclerView
  - Adapter
categories:
  - Android
---

使用 RecyclerView 实现下面的效果：

1. 有两种布局，红色布局和蓝色布局
2. 偶数位置的是红色布局，奇数位置的是蓝色布局

{% asset_img 3617116-10210f9aaf29bb93.png %}

按照一般的方法来实现 Adapter 的话，代码如下

<!-- more -->

```java
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import java.util.List;

public class MyAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {

    private static final int TYPE_RED = 0;
    private static final int TYPE_BLUE = 1;

    private List<String> mDataSource;

    public MyAdapter(List<String> dataSource) {
        mDataSource = dataSource;
    }

    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        if (viewType == TYPE_RED) {
            return new RedViewHolder(LayoutInflater.from(parent.getContext()).inflate(R.layout.item_red, parent, false));
        } else {
            return new BlueViewHolder(LayoutInflater.from(parent.getContext()).inflate(R.layout.item_blue, parent, false));
        }
    }

    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
        if (holder.getItemViewType() == TYPE_RED) {
            RedViewHolder redViewHolder = (RedViewHolder) holder;
            redViewHolder.tv.setText(mDataSource.get(position));
        } else {
            BlueViewHolder blueViewHolder = (BlueViewHolder) holder;
            blueViewHolder.tv.setText(mDataSource.get(position));
        }
    }

    @Override
    public int getItemCount() {
        return mDataSource == null ? 0 : mDataSource.size();
    }

    @Override
    public int getItemViewType(int position) {
        if (position % 2 == 0) {
            return TYPE_RED;
        } else {
            return TYPE_BLUE;
        }
    }

    private static class RedViewHolder extends RecyclerView.ViewHolder {

        TextView tv;

        RedViewHolder(View itemView) {
            super(itemView);
            tv = (TextView) itemView.findViewById(R.id.tv);
        }
    }

    private static class BlueViewHolder extends RecyclerView.ViewHolder {

        TextView tv;

        BlueViewHolder(View itemView) {
            super(itemView);
            tv = (TextView) itemView.findViewById(R.id.tv);
        }
    }
}
```

因为有多种布局，所以重点关注的是如何判断在某一个位置要如何展示哪一个布局，RecyclerView.Adapter 提供了 `int getItemViewType(int position)` 方法让我们重写，给每一个布局指定一个唯一标识 `ItemViewType`

按照效果图：偶数位置的是红色布局，奇数位置的是蓝色布局，所以方法重写如下

```java
// 布局的唯一标识
private static final int TYPE_RED = 0;
private static final int TYPE_BLUE = 1;

// 布局的判断
@Override
public int getItemViewType(int position) {
    if (position % 2 == 0) {
        return TYPE_RED;
    } else {
        return TYPE_BLUE;
    }
}
```

有了布局的唯一标识 `ItemViewType` 之后，就可以在 
`RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType)` 和
`void onBindViewHolder(RecyclerView.ViewHolder holder, int position)` 中根据这个唯一标识，创建和绑定不同的 ViewHolder 了。

## 缺陷

这个写法没什么毛病，但是在维护上会有一些问题。如果将来要增加一个新的布局，假设位置的尾数逢 5 的地方需要显示绿色布局，就需要改动几个地方

1. 增加或布局的唯一标识，也就是改动这个地方

```java
private static final int TYPE_RED = 0;
private static final int TYPE_BLUE = 1;
private static final int TYPE_GREEN = 2;  // 增加一个标识
```

2. 增加一个 ViewHolder

```java
private static class GreenViewHolder extends RecyclerView.ViewHolder {

    TextView tv;

    GreenViewHolder(View itemView) {
        super(itemView);
        tv = (TextView) itemView.findViewById(R.id.tv);
    }
}
```

3. 修改 `int getItemViewType(int position)` 的 if-else 判断

```java
@Override
public int getItemViewType(int position) {
    // 这里的 if - else 判断需要修改
    if (position % 2 == 0) {
        return TYPE_RED;
    } else if (position % 5 == 0){
      return TYPE_GREEN;  
    } else {
      return TYPE_BLUE;
    }
}
```

4. 修改 `RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType)` 方法，增加一个 ViewHolder

```java
@Override
public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
    if (viewType == TYPE_RED) {
        return new RedViewHolder(LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_red, parent, false));
    } else if (viewType == TYPE_GREEN){
        // 增加一个 GreenViewHolder
        return new GreenViewHolder(LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_green, parent, false));
    } else {
        return new BlueViewHolder(LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_blue, parent, false));
    }
}
```

5. 修改 `void onBindViewHolder(RecyclerView.ViewHolder holder, int position)` 方法，处理 GreenViewHolder 的逻辑

```java
@Override
public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
    if (holder.getItemViewType() == TYPE_RED) {
        RedViewHolder redViewHolder = (RedViewHolder) holder;
        redViewHolder.tv.setText(mDataSource.get(position));
    } else if (holder.getItemViewType() == TYPE_GREEN){
        // 处理 GreenViewHolder 的逻辑
        GreenViewHolder greenViewHolder = (GreenViewHolder) holder;
        greenViewHolder.tv.setText(mDataSource.get(position));
    } else {
        BlueViewHolder blueViewHolder = (BlueViewHolder) holder;
        blueViewHolder.tv.setText(mDataSource.get(position));
    }
}
```

## 发现问题

可以看到增加一个布局，需要改动很多地方。问题的根源就在于 if-else 判断这一块代码。如果让一个类来帮我们在 `int getItemViewType(int position)` 方法中返回唯一标识，同时在 `RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType)` 和
`void onBindViewHolder(RecyclerView.ViewHolder holder, int position)` 方法中帮我们根据 itemViewType 拿到 ViewHolder，这样就能大幅度地减少代码。

## 实现

可以看到 itemViewType 和 ViewHolder 是一一对应的关系，那么就可以使用 ` HashMap<Integer, RecyclerView.ViewHolder>` 来存放 itemViewType 和 ViewHolder。

接着再定义一个接口

```java
public interface AdapterDelegate<T> {

    /**
     * 确定唯一标识
     */
    boolean isForViewType(List<T> items, int position);

    /**
     * 创建 ViewHolder
     */
    RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent);

    /**
     * 绑定 ViewHolder
     */
    void onBindViewHolder(List<T> items, int position, RecyclerView.ViewHolder holder);
}
```

把确定唯一标识、创建 ViewHolder 和 绑定 ViewHolder 的工作交给实现接口的具体的 ViewHolder 来实现。

## 最终效果

MyAdapter 现在只有这些代码

```java
public class MyAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {

    private AdapterDelegatesManager<String> mAdapterDelegatesManager;

    private List<String> mDataSource;

    public MyAdapter(List<String> dataSource) {
        mDataSource = dataSource;
        mAdapterDelegatesManager = new AdapterDelegatesManager<>();
        mAdapterDelegatesManager  // 在这里绑定 ViewHolder
                .addDelegate(new RedViewHolder())
                .addDelegate(new GreenViewHolder())
                .addDelegate(new BlueViewHolder());
    }

    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        return mAdapterDelegatesManager.onCreateViewHolder(parent, viewType);
    }

    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
        mAdapterDelegatesManager.onBindViewHolder(mDataSource, position, holder);
    }

    @Override
    public int getItemCount() {
        return mDataSource == null ? 0 : mDataSource.size();
    }

    @Override
    public int getItemViewType(int position) {
        return mAdapterDelegatesManager.getItemViewType(mDataSource, position);
    }
}
```

剩下的就是定义 ViewHolder 实现 AdapterDelegate<T> 接口就行

```java
/**
 * RedViewHolder
 */
private static class RedViewHolder implements AdapterDelegate<String> {

    @Override
    public boolean isForViewType(List<String> items, int position) {
        return position % 2 == 0;
    }

    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent) {
        return new ViewHolder(LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_red, parent, false));
    }

    @Override
    public void onBindViewHolder(List<String> items, int position, RecyclerView.ViewHolder holder) {
        ViewHolder redViewHolder = (ViewHolder) holder;
        redViewHolder.tv.setText(items.get(position));
    }

    private class ViewHolder extends RecyclerView.ViewHolder {

        TextView tv;

        ViewHolder(View itemView) {
            super(itemView);
            tv = (TextView) itemView.findViewById(R.id.tv);
        }
    }
}


/**
 * BlueViewHolder
 */
private static class BlueViewHolder implements AdapterDelegate<String> {

    @Override
    public boolean isForViewType(List<String> items, int position) {
        return position % 2 != 0;
    }

    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent) {
        return new ViewHolder(LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_blue, parent, false));
    }

    @Override
    public void onBindViewHolder(List<String> items, int position, RecyclerView.ViewHolder holder) {
        ViewHolder viewHolder = (ViewHolder) holder;
        viewHolder.tv.setText(items.get(position));
    }

    private class ViewHolder extends RecyclerView.ViewHolder {

        TextView tv;

        ViewHolder(View itemView) {
            super(itemView);
            tv = (TextView) itemView.findViewById(R.id.tv);
        }
    }
}


/**
 * GreenViewHolder
 */
private static class GreenViewHolder implements AdapterDelegate<String> {

    @Override
    public boolean isForViewType(List<String> items, int position) {
        return position % 5 == 0;
    }

    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent) {
        return new ViewHolder(LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_green, parent, false));
    }

    @Override
    public void onBindViewHolder(List<String> items, int position, RecyclerView.ViewHolder holder) {
        ViewHolder viewHolder = (ViewHolder) holder;
        viewHolder.tv.setText(items.get(position));
    }

    private class ViewHolder extends RecyclerView.ViewHolder {

        TextView tv;

        ViewHolder(View itemView) {
            super(itemView);
            tv = (TextView) itemView.findViewById(R.id.tv);
        }
    }
}
```

## 核心代码的实现

这个类是按照 [JOE'S GREAT ADAPTER HELL ESCAPE](http://hannesdorfmann.com/android/adapter-delegates) 介绍的思想来实现的，核心指出就在于  **itemViewType 和 ViewHolder 是一一对应的关系**。

### AdapterDelegate 接口

```java
import android.support.v7.widget.RecyclerView;
import android.view.ViewGroup;

import java.util.List;

public interface AdapterDelegate<T> {

    boolean isForViewType(List<T> items, int position);

    RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent);

    void onBindViewHolder(List<T> items, int position, RecyclerView.ViewHolder holder);
}
```

### AdapterDelegatesManager 的实现

```java
import android.support.v7.widget.RecyclerView;
import android.util.SparseArray;
import android.view.ViewGroup;

import java.util.List;

public class AdapterDelegatesManager<T> {

    /**
     * Key: ItemViewType
     * Value: AdapterDelegate
     */
    private SparseArray<AdapterDelegate<T>> mDelegates = new SparseArray<>();

    public AdapterDelegatesManager<T> addDelegate(AdapterDelegate<T> delegate) {
        if (delegate == null) {
            throw new NullPointerException("AdapterDelegate of item can not be null");
        } else {
            int newItemViewType = mDelegates.size();
            mDelegates.put(newItemViewType, delegate);
        }
        return this;
    }

    public int getItemViewType(List<T> items, int position) {
        int delegateCount = mDelegates.size();
        for (int itemViewType = 0; itemViewType < delegateCount; itemViewType++) {
            AdapterDelegate<T> delegate = mDelegates.valueAt(itemViewType);
            if (delegate.isForViewType(items, position)) {
                return itemViewType;
            }
        }
        throw new IllegalArgumentException(
                "No AdapterDelegate added that matches position=" + position + " in data source");
    }

    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        AdapterDelegate delegate = mDelegates.get(viewType);
        if (delegate == null) {
            throw new NullPointerException("No AdapterDelegate added for ViewType " + viewType);
        } else {
            return delegate.onCreateViewHolder(parent);
        }
    }

    public void onBindViewHolder(List<T> items, int position, RecyclerView.ViewHolder viewHolder) {
        AdapterDelegate<T> delegate = mDelegates.get(viewHolder.getItemViewType());
        if (delegate == null) {
            throw new NullPointerException(
                    "No AdapterDelegate added for ViewType " + viewHolder.getItemViewType());
        } else {
            delegate.onBindViewHolder(items, position, viewHolder);
        }
    }
}
```

## 参考来源

http://hannesdorfmann.com/android/adapter-delegates
https://github.com/sockeqwe/AdapterDelegates
