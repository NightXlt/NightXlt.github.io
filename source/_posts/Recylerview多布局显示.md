title: Recylerview多布局显示
date: 2019-1-14
tags: [Android]
categories: Android
description: 一个Recylerview中多个布局显示
---
## 重写getItemViewType方法 

重写getItemViewType方法,根据条件返回条目的类型

```java
@Override
    public int getItemViewType(int position) {

        MoreTypeBean moreTypeBean = mData.get(position);
        if (moreTypeBean.type == 0) {
            return TYPE_PULL_IMAGE;
        } else if (moreTypeBean.type == 1) {
            return TYPE_RIGHT_IMAGE;
        } else {
            return TYPE_THREE_IMAGE;
        }


    }
```

## 新建对应数目的ViewHolder
```java
   private class PullImageHolder extends RecyclerView.ViewHolder {

        public PullImageHolder(View itemView) {
            super(itemView);
        }
    }

    private class RightImageHolder extends RecyclerView.ViewHolder {

        public RightImageHolder(View itemView) {
            super(itemView);
        }
    }

    private class ThreeImageHolder extends RecyclerView.ViewHolder {

        public ThreeImageHolder(View itemView) {
            super(itemView);
        }
    }
```
## 在onCreateViewHolder里根据viewtype来判断 所引用的item布局类型 

```java
@Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        //创建不同的 ViewHolder
        View view;
        //根据viewtype来判断

        if (viewType == TYPE_PULL_IMAGE) {
            view =View.inflate(parent.getContext(),R.layout.item_pull_img,null);
            return new PullImageHolder(view);
        } else if (viewType == TYPE_RIGHT_IMAGE) {
            view =View.inflate(parent.getContext(),R.layout.item_right_img,null);
            return new RightImageHolder(view);
        } else {
            view =View.inflate(parent.getContext(),R.layout.item_three_img,null);
            return new ThreeImageHolder(view);
        }
    }
```
