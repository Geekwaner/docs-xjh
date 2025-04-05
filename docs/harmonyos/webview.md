# 鸿蒙开发之 WebView

在鸿蒙应用开发中，WebView 组件是一个强大的工具，它允许应用内嵌入网页内容，实现混合应用开发。本文将介绍如何在鸿蒙应用中使用 WebView 组件及其各种功能。

## 目录

- [基础使用](#基础使用)
- [加载不同内容](#加载不同内容)
- [WebView 控制](#webview控制)
- [JavaScript 交互](#javascript交互)
- [高级功能](#高级功能)

## 基础使用

WebView 的基本使用非常简单，只需创建一个 WebView 组件并提供要加载的 URL。

```typescript
@Entry
@Component
struct BasicWebViewDemo {
  build() {
    Column() {
      Text('基础 WebView 示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 20 })

      // 基础 WebView
      Web({ src: 'https://developer.harmonyos.com' })
        .width('100%')
        .height('80%')
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

## 加载不同内容

WebView 可以加载各种内容，包括远程 URL、本地 HTML 文件、HTML 字符串等。

### 加载远程网页

```typescript
Web({ src: "https://developer.harmonyos.com" }).width("100%").height("80%");
```

### 加载本地 HTML 文件

```typescript
// 假设在 resources/rawfile 目录下有一个 local.html 文件
Web({ src: $rawfile("local.html") })
  .width("100%")
  .height("80%");
```

### 加载 HTML 字符串

```typescript
const htmlString = `
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>本地 HTML</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 20px; }
    h1 { color: #007DFF; }
  </style>
</head>
<body>
  <h1>这是通过 HTML 字符串加载的内容</h1>
  <p>当前时间: <span id="time"></span></p>
  <script>
    document.getElementById('time').textContent = new Date().toLocaleString();
  </script>
</body>
</html>
`;

Web({ src: htmlString, sandboxMode: true }).width("100%").height("80%");
```

## WebView 控制

WebView 提供了许多控制功能，如导航、刷新、获取信息等。

```typescript
@Entry
@Component
struct WebViewControlDemo {
  @State currentUrl: string = 'https://developer.harmonyos.com'
  @State pageTitle: string = ''
  @State canGoBack: boolean = false
  @State canGoForward: boolean = false
  @State isLoading: boolean = true
  private controller: WebController = new WebController()

  build() {
    Column({ space: 10 }) {
      Text('WebView 控制示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      // 网页标题
      if (this.pageTitle) {
        Text(`当前页面: ${this.pageTitle}`)
          .fontSize(16)
          .maxLines(1)
          .textOverflow({ overflow: TextOverflow.Ellipsis })
      }

      // 网址输入框和加载按钮
      Row({ space: 10 }) {
        TextInput({ text: this.currentUrl })
          .layoutWeight(1)
          .onChange((value) => {
            this.currentUrl = value
          })

        Button('前往')
          .onClick(() => {
            // 确保 URL 格式正确
            if (!this.currentUrl.startsWith('http://') && !this.currentUrl.startsWith('https://')) {
              this.currentUrl = 'https://' + this.currentUrl
            }
            this.controller.loadUrl(this.currentUrl)
          })
      }
      .width('100%')

      // 加载指示器
      if (this.isLoading) {
        Row() {
          LoadingProgress().width(24).height(24)
          Text('加载中...').margin({ left: 10 })
        }
        .justifyContent(FlexAlign.Center)
        .width('100%')
        .height(30)
      }

      // WebView
      Web({ src: this.currentUrl, controller: this.controller })
        .width('100%')
        .height('60%')
        .onPageBegin(e => {
          this.isLoading = true
          console.info('Page begin loading: ' + e.url)
        })
        .onPageEnd(e => {
          this.isLoading = false
          console.info('Page end loading: ' + e.url)
          // 更新当前URL
          this.currentUrl = e.url
        })
        .onTitleReceive(e => {
          this.pageTitle = e.title
          console.info('Title received: ' + e.title)
        })
        .onProgressChange(e => {
          console.info('Loading progress: ' + e.newProgress)
        })
        .onPageError(e => {
          this.isLoading = false
          console.error('Page error: ' + JSON.stringify(e))
        })
        .onHttpErrorReceive(e => {
          console.warn('HTTP error: ' + JSON.stringify(e))
        })

      // 导航控制按钮
      Row({ space: 15 }) {
        Button('返回')
          .enabled(this.canGoBack)
          .opacity(this.canGoBack ? 1 : 0.5)
          .onClick(() => {
            this.controller.backward()
          })

        Button('前进')
          .enabled(this.canGoForward)
          .opacity(this.canGoForward ? 1 : 0.5)
          .onClick(() => {
            this.controller.forward()
          })

        Button('刷新')
          .onClick(() => {
            this.controller.refresh()
          })

        Button('停止')
          .onClick(() => {
            this.controller.stop()
            this.isLoading = false
          })

        Button('首页')
          .onClick(() => {
            this.controller.loadUrl('https://developer.harmonyos.com')
          })
      }
      .width('100%')
      .justifyContent(FlexAlign.SpaceEvenly)

      // 检查导航状态
      Button('检查导航状态')
        .onClick(() => {
          this.controller.runJavaScript('history.length', result => {
            console.info('History length: ' + result)
          })
          this.controller.accessBackward(result => {
            this.canGoBack = result
            console.info('Can go back: ' + result)
          })
          this.controller.accessForward(result => {
            this.canGoForward = result
            console.info('Can go forward: ' + result)
          })
        })
    }
    .width('100%')
    .height('100%')
    .padding(15)
  }
}
```

## JavaScript 交互

WebView 允许 JavaScript 与鸿蒙应用之间进行双向通信，这对于混合应用开发非常有用。

```typescript
@Entry
@Component
struct JavaScriptInteractionDemo {
  @State messageFromJS: string = ''
  @State counter: number = 0
  private controller: WebController = new WebController()

  // 示例HTML内容
  private htmlContent: string = `
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>JavaScript 交互示例</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 20px; }
    button { padding: 10px; margin: 10px 0; background-color: #007DFF; color: white; border: none; border-radius: 4px; }
    #result { margin-top: 20px; padding: 10px; background-color: #f0f0f0; border-radius: 4px; }
  </style>
</head>
<body>
  <h1>JavaScript 与鸿蒙交互</h1>
  <button onclick="sendMessageToHarmonyOS()">发送消息到鸿蒙</button>
  <button onclick="callHarmonyOSMethod()">调用鸿蒙方法</button>
  <button onclick="getCounterValue()">获取鸿蒙计数器值</button>
  <div id="result">结果将显示在这里</div>

  <script>
    // 发送消息到鸿蒙
    function sendMessageToHarmonyOS() {
      // 调用鸿蒙注册的方法
      const timestamp = new Date().toLocaleString();
      window.harmonyOSBridge.receiveMessage('来自网页的消息，时间: ' + timestamp);
      document.getElementById('result').innerText = '消息已发送到鸿蒙';
    }

    // 调用鸿蒙方法
    function callHarmonyOSMethod() {
      // 调用鸿蒙注册的方法，可以传递参数
      window.harmonyOSBridge.performAction('增加计数器', 1);
      document.getElementById('result').innerText = '已请求鸿蒙增加计数器';
    }

    // 获取鸿蒙数据
    function getCounterValue() {
      // 尝试获取鸿蒙的计数器值
      window.harmonyOSBridge.getCounter(function(value) {
        document.getElementById('result').innerText = '当前计数器值: ' + value;
      });
    }

    // 提供一个方法让鸿蒙调用
    window.updateFromHarmonyOS = function(message) {
      document.getElementById('result').innerText = message;
      return '已更新页面内容';
    }
  </script>
</body>
</html>
  `

  build() {
    Column({ space: 15 }) {
      Text('JavaScript 交互示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      // 显示来自JavaScript的消息
      if (this.messageFromJS) {
        Text('来自JavaScript的消息:')
          .fontSize(16)
          .fontWeight(FontWeight.Bold)

        Text(this.messageFromJS)
          .fontSize(14)
          .padding(10)
          .backgroundColor('#f0f0f0')
          .width('100%')
      }

      // 显示计数器
      Text(`当前计数器: ${this.counter}`)
        .fontSize(16)

      // 操作按钮
      Row({ space: 10 }) {
        Button('增加计数器')
          .onClick(() => {
            this.counter++
          })

        Button('调用JS方法')
          .onClick(() => {
            // 调用网页中的JavaScript方法
            this.controller.runJavaScript(
              `updateFromHarmonyOS('鸿蒙更新: ${new Date().toLocaleString()}')`,
              (result) => {
                console.info('JS方法返回: ' + result)
              }
            )
          })

        Button('重置')
          .onClick(() => {
            this.counter = 0
            this.messageFromJS = ''
          })
      }
      .width('100%')
      .justifyContent(FlexAlign.SpaceEvenly)

      // WebView
      Web({
        src: this.htmlContent,
        controller: this.controller
      })
        .width('100%')
        .layoutWeight(1)
        .javaScriptAccess(true) // 启用JavaScript
        .onPageEnd(() => {
          // 页面加载完成后注册JavaScript接口
          this.registerJavaScriptInterface()
        })
    }
    .width('100%')
    .height('100%')
    .padding(15)
  }

  // 注册JavaScript接口
  registerJavaScriptInterface() {
    // 创建一个JavaScript接口对象
    const jsInterface = {
      // 接收来自JavaScript的消息
      receiveMessage: (message) => {
        this.messageFromJS = message
        return 'Message received'
      },

      // 执行操作
      performAction: (action, value) => {
        console.info(`接收到动作请求: ${action}, 值: ${value}`)
        if (action === '增加计数器') {
          this.counter += value
          return `计数器已增加${value}`
        }
        return 'Unknown action'
      },

      // 获取计数器值
      getCounter: (callback) => {
        // 使用回调返回计数器值
        this.controller.runJavaScript(`${callback}(${this.counter})`)
        return true
      }
    }

    // 注册JavaScript接口
    this.controller.addJavaScriptInterface('harmonyOSBridge', jsInterface)
  }
}
```

## 高级功能

WebView 还提供了许多高级功能，如文件上传、HTTP 请求拦截、Cookie 管理等。

### 自定义 WebView 设置

```typescript
@Entry
@Component
struct AdvancedWebViewDemo {
  private controller: WebController = new WebController()

  build() {
    Column({ space: 15 }) {
      Text('高级 WebView 设置')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Web({
        src: 'https://developer.harmonyos.com',
        controller: this.controller
      })
        .width('100%')
        .layoutWeight(1)
        // 设置 User-Agent
        .userAgent('HarmonyOS WebView Demo')
        // 启用缩放
        .zoomAccess(true)
        // 自适应屏幕宽度
        .textZoomRatio(100)
        // 启用地理位置
        .geolocationAccess(true)
        // 是否允许混合内容（HTTP 和 HTTPS 混合）
        .mixedMode(MixedMode.MIXED_CONTENT_ALWAYS_ALLOW)
        // 是否允许表单自动填充
        .autoFillForm(true)
        // 是否支持多窗口
        .multiWindowAccess(false)
        // 设置缓存模式
        .cacheMode(CacheMode.DEFAULT)
        // 文件访问模式
        .fileAccess(true)
        // 是否允许执行JavaScript
        .javaScriptAccess(true)
        // DOM Storage API 是否可用
        .domStorageAccess(true)
        // 是否显示水平滚动条
        .horizontalScrollBarAccess(true)
        // 是否启用媒体播放手势
        .mediaPlayGestureAccess(true)
        // 是否允许使用内置缩放机制
        .zoomRulesAccess(true)
        // 是否开启Web调试
        .debugEnabled(true)
        // 设置默认编码
        .defaultEncoding('UTF-8')
        // 是否使用WideViewport模式
        .wideViewModeAccess(true)
        // 初始缩放比例
        .initialScale(100)
        // 设置背景颜色
        .backgroundColor('#FFFFFF')
    }
    .width('100%')
    .height('100%')
    .padding(15)
  }
}
```

### HTTP 请求和 Cookie 拦截

```typescript
@Entry
@Component
struct WebViewInterceptionDemo {
  private controller: WebController = new WebController()
  @State logs: string[] = []

  build() {
    Column({ space: 15 }) {
      Text('WebView 请求拦截')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Web({
        src: 'https://example.com',
        controller: this.controller
      })
        .width('100%')
        .height('60%')
        // 监听 HTTP 请求相关事件
        .onHttpAuthRequest(event => {
          // HTTP 认证请求
          this.logs.push(`HTTP Auth Request: ${event.host}:${event.realm}`)
          // 可以选择提供认证信息
          // event.handler.onConfirm('username', 'password')
          // 或者取消认证
          event.handler.onCancel()
        })
        .onSslErrorReceive(event => {
          // SSL 证书错误
          this.logs.push(`SSL Error: ${event.error}`)
          // 可以选择继续加载
          // event.handler.onConfirm()
          // 或者取消加载
          event.handler.onCancel()
        })
        .onClientAuthenticationRequest(event => {
          // 客户端证书认证请求
          this.logs.push(`Client Auth Request: ${event.host}:${event.port}`)
          // 可以选择提供证书
          // event.handler.onConfirm('keyAlias')
          // 或者取消认证
          event.handler.onCancel()
        })
        .onDownloadStart(event => {
          // 文件下载开始
          this.logs.push(`Download Start: ${event.url}, MIME: ${event.mimeType}`)
        })
        .onInterceptRequest(event => {
          // 拦截网络请求
          this.logs.push(`Intercept Request: ${event.request.getRequestUrl()}`)
          // 可以修改请求或返回自定义响应
          return null // 返回null则继续正常请求
        })
        .onInterceptKeyDown(keyEvent => {
          // 拦截键盘事件
          this.logs.push(`Key Down: ${keyEvent.code}`)
          return false // 返回false则继续传递事件，返回true则拦截
        })

      // 显示日志
      Text('请求日志:')
        .fontSize(16)
        .fontWeight(FontWeight.Bold)

      List() {
        ForEach(this.logs, (log, index) => {
          ListItem() {
            Text(`${index + 1}. ${log}`)
              .fontSize(14)
              .width('100%')
          }
        })
      }
      .width('100%')
      .layoutWeight(1)
      .backgroundColor('#f5f5f5')
      .padding(10)

      // 控制按钮
      Row({ space: 10 }) {
        Button('清除日志')
          .onClick(() => {
            this.logs = []
          })

        Button('设置Cookie')
          .onClick(() => {
            this.controller.setCookie('https://example.com', 'name=value; path=/;')
            this.logs.push('Set Cookie: name=value')
          })

        Button('获取Cookie')
          .onClick(() => {
            this.controller.getCookies('https://example.com', cookies => {
              this.logs.push(`Cookies: ${cookies.join('; ')}`)
            })
          })
      }
      .width('100%')
      .justifyContent(FlexAlign.SpaceEvenly)
    }
    .width('100%')
    .height('100%')
    .padding(15)
  }
}
```

这些示例展示了鸿蒙 WebView 组件的强大功能和灵活性，可以满足各种混合应用开发需求。通过使用 WebView，开发者可以将 Web 内容无缝集成到鸿蒙应用中，实现丰富的跨平台用户体验。
