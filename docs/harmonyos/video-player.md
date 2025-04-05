# 鸿蒙开发之简易视频播放器

在鸿蒙应用开发中，视频播放是常见的功能需求。本文介绍如何使用鸿蒙的 ArkUI 框架和多媒体能力开发一个简易的视频播放器。

## 目录

- [基础视频播放](#基础视频播放)
- [视频播放控制](#视频播放控制)
- [自定义播放器界面](#自定义播放器界面)
- [完整视频播放器](#完整视频播放器)

## 基础视频播放

鸿蒙系统提供了 `Video` 组件用于视频播放，基本用法如下：

```typescript
@Entry
@Component
struct BasicVideoPlayer {
  build() {
    Column({ space: 20 }) {
      Text('基础视频播放器')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      // 基础视频播放组件
      Video({
        src: 'https://www.w3schools.com/html/mov_bbb.mp4',
        previewUri: $r('app.media.video_preview'), // 可选的预览图
        controller: new VideoController() // 视频控制器
      })
        .width('100%')
        .height(260)
        .objectFit(ImageFit.Contain) // 视频适应方式
        .autoPlay(false) // 是否自动播放
        .controls(true) // 是否显示默认控制栏
        .loop(false) // 是否循环播放
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

## 视频播放控制

通过 `VideoController` 可以控制视频的播放、暂停、跳转等行为。

```typescript
@Entry
@Component
struct VideoControlDemo {
  private videoController: VideoController = new VideoController()
  @State isPlaying: boolean = false
  @State currentTime: number = 0
  @State duration: number = 0

  build() {
    Column({ space: 20 }) {
      Text('视频播放控制')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Video({
        src: 'https://www.w3schools.com/html/mov_bbb.mp4',
        controller: this.videoController
      })
        .width('100%')
        .height(260)
        .objectFit(ImageFit.Contain)
        .autoPlay(false)
        .controls(false) // 关闭默认控制栏，使用自定义控制
        .onPrepared((event) => {
          // 视频准备完成事件
          this.duration = event.duration
          console.info(`Video prepared, duration: ${this.duration}ms`)
        })
        .onUpdate((event) => {
          // 视频播放进度更新事件
          this.currentTime = event.time
        })
        .onStart(() => {
          // 视频开始播放事件
          this.isPlaying = true
          console.info('Video playback started')
        })
        .onPause(() => {
          // 视频暂停事件
          this.isPlaying = false
          console.info('Video playback paused')
        })
        .onFinish(() => {
          // 视频播放完成事件
          this.isPlaying = false
          console.info('Video playback finished')
        })
        .onError(() => {
          // 视频播放错误事件
          console.error('Video playback error')
        })

      // 自定义控制栏
      Row({ space: 20 }) {
        Button(this.isPlaying ? '暂停' : '播放')
          .onClick(() => {
            if (this.isPlaying) {
              this.videoController.pause()
            } else {
              this.videoController.start()
            }
          })

        Button('停止')
          .onClick(() => {
            this.videoController.stop()
            this.isPlaying = false
          })

        Button('向前10秒')
          .onClick(() => {
            const newTime = Math.min(this.currentTime + 10000, this.duration)
            this.videoController.seekTo(newTime)
          })

        Button('向后10秒')
          .onClick(() => {
            const newTime = Math.max(this.currentTime - 10000, 0)
            this.videoController.seekTo(newTime)
          })
      }
      .width('100%')
      .justifyContent(FlexAlign.Center)

      // 显示播放进度
      Text(`当前播放进度: ${(this.currentTime / 1000).toFixed(1)}s / ${(this.duration / 1000).toFixed(1)}s`)
        .fontSize(16)

      // 进度条
      Slider({
        value: this.currentTime,
        min: 0,
        max: this.duration > 0 ? this.duration : 100,
        step: 1000
      })
        .width('100%')
        .onChange((value) => {
          this.videoController.seekTo(value)
        })
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```

## 自定义播放器界面

创建一个带有更多功能的自定义视频播放器界面，包括全屏、音量控制等。

```typescript
@Entry
@Component
struct CustomVideoPlayer {
  private videoController: VideoController = new VideoController()
  @State isPlaying: boolean = false
  @State currentTime: number = 0
  @State duration: number = 0
  @State volume: number = 0.5 // 音量 0-1
  @State isMuted: boolean = false // 是否静音
  @State isFullScreen: boolean = false // 是否全屏
  @State showControls: boolean = true // 是否显示控制栏

  // 控制栏自动隐藏计时器
  private controlsTimer: number = -1

  // 格式化时间为 mm:ss 格式
  formatTime(ms: number): string {
    const totalSeconds = Math.floor(ms / 1000)
    const minutes = Math.floor(totalSeconds / 60)
    const seconds = totalSeconds % 60
    return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
  }

  // 重置控制栏隐藏计时器
  resetControlsTimer() {
    if (this.controlsTimer !== -1) {
      clearTimeout(this.controlsTimer)
    }

    // 5秒后自动隐藏控制栏
    if (this.isPlaying) {
      this.controlsTimer = setTimeout(() => {
        this.showControls = false
        this.controlsTimer = -1
      }, 5000)
    }
  }

  // 组件生命周期
  aboutToDisappear() {
    // 清除计时器
    if (this.controlsTimer !== -1) {
      clearTimeout(this.controlsTimer)
      this.controlsTimer = -1
    }
  }

  build() {
    Stack({ alignContent: Alignment.Bottom }) {
      // 视频组件
      Video({
        src: 'https://www.w3schools.com/html/mov_bbb.mp4',
        controller: this.videoController
      })
        .width('100%')
        .height(this.isFullScreen ? '100%' : '260vp')
        .objectFit(ImageFit.Contain)
        .autoPlay(false)
        .controls(false)
        .muted(this.isMuted)
        .onPrepared((event) => {
          this.duration = event.duration
          // 设置音量
          this.videoController.setVolume(this.volume)
        })
        .onUpdate((event) => {
          this.currentTime = event.time
        })
        .onStart(() => {
          this.isPlaying = true
          this.resetControlsTimer()
        })
        .onPause(() => {
          this.isPlaying = false
        })
        .onClick(() => {
          // 点击视频区域切换控制栏显示状态
          this.showControls = !this.showControls
          if (this.showControls) {
            this.resetControlsTimer()
          }
        })

      if (this.showControls) {
        // 控制栏背景 - 半透明渐变
        Column()
          .width('100%')
          .height(this.isFullScreen ? '100%' : '260vp')
          .backgroundColor(this.isFullScreen ? '#00000080' : 'transparent')
          .onClick(() => {
            // 点击事件穿透到视频组件
          })

        // 顶部控制栏
        if (this.isFullScreen) {
          Row({ space: 10 }) {
            Button('返回')
              .onClick(() => {
                this.isFullScreen = false
              })

            Text('视频标题')
              .fontSize(16)
              .fontColor(Color.White)
              .layoutWeight(1)

            Toggle({ type: ToggleType.Switch, isOn: false })
              .onChange((isOn) => {
                // 自动播放开关
              })
          }
          .width('100%')
          .padding(10)
          .position({ x: 0, y: 0 })
        }

        // 底部控制栏
        Column({ space: 10 }) {
          // 进度条
          Slider({
            value: this.currentTime,
            min: 0,
            max: this.duration > 0 ? this.duration : 100,
            step: 1000
          })
            .width('100%')
            .onChange((value) => {
              this.videoController.seekTo(value)
              this.resetControlsTimer()
            })

          // 控制按钮和时间信息
          Row({ space: 20 }) {
            Button(this.isPlaying ? '暂停' : '播放')
              .onClick(() => {
                if (this.isPlaying) {
                  this.videoController.pause()
                } else {
                  this.videoController.start()
                }
                this.resetControlsTimer()
              })

            Text(`${this.formatTime(this.currentTime)} / ${this.formatTime(this.duration)}`)
              .fontSize(14)
              .fontColor(Color.White)

            Row({ space: 5 }) {
              Button(this.isMuted ? '取消静音' : '静音')
                .onClick(() => {
                  this.isMuted = !this.isMuted
                  this.resetControlsTimer()
                })

              Slider({
                value: this.volume * 100,
                min: 0,
                max: 100,
                step: 1
              })
                .width(100)
                .onChange((value) => {
                  this.volume = value / 100
                  this.videoController.setVolume(this.volume)
                  this.isMuted = this.volume === 0
                  this.resetControlsTimer()
                })
            }

            Button(this.isFullScreen ? '退出全屏' : '全屏')
              .onClick(() => {
                this.isFullScreen = !this.isFullScreen
                this.resetControlsTimer()
              })
          }
          .width('100%')
          .justifyContent(FlexAlign.SpaceBetween)
          .padding({ left: 10, right: 10, bottom: 10 })
        }
        .width('100%')
        .backgroundColor('#00000080')
      }
    }
    .width('100%')
    .height(this.isFullScreen ? '100%' : '260vp')
    .position({ x: 0, y: 0 })
  }
}
```

## 完整视频播放器

以下是一个功能完整的视频播放器组件，支持多种视频源、播放列表等高级功能。

```typescript
// 视频项数据结构
interface VideoItem {
  id: string;
  title: string;
  description: string;
  source: string;
  thumbnail?: string;
  duration?: number;
}

// 视频播放器组件
@Component
export struct VideoPlayer {
  // 视频控制器
  private videoController: VideoController = new VideoController()
  // 视频列表
  @Link videoList: Array<VideoItem>
  // 当前播放的视频索引
  @State currentIndex: number = 0
  // 播放状态
  @State isPlaying: boolean = false
  @State currentTime: number = 0
  @State duration: number = 0
  @State volume: number = 0.7
  @State isMuted: boolean = false
  @State isFullScreen: boolean = false
  @State showControls: boolean = true
  @State isLoading: boolean = true
  @State playbackRate: number = 1.0 // 播放速率
  @State isPlaylistVisible: boolean = false

  // 播放质量选项
  private qualityOptions: string[] = ['自动', '720p', '1080p', '4K']
  @State selectedQuality: number = 0

  private controlsTimer: number = -1

  // 格式化时间
  formatTime(ms: number): string {
    const totalSeconds = Math.floor(ms / 1000)
    const minutes = Math.floor(totalSeconds / 60)
    const seconds = totalSeconds % 60
    return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
  }

  // 重置控制栏计时器
  resetControlsTimer() {
    if (this.controlsTimer !== -1) {
      clearTimeout(this.controlsTimer)
    }

    if (this.isPlaying) {
      this.controlsTimer = setTimeout(() => {
        this.showControls = false
        this.controlsTimer = -1
      }, 5000)
    }
  }

  // 播放下一个视频
  playNext() {
    if (this.currentIndex < this.videoList.length - 1) {
      this.currentIndex++
      this.currentTime = 0
      this.isLoading = true
      this.videoController.start()
    }
  }

  // 播放上一个视频
  playPrevious() {
    if (this.currentIndex > 0) {
      this.currentIndex--
      this.currentTime = 0
      this.isLoading = true
      this.videoController.start()
    }
  }

  // 切换到指定视频
  playVideo(index: number) {
    if (index >= 0 && index < this.videoList.length) {
      this.currentIndex = index
      this.currentTime = 0
      this.isLoading = true
      this.isPlaylistVisible = false
      this.videoController.start()
    }
  }

  // 设置播放速率
  setPlaybackRate(rate: number) {
    this.playbackRate = rate
    this.videoController.setSpeed(rate)
  }

  aboutToAppear() {
    // 初始设置
    setTimeout(() => {
      this.videoController.setVolume(this.volume)
    }, 200)
  }

  aboutToDisappear() {
    // 清理
    if (this.controlsTimer !== -1) {
      clearTimeout(this.controlsTimer)
      this.controlsTimer = -1
    }
    // 停止视频播放
    this.videoController.stop()
  }

  build() {
    Stack({ alignContent: Alignment.Center }) {
      // 视频播放区域
      Video({
        src: this.videoList[this.currentIndex].source,
        controller: this.videoController
      })
        .width('100%')
        .height(this.isFullScreen ? '100%' : '260vp')
        .objectFit(ImageFit.Contain)
        .autoPlay(true)
        .controls(false)
        .loop(false)
        .muted(this.isMuted)
        .onPrepared((event) => {
          this.duration = event.duration
          this.isLoading = false
        })
        .onUpdate((event) => {
          this.currentTime = event.time
        })
        .onStart(() => {
          this.isPlaying = true
          this.resetControlsTimer()
        })
        .onPause(() => {
          this.isPlaying = false
        })
        .onFinish(() => {
          this.isPlaying = false
          // 自动播放下一个视频
          this.playNext()
        })
        .onClick(() => {
          this.showControls = !this.showControls
          if (this.showControls) {
            this.resetControlsTimer()
          }
        })

      // 加载指示器
      if (this.isLoading) {
        LoadingProgress()
          .width(50)
          .height(50)
          .color(Color.White)
      }

      if (this.showControls) {
        // 控制界面背景
        Column()
          .width('100%')
          .height(this.isFullScreen ? '100%' : '260vp')
          .backgroundColor('#00000080')
          .onClick(() => {
            // 事件穿透
          })

        // 顶部控制栏
        Row({ space: 10 }) {
          Button('返回')
            .onClick(() => {
              if (this.isFullScreen) {
                this.isFullScreen = false
              } else {
                // 返回上一页或关闭播放器
                // router.back()
              }
            })

          Text(this.videoList[this.currentIndex].title)
            .fontSize(16)
            .fontColor(Color.White)
            .layoutWeight(1)

          Button('播放列表')
            .onClick(() => {
              this.isPlaylistVisible = !this.isPlaylistVisible
              this.resetControlsTimer()
            })
        }
        .width('100%')
        .padding(10)
        .position({ x: 0, y: 0 })

        // 播放列表面板
        if (this.isPlaylistVisible) {
          Column({ space: 10 }) {
            Text('播放列表')
              .fontSize(18)
              .fontColor(Color.White)
              .fontWeight(FontWeight.Bold)

            List() {
              ForEach(this.videoList, (item, index) => {
                ListItem() {
                  Row({ space: 10 }) {
                    Text(item.title)
                      .fontSize(14)
                      .fontColor(index === this.currentIndex ? Color.Blue : Color.White)

                    if (index === this.currentIndex) {
                      Text('播放中')
                        .fontSize(12)
                        .fontColor(Color.Blue)
                    }
                  }
                  .width('100%')
                  .padding(10)
                  .backgroundColor(index === this.currentIndex ? '#ffffff30' : 'transparent')
                  .borderRadius(5)
                  .onClick(() => {
                    this.playVideo(index)
                  })
                }
              })
            }
            .width('100%')
            .height(200)
          }
          .width('80%')
          .padding(10)
          .backgroundColor('#000000B0')
          .borderRadius(10)
          .position({ x: '10%', y: '25%' })
        }

        // 中央控制按钮
        Row({ space: 20 }) {
          Button('上一个')
            .onClick(() => {
              this.playPrevious()
              this.resetControlsTimer()
            })

          Button(this.isPlaying ? '暂停' : '播放')
            .fontSize(18)
            .onClick(() => {
              if (this.isPlaying) {
                this.videoController.pause()
              } else {
                this.videoController.start()
              }
              this.resetControlsTimer()
            })

          Button('下一个')
            .onClick(() => {
              this.playNext()
              this.resetControlsTimer()
            })
        }
        .position({ x: '50%', y: '50%' })
        .translate({ x: '-50%', y: '-50%' })

        // 底部控制栏
        Column({ space: 5 }) {
          // 进度条
          Row({ space: 10 }) {
            Text(this.formatTime(this.currentTime))
              .fontSize(12)
              .fontColor(Color.White)

            Slider({
              value: this.currentTime,
              min: 0,
              max: this.duration > 0 ? this.duration : 100,
              step: 1000
            })
              .layoutWeight(1)
              .onChange((value) => {
                this.videoController.seekTo(value)
                this.resetControlsTimer()
              })

            Text(this.formatTime(this.duration))
              .fontSize(12)
              .fontColor(Color.White)
          }
          .width('100%')
          .padding({ left: 10, right: 10 })

          // 底部功能按钮
          Row({ space: 10 }) {
            // 音量控制
            Row({ space: 5 }) {
              Button(this.isMuted ? '取消静音' : '静音')
                .fontSize(14)
                .onClick(() => {
                  this.isMuted = !this.isMuted
                  this.resetControlsTimer()
                })

              Slider({
                value: this.volume * 100,
                min: 0,
                max: 100,
                step: 1
              })
                .width(80)
                .onChange((value) => {
                  this.volume = value / 100
                  this.videoController.setVolume(this.volume)
                  this.isMuted = this.volume === 0
                  this.resetControlsTimer()
                })
            }

            // 播放速率选择
            Select([
              { value: '0.5x' },
              { value: '1.0x' },
              { value: '1.5x' },
              { value: '2.0x' }
            ])
              .selected(1) // 默认选择 1.0x
              .value('速率: ' + this.playbackRate + 'x')
              .fontSize(14)
              .onSelect((index, value) => {
                const rate = parseFloat(value.replace('x', ''))
                this.setPlaybackRate(rate)
                this.resetControlsTimer()
              })

            // 清晰度选择
            Select(this.qualityOptions.map(option => ({ value: option })))
              .selected(this.selectedQuality)
              .value('清晰度: ' + this.qualityOptions[this.selectedQuality])
              .fontSize(14)
              .onSelect((index, value) => {
                this.selectedQuality = index
                // 这里可以根据不同清晰度切换视频源
                this.resetControlsTimer()
              })

            // 全屏按钮
            Button(this.isFullScreen ? '退出全屏' : '全屏')
              .fontSize(14)
              .onClick(() => {
                this.isFullScreen = !this.isFullScreen
                this.resetControlsTimer()
              })
          }
          .width('100%')
          .justifyContent(FlexAlign.SpaceBetween)
          .padding({ left: 10, right: 10, bottom: 10 })
        }
        .width('100%')
        .position({ x: 0, y: '100%' })
        .translate({ y: '-100%' })
      }
    }
    .width('100%')
    .height(this.isFullScreen ? '100%' : '260vp')
    .backgroundColor(Color.Black)
  }
}

// 使用示例
@Entry
@Component
struct VideoPlayerDemo {
  @State videoList: VideoItem[] = [
    {
      id: '1',
      title: '示例视频1',
      description: '这是一个示例视频',
      source: 'https://www.w3schools.com/html/mov_bbb.mp4',
      thumbnail: 'https://www.w3schools.com/html/img_girl.jpg'
    },
    {
      id: '2',
      title: '示例视频2',
      description: '这是另一个示例视频',
      source: 'https://www.w3schools.com/html/movie.mp4',
      thumbnail: 'https://www.w3schools.com/html/img_girl.jpg'
    }
  ]

  build() {
    Column({ space: 20 }) {
      Text('视频播放器示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      VideoPlayer({ videoList: $videoList })
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}
```
