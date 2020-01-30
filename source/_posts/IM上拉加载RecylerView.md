title: 关于IM上拉加载RecylerView闪烁至错误位置
date: 2017-06-06 00:00:00
tags: [Android]
categories: bug
description: IM上拉加载RecylerView闪烁至错误位置
---

  在上拉刷新时采取 mChatAdapter.notifyItemRangeInserted(0, size)，而不是mChatAdapter.notifyDataSetChanged();