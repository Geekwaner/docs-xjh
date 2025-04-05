# 鸿蒙开发之路由控制

在鸿蒙应用开发中，路由控制是实现页面导航和跳转的关键机制。本文介绍鸿蒙系统中的路由控制方式和常见用法。

## 目录

- [基本路由机制](#基本路由机制)
- [页面参数传递](#页面参数传递)
- [路由栈管理](#路由栈管理)
- [路由拦截与守卫](#路由拦截与守卫)
- [高级路由应用](#高级路由应用)

## 基本路由机制

鸿蒙系统提供了 `router` 模块来处理页面间的导航。

```typescript
import router from '@ohos.router'

@Entry
@Component
struct RoutingBasics {
  build() {
    Column({ space: 20 }) {
      Text('路由基础示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      // 跳转到新页面
      Button('跳转到详情页')
        .onClick(() => {
          router.pushUrl({
            url: 'pages/DetailPage',
          })
        })

      // 返回上一页
      Button('返回上一页')
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

## 页面参数传递

路由跳转时可以传递参数，接收页面可以获取这些参数。

```typescript
// 发送页面
@Entry
@Component
struct SenderPage {
  build() {
    Column({ space: 20 }) {
      Text('发送页面')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Button('跳转并传递参数')
        .onClick(() => {
          // 传递多种类型的参数
          router.pushUrl({
            url: 'pages/ReceiverPage',
            params: {
              id: 12345,
              name: '商品名称',
              isVip: true,
              items: [1, 2, 3],
              metadata: {
                category: '电子产品',
                price: 999.99
              }
            }
          })
        })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}

// 接收页面
@Entry
@Component
struct ReceiverPage {
  @State params: any = {}

  aboutToAppear() {
    // 获取路由参数
    this.params = router.getParams() as Record<string, any>
    console.info(`Received params: ${JSON.stringify(this.params)}`)
  }

  build() {
    Column({ space: 10 }) {
      Text('接收页面')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Text(`ID: ${this.params.id ?? 'N/A'}`)
        .fontSize(16)

      Text(`名称: ${this.params.name ?? 'N/A'}`)
        .fontSize(16)

      Text(`VIP状态: ${this.params.isVip ? '是' : '否'}`)
        .fontSize(16)

      Text(`数组项: ${this.params.items ? JSON.stringify(this.params.items) : 'N/A'}`)
        .fontSize(16)

      if (this.params.metadata) {
        Text(`分类: ${this.params.metadata.category ?? 'N/A'}`)
          .fontSize(16)

        Text(`价格: ${this.params.metadata.price ?? 'N/A'}`)
          .fontSize(16)
      }

      Button('返回')
        .onClick(() => {
          router.back()
        })
    }
    .width('100%')
    .height('100%')
    .padding(20)
    .justifyContent(FlexAlign.Center)
  }
}
```

## 路由栈管理

鸿蒙系统的路由栈管理，包括路由历史记录的查看和复杂导航场景。

```typescript
@Entry
@Component
struct RouteStackManagement {
  @State routeParams: Array<string> = []

  aboutToAppear() {
    // 获取路由栈信息
    const routeSize = router.getRouteSize()
    console.info(`Current route stack size: ${routeSize}`)

    // 获取当前页面的路由索引
    const currentRouteIndex = router.getRouteSize() - 1
    console.info(`Current route index: ${currentRouteIndex}`)

    // 获取当前页面的路由信息
    const currentRoute = router.getState()
    console.info(`Current route: ${JSON.stringify(currentRoute)}`)
  }

  build() {
    Column({ space: 20 }) {
      Text('路由栈管理')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      // 清除路由历史并跳转到新页面
      Button('清除历史并跳转')
        .onClick(() => {
          router.clear()
          router.pushUrl({
            url: 'pages/HomePage',
          })
        })

      // 返回到指定页面
      Button('返回到首页')
        .onClick(() => {
          router.back({ url: 'pages/HomePage' })
        })

      // 替换当前页面
      Button('替换当前页面')
        .onClick(() => {
          router.replaceUrl({
            url: 'pages/ReplacementPage',
          })
        })

      // 显示当前路由状态
      Button('显示路由状态')
        .onClick(() => {
          const state = router.getState()
          this.routeParams = [
            `索引: ${state.index}`,
            `名称: ${state.name}`,
            `路径: ${state.path}`
          ]
        })

      ForEach(this.routeParams, (param) => {
        Text(param)
          .fontSize(16)
      })
    }
    .width('100%')
    .height('100%')
    .padding(20)
    .justifyContent(FlexAlign.Center)
  }
}
```

## 路由拦截与守卫

通过在 `aboutToAppear` 和 `onBackPress` 生命周期函数中添加逻辑，可以实现路由拦截和守卫。

```typescript
@Entry
@Component
struct RouteGuards {
  @State hasUnsavedChanges: boolean = true

  // 返回按钮拦截
  onBackPress(): boolean {
    if (this.hasUnsavedChanges) {
      // 显示确认对话框
      AlertDialog.show({
        title: '未保存的更改',
        message: '您有未保存的更改，确定要离开吗？',
        primaryButton: {
          value: '保存并离开',
          action: () => {
            // 保存逻辑
            console.info('Saving changes...')
            setTimeout(() => {
              router.back()
            }, 500)
          }
        },
        secondaryButton: {
          value: '不保存离开',
          action: () => {
            router.back()
          }
        },
        cancel: () => {
          console.info('User canceled')
        }
      })
      return true // 返回true表示已处理返回操作
    }
    return false // 返回false让系统处理返回操作
  }

  // 进入页面拦截
  aboutToAppear() {
    const params = router.getParams() as Record<string, any>

    // 检查用户是否有权限访问
    if (params.requiresAuth && !params.isLoggedIn) {
      console.info('User not authenticated, redirecting to login')
      // 重定向到登录页面
      router.replaceUrl({
        url: 'pages/Login',
        params: {
          redirect: router.getState().path
        }
      })
    }
  }

  build() {
    Column({ space: 20 }) {
      Text('路由守卫示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Text('此页面有未保存的更改，尝试返回将触发确认对话框')
        .fontSize(16)
        .margin({ top: 20 })

      Switch()
        .checked(this.hasUnsavedChanges)
        .onChange((value) => {
          this.hasUnsavedChanges = value
        })
      Text('有未保存的更改')
        .fontSize(16)

      Button('尝试返回')
        .onClick(() => {
          // 模拟按下返回键
          this.onBackPress()
        })
    }
    .width('100%')
    .height('100%')
    .padding(20)
    .justifyContent(FlexAlign.Center)
  }
}
```

## 高级路由应用

更复杂的路由场景，例如有条件导航、跨页面数据共享等。

```typescript
// 路由服务
class RouterService {
  // 存储共享数据
  private static sharedData: Map<string, any> = new Map()

  // 设置共享数据
  static setSharedData(key: string, value: any): void {
    this.sharedData.set(key, value)
  }

  // 获取共享数据
  static getSharedData(key: string): any {
    return this.sharedData.get(key)
  }

  // 删除共享数据
  static removeSharedData(key: string): void {
    this.sharedData.delete(key)
  }

  // 条件导航
  static conditionalNavigate(condition: boolean, trueUrl: string, falseUrl: string, params?: Record<string, any>): void {
    router.pushUrl({
      url: condition ? trueUrl : falseUrl,
      params
    })
  }

  // 带回调的导航
  static navigateForResult(url: string, params: Record<string, any>, callback: (result: any) => void): void {
    // 生成唯一回调ID
    const callbackId = `callback_${Date.now()}`

    // 存储回调函数
    this.setSharedData(callbackId, callback)

    // 导航到目标页面
    router.pushUrl({
      url,
      params: {
        ...params,
        callbackId
      }
    })
  }

  // 返回并传递结果
  static returnWithResult(callbackId: string, result: any): void {
    // 获取回调函数
    const callback = this.getSharedData(callbackId) as (result: any) => void

    // 如果有回调函数，执行它
    if (callback) {
      callback(result)
      // 删除回调函数
      this.removeSharedData(callbackId)
    }

    // 返回上一页
    router.back()
  }
}

// 使用示例
@Entry
@Component
struct AdvancedRouting {
  build() {
    Column({ space: 20 }) {
      Text('高级路由示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      // 条件导航
      Button('条件导航')
        .onClick(() => {
          const userRole = 'admin' // 假设这是从某处获取的用户角色
          RouterService.conditionalNavigate(
            userRole === 'admin',
            'pages/AdminPage',
            'pages/UserPage',
            { username: 'John' }
          )
        })

      // 带回调的导航
      Button('选择商品')
        .onClick(() => {
          RouterService.navigateForResult(
            'pages/ProductSelector',
            { category: 'electronics' },
            (selectedProducts) => {
              // 处理选择结果
              console.info(`Selected products: ${JSON.stringify(selectedProducts)}`)
              // 这里可以更新UI等
            }
          )
        })
    }
    .width('100%')
    .height('100%')
    .padding(20)
    .justifyContent(FlexAlign.Center)
  }
}

// 接收回调的页面
@Entry
@Component
struct ProductSelector {
  @State params: Record<string, any> = {}

  aboutToAppear() {
    this.params = router.getParams() as Record<string, any>
  }

  build() {
    Column({ space: 20 }) {
      Text('商品选择器')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Text(`分类: ${this.params.category}`)
        .fontSize(16)

      Button('选择iPhone')
        .onClick(() => {
          if (this.params.callbackId) {
            RouterService.returnWithResult(this.params.callbackId, {
              id: 1,
              name: 'iPhone 13',
              price: 799
            })
          }
        })

      Button('选择MacBook')
        .onClick(() => {
          if (this.params.callbackId) {
            RouterService.returnWithResult(this.params.callbackId, {
              id: 2,
              name: 'MacBook Pro',
              price: 1299
            })
          }
        })

      Button('取消')
        .onClick(() => {
          router.back()
        })
    }
    .width('100%')
    .height('100%')
    .padding(20)
    .justifyContent(FlexAlign.Center)
  }
}
```
