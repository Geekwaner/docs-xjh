# 鸿蒙开发之 Canvas 绘图

在鸿蒙应用开发中，Canvas（画布）组件是实现自定义绘图和图形处理的重要工具。通过 Canvas，开发者可以绘制各种图形、文本和图像，实现复杂的自定义 UI 效果。本文将介绍如何在鸿蒙应用中使用 Canvas 组件进行绘图。

## 目录

- [Canvas 基础](#Canvas基础)
- [绘制基本图形](#绘制基本图形)
- [绘制文本](#绘制文本)
- [绘制图像](#绘制图像)
- [绘制路径](#绘制路径)
- [实际应用场景](#实际应用场景)

## Canvas 基础

首先，我们来了解 Canvas 的基本使用方法：

```typescript
@Entry
@Component
struct CanvasBasicsDemo {
  private settings: RenderingContextSettings = new RenderingContextSettings(true)
  private context: CanvasRenderingContext2D = new CanvasRenderingContext2D(this.settings)

  build() {
    Flex({ direction: FlexDirection.Column, alignItems: ItemAlign.Center, justifyContent: FlexAlign.Center }) {
      Canvas(this.context)
        .width('100%')
        .height(300)
        .backgroundColor('#F5F5F5')
        .onReady(() => {
          // 清空画布
          this.context.clearRect(0, 0, 1000, 1000)

          // 设置填充颜色
          this.context.fillStyle = '#007DFF'

          // 绘制一个矩形
          this.context.fillRect(50, 50, 150, 100)

          // 设置描边颜色和宽度
          this.context.strokeStyle = '#FF0000'
          this.context.lineWidth = 5

          // 绘制一个描边矩形
          this.context.strokeRect(250, 50, 150, 100)
        })

      Text('Canvas 基础示例')
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
        .margin({top: 20})
    }
    .width('100%')
    .height('100%')
  }
}
```

## 绘制基本图形

Canvas 支持绘制各种基本图形，如矩形、圆形、线条等：

```typescript
@Entry
@Component
struct BasicShapesDemo {
  private settings: RenderingContextSettings = new RenderingContextSettings(true)
  private context: CanvasRenderingContext2D = new CanvasRenderingContext2D(this.settings)

  build() {
    Column({ space: 20 }) {
      Text('基本图形绘制')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Canvas(this.context)
        .width('100%')
        .height(500)
        .backgroundColor('#F5F5F5')
        .onReady(() => {
          // 清空画布
          this.context.clearRect(0, 0, 1000, 1000)

          // 1. 绘制矩形
          this.context.fillStyle = '#1890FF'
          this.context.fillRect(50, 50, 100, 80)

          // 2. 绘制描边矩形
          this.context.strokeStyle = '#FF4D4F'
          this.context.lineWidth = 3
          this.context.strokeRect(180, 50, 100, 80)

          // 3. 绘制圆形
          this.context.fillStyle = '#52C41A'
          this.context.beginPath()
          this.context.arc(100, 200, 40, 0, 2 * Math.PI)
          this.context.fill()

          // 4. 绘制描边圆形
          this.context.strokeStyle = '#FAAD14'
          this.context.lineWidth = 3
          this.context.beginPath()
          this.context.arc(230, 200, 40, 0, 2 * Math.PI)
          this.context.stroke()

          // 5. 绘制线条
          this.context.strokeStyle = '#722ED1'
          this.context.lineWidth = 5
          this.context.beginPath()
          this.context.moveTo(50, 300)
          this.context.lineTo(150, 350)
          this.context.lineTo(250, 300)
          this.context.stroke()

          // 6. 绘制三角形
          this.context.fillStyle = '#EB2F96'
          this.context.beginPath()
          this.context.moveTo(50, 400)
          this.context.lineTo(150, 450)
          this.context.lineTo(250, 400)
          this.context.closePath()
          this.context.fill()
        })

      Text('说明：展示了矩形、圆形、线条和三角形的绘制')
        .fontSize(14)
        .fontColor('#666')
        .width('90%')
        .textAlign(TextAlign.Center)
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

## 绘制文本

Canvas 还支持文本绘制，可以设置字体、对齐方式等属性：

```typescript
@Entry
@Component
struct TextRenderingDemo {
  private settings: RenderingContextSettings = new RenderingContextSettings(true)
  private context: CanvasRenderingContext2D = new CanvasRenderingContext2D(this.settings)

  build() {
    Column({ space: 20 }) {
      Text('文本绘制')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Canvas(this.context)
        .width('100%')
        .height(400)
        .backgroundColor('#F5F5F5')
        .onReady(() => {
          // 清空画布
          this.context.clearRect(0, 0, 1000, 1000)

          // 1. 基本文本
          this.context.fillStyle = '#333333'
          this.context.font = '20px sans-serif'
          this.context.fillText('基本文本绘制', 50, 50)

          // 2. 设置字体大小和样式
          this.context.fillStyle = '#1890FF'
          this.context.font = 'bold 24px Arial'
          this.context.fillText('粗体文本', 50, 100)

          // 3. 斜体文本
          this.context.fillStyle = '#52C41A'
          this.context.font = 'italic 24px Georgia'
          this.context.fillText('斜体文本', 50, 150)

          // 4. 描边文本
          this.context.strokeStyle = '#FF4D4F'
          this.context.lineWidth = 1
          this.context.font = '24px sans-serif'
          this.context.strokeText('描边文本', 50, 200)

          // 5. 文本对齐
          this.context.fillStyle = '#333333'
          this.context.font = '20px sans-serif'

          // 设置文本水平对齐方式
          this.context.textAlign = 'start'
          this.context.fillText('左对齐', 200, 250)

          this.context.textAlign = 'center'
          this.context.fillText('居中对齐', 200, 280)

          this.context.textAlign = 'end'
          this.context.fillText('右对齐', 200, 310)

          // 6. 文本基线
          this.context.textAlign = 'start'
          this.context.fillStyle = '#FAAD14'

          // 绘制参考线
          this.context.strokeStyle = '#CCCCCC'
          this.context.beginPath()
          this.context.moveTo(50, 350)
          this.context.lineTo(300, 350)
          this.context.stroke()

          this.context.textBaseline = 'top'
          this.context.fillText('Top', 50, 350)

          this.context.textBaseline = 'middle'
          this.context.fillText('Middle', 100, 350)

          this.context.textBaseline = 'bottom'
          this.context.fillText('Bottom', 200, 350)
        })

      Text('说明：展示了不同样式、大小、对齐方式和基线的文本绘制')
        .fontSize(14)
        .fontColor('#666')
        .width('90%')
        .textAlign(TextAlign.Center)
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

## 绘制图像

Canvas 可以绘制图像，支持裁剪、缩放等操作：

```typescript
@Entry
@Component
struct ImageRenderingDemo {
  private settings: RenderingContextSettings = new RenderingContextSettings(true)
  private context: CanvasRenderingContext2D = new CanvasRenderingContext2D(this.settings)
  @State imageLoaded: boolean = false

  // 图像对象
  private logoImage = new Image()

  aboutToAppear() {
    // 加载图像
    this.logoImage.src = $r('app.media.logo')
    this.logoImage.onload = () => {
      this.imageLoaded = true
    }
  }

  build() {
    Column({ space: 20 }) {
      Text('图像绘制')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      if (!this.imageLoaded) {
        Text('图像加载中...')
          .fontSize(16)
      }

      Canvas(this.context)
        .width('100%')
        .height(500)
        .backgroundColor('#F5F5F5')
        .onReady(() => {
          // 清空画布
          this.context.clearRect(0, 0, 1000, 1000)

          if (!this.imageLoaded) return

          // 1. 基本图像绘制
          this.context.drawImage(this.logoImage, 50, 50, 150, 150)

          // 2. 裁剪部分图像绘制
          // 参数：原图像, 裁剪的x坐标, 裁剪的y坐标, 裁剪的宽度, 裁剪的高度, 目标x坐标, 目标y坐标, 目标宽度, 目标高度
          this.context.drawImage(
            this.logoImage,
            0, 0, this.logoImage.width / 2, this.logoImage.height / 2,
            230, 50, 100, 100
          )

          // 3. 添加特效 - 绘制带边框的图像
          this.context.drawImage(this.logoImage, 50, 230, 150, 150)
          this.context.strokeStyle = '#1890FF'
          this.context.lineWidth = 5
          this.context.strokeRect(50, 230, 150, 150)

          // 4. 添加特效 - 圆形裁剪
          this.context.save() // 保存当前状态

          this.context.beginPath()
          this.context.arc(280, 300, 75, 0, Math.PI * 2)
          this.context.clip() // 设置裁剪区域

          this.context.drawImage(this.logoImage, 205, 225, 150, 150)

          // 添加边框
          this.context.strokeStyle = '#52C41A'
          this.context.lineWidth = 5
          this.context.stroke()

          this.context.restore() // 恢复之前的状态

          // 5. 添加阴影效果
          this.context.save()

          this.context.shadowColor = 'rgba(0, 0, 0, 0.5)'
          this.context.shadowBlur = 15
          this.context.shadowOffsetX = 5
          this.context.shadowOffsetY = 5

          this.context.drawImage(this.logoImage, 50, 400, 100, 100)

          this.context.restore()
        })

      Text('说明：展示了基本图像绘制、裁剪、边框、圆形裁剪和阴影效果')
        .fontSize(14)
        .fontColor('#666')
        .width('90%')
        .textAlign(TextAlign.Center)
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

## 绘制路径

使用路径可以绘制更复杂的图形：

```typescript
@Entry
@Component
struct PathDrawingDemo {
  private settings: RenderingContextSettings = new RenderingContextSettings(true)
  private context: CanvasRenderingContext2D = new CanvasRenderingContext2D(this.settings)

  build() {
    Column({ space: 20 }) {
      Text('路径绘制')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Canvas(this.context)
        .width('100%')
        .height(500)
        .backgroundColor('#F5F5F5')
        .onReady(() => {
          // 清空画布
          this.context.clearRect(0, 0, 1000, 1000)

          // 1. 基本路径 - 多边形
          this.context.beginPath()
          this.context.moveTo(50, 50)
          this.context.lineTo(150, 50)
          this.context.lineTo(200, 100)
          this.context.lineTo(150, 150)
          this.context.lineTo(50, 150)
          this.context.lineTo(0, 100)
          this.context.closePath()

          this.context.fillStyle = '#1890FF'
          this.context.fill()

          this.context.strokeStyle = '#333333'
          this.context.lineWidth = 2
          this.context.stroke()

          // 2. 二次贝塞尔曲线
          this.context.beginPath()
          this.context.moveTo(250, 50)
          this.context.quadraticCurveTo(300, 0, 350, 50)
          this.context.quadraticCurveTo(400, 100, 350, 150)
          this.context.quadraticCurveTo(300, 200, 250, 150)
          this.context.quadraticCurveTo(200, 100, 250, 50)

          this.context.fillStyle = '#52C41A'
          this.context.fill()

          // 3. 绘制圆弧路径
          this.context.beginPath()
          this.context.moveTo(50, 200)
          this.context.arcTo(50, 300, 150, 300, 50)
          this.context.arcTo(150, 300, 150, 200, 50)
          this.context.arcTo(150, 200, 50, 200, 50)
          this.context.arcTo(50, 200, 50, 300, 50)

          this.context.fillStyle = '#FAAD14'
          this.context.fill()

          // 4. 三次贝塞尔曲线
          this.context.beginPath()
          this.context.moveTo(200, 250)
          this.context.bezierCurveTo(250, 200, 300, 300, 350, 250)

          this.context.strokeStyle = '#FF4D4F'
          this.context.lineWidth = 3
          this.context.stroke()

          // 5. 绘制心形
          this.context.beginPath()

          // 绘制心形的左半部分
          this.context.moveTo(100, 350)
          this.context.bezierCurveTo(100, 320, 50, 320, 50, 350)
          this.context.bezierCurveTo(50, 380, 100, 400, 100, 430)

          // 绘制心形的右半部分
          this.context.bezierCurveTo(100, 400, 150, 380, 150, 350)
          this.context.bezierCurveTo(150, 320, 100, 320, 100, 350)

          this.context.fillStyle = '#FF4D4F'
          this.context.fill()

          // 6. 绘制星形
          this.context.beginPath()
          const centerX = 250
          const centerY = 390
          const spikes = 5
          const outerRadius = 50
          const innerRadius = 25

          let rot = Math.PI / 2 * 3
          let x = centerX
          let y = centerY
          let step = Math.PI / spikes

          this.context.moveTo(centerX, centerY - outerRadius)

          for (let i = 0; i < spikes; i++) {
            x = centerX + Math.cos(rot) * outerRadius
            y = centerY + Math.sin(rot) * outerRadius
            this.context.lineTo(x, y)
            rot += step

            x = centerX + Math.cos(rot) * innerRadius
            y = centerY + Math.sin(rot) * innerRadius
            this.context.lineTo(x, y)
            rot += step
          }

          this.context.lineTo(centerX, centerY - outerRadius)
          this.context.closePath()

          this.context.fillStyle = '#722ED1'
          this.context.fill()

          this.context.strokeStyle = '#333333'
          this.context.lineWidth = 1
          this.context.stroke()

          // 7. 复杂路径 - 闪电形状
          this.context.beginPath()
          this.context.moveTo(350, 350)
          this.context.lineTo(320, 400)
          this.context.lineTo(340, 400)
          this.context.lineTo(310, 450)
          this.context.lineTo(375, 390)
          this.context.lineTo(350, 390)
          this.context.lineTo(380, 350)
          this.context.closePath()

          this.context.fillStyle = '#FAAD14'
          this.context.fill()
        })

      Text('说明：展示了多边形、贝塞尔曲线、圆弧、心形、星形和闪电等路径绘制技术')
        .fontSize(14)
        .fontColor('#666')
        .width('90%')
        .textAlign(TextAlign.Center)
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

## 实际应用场景

下面是一个实际应用场景的例子 - 绘制一个简单的图表：

```typescript
@Entry
@Component
struct ChartDemo {
  private settings: RenderingContextSettings = new RenderingContextSettings(true)
  private context: CanvasRenderingContext2D = new CanvasRenderingContext2D(this.settings)

  // 图表数据
  private data: number[] = [30, 70, 45, 90, 60, 80, 50]
  private labels: string[] = ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
  private colors: string[] = ['#1890FF', '#2FC25B', '#FACC14', '#223273', '#8543E0', '#13C2C2', '#3436C7']

  // 图表尺寸和位置
  private chartPadding: number = 50
  private barWidth: number = 30
  private barSpacing: number = 20
  private maxValue: number = 100

  build() {
    Column({ space: 20 }) {
      Text('每周工作量柱状图')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Canvas(this.context)
        .width('100%')
        .height(500)
        .backgroundColor('#F5F5F5')
        .onReady(() => {
          // 清空画布
          this.context.clearRect(0, 0, 1000, 1000)

          // 绘制坐标轴
          this.drawAxis()

          // 绘制柱状图
          this.drawBarChart()

          // 绘制图例
          this.drawLegend()
        })

      Text('说明：展示了如何使用Canvas绘制一个带有坐标轴、数据标签和图例的柱状图')
        .fontSize(14)
        .fontColor('#666')
        .width('90%')
        .textAlign(TextAlign.Center)
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }

  // 绘制坐标轴
  drawAxis() {
    const canvasWidth = 350
    const canvasHeight = 400

    this.context.strokeStyle = '#333333'
    this.context.lineWidth = 2

    // X轴
    this.context.beginPath()
    this.context.moveTo(this.chartPadding, canvasHeight - this.chartPadding)
    this.context.lineTo(canvasWidth - this.chartPadding, canvasHeight - this.chartPadding)
    this.context.stroke()

    // Y轴
    this.context.beginPath()
    this.context.moveTo(this.chartPadding, this.chartPadding)
    this.context.lineTo(this.chartPadding, canvasHeight - this.chartPadding)
    this.context.stroke()

    // X轴刻度和标签
    this.context.textAlign = 'center'
    this.context.textBaseline = 'top'
    this.context.font = '14px sans-serif'
    this.context.fillStyle = '#333333'

    for (let i = 0; i < this.data.length; i++) {
      const x = this.chartPadding + (i + 0.5) * (this.barWidth + this.barSpacing)

      // 刻度线
      this.context.beginPath()
      this.context.moveTo(x, canvasHeight - this.chartPadding)
      this.context.lineTo(x, canvasHeight - this.chartPadding + 5)
      this.context.stroke()

      // 标签
      this.context.fillText(this.labels[i], x, canvasHeight - this.chartPadding + 10)
    }

    // Y轴刻度和标签
    this.context.textAlign = 'right'
    this.context.textBaseline = 'middle'

    const yAxisHeight = canvasHeight - 2 * this.chartPadding
    const yStep = 20

    for (let value = 0; value <= this.maxValue; value += yStep) {
      const y = canvasHeight - this.chartPadding - (value / this.maxValue) * yAxisHeight

      // 刻度线
      this.context.beginPath()
      this.context.moveTo(this.chartPadding - 5, y)
      this.context.lineTo(this.chartPadding, y)
      this.context.stroke()

      // 标签
      this.context.fillText(value.toString(), this.chartPadding - 10, y)

      // 网格线（虚线）
      if (value > 0) {
        this.context.beginPath()
        this.context.setLineDash([5, 5])
        this.context.moveTo(this.chartPadding, y)
        this.context.lineTo(canvasWidth - this.chartPadding, y)
        this.context.strokeStyle = '#CCCCCC'
        this.context.stroke()
        this.context.setLineDash([])
        this.context.strokeStyle = '#333333'
      }
    }
  }

  // 绘制柱状图
  drawBarChart() {
    const canvasHeight = 400
    const yAxisHeight = canvasHeight - 2 * this.chartPadding

    // 绘制数据条
    for (let i = 0; i < this.data.length; i++) {
      const x = this.chartPadding + i * (this.barWidth + this.barSpacing)
      const barHeight = (this.data[i] / this.maxValue) * yAxisHeight
      const y = canvasHeight - this.chartPadding - barHeight

      // 设置填充颜色
      this.context.fillStyle = this.colors[i]

      // 绘制柱子
      this.context.fillRect(x + this.barSpacing / 2, y, this.barWidth, barHeight)

      // 绘制数值标签
      this.context.textAlign = 'center'
      this.context.textBaseline = 'bottom'
      this.context.fillStyle = '#333333'
      this.context.font = '14px sans-serif'
      this.context.fillText(this.data[i].toString(), x + this.barSpacing / 2 + this.barWidth / 2, y - 5)
    }
  }

  // 绘制图例
  drawLegend() {
    const legendX = 50
    const legendY = 420
    const legendItemHeight = 20
    const legendItemWidth = 100
    const legendColorSize = 15

    this.context.textAlign = 'left'
    this.context.textBaseline = 'middle'
    this.context.font = '14px sans-serif'
    this.context.fillStyle = '#333333'

    // 绘制"图例"标题
    this.context.font = 'bold 16px sans-serif'
    this.context.fillText('图例:', legendX, legendY)
    this.context.font = '14px sans-serif'

    // 绘制图例项
    for (let i = 0; i < Math.min(4, this.labels.length); i++) {
      const x = legendX + Math.floor(i / 2) * legendItemWidth
      const y = legendY + 30 + (i % 2) * legendItemHeight

      // 绘制颜色框
      this.context.fillStyle = this.colors[i]
      this.context.fillRect(x, y - legendColorSize / 2, legendColorSize, legendColorSize)

      // 绘制标签
      this.context.fillStyle = '#333333'
      this.context.fillText(this.labels[i], x + legendColorSize + 5, y)
    }

    // 继续绘制剩余图例项
    for (let i = 4; i < this.labels.length; i++) {
      const x = legendX + 210 + Math.floor((i - 4) / 2) * legendItemWidth
      const y = legendY + 30 + ((i - 4) % 2) * legendItemHeight

      // 绘制颜色框
      this.context.fillStyle = this.colors[i]
      this.context.fillRect(x, y - legendColorSize / 2, legendColorSize, legendColorSize)

      // 绘制标签
      this.context.fillStyle = '#333333'
      this.context.fillText(this.labels[i], x + legendColorSize + 5, y)
    }
  }
}
```

通过本文的示例，我们展示了在鸿蒙应用开发中如何使用 Canvas 组件绘制各种图形、文本、图像和路径，以及如何将这些技术应用到实际场景中。Canvas 绘图功能强大而灵活，可以实现许多原生组件难以实现的自定义 UI 效果，如各类图表、自定义动画、游戏界面等。在实际开发中，合理利用 Canvas 可以大大提升应用的表现力和用户体验。
