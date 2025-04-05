# 鸿蒙开发之封装懒加载 list 组件

在鸿蒙应用开发中，列表是最常用的 UI 组件之一。为了提高性能，我们需要实现列表的懒加载功能，使得只有在用户滚动到视图范围内的内容才会被加载和渲染。本文将介绍如何封装一个功能完善的懒加载列表组件。

## 目录

- [基础列表实现](#基础列表实现)
- [懒加载机制](#懒加载机制)
- [上拉加载更多](#上拉加载更多)
- [下拉刷新](#下拉刷新)
- [完整组件实现](#完整组件实现)

## 基础列表实现

首先，我们来实现一个基础的列表组件：

```typescript
@Component
struct BasicList {
  @State dataList: Array<any> = []

  build() {
    List() {
      ForEach(this.dataList, (item, index) => {
        ListItem() {
          Text(`${item.title}`)
            .width('100%')
            .height(50)
            .backgroundColor(Color.White)
            .borderRadius(10)
            .padding(10)
        }
        .margin({ bottom: 10 })
      })
    }
    .width('100%')
    .height('100%')
    .divider({ strokeWidth: 1, color: '#f1f1f1' })
  }
}
```

## 懒加载机制

接下来，我们添加懒加载功能，使用`LazyForEach`代替`ForEach`：

```typescript
// 定义数据源类
class MyDataSource implements IDataSource {
  private dataArray: Array<any> = []
  private listener: DataChangeListener

  constructor(data: Array<any>) {
    this.dataArray = data
  }

  // 获取索引对应的数据
  public getData(index: number): any {
    return this.dataArray[index]
  }

  // 获取数据总数
  public totalCount(): number {
    return this.dataArray.length
  }

  // 注册数据变化监听器
  registerDataChangeListener(listener: DataChangeListener): void {
    this.listener = listener
  }

  // 注销数据变化监听器
  unregisterDataChangeListener() {
    this.listener = undefined
  }

  // 通知数据变化
  notifyDataReload(): void {
    if (this.listener) {
      this.listener.onDataReloaded()
    }
  }

  // 通知数据增加
  notifyDataAdd(index: number): void {
    if (this.listener) {
      this.listener.onDataAdd(index)
    }
  }

  // 通知数据变化
  notifyDataChange(index: number): void {
    if (this.listener) {
      this.listener.onDataChange(index)
    }
  }

  // 通知数据删除
  notifyDataDelete(index: number): void {
    if (this.listener) {
      this.listener.onDataDelete(index)
    }
  }
}

@Component
struct LazyLoadList {
  @State dataArray: Array<any> = []
  private dataSource: MyDataSource = new MyDataSource([])

  aboutToAppear() {
    // 初始化数据
    this.dataArray = Array.from({ length: 100 }, (_, i) => ({
      id: i,
      title: `Item ${i}`
    }))
    // 创建数据源
    this.dataSource = new MyDataSource(this.dataArray)
  }

  build() {
    List() {
      LazyForEach(this.dataSource, (item, index) => {
        ListItem() {
          Row() {
            Text(`${item.title}`)
              .fontSize(16)
          }
          .width('100%')
          .height(60)
          .backgroundColor(Color.White)
          .borderRadius(8)
          .padding(10)
        }
        .margin({ bottom: 10 })
      }, item => item.id.toString()) // 提供唯一标识
    }
    .width('100%')
    .height('100%')
    .divider({ strokeWidth: 1, color: '#f1f1f1' })
  }
}
```

## 上拉加载更多

现在，我们添加上拉加载更多的功能：

```typescript
@Component
struct LoadMoreList {
  @State dataArray: Array<any> = []
  @State isLoading: boolean = false
  @State hasMoreData: boolean = true
  private dataSource: MyDataSource = new MyDataSource([])
  private pageSize: number = 20
  private currentPage: number = 1

  aboutToAppear() {
    // 初始加载第一页数据
    this.loadData()
  }

  async loadData() {
    if (this.isLoading || !this.hasMoreData) {
      return
    }

    this.isLoading = true

    try {
      // 模拟网络请求
      await new Promise(resolve => setTimeout(resolve, 1000))

      // 生成新数据
      const newItems = Array.from({ length: this.pageSize }, (_, i) => {
        const id = (this.currentPage - 1) * this.pageSize + i
        return {
          id,
          title: `Item ${id}`
        }
      })

      // 如果没有更多数据
      if (this.currentPage >= 5) {
        this.hasMoreData = false
      }

      // 添加新数据到数组
      this.dataArray = [...this.dataArray, ...newItems]
      this.dataSource = new MyDataSource(this.dataArray)
      this.currentPage++
    } finally {
      this.isLoading = false
    }
  }

  build() {
    Column() {
      List() {
        LazyForEach(this.dataSource, (item, index) => {
          ListItem() {
            Row() {
              Text(`${item.title}`)
                .fontSize(16)
            }
            .width('100%')
            .height(60)
            .backgroundColor(Color.White)
            .borderRadius(8)
            .padding(10)
          }
          .margin({ bottom: 10 })
        }, item => item.id.toString())

        // 底部加载更多视图
        if (this.dataArray.length > 0) {
          ListItem() {
            Row() {
              if (this.isLoading) {
                LoadingProgress()
                  .width(24)
                  .height(24)
                Text('加载中...')
                  .fontSize(14)
                  .margin({ left: 10 })
              } else if (!this.hasMoreData) {
                Text('没有更多数据')
                  .fontSize(14)
              } else {
                Text('上拉加载更多')
                  .fontSize(14)
              }
            }
            .width('100%')
            .height(50)
            .justifyContent(FlexAlign.Center)
          }
        }
      }
      .width('100%')
      .height('100%')
      .divider({ strokeWidth: 1, color: '#f1f1f1' })
      .onReachEnd(() => {
        if (this.hasMoreData && !this.isLoading) {
          this.loadData()
        }
      })
    }
    .width('100%')
    .height('100%')
  }
}
```

## 下拉刷新

接下来，我们添加下拉刷新功能：

```typescript
@Component
struct PullToRefreshList {
  @State dataArray: Array<any> = []
  @State isLoading: boolean = false
  @State isRefreshing: boolean = false
  @State hasMoreData: boolean = true
  private dataSource: MyDataSource = new MyDataSource([])
  private pageSize: number = 20
  private currentPage: number = 1

  aboutToAppear() {
    // 初始加载第一页数据
    this.loadData()
  }

  async loadData(isRefresh: boolean = false) {
    if (this.isLoading || (this.isRefreshing && !isRefresh)) {
      return
    }

    if (isRefresh) {
      this.isRefreshing = true
      this.currentPage = 1
    } else {
      this.isLoading = true
    }

    try {
      // 模拟网络请求
      await new Promise(resolve => setTimeout(resolve, 1000))

      // 生成新数据
      const newItems = Array.from({ length: this.pageSize }, (_, i) => {
        const id = (this.currentPage - 1) * this.pageSize + i
        return {
          id,
          title: `Item ${id} ${isRefresh ? '(Refreshed)' : ''}`
        }
      })

      // 如果没有更多数据
      if (this.currentPage >= 5) {
        this.hasMoreData = false
      } else {
        this.hasMoreData = true
      }

      // 更新数据
      if (isRefresh) {
        this.dataArray = [...newItems]
      } else {
        this.dataArray = [...this.dataArray, ...newItems]
      }

      this.dataSource = new MyDataSource(this.dataArray)
      this.currentPage++
    } finally {
      if (isRefresh) {
        this.isRefreshing = false
      } else {
        this.isLoading = false
      }
    }
  }

  build() {
    Column() {
      Refresh({ refreshing: $$this.isRefreshing }) {
        List() {
          LazyForEach(this.dataSource, (item, index) => {
            ListItem() {
              Row() {
                Text(`${item.title}`)
                  .fontSize(16)
              }
              .width('100%')
              .height(60)
              .backgroundColor(Color.White)
              .borderRadius(8)
              .padding(10)
            }
            .margin({ bottom: 10 })
          }, item => item.id.toString())

          // 底部加载更多视图
          if (this.dataArray.length > 0) {
            ListItem() {
              Row() {
                if (this.isLoading) {
                  LoadingProgress()
                    .width(24)
                    .height(24)
                  Text('加载中...')
                    .fontSize(14)
                    .margin({ left: 10 })
                } else if (!this.hasMoreData) {
                  Text('没有更多数据')
                    .fontSize(14)
                } else {
                  Text('上拉加载更多')
                    .fontSize(14)
                }
              }
              .width('100%')
              .height(50)
              .justifyContent(FlexAlign.Center)
            }
          }
        }
        .width('100%')
        .divider({ strokeWidth: 1, color: '#f1f1f1' })
        .onReachEnd(() => {
          if (this.hasMoreData && !this.isLoading) {
            this.loadData()
          }
        })
      }
      .onRefresh(() => {
        this.loadData(true)
      })
    }
    .width('100%')
    .height('100%')
  }
}
```

## 完整组件实现

最后，我们封装一个可复用的懒加载列表组件：

```typescript
// 通用数据源类
export class LazyDataSource implements IDataSource {
  private dataArray: Array<any> = []
  private listener: DataChangeListener

  constructor(data: Array<any> = []) {
    this.dataArray = data
  }

  public getData(index: number): any {
    return this.dataArray[index]
  }

  public totalCount(): number {
    return this.dataArray.length
  }

  public updateData(data: Array<any>): void {
    this.dataArray = data
    this.notifyDataReload()
  }

  public addData(data: Array<any>): void {
    const startIndex = this.dataArray.length
    this.dataArray = [...this.dataArray, ...data]
    for (let i = 0; i < data.length; i++) {
      this.notifyDataAdd(startIndex + i)
    }
  }

  registerDataChangeListener(listener: DataChangeListener): void {
    this.listener = listener
  }

  unregisterDataChangeListener() {
    this.listener = undefined
  }

  notifyDataReload(): void {
    if (this.listener) {
      this.listener.onDataReloaded()
    }
  }

  notifyDataAdd(index: number): void {
    if (this.listener) {
      this.listener.onDataAdd(index)
    }
  }

  notifyDataChange(index: number): void {
    if (this.listener) {
      this.listener.onDataChange(index)
    }
  }

  notifyDataDelete(index: number): void {
    if (this.listener) {
      this.listener.onDataDelete(index)
    }
  }
}

// 懒加载列表组件
@Component
export struct LazyLoadingList {
  // 是否启用下拉刷新
  @Prop enablePullToRefresh: boolean = true
  // 是否启用上拉加载更多
  @Prop enableLoadMore: boolean = true
  // 数据源
  @Link dataSource: LazyDataSource
  // 自定义列表项渲染器
  @BuilderParam itemBuilder: (item: any, index?: number) => void
  // 自定义空数据视图
  @BuilderParam emptyBuilder?: () => void
  // 自定义加载中视图
  @BuilderParam loadingBuilder?: () => void
  // 自定义加载更多视图
  @BuilderParam loadMoreBuilder?: () => void
  // 自定义无更多数据视图
  @BuilderParam noMoreDataBuilder?: () => void
  // 加载中状态
  @Link isLoading: boolean
  // 刷新中状态
  @Link isRefreshing: boolean
  // 是否有更多数据
  @Link hasMoreData: boolean
  // 加载更多回调
  private onLoadMore?: () => void
  // 下拉刷新回调
  private onRefresh?: () => void
  // 列表项唯一标识回调
  private keyGenerator: (item: any) => string = item => item.id?.toString() || item.toString()

  build() {
    Column() {
      if (this.enablePullToRefresh) {
        Refresh({ refreshing: $$this.isRefreshing }) {
          this.buildList()
        }
        .onRefresh(() => {
          if (this.onRefresh) {
            this.onRefresh()
          }
        })
      } else {
        this.buildList()
      }
    }
    .width('100%')
    .height('100%')
  }

  @Builder
  buildList() {
    List() {
      if (this.dataSource.totalCount() === 0 && !this.isLoading && !this.isRefreshing) {
        // 空数据视图
        ListItem() {
          if (this.emptyBuilder) {
            this.emptyBuilder()
          } else {
            Column() {
              Text('暂无数据')
                .fontSize(16)
                .fontColor('#999')
            }
            .width('100%')
            .height(200)
            .justifyContent(FlexAlign.Center)
          }
        }
      } else {
        // 数据列表
        LazyForEach(this.dataSource, (item, index) => {
          ListItem() {
            this.itemBuilder(item, index)
          }
        }, this.keyGenerator)

        // 加载更多视图
        if (this.enableLoadMore && this.dataSource.totalCount() > 0) {
          ListItem() {
            if (this.isLoading) {
              if (this.loadingBuilder) {
                this.loadingBuilder()
              } else {
                Row() {
                  LoadingProgress()
                    .width(24)
                    .height(24)
                  Text('加载中...')
                    .fontSize(14)
                    .margin({ left: 10 })
                }
                .width('100%')
                .height(50)
                .justifyContent(FlexAlign.Center)
              }
            } else if (!this.hasMoreData) {
              if (this.noMoreDataBuilder) {
                this.noMoreDataBuilder()
              } else {
                Row() {
                  Text('没有更多数据')
                    .fontSize(14)
                    .fontColor('#999')
                }
                .width('100%')
                .height(50)
                .justifyContent(FlexAlign.Center)
              }
            } else {
              if (this.loadMoreBuilder) {
                this.loadMoreBuilder()
              } else {
                Row() {
                  Text('上拉加载更多')
                    .fontSize(14)
                    .fontColor('#999')
                }
                .width('100%')
                .height(50)
                .justifyContent(FlexAlign.Center)
              }
            }
          }
        }
      }
    }
    .width('100%')
    .height('100%')
    .listDirection(Axis.Vertical)
    .onReachEnd(() => {
      if (this.enableLoadMore && this.hasMoreData && !this.isLoading && this.onLoadMore) {
        this.onLoadMore()
      }
    })
  }
}
```

## 使用示例

```typescript
@Entry
@Component
struct ListDemoPage {
  @State dataArray: Array<any> = []
  @State dataSource: LazyDataSource = new LazyDataSource([])
  @State isLoading: boolean = false
  @State isRefreshing: boolean = false
  @State hasMoreData: boolean = true
  private pageSize: number = 20
  private currentPage: number = 1

  aboutToAppear() {
    // 初始加载第一页数据
    this.loadData()
  }

  async loadData(isRefresh: boolean = false) {
    if (this.isLoading || (this.isRefreshing && !isRefresh)) {
      return
    }

    if (isRefresh) {
      this.isRefreshing = true
      this.currentPage = 1
    } else {
      this.isLoading = true
    }

    try {
      // 模拟网络请求
      await new Promise(resolve => setTimeout(resolve, 1000))

      // 生成新数据
      const newItems = Array.from({ length: this.pageSize }, (_, i) => {
        const id = (this.currentPage - 1) * this.pageSize + i
        return {
          id,
          title: `Item ${id} ${isRefresh ? '(Refreshed)' : ''}`
        }
      })

      // 如果没有更多数据
      if (this.currentPage >= 5) {
        this.hasMoreData = false
      } else {
        this.hasMoreData = true
      }

      // 更新数据
      if (isRefresh) {
        this.dataArray = [...newItems]
        this.dataSource.updateData(this.dataArray)
      } else {
        this.dataArray = [...this.dataArray, ...newItems]
        this.dataSource.updateData(this.dataArray)
      }

      this.currentPage++
    } finally {
      if (isRefresh) {
        this.isRefreshing = false
      } else {
        this.isLoading = false
      }
    }
  }

  build() {
    Column() {
      Text('懒加载列表示例')
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
        .padding(10)

      LazyLoadingList({
        dataSource: $dataSource,
        isLoading: $isLoading,
        isRefreshing: $isRefreshing,
        hasMoreData: $hasMoreData,
        enablePullToRefresh: true,
        enableLoadMore: true,
        onRefresh: () => this.loadData(true),
        onLoadMore: () => this.loadData(),
      }) {
        itemBuilder: (item, index) => {
          Column() {
            Text(`${item.title}`)
              .fontSize(16)
            Text(`索引: ${index}`)
              .fontSize(12)
              .fontColor('#999')
          }
          .width('100%')
          .padding(16)
          .backgroundColor(Color.White)
          .borderRadius(8)
          .margin({ bottom: 10 })
        }

        emptyBuilder: () => {
          Column() {
            Image($r('app.media.empty'))
              .width(100)
              .height(100)
            Text('暂无数据，下拉刷新试试')
              .fontSize(16)
              .fontColor('#999')
              .margin({ top: 10 })
          }
          .width('100%')
          .height(300)
          .justifyContent(FlexAlign.Center)
        }
      }
    }
    .width('100%')
    .height('100%')
  }
}
```
