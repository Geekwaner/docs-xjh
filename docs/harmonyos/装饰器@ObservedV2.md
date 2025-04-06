# @ObservedV2 状态管理详解

## 引言

在鸿蒙（HarmonyOS）应用开发中，状态管理始终是构建复杂 UI 的核心挑战。随着 HarmonyOS 4.0 的发布，华为推出了新一代状态管理装饰器@ObservedV2，该特性在原有@Observed/@ObjectLink 方案基础上进行了全面升级。本文将解析@ObservedV2 的技术优势和实践方法。

## 与旧版方案对比

| 特性         | @ObservedV2 | @Observed/@ObjectLink |
| ------------ | ----------- | --------------------- |
| 依赖追踪粒度 | 属性级      | 对象级                |
| 数组支持     | 元素级更新  | 全量更新              |
| 初始渲染性能 | 快 30%      | 基准值                |
| 内存占用     | 低 20%      | 较高                  |
| TS 类型支持  | 完整        | 部分                  |
| 嵌套对象监听 | 默认支持    | 需要手动拆分          |

## @ObservedV2 的核心改进

### 依赖追踪机制升级

[官方文档- v2 相比 v1 详解](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V13/arkts-state-management-overview-V13#%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86v1%E7%8E%B0%E7%8A%B6%E4%BB%A5%E5%8F%8Av2%E4%BC%98%E7%82%B9)

- **精准依赖收集**：支持嵌套属性路径监听（如`obj.a.b.c`）
- **数组索引感知**：自动追踪数组元素变化，支持`array[index]`级更新

```ts
@ObservedV2
export class DynamicStylesModel {
  @Trace value?: ESObject;
  @Trace width?: Length;
  @Trace height?: Length;
  @Trace backgroundColor?: string;

  constructor(dynamicStylesModel?: dynamicStyles) {
    this.value = dynamicStylesModel?.value;
    this.width = dynamicStylesModel?.width;
    this.height = dynamicStylesModel?.height;
    this.backgroundColor = dynamicStylesModel?.backgroundColor;
  }
}
```

### 渲染性能优化

- 差分更新算法升级，减少无效渲染
- 依赖关系树形结构存储，比对效率提升 40%
- 支持批量异步更新（类似 React 的 automatic batching）
- 兼容 Map/Set 等复杂数据结构

## 开发实践指南

### 基础使用

- 要监听的属性要添加 `@Trace` 装饰器
- 被监听的属性所在的类要添加 `@ObservedV2`
- 继承类，继承其中的被监听的属性时，可以等价视为是给出自己的类添加了 `@Trace` 装饰器监听

```ts
// 样式
@ObservedV2
export class DynamicStylesModel {
  @Trace value?: ESObject;
  @Trace width?: Length;
  @Trace height?: Length;
  @Trace backgroundColor?: string;

  constructor(dynamicStylesModel?: DynamicStylesModel) {
    this.value = dynamicStylesModel?.value;
    this.width = dynamicStylesModel?.width;
    this.height = dynamicStylesModel?.height;
    this.backgroundColor = dynamicStylesModel?.backgroundColor;
  }
}

// 第一层
export class DynamicComponentModel {
  type?: string;
  styles?: DynamicStylesModel;
  children?: DynamicComponentModel[] = [];
  event?: Function;

  constructor(DynamicComponentModel?: DynamicComponentModel) {
    this.type = DynamicComponentModel?.type;
    this.styles = new DynamicStylesModel(DynamicComponentModel?.styles);
    this.children = DynamicComponentModel?.children;
    this.event = DynamicComponentModel?.event;
  }
}
```

### 使用细节一：class 需要 constructor 赋值

- 被 `@ObservedV2` 装饰的类必须通过 constructor 显式初始化 `@Trace` 属性，否则无法触发深度监听。如示例中 DynamicStylesModel 通过 constructor 显式赋值
- 只有当数据被 new class(源数据)，进行赋值，此时数据才会被绑定监听

```ts
constructor(dynamicStylesModel?: DynamicStylesModel) {
  this.value = dynamicStylesModel?.value // 显式初始化@Trace属性
  this.width = dynamicStylesModel?.width
  ...
}
```

### 使用细节二：需要递归创建 class

- 嵌套类结构必须逐层实例化（如 DynamicComponentModel 中的 styles 属性）

```ts
// 递归对象, 每一层添加new DynamicComponentModel
transformToDynamicComponent(obj: dynamicComponent): DynamicComponentModel {
  // 递归处理 children
  if (obj.children && Array.isArray(obj.children)) {
    obj.children = obj.children.map((child: dynamicComponent) => this.transformToDynamicComponent(child));
  }
  return new DynamicComponentModel(obj);
}
```

### 使用细节三：父子组件使用均需要 new class 初始化

#### 父组件

进行数据初始化，无需任何装饰器修饰

```ts
@Entry
@Component
struct Index {
  dynamicData: DynamicComponentModel = new DynamicComponentModel()
  aboutToAppear(): void {
    // 函数递归数据，进行
    this.dynamicData = this.transformToDynamicComponent(this.dynamicDataOld)

    // 递归对象, 每一层添加new DynamicComponentModel
    transformToDynamicComponent(obj: dynamicComponent): DynamicComponentModel {
      // 递归处理 children
      if (obj.children && Array.isArray(obj.children)) {
        obj.children = obj.children.map((child: dynamicComponent) => this.transformToDynamicComponent(child));
      }
      return new DynamicComponentModel(obj);
    }
  }
}
```

#### 子组件

初始化，无需任何装饰器修饰

```ts
@Component
export struct dynamicComponent {
  dynamicData: DynamicComponentModel = new DynamicComponentModel()
  .....
}
```

## 结语

@ObservedV2 的推出标志着鸿蒙状态管理进入新时代。通过本文的技术解析和实践指导，开发者可以更好地利用这一利器构建高性能应用。

> 最新技术动态请关注：
>
> - [华为开发者联盟](https://developer.harmonyos.com)

## 源代码

### `index.ets` 父组件

```ts
import { dynamicStyles, DynamicComponentModel } from '../viewModels/dynamicModel'
import { cardComponent } from './cardComponent'

// 三方调用处
@Entry
@Component
struct Index {
  dynamicData: DynamicComponentModel = new DynamicComponentModel({
    type: 'Column',
    styles: {
      value: '我是column组件',
      backgroundColor: '#FF0000',
      width: '100%',
      height: '100'
    },
    children: [
      new DynamicComponentModel({
        type: 'Text',
        styles: {
          value: '我是text组件',
          width: 50,
          height: 50
        }
      }),
      new DynamicComponentModel({
        type: 'Button',
        styles: {
          value: '我是button组件',
        },
        event: (dynamicStyles: dynamicStyles) => {
          dynamicStyles.value = '我是button组件被点击了'
          console.log('dynamicComponent', 'this.dynamicStyles:', JSON.stringify(dynamicStyles))
        }
      })
    ]
  })

  aboutToAppear(): void {
  }

  build() {
    Column() {
      cardComponent({
        dynamicData: this.dynamicData,
        dynamicStyles: this.dynamicData.styles
      })
        .height('100%')
        .width('100%')
    }
    .height('100%')
    .width('100%')
  }
}
```

### `src/main/ets/pages/cardComponent.ets`

```ts
import { DynamicComponentModel, DynamicStylesModel } from "../viewModels/dynamicModel"
import { dynamicComponent } from "./DynamicComponent"

@Component
export struct cardComponent {
  dynamicData: DynamicComponentModel = new DynamicComponentModel({})
  dynamicStyles: DynamicStylesModel = new DynamicStylesModel({})

  build() {
    dynamicComponent({
      dynamicData: this.dynamicData,
      dynamicStyles: this.dynamicStyles
    })
      .height('100%')
      .width('100%')
  }
}
```

### `src/main/ets/pages/DynamicComponent.ets`

```ts
import { DynamicComponentModel, DynamicStylesModel } from "../viewModels/dynamicModel"

const Tag = 'dynamicComponent'

@Component
export struct dynamicComponent {
  dynamicData: DynamicComponentModel = new DynamicComponentModel({})
  dynamicStyles: DynamicStylesModel = new DynamicStylesModel({})

  aboutToAppear(): void {
    console.log(Tag, 'aboutToAppear')
    console.log(Tag, 'this.dynamicData:', JSON.stringify(this.dynamicData))
    console.log(Tag, 'this.dynamicStyles:', JSON.stringify(this.dynamicStyles))
  }

  @Builder
  buildChildren() {
    ForEach(this.dynamicData.children, (item: DynamicComponentModel, index) => {
      dynamicComponent({
        dynamicData: item as DynamicComponentModel,
        dynamicStyles: item.styles as DynamicStylesModel
      })
    })
  }

  @Builder
  buildColumn() {
    Column() {
      this.buildChildren()
    }
    .width(this.dynamicStyles.width)
    .height(this.dynamicStyles.height)
  }

  @Builder
  buildRow() {
    Row() {
      this.buildChildren()
    }
  }

  @Builder
  buildText() {
    Text(this.dynamicStyles.value + '')
  }

  @Builder
  buildImage() {
    // Image()
  }

  @Builder
  buildButton() {
    Button(this.dynamicStyles.value + '')
      .width(this.dynamicStyles.width)
      .height(this.dynamicStyles.height)
      .backgroundColor(this.dynamicStyles.backgroundColor)
      .onClick(() => {
        this.dynamicData.event?.(this.dynamicStyles)
      })
  }

  build() {
    if (this.dynamicData?.type == 'Column') {
      this.buildColumn()
    } else if (this.dynamicData?.type == 'Row') {
      this.buildRow()
    } else if (this.dynamicData?.type == 'Text') {
      this.buildText()
    } else if (this.dynamicData?.type == 'Image') {
      this.buildImage()
    } else if (this.dynamicData?.type === 'Button') {
      this.buildButton()
    }
  }
}
```

- `src/main/ets/viewModels/dynamicModel.ets`

```typescript
export interface dynamicStyles {
  value?: ESObject;
  width?: Length;
  height?: Length;
  backgroundColor?: string;
}

export interface dynamicComponent {
  type?: string;
  styles?: dynamicStyles;
  children?: Array<dynamicComponent>;
  event?: Function;
}

// 样式
@ObservedV2
export class DynamicStylesModel {
  @Trace value?: ESObject;
  @Trace width?: Length;
  @Trace height?: Length;
  @Trace backgroundColor?: string;

  constructor(dynamicStylesModel?: dynamicStyles) {
    this.value = dynamicStylesModel?.value;
    this.width = dynamicStylesModel?.width;
    this.height = dynamicStylesModel?.height;
    this.backgroundColor = dynamicStylesModel?.backgroundColor;
  }
}

// 第一层
export class DynamicComponentModel {
  type?: string;
  styles?: DynamicStylesModel;
  children?: DynamicComponentModel[] = [];
  event?: Function;

  constructor(DynamicComponentModel?: dynamicComponent) {
    this.type = DynamicComponentModel?.type;
    this.styles = new DynamicStylesModel(DynamicComponentModel?.styles);
    this.children = DynamicComponentModel?.children;
    this.event = DynamicComponentModel?.event;
  }
}
```
