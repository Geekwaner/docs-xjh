# 鸿蒙开发之拖拽表格

在鸿蒙应用开发中，表格是常用的数据展示方式，而支持拖拽功能的表格可以提供更灵活的用户交互体验。本文将介绍如何在鸿蒙应用中实现一个支持拖拽排序的表格组件。

## 目录

- [基础表格实现](#基础表格实现)
- [拖拽功能实现](#拖拽功能实现)
- [完整拖拽表格组件](#完整拖拽表格组件)
- [实际应用场景](#实际应用场景)

## 基础表格实现

首先，我们需要创建一个基础的表格组件，使用 `List` 和 `Grid` 组件来实现：

```typescript
@Entry
@Component
struct BasicTableDemo {
  // 表格数据
  @State tableData: Array<{
    id: number,
    name: string,
    age: number,
    address: string
  }> = [
    { id: 1, name: '张三', age: 28, address: '北京市朝阳区' },
    { id: 2, name: '李四', age: 32, address: '上海市浦东新区' },
    { id: 3, name: '王五', age: 24, address: '广州市天河区' },
    { id: 4, name: '赵六', age: 35, address: '深圳市南山区' },
    { id: 5, name: '钱七', age: 29, address: '杭州市西湖区' }
  ]

  // 表格列定义
  private columns: Array<{
    title: string,
    dataIndex: string,
    width: string
  }> = [
    { title: '姓名', dataIndex: 'name', width: '25%' },
    { title: '年龄', dataIndex: 'age', width: '25%' },
    { title: '地址', dataIndex: 'address', width: '50%' }
  ]

  build() {
    Column() {
      Text('基础表格示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 20 })

      // 表头
      Row() {
        ForEach(this.columns, (column) => {
          Text(column.title)
            .fontWeight(FontWeight.Bold)
            .width(column.width)
            .textAlign(TextAlign.Center)
            .padding(10)
            .backgroundColor('#f0f0f0')
            .border({
              width: { right: 1, bottom: 1 },
              color: '#ddd',
              style: BorderStyle.Solid
            })
        })
      }
      .width('100%')
      .border({
        width: { top: 1, left: 1 },
        color: '#ddd',
        style: BorderStyle.Solid
      })

      // 表格内容
      List() {
        ForEach(this.tableData, (row) => {
          ListItem() {
            Row() {
              ForEach(this.columns, (column) => {
                Text(row[column.dataIndex])
                  .width(column.width)
                  .textAlign(TextAlign.Center)
                  .padding(10)
                  .border({
                    width: { right: 1, bottom: 1 },
                    color: '#ddd',
                    style: BorderStyle.Solid
                  })
              })
            }
            .width('100%')
            .border({
              width: { left: 1 },
              color: '#ddd',
              style: BorderStyle.Solid
            })
          }
        })
      }
      .width('100%')
      .height('80%')
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

## 拖拽功能实现

接下来，我们将为表格增加拖拽排序功能，使用 `onDragStart`、`onDragEnter` 和 `onDrop` 事件：

```typescript
@Entry
@Component
struct DraggableTableDemo {
  // 表格数据
  @State tableData: Array<{
    id: number,
    name: string,
    age: number,
    address: string
  }> = [
    { id: 1, name: '张三', age: 28, address: '北京市朝阳区' },
    { id: 2, name: '李四', age: 32, address: '上海市浦东新区' },
    { id: 3, name: '王五', age: 24, address: '广州市天河区' },
    { id: 4, name: '赵六', age: 35, address: '深圳市南山区' },
    { id: 5, name: '钱七', age: 29, address: '杭州市西湖区' }
  ]

  // 表格列定义
  private columns: Array<{
    title: string,
    dataIndex: string,
    width: string
  }> = [
    { title: '姓名', dataIndex: 'name', width: '25%' },
    { title: '年龄', dataIndex: 'age', width: '25%' },
    { title: '地址', dataIndex: 'address', width: '50%' }
  ]

  // 拖拽状态
  @State dragIndex: number = -1
  @State dragEnterIndex: number = -1

  build() {
    Column() {
      Text('可拖拽表格示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 20 })

      // 表头
      Row() {
        ForEach(this.columns, (column) => {
          Text(column.title)
            .fontWeight(FontWeight.Bold)
            .width(column.width)
            .textAlign(TextAlign.Center)
            .padding(10)
            .backgroundColor('#f0f0f0')
            .border({
              width: { right: 1, bottom: 1 },
              color: '#ddd',
              style: BorderStyle.Solid
            })
        })
      }
      .width('100%')
      .border({
        width: { top: 1, left: 1 },
        color: '#ddd',
        style: BorderStyle.Solid
      })

      // 表格内容
      List() {
        ForEach(this.tableData, (row, index) => {
          ListItem() {
            Row() {
              ForEach(this.columns, (column) => {
                Text(row[column.dataIndex])
                  .width(column.width)
                  .textAlign(TextAlign.Center)
                  .padding(10)
                  .border({
                    width: { right: 1, bottom: 1 },
                    color: '#ddd',
                    style: BorderStyle.Solid
                  })
              })
            }
            .width('100%')
            .height(50)
            .border({
              width: { left: 1 },
              color: '#ddd',
              style: BorderStyle.Solid
            })
            .backgroundColor(index === this.dragEnterIndex ? '#e6f7ff' :
                             index === this.dragIndex ? '#f9f9f9' : '#ffffff')
            .opacity(index === this.dragIndex ? 0.6 : 1)
            // 拖拽相关事件
            .draggable(true)
            .onDragStart(() => {
              this.dragIndex = index
              return this.tableData[index]
            })
            .onDragEnter(() => {
              this.dragEnterIndex = index
            })
            .onDragLeave(() => {
              if (this.dragEnterIndex === index) {
                this.dragEnterIndex = -1
              }
            })
            .onDrop((event) => {
              // 当拖拽结束时，交换两行数据
              if (this.dragIndex !== -1 && this.dragIndex !== index) {
                const draggedItem = this.tableData[this.dragIndex]
                const newTableData = [...this.tableData]

                // 移除拖拽项
                newTableData.splice(this.dragIndex, 1)

                // 插入到目标位置
                newTableData.splice(index, 0, draggedItem)

                // 更新数据
                this.tableData = newTableData
              }

              // 重置拖拽状态
              this.dragIndex = -1
              this.dragEnterIndex = -1
            })
          }
        })
      }
      .width('100%')
      .height('80%')

      // 显示当前数据顺序
      Text('当前数据顺序:')
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .margin({ top: 20 })

      Text(JSON.stringify(this.tableData.map(item => item.name)))
        .fontSize(14)
        .margin({ top: 10 })
        .width('100%')
        .textAlign(TextAlign.Center)
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

## 完整拖拽表格组件

下面是一个完整的、可复用的拖拽表格组件：

```typescript
// 表格列定义接口
interface ColumnDef {
  title: string;        // 列标题
  dataIndex: string;    // 数据字段名
  width: string;        // 列宽度
  render?: (text: any, record: any, index: number) => void;  // 自定义渲染函数
}

// 拖拽表格组件
@Component
export struct DraggableTable {
  // 表格数据
  @Link tableData: Array<Record<string, any>>;
  // 列定义
  private columns: Array<ColumnDef>;
  // 是否允许拖拽
  @Prop allowDrag: boolean = true;
  // 拖拽回调
  private onRowDragEnd?: (fromIndex: number, toIndex: number) => void;

  // 拖拽状态
  @State dragIndex: number = -1;
  @State dragEnterIndex: number = -1;

  // 处理拖拽结束
  handleDrop(fromIndex: number, toIndex: number) {
    if (fromIndex !== -1 && fromIndex !== toIndex) {
      // 创建数据副本
      const newTableData = [...this.tableData];

      // 移除拖拽项
      const draggedItem = newTableData.splice(fromIndex, 1)[0];

      // 插入到目标位置
      newTableData.splice(toIndex, 0, draggedItem);

      // 更新数据
      this.tableData = newTableData;

      // 调用回调
      if (this.onRowDragEnd) {
        this.onRowDragEnd(fromIndex, toIndex);
      }
    }

    // 重置拖拽状态
    this.dragIndex = -1;
    this.dragEnterIndex = -1;
  }

  build() {
    Column() {
      // 表头
      Row() {
        ForEach(this.columns, (column) => {
          Text(column.title)
            .fontWeight(FontWeight.Bold)
            .width(column.width)
            .textAlign(TextAlign.Center)
            .padding(10)
            .backgroundColor('#f0f0f0')
            .border({
              width: { right: 1, bottom: 1 },
              color: '#ddd',
              style: BorderStyle.Solid
            })
        })
      }
      .width('100%')
      .border({
        width: { top: 1, left: 1 },
        color: '#ddd',
        style: BorderStyle.Solid
      })

      // 表格内容
      List() {
        ForEach(this.tableData, (row, index) => {
          ListItem() {
            Row() {
              ForEach(this.columns, (column) => {
                if (column.render) {
                  // 使用自定义渲染函数
                  column.render(row[column.dataIndex], row, index)
                } else {
                  // 默认渲染
                  Text(row[column.dataIndex] !== undefined ? row[column.dataIndex].toString() : '')
                    .width(column.width)
                    .textAlign(TextAlign.Center)
                    .padding(10)
                    .border({
                      width: { right: 1, bottom: 1 },
                      color: '#ddd',
                      style: BorderStyle.Solid
                    })
                }
              })
            }
            .width('100%')
            .height(50)
            .border({
              width: { left: 1 },
              color: '#ddd',
              style: BorderStyle.Solid
            })
            .backgroundColor(index === this.dragEnterIndex ? '#e6f7ff' :
                             index === this.dragIndex ? '#f9f9f9' : '#ffffff')
            .opacity(index === this.dragIndex ? 0.6 : 1)
            // 拖拽相关事件
            .draggable(this.allowDrag)
            .onDragStart(() => {
              if (this.allowDrag) {
                this.dragIndex = index;
                return row;
              }
              return undefined;
            })
            .onDragEnter(() => {
              if (this.allowDrag) {
                this.dragEnterIndex = index;
              }
            })
            .onDragLeave(() => {
              if (this.allowDrag && this.dragEnterIndex === index) {
                this.dragEnterIndex = -1;
              }
            })
            .onDrop(() => {
              if (this.allowDrag) {
                this.handleDrop(this.dragIndex, index);
              }
            })
          }
        })
      }
      .width('100%')
      .layoutWeight(1)
    }
    .width('100%')
    .height('100%')
  }
}
```

## 实际应用场景

下面是一个使用拖拽表格组件的实际应用示例，实现了一个任务管理器：

```typescript
@Entry
@Component
struct TaskManagerDemo {
  // 任务数据
  @State tasks: Array<{
    id: number,
    title: string,
    priority: string,
    status: string,
    dueDate: string
  }> = [
    { id: 1, title: '完成项目报告', priority: '高', status: '进行中', dueDate: '2023-12-10' },
    { id: 2, title: '客户会议准备', priority: '中', status: '未开始', dueDate: '2023-12-15' },
    { id: 3, title: '更新网站内容', priority: '低', status: '已完成', dueDate: '2023-12-05' },
    { id: 4, title: '发布新功能', priority: '高', status: '未开始', dueDate: '2023-12-20' },
    { id: 5, title: '团队周会', priority: '中', status: '进行中', dueDate: '2023-12-12' }
  ]

  // 表格列定义
  private columns: Array<{
    title: string,
    dataIndex: string,
    width: string,
    render?: (text: any, record: any, index: number) => void
  }> = [
    {
      title: '任务名称',
      dataIndex: 'title',
      width: '30%'
    },
    {
      title: '优先级',
      dataIndex: 'priority',
      width: '15%',
      render: (text) => {
        Column() {
          Text(text)
            .fontColor(text === '高' ? '#ff4d4f' :
                      text === '中' ? '#faad14' : '#52c41a')
            .fontWeight(text === '高' ? FontWeight.Bold : FontWeight.Normal)
        }
        .width('100%')
        .height('100%')
        .justifyContent(FlexAlign.Center)
        .border({
          width: { right: 1, bottom: 1 },
          color: '#ddd',
          style: BorderStyle.Solid
        })
      }
    },
    {
      title: '状态',
      dataIndex: 'status',
      width: '20%',
      render: (text) => {
        Column() {
          Text(text)
            .backgroundColor(text === '已完成' ? '#d9f7be' :
                           text === '进行中' ? '#e6f7ff' : '#fff7e6')
            .padding(5)
            .borderRadius(5)
        }
        .width('100%')
        .height('100%')
        .justifyContent(FlexAlign.Center)
        .border({
          width: { right: 1, bottom: 1 },
          color: '#ddd',
          style: BorderStyle.Solid
        })
      }
    },
    {
      title: '截止日期',
      dataIndex: 'dueDate',
      width: '20%'
    },
    {
      title: '操作',
      dataIndex: 'operation',
      width: '15%',
      render: (_, record, index) => {
        Row({ space: 5 }) {
          Button({ type: ButtonType.Circle, stateEffect: true }) {
            Image($r('app.media.ic_public_edit')).width(20).height(20)
          }
          .width(30)
          .height(30)
          .backgroundColor('#e6f7ff')
          .onClick(() => {
            // 编辑任务
            promptAction.showDialog({
              title: '编辑任务',
              message: `正在编辑: ${record.title}`,
              buttons: [
                { text: '取消', color: '#999' },
                { text: '确定', color: '#007DFF' }
              ]
            })
          })

          Button({ type: ButtonType.Circle, stateEffect: true }) {
            Image($r('app.media.ic_public_delete')).width(20).height(20)
          }
          .width(30)
          .height(30)
          .backgroundColor('#fff1f0')
          .onClick(() => {
            // 删除任务
            this.tasks = this.tasks.filter((_, i) => i !== index)
          })
        }
        .width('100%')
        .height('100%')
        .justifyContent(FlexAlign.Center)
        .border({
          width: { right: 1, bottom: 1 },
          color: '#ddd',
          style: BorderStyle.Solid
        })
      }
    }
  ]

  build() {
    Column({ space: 20 }) {
      Text('任务管理器')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      // 添加任务按钮
      Button('添加任务')
        .width(120)
        .onClick(() => {
          // 添加新任务
          const newTask = {
            id: this.tasks.length + 1,
            title: '新任务 ' + (this.tasks.length + 1),
            priority: '中',
            status: '未开始',
            dueDate: '2023-12-31'
          }
          this.tasks = [...this.tasks, newTask]
        })

      // 使用拖拽表格组件
      DraggableTable({
        tableData: $tasks,
        columns: this.columns,
        allowDrag: true,
        onRowDragEnd: (fromIndex, toIndex) => {
          console.info(`任务从第 ${fromIndex + 1} 位移动到第 ${toIndex + 1} 位`)
        }
      })
      .width('100%')
      .layoutWeight(1)

      // 拖拽提示
      Text('提示: 拖拽任务行可以调整优先顺序')
        .fontSize(14)
        .fontColor('#999')
        .margin({ top: 10 })
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

通过以上示例，我们展示了如何在鸿蒙应用中创建一个支持拖拽排序的表格组件，这对于需要灵活数据展示和交互的应用场景非常有用，如任务管理、待办事项、优先级排序等。通过合理应用拖拽功能，可以大大提升用户体验。
