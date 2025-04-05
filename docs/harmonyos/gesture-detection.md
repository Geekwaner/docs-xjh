# 鸿蒙开发之手势检测

手势是用户与应用交互的重要方式，在鸿蒙应用开发中，ArkUI 框架提供了丰富的手势识别和处理机制。本文将介绍如何在应用中实现各种手势检测和交互效果。

## 目录

- [基础手势](#基础手势)
- [组合手势](#组合手势)
- [自定义手势](#自定义手势)
- [实际应用场景](#实际应用场景)

## 基础手势

鸿蒙 ArkUI 框架内置了多种基础手势识别器，可以直接使用：

### 点击手势（TapGesture）

```typescript
@Entry
@Component
struct TapGestureDemo {
  @State tapCount: number = 0
  @State tapInfo: string = ''

  build() {
    Column({ space: 20 }) {
      Text('点击手势示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Text(`点击次数: ${this.tapCount}`)
        .fontSize(18)

      Text(this.tapInfo)
        .fontSize(16)
        .fontColor('#666')

      // 单击手势
      Text('点击此区域')
        .width('80%')
        .height(100)
        .fontSize(20)
        .textAlign(TextAlign.Center)
        .borderRadius(10)
        .backgroundColor('#f0f0f0')
        .gesture(
          TapGesture()
            .onAction((event: GestureEvent) => {
              this.tapCount++
              this.tapInfo = `单击坐标: (${event.x.toFixed(2)}, ${event.y.toFixed(2)})`
            })
        )
        .margin({ top: 20 })

      // 多击手势
      Text('双击此区域')
        .width('80%')
        .height(100)
        .fontSize(20)
        .textAlign(TextAlign.Center)
        .borderRadius(10)
        .backgroundColor('#e6f7ff')
        .gesture(
          TapGesture({ count: 2 })
            .onAction(() => {
              this.tapCount += 2
              this.tapInfo = '检测到双击'
            })
        )
        .margin({ top: 20 })

      Button('重置计数')
        .onClick(() => {
          this.tapCount = 0
          this.tapInfo = ''
        })
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

### 长按手势（LongPressGesture）

```typescript
@Entry
@Component
struct LongPressGestureDemo {
  @State pressInfo: string = '请长按下方区域'
  @State pressTime: number = 0
  @State pressColor: string = '#f0f0f0'

  build() {
    Column({ space: 20 }) {
      Text('长按手势示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Text(this.pressInfo)
        .fontSize(18)

      Text(`长按时间: ${this.pressTime}ms`)
        .fontSize(16)
        .fontColor('#666')

      // 长按手势
      Text('长按此区域')
        .width('80%')
        .height(100)
        .fontSize(20)
        .textAlign(TextAlign.Center)
        .borderRadius(10)
        .backgroundColor(this.pressColor)
        .gesture(
          LongPressGesture({ repeat: true, duration: 1000 })
            .onAction((event: GestureEvent) => {
              this.pressInfo = '检测到长按'
              this.pressTime = event.timeStamp
              this.pressColor = '#ffd591'
            })
            .onActionEnd(() => {
              this.pressInfo = '长按结束'
              this.pressColor = '#f0f0f0'
            })
        )
        .margin({ top: 20 })

      Button('重置')
        .onClick(() => {
          this.pressInfo = '请长按下方区域'
          this.pressTime = 0
          this.pressColor = '#f0f0f0'
        })
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

### 平移手势（PanGesture）

```typescript
@Entry
@Component
struct PanGestureDemo {
  @State boxPosition: { x: number, y: number } = { x: 100, y: 100 }
  @State panInfo: string = '拖动下方方块'

  build() {
    Column({ space: 20 }) {
      Text('平移手势示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Text(this.panInfo)
        .fontSize(18)

      // 创建一个容器作为移动区域
      Stack({ alignContent: Alignment.TopStart }) {
        // 可拖动的方块
        Rectangle()
          .width(100)
          .height(100)
          .fill('#3478f6')
          .borderRadius(10)
          .position({ x: this.boxPosition.x, y: this.boxPosition.y })
          .gesture(
            PanGesture()
              .onActionStart((event: GestureEvent) => {
                this.panInfo = '开始拖动'
              })
              .onActionUpdate((event: GestureEvent) => {
                // 更新方块位置
                this.boxPosition.x += event.offsetX
                this.boxPosition.y += event.offsetY

                // 确保不超出边界
                const containerWidth = 300
                const containerHeight = 400
                const boxSize = 100

                this.boxPosition.x = Math.max(0, Math.min(this.boxPosition.x, containerWidth - boxSize))
                this.boxPosition.y = Math.max(0, Math.min(this.boxPosition.y, containerHeight - boxSize))

                this.panInfo = `位置: (${this.boxPosition.x.toFixed(0)}, ${this.boxPosition.y.toFixed(0)})`
              })
              .onActionEnd(() => {
                this.panInfo = '拖动结束'
              })
          )
      }
      .width(300)
      .height(400)
      .backgroundColor('#f5f5f5')
      .borderRadius(20)
      .margin({ top: 20 })

      Button('重置位置')
        .onClick(() => {
          this.boxPosition = { x: 100, y: 100 }
          this.panInfo = '拖动下方方块'
        })
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

### 滑动手势（SwipeGesture）

```typescript
@Entry
@Component
struct SwipeGestureDemo {
  @State currentIndex: number = 0
  @State swipeInfo: string = '向左或向右滑动'
  @State opacity: number = 1.0

  private colors: string[] = ['#f5222d', '#fa8c16', '#fadb14', '#52c41a', '#1890ff', '#722ed1']
  private colorNames: string[] = ['红色', '橙色', '黄色', '绿色', '蓝色', '紫色']

  build() {
    Column({ space: 20 }) {
      Text('滑动手势示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Text(`当前: ${this.colorNames[this.currentIndex]}`)
        .fontSize(18)

      Text(this.swipeInfo)
        .fontSize(16)
        .fontColor('#666')

      // 滑动区域
      Column() {
        Rectangle()
          .width('100%')
          .height('100%')
          .fill(this.colors[this.currentIndex])
          .opacity(this.opacity)
      }
      .width('80%')
      .height(200)
      .borderRadius(20)
      .gesture(
        SwipeGesture({ direction: SwipeDirection.Horizontal })
          .onAction((event: SwipeGestureEvent) => {
            if (event.direction === SwipeDirection.Right) {
              // 向右滑动，显示前一个颜色
              this.currentIndex = (this.currentIndex - 1 + this.colors.length) % this.colors.length
              this.swipeInfo = '向右滑动'
            } else if (event.direction === SwipeDirection.Left) {
              // 向左滑动，显示后一个颜色
              this.currentIndex = (this.currentIndex + 1) % this.colors.length
              this.swipeInfo = '向左滑动'
            }

            // 创建滑动动画效果
            this.opacity = 0.5
            animateTo({ duration: 300, curve: Curve.EaseOut }, () => {
              this.opacity = 1.0
            })
          })
      )
      .margin({ top: 20 })

      Row({ space: 10 }) {
        ForEach(this.colors, (color, index) => {
          Circle()
            .width(20)
            .height(20)
            .fill(color)
            .opacity(this.currentIndex === index ? 1.0 : 0.5)
        })
      }
      .margin({ top: 20 })
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

### 捏合手势（PinchGesture）

```typescript
@Entry
@Component
struct PinchGestureDemo {
  @State scale: number = 1.0
  @State pinchInfo: string = '捏合或张开以缩放图像'

  build() {
    Column({ space: 20 }) {
      Text('捏合手势示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Text(`缩放比例: ${this.scale.toFixed(2)}x`)
        .fontSize(18)

      Text(this.pinchInfo)
        .fontSize(16)
        .fontColor('#666')

      // 捏合缩放区域
      Column() {
        Image($r('app.media.sample_image'))
          .width(300 * this.scale)
          .height(200 * this.scale)
          .objectFit(ImageFit.Contain)
      }
      .width('100%')
      .height(300)
      .justifyContent(FlexAlign.Center)
      .gesture(
        PinchGesture()
          .onActionStart(() => {
            this.pinchInfo = '开始缩放'
          })
          .onActionUpdate((event: GestureEvent) => {
            // 更新缩放比例
            this.scale *= event.scale
            // 限制缩放范围
            this.scale = Math.max(0.5, Math.min(this.scale, 3.0))
            this.pinchInfo = `缩放因子: ${event.scale.toFixed(2)}`
          })
          .onActionEnd(() => {
            this.pinchInfo = '缩放结束'
          })
      )
      .margin({ top: 20 })

      Button('重置缩放')
        .onClick(() => {
          this.scale = 1.0
          this.pinchInfo = '捏合或张开以缩放图像'
        })
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

### 旋转手势（RotationGesture）

```typescript
@Entry
@Component
struct RotationGestureDemo {
  @State angle: number = 0
  @State rotationInfo: string = '使用两指旋转图像'

  build() {
    Column({ space: 20 }) {
      Text('旋转手势示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Text(`旋转角度: ${this.angle.toFixed(2)}°`)
        .fontSize(18)

      Text(this.rotationInfo)
        .fontSize(16)
        .fontColor('#666')

      // 旋转区域
      Column() {
        Image($r('app.media.sample_image'))
          .width(200)
          .height(200)
          .objectFit(ImageFit.Contain)
          .rotate({ angle: this.angle })
      }
      .width('100%')
      .height(300)
      .justifyContent(FlexAlign.Center)
      .gesture(
        RotationGesture()
          .onActionStart(() => {
            this.rotationInfo = '开始旋转'
          })
          .onActionUpdate((event: GestureEvent) => {
            // 更新旋转角度
            this.angle += event.angle
            this.rotationInfo = `旋转增量: ${event.angle.toFixed(2)}°`
          })
          .onActionEnd(() => {
            this.rotationInfo = '旋转结束'
          })
      )
      .margin({ top: 20 })

      Button('重置旋转')
        .onClick(() => {
          this.angle = 0
          this.rotationInfo = '使用两指旋转图像'
        })
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

## 组合手势

在实际应用中，我们经常需要组合多种手势来实现更复杂的交互效果。鸿蒙 ArkUI 框架支持将多种手势组合在一起，通过 GestureGroup 实现：

```typescript
@Entry
@Component
struct CombinedGestureDemo {
  @State scale: number = 1.0
  @State angle: number = 0
  @State position: { x: number, y: number } = { x: 0, y: 0 }
  @State gestureInfo: string = '可拖动、缩放和旋转下方图像'

  build() {
    Column({ space: 20 }) {
      Text('组合手势示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Text(this.gestureInfo)
        .fontSize(16)
        .width('80%')
        .textAlign(TextAlign.Center)

      // 组合手势区域
      Stack({ alignContent: Alignment.Center }) {
        Image($r('app.media.sample_image'))
          .width(200 * this.scale)
          .height(200 * this.scale)
          .rotate({ angle: this.angle })
          .translate({ x: this.position.x, y: this.position.y })
          .gesture(
            GestureGroup(GestureMode.Parallel)
              .gesture(
                PanGesture()
                  .onActionUpdate((event: GestureEvent) => {
                    this.position.x += event.offsetX
                    this.position.y += event.offsetY
                    this.gestureInfo = `位置: (${this.position.x.toFixed(0)}, ${this.position.y.toFixed(0)})`
                  })
              )
              .gesture(
                PinchGesture()
                  .onActionUpdate((event: GestureEvent) => {
                    this.scale *= event.scale
                    this.scale = Math.max(0.5, Math.min(this.scale, 3.0))
                    this.gestureInfo = `缩放: ${this.scale.toFixed(2)}x`
                  })
              )
              .gesture(
                RotationGesture()
                  .onActionUpdate((event: GestureEvent) => {
                    this.angle += event.angle
                    this.gestureInfo = `旋转: ${this.angle.toFixed(2)}°`
                  })
              )
          )
      }
      .width('100%')
      .height(400)
      .backgroundColor('#f5f5f5')
      .borderRadius(20)
      .margin({ top: 20 })

      Button('重置')
        .onClick(() => {
          this.scale = 1.0
          this.angle = 0
          this.position = { x: 0, y: 0 }
          this.gestureInfo = '可拖动、缩放和旋转下方图像'
        })
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

## 自定义手势

除了内置的手势识别器外，鸿蒙 ArkUI 还允许开发者自定义手势识别逻辑，基于触摸事件实现更加灵活的手势交互：

```typescript
@Entry
@Component
struct CustomGestureDemo {
  @State points: Array<{ x: number, y: number }> = []
  @State strokePath: string = ''
  @State gestureInfo: string = '在下方区域绘制'

  // 构建SVG路径
  buildPath(points: Array<{ x: number, y: number }>): string {
    if (points.length === 0) return ''

    let path = `M ${points[0].x} ${points[0].y}`
    for (let i = 1; i < points.length; i++) {
      path += ` L ${points[i].x} ${points[i].y}`
    }

    return path
  }

  build() {
    Column({ space: 20 }) {
      Text('自定义手势示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Text(this.gestureInfo)
        .fontSize(16)
        .fontColor('#666')

      // 绘制区域
      Stack({ alignContent: Alignment.TopStart }) {
        // 绘制线条
        Shape() {
          Path()
            .stroke('#3478f6')
            .strokeWidth(5)
            .antiAlias(true)
            .commands(this.strokePath)
        }
        .width('100%')
        .height('100%')

        // 绘制点
        ForEach(this.points, (point) => {
          Circle()
            .width(10)
            .height(10)
            .fill('#3478f6')
            .position({ x: point.x - 5, y: point.y - 5 })
        })
      }
      .width('90%')
      .height(300)
      .backgroundColor('#f5f5f5')
      .borderRadius(20)
      // 使用触摸事件实现自定义手势
      .onTouch((event: TouchEvent) => {
        switch (event.type) {
          case TouchType.Down:
            // 开始新的绘制
            this.points = [{ x: event.touches[0].x, y: event.touches[0].y }]
            this.strokePath = this.buildPath(this.points)
            this.gestureInfo = '开始绘制'
            break

          case TouchType.Move:
            // 添加新的点
            this.points.push({ x: event.touches[0].x, y: event.touches[0].y })
            this.strokePath = this.buildPath(this.points)
            this.gestureInfo = `点数: ${this.points.length}`
            break

          case TouchType.Up:
            // 完成绘制
            this.gestureInfo = '绘制完成'
            break
        }
      })
      .margin({ top: 20 })

      Button('清除绘制')
        .onClick(() => {
          this.points = []
          this.strokePath = ''
          this.gestureInfo = '在下方区域绘制'
        })
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

## 实际应用场景

下面是一个结合多种手势实现的图片查看器示例，包含缩放、旋转、拖动和双击还原等交互功能：

```typescript
@Entry
@Component
struct ImageViewerDemo {
  @State scale: number = 1.0
  @State angle: number = 0
  @State position: { x: number, y: number } = { x: 0, y: 0 }
  @State statusInfo: string = '使用手势操作图片'

  // 图片集
  private images: ResourceStr[] = [
    $r('app.media.image1'),
    $r('app.media.image2'),
    $r('app.media.image3'),
    $r('app.media.image4')
  ]
  @State currentImageIndex: number = 0

  // 还原图片状态
  resetImageState() {
    animateTo({ duration: 300, curve: Curve.EaseOut }, () => {
      this.scale = 1.0
      this.angle = 0
      this.position = { x: 0, y: 0 }
    })
    this.statusInfo = '图片已还原'
  }

  // 切换图片
  switchImage(direction: number) {
    this.currentImageIndex = (this.currentImageIndex + direction + this.images.length) % this.images.length
    this.resetImageState()
    this.statusInfo = `当前图片: ${this.currentImageIndex + 1}/${this.images.length}`
  }

  build() {
    Column({ space: 20 }) {
      Text('图片查看器')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Text(this.statusInfo)
        .fontSize(16)
        .fontColor('#666')
        .textAlign(TextAlign.Center)

      // 图片查看区域
      Stack({ alignContent: Alignment.Center }) {
        Image(this.images[this.currentImageIndex])
          .width(280 * this.scale)
          .height(280 * this.scale)
          .objectFit(ImageFit.Contain)
          .rotate({ angle: this.angle })
          .translate({ x: this.position.x, y: this.position.y })
          .gesture(
            GestureGroup(GestureMode.Parallel)
              // 拖动手势
              .gesture(
                PanGesture()
                  .onActionUpdate((event: GestureEvent) => {
                    this.position.x += event.offsetX
                    this.position.y += event.offsetY
                    this.statusInfo = '拖动图片中'
                  })
              )
              // 捏合缩放
              .gesture(
                PinchGesture()
                  .onActionUpdate((event: GestureEvent) => {
                    this.scale *= event.scale
                    this.scale = Math.max(0.5, Math.min(this.scale, 5.0))
                    this.statusInfo = `缩放比例: ${this.scale.toFixed(2)}x`
                  })
              )
              // 旋转手势
              .gesture(
                RotationGesture()
                  .onActionUpdate((event: GestureEvent) => {
                    this.angle += event.angle
                    this.statusInfo = `旋转角度: ${this.angle.toFixed(2)}°`
                  })
              )
          )
          // 双击还原
          .gesture(
            TapGesture({ count: 2 })
              .onAction(() => {
                this.resetImageState()
              })
          )
      }
      .width('100%')
      .height(400)
      .backgroundColor('#f5f5f5')
      .borderRadius(20)
      // 左右滑动切换图片
      .gesture(
        SwipeGesture({ direction: SwipeDirection.Horizontal })
          .onAction((event: SwipeGestureEvent) => {
            if (event.direction === SwipeDirection.Left) {
              this.switchImage(1) // 下一张
            } else if (event.direction === SwipeDirection.Right) {
              this.switchImage(-1) // 上一张
            }
          })
      )
      .margin({ top: 20 })

      // 底部控制栏
      Row({ space: 20 }) {
        Button('上一张')
          .onClick(() => this.switchImage(-1))

        Button('还原')
          .onClick(() => this.resetImageState())

        Button('下一张')
          .onClick(() => this.switchImage(1))
      }
      .margin({ top: 20 })

      // 图片指示器
      Row({ space: 10 }) {
        ForEach(this.images, (_, index) => {
          Circle()
            .width(10)
            .height(10)
            .fill(this.currentImageIndex === index ? '#3478f6' : '#d9d9d9')
        })
      }
      .margin({ top: 10 })
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

通过本文的示例，我们展示了鸿蒙 ArkUI 中各种手势的使用方法和应用场景。熟练掌握这些手势交互技巧，可以为用户提供更加自然、流畅的应用体验。在实际开发中，我们应该根据产品需求选择合适的手势类型，并合理组合使用，同时注意考虑无障碍设计，为不同用户提供更加包容的交互方式。
