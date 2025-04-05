# 鸿蒙开发之生命周期

在鸿蒙应用开发中，了解组件和应用的生命周期对于正确管理资源和实现功能至关重要。本文介绍鸿蒙系统中的各种生命周期钩子函数及其用法。

## 目录

- [组件生命周期](#组件生命周期)
- [应用生命周期](#应用生命周期)
- [页面生命周期](#页面生命周期)
- [生命周期最佳实践](#生命周期最佳实践)

## 组件生命周期

鸿蒙组件（ArkUI）的生命周期包括创建、渲染、更新和销毁等阶段，每个阶段都有相应的回调函数。

### 主要生命周期钩子函数

```typescript
@Component
struct LifecycleComponent {
  @State message: string = 'Hello World'

  // 组件即将出现时调用
  aboutToAppear() {
    console.info('Component aboutToAppear')
    // 适合执行初始化操作、数据请求等
  }

  // 组件即将消失时调用
  aboutToDisappear() {
    console.info('Component aboutToDisappear')
    // 适合执行清理操作、释放资源等
  }

  // 组件渲染完成后调用
  onPageShow() {
    console.info('Component onPageShow')
    // 页面显示时调用，适合启动动画、继续播放等
  }

  // 页面隐藏时调用
  onPageHide() {
    console.info('Component onPageHide')
    // 页面隐藏时调用，适合暂停操作、停止刷新等
  }

  // 组件更新前调用
  onBackPress() {
    console.info('Component onBackPress')
    // 用户点击返回按钮时调用，可以拦截返回操作
    return false; // 返回true表示已处理返回事件，false表示未处理
  }

  build() {
    Column() {
      Text(this.message)
        .fontSize(20)

      Button('更新消息')
        .onClick(() => {
          this.message = 'Updated Message'
        })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}
```

### 生命周期调用顺序

1. 创建组件实例
2. 调用 `aboutToAppear()`
3. 执行 `build()` 函数渲染 UI
4. 完成渲染，显示界面
5. 如果组件更新，再次执行 `build()`
6. 组件将要销毁，调用 `aboutToDisappear()`
7. 组件销毁

## 应用生命周期

鸿蒙应用的生命周期管理了应用从启动到退出的整个过程。

```typescript
import AbilityStage from "@ohos.app.ability.AbilityStage";
import UIAbility from "@ohos.app.ability.UIAbility";
import window from "@ohos.window";

export default class MyAbility extends UIAbility {
  // 应用创建时调用
  onCreate(want, launchParam) {
    console.info("Ability onCreate");
    // 适合进行应用级初始化
  }

  // 窗口创建完成时调用
  onWindowStageCreate(windowStage) {
    console.info("Ability onWindowStageCreate");

    // 设置UI内容
    windowStage.loadContent("pages/Index");

    // 窗口获取焦点事件监听
    windowStage.on("windowStageEvent", (data) => {
      console.info(`windowStageEvent: ${JSON.stringify(data)}`);
    });
  }

  // 窗口获得前台焦点时调用
  onForeground() {
    console.info("Ability onForeground");
    // 应用进入前台，适合恢复之前暂停的任务
  }

  // 窗口失去焦点进入后台时调用
  onBackground() {
    console.info("Ability onBackground");
    // 应用进入后台，适合暂停任务、保存状态等
  }

  // 窗口销毁时调用
  onWindowStageDestroy() {
    console.info("Ability onWindowStageDestroy");
    // 窗口销毁，适合释放窗口相关资源
  }

  // 应用销毁时调用
  onDestroy() {
    console.info("Ability onDestroy");
    // 应用销毁，适合释放全局资源
  }
}
```

## 页面生命周期

在鸿蒙应用中，页面生命周期与组件生命周期结合使用，可以更好地管理页面状态。

```typescript
import router from '@ohos.router';

@Entry
@Component
struct PageLifecycle {
  @State message: string = 'Page Lifecycle Demo'
  private timer: number = -1

  aboutToAppear() {
    console.info('Page aboutToAppear')
    // 获取路由参数
    const params = router.getParams() as Record<string, string>
    if (params && params.id) {
      console.info(`Received params: id=${params.id}`)
    }

    // 启动定时器或其他任务
    this.timer = setInterval(() => {
      console.info('Timer tick')
    }, 1000)
  }

  onPageShow() {
    console.info('Page onPageShow')
    // 页面显示时的操作
  }

  onPageHide() {
    console.info('Page onPageHide')
    // 页面隐藏时的操作
  }

  onBackPress() {
    console.info('Page onBackPress')
    // 自定义返回行为
    AlertDialog.show({
      title: '确认退出',
      message: '是否要离开当前页面？',
      primaryButton: {
        value: '确认',
        action: () => {
          router.back()
        }
      },
      secondaryButton: {
        value: '取消',
        action: () => {
          console.info('用户取消退出')
        }
      }
    })
    return true // 表示已处理返回事件
  }

  aboutToDisappear() {
    console.info('Page aboutToDisappear')
    // 清理资源
    if (this.timer !== -1) {
      clearInterval(this.timer)
      this.timer = -1
    }
  }

  build() {
    Column({ space: 20 }) {
      Text(this.message)
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Button('跳转到新页面')
        .onClick(() => {
          router.pushUrl({
            url: 'pages/NextPage',
            params: {
              id: '12345',
              from: 'homepage'
            }
          }, (err) => {
            if (err) {
              console.error(`路由错误: ${JSON.stringify(err)}`)
            }
          })
        })

      Button('返回')
        .onClick(() => {
          router.back()
        })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}
```

## 生命周期最佳实践

### 资源管理

```typescript
@Component
struct ResourceManagement {
  private resource: any = null
  private listener: (data: any) => void = () => {}
  private eventSubscription: any = null

  aboutToAppear() {
    // 初始化资源
    this.resource = this.createResource()

    // 注册事件监听
    this.listener = (data) => {
      console.info(`Received event: ${JSON.stringify(data)}`)
    }
    // 假设有一个事件订阅系统
    this.eventSubscription = EventBus.subscribe('myEvent', this.listener)
  }

  aboutToDisappear() {
    // 释放资源
    if (this.resource) {
      this.resource.release()
      this.resource = null
    }

    // 取消事件监听
    if (this.eventSubscription) {
      this.eventSubscription.unsubscribe()
      this.eventSubscription = null
    }
  }

  private createResource() {
    // 创建资源的逻辑
    return {
      data: 'some data',
      release: () => {
        console.info('Resource released')
      }
    }
  }

  build() {
    Column() {
      Text('资源管理示例')
    }
  }
}
```

### 数据加载

```typescript
@Component
struct DataLoading {
  @State isLoading: boolean = true
  @State data: any[] = []
  @State error: string = ''

  aboutToAppear() {
    this.loadData()
  }

  async loadData() {
    try {
      this.isLoading = true
      // 模拟网络请求
      const response = await fetch('https://api.example.com/data')
      const result = await response.json()
      this.data = result.items
      this.error = ''
    } catch (err) {
      this.error = `加载失败: ${err.message}`
      this.data = []
    } finally {
      this.isLoading = false
    }
  }

  build() {
    Column({ space: 10 }) {
      if (this.isLoading) {
        LoadingProgress()
          .width(50)
          .height(50)
      } else if (this.error) {
        Text(this.error)
          .fontSize(16)
          .fontColor(Color.Red)

        Button('重试')
          .onClick(() => {
            this.loadData()
          })
      } else {
        List() {
          ForEach(this.data, (item) => {
            ListItem() {
              Text(item.name)
                .fontSize(16)
            }
          })
        }
      }
    }
    .width('100%')
    .padding(20)
  }
}
```
