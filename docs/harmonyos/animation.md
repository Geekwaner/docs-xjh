# 鸿蒙开发之动画

在鸿蒙开发中，动画是提升用户体验的重要元素。本文介绍鸿蒙应用中几种常用的动画实现方式。

## 目录

- [属性动画](#属性动画)
- [显式动画](#显式动画)
- [转场动画](#转场动画)
- [组件内置转场](#组件内置转场)

## 属性动画

属性动画是通过改变组件的属性值来实现动画效果。

```typescript
@Entry
@Component
struct AnimationDemo {
  @State widthSize: number = 250
  @State heightSize: number = 100
  @State opacitySize: number = 1
  @State rotateAngle: number = 0

  build() {
    Column({ space: 10 }) {
      Text('属性动画示例').fontSize(20).fontWeight(FontWeight.Bold)

      // 使用animation属性定义动画
      Image($r('app.media.icon'))
        .width(this.widthSize)
        .height(this.heightSize)
        .opacity(this.opacitySize)
        .rotate({ x: 0, y: 0, z: 1, angle: this.rotateAngle })
        .animation({
          duration: 1000, // 动画持续时间
          curve: Curve.EaseInOut, // 动画曲线
          delay: 100, // 动画延迟
          iterations: 1, // 动画重复次数，-1为无限重复
          playMode: PlayMode.Normal // 动画播放模式
        })

      // 控制按钮
      Row({ space: 20 }) {
        Button('放大')
          .onClick(() => {
            this.widthSize = 350
            this.heightSize = 200
          })
        Button('缩小')
          .onClick(() => {
            this.widthSize = 150
            this.heightSize = 50
          })
      }

      Row({ space: 20 }) {
        Button('旋转')
          .onClick(() => {
            this.rotateAngle = this.rotateAngle + 90
          })
        Button('重置')
          .onClick(() => {
            this.widthSize = 250
            this.heightSize = 100
            this.opacitySize = 1
            this.rotateAngle = 0
          })
      }
    }
    .width('100%')
    .padding(20)
  }
}
```

## 显式动画

显式动画是指通过`animateTo`函数明确指定动画过程的方式。

```typescript
@Entry
@Component
struct ExplicitAnimationDemo {
  @State positionX: number = 0
  @State color: Color = Color.Blue
  @State scale: number = 1

  build() {
    Column({ space: 20 }) {
      Text('显式动画示例').fontSize(20).fontWeight(FontWeight.Bold)

      Row() {
        // 用于动画展示的元素
        Circle({ width: 50, height: 50 })
          .fill(this.color)
          .margin({ left: this.positionX })
          .scale({ x: this.scale, y: this.scale })
      }
      .width('100%')
      .height(100)

      // 控制按钮
      Button('开始动画')
        .onClick(() => {
          // 使用animateTo函数创建显式动画
          animateTo({
            duration: 1000, // 动画时长
            curve: Curve.Ease, // 动画曲线
            delay: 100, // 延迟时间
            onFinish: () => { // 动画完成回调
              console.info('Animation finished')
            }
          }, () => {
            // 状态改变，这些改变会以动画形式呈现
            this.positionX = this.positionX === 0 ? 300 : 0
            this.color = this.color === Color.Blue ? Color.Red : Color.Blue
            this.scale = this.scale === 1 ? 1.5 : 1
          })
        })
    }
    .width('100%')
    .padding(20)
  }
}
```

## 转场动画

转场动画用于页面切换或组件出现/消失时的动画效果。

```typescript
@Entry
@Component
struct TransitionDemo {
  @State isShow: boolean = true

  build() {
    Column({ space: 20 }) {
      Text('转场动画示例').fontSize(20).fontWeight(FontWeight.Bold)

      // 转场动画控制
      Button(this.isShow ? '隐藏元素' : '显示元素')
        .onClick(() => {
          this.isShow = !this.isShow
        })

      if (this.isShow) {
        // 使用transition定义转场动画
        Image($r('app.media.icon'))
          .width(200)
          .height(200)
          .transition({
            type: TransitionType.Insert, // 插入类型转场
            opacity: 0, // 初始透明度
            scale: { x: 0.2, y: 0.2 }, // 初始缩放比例
            translate: { x: 100, y: 100 } // 初始位置偏移
          })
          .transition({
            type: TransitionType.Delete, // 删除类型转场
            opacity: 0, // 最终透明度
            scale: { x: 0.2, y: 0.2 }, // 最终缩放比例
            translate: { x: 100, y: 100 } // 最终位置偏移
          })
      }
    }
    .width('100%')
    .padding(20)
  }
}
```

## 组件内置转场

某些组件有内置的转场能力，如 List、Grid 等。

```typescript
@Entry
@Component
struct ComponentTransitionDemo {
  @State dataList: number[] = [1, 2, 3, 4, 5]

  build() {
    Column({ space: 20 }) {
      Text('组件内置转场示例').fontSize(20).fontWeight(FontWeight.Bold)

      // 添加/删除项目的按钮
      Row({ space: 20 }) {
        Button('添加项目')
          .onClick(() => {
            this.dataList.push(this.dataList.length + 1)
          })
        Button('删除项目')
          .onClick(() => {
            if (this.dataList.length > 0) {
              this.dataList.pop()
            }
          })
      }

      // 使用内置转场的List组件
      List({ space: 10 }) {
        ForEach(this.dataList, (item) => {
          ListItem() {
            Text(`项目 ${item}`)
              .width('100%')
              .height(60)
              .backgroundColor(Color.Blue)
              .textAlign(TextAlign.Center)
              .borderRadius(10)
          }
          .transition({ // ListItem的转场动画
            type: TransitionType.All, // 所有类型的转场
            opacity: 0.2, // 初始/最终透明度
            scale: { x: 0.5, y: 0.5 }, // 初始/最终缩放比例
            translate: { x: 100, y: 0 } // 初始/最终位置偏移
          })
        })
      }
      .width('100%')
      .height(300)
      .divider({ strokeWidth: 1, color: Color.Gray })
      .listDirection(Axis.Vertical)
    }
    .width('100%')
    .padding(20)
  }
}
```

## 综合实例：自定义页面转场

```typescript
import router from '@ohos.router'

@Entry
@Component
struct CustomPageTransition {
  @State scale: number = 0.5
  @State opacity: number = 0

  aboutToAppear() {
    // 页面即将出现时，启动动画
    setTimeout(() => {
      animateTo({
        duration: 1000,
        curve: Curve.EaseInOut,
      }, () => {
        this.scale = 1
        this.opacity = 1
      })
    }, 100)
  }

  build() {
    Column({ space: 20 }) {
      Text('自定义页面转场效果')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
        .opacity(this.opacity)
        .scale({ x: this.scale, y: this.scale })

      Image($r('app.media.icon'))
        .width(200)
        .height(200)
        .opacity(this.opacity)
        .scale({ x: this.scale, y: this.scale })

      Button('返回')
        .opacity(this.opacity)
        .scale({ x: this.scale, y: this.scale })
        .onClick(() => {
          // 退出时的动画
          animateTo({
            duration: 800,
            curve: Curve.EaseIn,
            onFinish: () => {
              router.back()
            }
          }, () => {
            this.opacity = 0
            this.scale = 0.5
          })
        })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
  }
}
```
