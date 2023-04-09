---
title: "Android一句话集成自定义表格"
date: 2023-04-07T10:29:12+08:00
tags: ["Android"]
categories: [ "移动端"]
draft: false
---
# LockTableView
自定义表格,可锁定双向表头,自适应列宽、自适应行高、下拉刷新、上拉加载、链式调用、快速集成。<br>

## 效果展示
![](/images/lock_table.webp)

## Github
[Github-LockTableView](https://github.com/RmondJone/LockTableView)<br>
欢迎大家点赞(Star)，你的鼓励是我前行的动力！我的宗旨：简单！实用！

## 工程集成说明
* 第一步
```java
//在工程gradle文件里
allprojects {
    repositories {
        .......
        maven { url 'https://jitpack.io' }
        ......
    }
}
```

```java
//如果不在工程gradle文件里加入，也可以加入模块gradle文件中
repositories {
    maven {
        url  "https://jitpack.io"
    }
}
```

* 第二步
```java
dependencies {
	compile 'com.github.RmondJone:LockTableView:1.0.9'
}
```

## API使用说明

```java
final LockTableView mLockTableView = new LockTableView(this, mContentView, mTableDatas);
Log.e("表格加载开始", "当前线程：" + Thread.currentThread());
mLockTableView.setLockFristColumn(true) //是否锁定第一列
      .setLockFristRow(true) //是否锁定第一行
      .setMaxColumnWidth(100) //列最大宽度
      .setMinColumnWidth(60) //列最小宽度
      .setMinRowHeight(20)//行最小高度
      .setMaxRowHeight(60)//行最大高度
      .setTextViewSize(16) //单元格字体大小
      .setFristRowBackGroudColor(R.color.table_head)//表头背景色
      .setTableHeadTextColor(R.color.beijin)//表头字体颜色
      .setTableContentTextColor(R.color.border_color)//单元格字体颜色
      .setNullableString("N/A") //空值替换值
      .setTableViewListener(new LockTableView.OnTableViewListener() {
          //设置横向滚动监听
          @Override
          public void onTableViewScrollChange(int x, int y) {
              Log.e("滚动值","["+x+"]"+"["+y+"]");
          }
      })
      .setTableViewRangeListener(new LockTableView.OnTableViewRangeListener() {
                    //设置横向滚动边界监听
                    @Override
                    public void onLeft(HorizontalScrollView view) {
                        Log.e("滚动边界","滚动到最左边");
                    }

                    @Override
                    public void onRight(HorizontalScrollView view) {
                        Log.e("滚动边界","滚动到最右边");
                    }
                })
      .setOnLoadingListener(new LockTableView.OnLoadingListener() {
          //下拉刷新、上拉加载监听
          @Override
          public void onRefresh(final XRecyclerView mXRecyclerView, final ArrayList<ArrayList<String>> mTableDatas) {
              Log.e("表格主视图",mXRecyclerView);
              Log.e("表格所有数据",mTableDatas);
              //如需更新表格数据调用,部分刷新不会全部重绘
              mLockTableView.setTableDatas(mTableDatas);
              //停止刷新
              mXRecyclerView.refreshComplete();
          }

          @Override
          public void onLoadMore(final XRecyclerView mXRecyclerView, final ArrayList<ArrayList<String>> mTableDatas) {
              Log.e("表格主视图",mXRecyclerView);
              Log.e("表格所有数据",mTableDatas);
              //如需更新表格数据调用,部分刷新不会全部重绘
              mLockTableView.setTableDatas(mTableDatas);
              //停止刷新
              mXRecyclerView.loadMoreComplete();
              //如果没有更多数据调用
              mXRecyclerView.setNoMore(true);
          }
      })
      .setOnItemClickListenter(new LockTableView.OnItemClickListenter() {
          @Override
          public void onItemClick(View item, int position) {
              Log.e("点击事件",position+"");
          }
      })
      .setOnItemLongClickListenter(new LockTableView.OnItemLongClickListenter() {
          @Override
          public void onItemLongClick(View item, int position) {
             Log.e("长按事件",position+"");
          }
      })
      .setOnItemSeletor(R.color.dashline_color)//设置Item被选中颜色
      .show(); //显示表格,此方法必须调用
mLockTableView.getTableScrollView().setPullRefreshEnabled(true);
mLockTableView.getTableScrollView().setLoadingMoreEnabled(true);
mLockTableView.getTableScrollView().setRefreshProgressStyle(ProgressStyle.SquareSpin);
//属性值获取
Log.e("每列最大宽度(dp)", mLockTableView.getColumnMaxWidths().toString());
Log.e("每行最大高度(dp)", mLockTableView.getRowMaxHeights().toString());
Log.e("表格所有的滚动视图", mLockTableView.getScrollViews().toString());
Log.e("表格头部固定视图(锁列)", mLockTableView.getLockHeadView().toString());
Log.e("表格头部固定视图(不锁列)", mLockTableView.getUnLockHeadView().toString());

/**
 * 构造方法
 *
 * @param mContext     上下文
 * @param mContentView 表格父视图
 * @param mTableDatas  表格数据
 */
public LockTableView(Context mContext, ViewGroup mContentView, ArrayList<ArrayList<String>> mTableDatas) {
    this.mContext = mContext;
    this.mContentView = mContentView;
    this.mTableDatas = mTableDatas;
    initAttrs();
}

```
## 目前支持可自定义属性

```java
/**
 * 是否锁定首行
 */
private boolean isLockFristRow = true;
/**
 * 是否锁定首列
 */
private boolean isLockFristColumn = true;
/**
 * 最大列宽(dp)
 */
private int maxColumnWidth;
/**
 * 最小列宽(dp)
 */
private int minColumnWidth;
/**
 * 最大行高(dp)
 */
private int maxRowHeight;
/**
  * 最小行高dp)
  */
private int minRowHeight;
/**
 * 第一行背景颜色
 */
private int mFristRowBackGroudColor;
/**
 * 数据为空时的缺省值
 */
private String mNullableString;
/**
 * 单元格字体大小
 */
private int mTextViewSize;
/**
 * 表格头部字体颜色
 */
private int mTableHeadTextColor;
/**
 * 表格内容字体颜色
 */
private int mTableContentTextColor;
/**
 * 表格横向滚动监听事件
 */
private OnTableViewListener mTableViewListener;
/**
 * 表格横向滚动到边界监听事件
 */
private OnTableViewRangeListener mTableViewRangeListener;
/**
 * 表格上拉刷新、下拉加载监听事件
 */
private OnLoadingListener mOnLoadingListener;
/**
 * Item点击事件
 */
private OnItemClickListenter mOnItemClickListenter;
/**
 * Item长按事件
 */
private OnItemLongClickListenter mOnItemLongClickListenter;
/**
 * Item选中颜色
 */
private int mOnItemSeletor;


```

## 问题反馈
* 技术交流群：QQ(264587303)
* Demo作者：郭翰林
* 注：有定制化需求自己下源码根据自己的需求改动，不要指望别人给你实现，这样永远没有成长！
* 本控件实现没有难度，只要静心看代码都能看的懂。我只提供最基础的功能，尽量满足大部分的开发需求。

## 关于作者
掘金：[https://juejin.im/user/5a2b6b306fb9a04522076dd0/posts](https://juejin.im/user/5a2b6b306fb9a04522076dd0/posts)
简书：[https://www.jianshu.com/u/7566e4604f87](https://www.jianshu.com/u/7566e4604f87)
GitHub:[https://github.com/RmondJone](https://github.com/RmondJone)

## License
```java
Copyright (c) 2018 Guohanlin

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

