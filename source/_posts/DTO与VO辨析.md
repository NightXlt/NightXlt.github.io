title: DTO与VO辨析
date: 2019-2-26
tags: [Android]
categories: Android
description: 为毛多写个VO呐
---
## VO 与 DTO概念
VO: ViewObject表现层对象。通俗地说即UI层的View需要的对象。
DTO：Data Transfer Object数据传输对象。通俗地说即用于展示层与服务层之间的数据传输对象。

## 加入VO原因
在学校的时候，开发中我一直使用的都是单DTO层。View展示的Model对象和接口返回的Model是同一个对象。
进入公司后，发现这样的代码存在四个问题

1. ViewModel绑定时，无休止的判空操作
2. 接口返回的数据结构或字段改变啦
3. 接口返回的数据结构不是我在View中显示想要的啊
4. 姗姗来迟的接口文档

迫于此情此景，VO应运而生啦。采用VO + Mapper转换接口。模拟场景在Retrofit请求访问网络数据下进行转换。

## Demo
Mapper接口 + listHelper转换接口
```java
public interface Transformable1<VO> {

  VO dto2vo();
}

public final class ListTransformHelper {

  private ListTransformHelper() {

  }

  @SuppressWarnings("unchecked")
  public static <T extends Transformable1, VO> List<VO>


  dto2vo(List<T> dtoList) {

    List<VO> voList = new ArrayList<>();

    for (T dto : dtoList) {

      voList.add((VO) dto.dto2vo());
    }

    return voList;
  }
}
```
Module层：DTO + VO
```java
public class WorksGroupDTO implements Transformable1<WorksGroupVO> {

  public String groupTitle;

  public List<PostDTO> posts;

  @Override public WorksGroupVO dto2vo() {

    return new WorksGroupVO.Builder().setGroupTitle(groupTitle)
        .setPostVOList(translatePosts())
        .setExpand(false)
        .build();
  }

  private List<PostVO> translatePosts() {

    return ListTransformHelper.dto2vo(posts);
  }
}

```

```java
public class WorksGroupVO {

  private String mGroupTitle;

  private List<PostVO> mPostVOList;

  private List<PostVO> mMorePostVOList;

  private boolean mExpand;

  private boolean mIsLoading; 

  public WorksGroupVO(Builder builder) {

    mGroupTitle = builder.mGroupTitle;
    mPostVOList = builder.mPostVOList;
    mMorePostVOList = builder.mMorePostVOList;
    mExpand = builder.mExpand;
    mIsLoading = builder.mIsLoading;
  }

  public String getGroupTitle() {
    return mGroupTitle;
  }

  public List<PostVO> getPostVOList() {
    return mPostVOList;
  }

  public List<PostVO> getMorePostVOList() {
    return mMorePostVOList;
  }

  public boolean isExpand() {
    return mExpand;
  }

  public boolean isLoading() {
    return mIsLoading;
  }

  public static final class Builder {

    private String mGroupTitle;

    private List<PostVO> mPostVOList;

    private List<PostVO> mMorePostVOList; 

    private boolean mExpand;

    private boolean mIsLoading; 

    public Builder() {

    }

    public Builder(WorksGroupVO worksGroupVO) {

      mGroupTitle = worksGroupVO.mGroupTitle;
      mPostVOList = worksGroupVO.mPostVOList;
      mMorePostVOList = worksGroupVO.mMorePostVOList;
      mExpand = worksGroupVO.mExpand;
      mIsLoading = worksGroupVO.mIsLoading;
    }

    public Builder setGroupTitle(String groupTitle) {
      mGroupTitle = groupTitle  == null ? "" : groupTitle;
      return this;
    }

    public Builder setPostVOList(List<PostVO> postVOList) {
      mPostVOList = postVOList == null ? new ArrayList<PostVo>()  : postVOList;
      return this;
    }

    public Builder setMorePostVOList(List<PostVO> morePostVOList) {
      mMorePostVOList = morePostVOList == null ? new ArrayList<PostVo>() : morePostVOList;;
      return this;
    }

    public Builder setExpand(boolean expand) {
      mExpand = expand;
      return this;
    }

    public Builder setLoading(boolean loading) {
      mIsLoading = loading;
      return this;
    }

    public WorksGroupVO build() {

      return new WorksGroupVO(this);
    }
  }
}

```

ViewModule层
```java
  public Observable<List<WorksGroupVO>> fetchWorksGroupList(String groupId, int pageNum) {

    return RetrofitWrapperStore.getInstance()
        .provideApi(Api.class)
        .fetchWorksGroupList(groupId, pageNum)
        .compose(RxServerResultTransformer.<List<WorksGroupDTO>>transform())
        .map(new Func1<List<WorksGroupDTO>, List<WorksGroupVO>>() {

          @Override public List<WorksGroupVO> call(List<WorksGroupDTO> worksGroupDTOs) {

            return ListTransformHelper.dto2vo(worksGroupDTOs);
          }
        })
        .subscribeOn(Schedulers.io());
  }

```
## 解决痛点
1. 后台返回数据为null，在dto2vo()中已经进行了处理。
2. 接口返回的数据结构或字段突然变更，使用这种Mappe之后，大多数情况，只需要更改DTO里的dto2vo()的实现即可
3. 接口返回的数据结构不是我想要的？把它转成我们想要的VO对象即可如int形的gender转换成boolean形的isMale.又比如在RecyclerView中显示的View不同，即属于不同的ItemType。
```java
 @Override
 public int getItemViewType(int position) {
    WorksGroupVO worksgroup = mDatas.get(position);
    return worksgroup.isExpand() ? 0 : 1;
 }
```
这样增加了View层的逻辑判断。可在Bean中增加itemType
```java
public class WorksGroupVO{
   ...
   public int itemType;
}
  @Override public WorksGroupVO dto2vo() {

    return new WorksGroupVO.Builder().setGroupTitle(groupTitle)
        .setPostVOList(translatePosts())
        .setExpand(mExpand)
		.setItemType(mExpand ? 0 : 1)
        .build();
  }
```
V层应该专注V应该做的事，而不是数据的处理、业务逻辑！

4 接口文档迟迟没有,在事先拿到设计稿时，可以在VO中先把数据结构搭好，利用VO模拟数据。后台好了后直接dto2vo()


## 总结
从实用角度出发：Mapper可以让开发过程更高效，更从容面对需求变更、接口变更等外在因素。

从架构角度出发：Mapper的出现可以让M层可以承担更多的职责，来减轻中间层的压力，更利于构造一个V，M的分离的架构。

