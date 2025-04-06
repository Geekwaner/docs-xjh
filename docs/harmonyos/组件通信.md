# 鸿蒙开发之组件通信

### 状态共享-父子单向

> `@Prop` 装饰的变量可以和父组件建立单向的同步关系。`@Prop` 装饰的变量是可变的，但是变化不会同步回其父组件。-Prop 是用在子组件中的
>
> Prop 支持类型和 State 修饰符基本一致，并且**<font style="color:#DF2A3F;">Prop 可以给初始值，也可以不给</font>**

#### 完成父-子的单向同步

```typescript
@Entry
@Component
struct PropCase {
  @State pnum: number = 0
  build() {
    Row() {
      Column() {
        Text(this.pnum.toString())
        Button("+1")
          .onClick(() => {
            this.pnum++
          })

        Divider()
        Child({ num: this.pnum })
      }
      .width('100%')
    }
    .height('100%')
  }
}

@Component
struct Child {
  @Prop num: number
  build() {
    Column() {
      Text("子组件")
      Text(this.num.toString())
    }.height(60)
    .width('100%')
    .backgroundColor(Color.Pink)
  }
}
```

![](https://cdn.nlark.com/yuque/0/2023/png/8435673/1702104880851-f31b31d5-c0af-4476-81f4-75ea6fc8ff8a.png)

- 如果子组件修改这个 Prop 呢？我们来试试

```typescript
Button("修改子组件Prop").onClick(() => {
  this.num++;
});
```

![](https://cdn.nlark.com/yuque/0/2023/png/8435673/1702104999557-3cd3d471-32ca-46e2-b73c-4cb4d5a7eec7.png)

- 我们发现使用 Prop 修饰的状态，只会在当前子组件生效，不会传导到父组件，所以它属于一种单向传递
- 子组件可修改 `Prop` 数据值，但不同步到父组件，父组件更新后覆盖子组件 `Prop` 数据
- 子组件可以初始化默认值
- Prop 如果传递是对象类型，它只会在子组件内部生效，不会延伸到父组件-

#### 网络相册案例

- 点个按钮，出现选择照片的相册
- 选择完成之后， 点击完成图片回显示到页面
- 基础相册封装和使用

```typescript
import { GoodItem } from '../04/models'
@Entry
@Component
struct PropBigCase {
  @State
  showAlbum: boolean = false
  @State
  list: GoodItem[] = [
    {
      "id": 1,
      "goods_name": "班俏BANQIAO超火ins潮卫衣女士2020秋季新款韩版宽松慵懒风薄款外套带帽上衣",
      "goods_img": "assets/1.webp",
      "goods_price": 108,
      "goods_count": 1,
    },
    {
      "id": 2,
      "goods_name": "嘉叶希连帽卫衣女春秋薄款2020新款宽松bf韩版字母印花中长款外套ins潮",
      "goods_img": "assets/2.webp",
      "goods_price": 129,
      "goods_count": 1,
    },
    {
      "id": 3,
      "goods_name": "思蜜怡2020休闲运动套装女春秋季新款时尚大码宽松长袖卫衣两件套",
      "goods_img": "assets/3.webp",
      "goods_price": 198,
      "goods_count": 1,
    },
    {
      "id": 4,
      "goods_name": "思蜜怡卫衣女加绒加厚2020秋冬装新款韩版宽松上衣连帽中长款外套",
      "goods_img": "assets/4.webp",
      "goods_price": 99,
      "goods_count": 1,
    },
    {
      "id": 5,
      "goods_name": "幂凝早秋季卫衣女春秋装韩版宽松中长款假两件上衣薄款ins盐系外套潮",
      "goods_img": "assets/5.webp",
      "goods_price": 156,
      "goods_count": 1,
    },
    {
      "id": 6,
      "goods_name": "ME&CITY女装冬季新款针织抽绳休闲连帽卫衣女",
      "goods_img": "assets/6.webp",
      "goods_price": 142.8,
      "goods_count": 1,
    },
    {
      "id": 7,
      "goods_name": "幂凝假两件女士卫衣秋冬女装2020年新款韩版宽松春秋季薄款ins潮外套",
      "goods_img": "assets/7.webp",
      "goods_price": 219,
      "goods_count": 2,
    },
    {
      "id": 8,
      "goods_name": "依魅人2020休闲运动衣套装女秋季新款秋季韩版宽松卫衣 时尚两件套",
      "goods_img": "assets/8.webp",
      "goods_price": 178,
      "goods_count": 1,
    },
    {
      "id": 9,
      "goods_name": "芷臻(zhizhen)加厚卫衣2020春秋季女长袖韩版宽松短款加绒春秋装连帽开衫外套冬",
      "goods_img": "assets/9.webp",
      "goods_price": 128,
      "goods_count": 1,
    },
    {
      "id": 10,
      "goods_name": "Semir森马卫衣女冬装2019新款可爱甜美大撞色小清新连帽薄绒女士套头衫",
      "goods_img": "assets/10.webp",
      "goods_price": 153,
      "goods_count": 1,
    }
  ]

  build() {
     Column() {
       Button("选择图片")
         .onClick(() => {
           this.showAlbum = true
         })
       if(this.showAlbum) {
         PhotoAlbum({
           list: this.list,
           maxSelectNumber: 2,
           close: () => {
             this.showAlbum = false
           }
         })
       }
     }
    .width('100%')
    .height('100%')
    .padding(2)
  }
}

@Component
struct PhotoAlbum {
  @Prop
  maxSelectNumber: number = 9 // 设置可选择的图片的张数
  @Prop
  list: GoodItem[] = []
  close: () => void = () => {}
  build() {
    Column() {
      Grid() {
        // 数据的
        ForEach(this.list, (item: GoodItem) => {
          GridItem() {
            Image(item.goods_img)
              .aspectRatio(1)
              .onClick(() => {
                // 通过一个标记 能够知道当前的图片到底是选中还是没选中 如果选中 取消选中
                // 如果没选中 则选中 可以设置最多选择9张图片
              })
          }
        })
      }
      .columnsGap(2)
      .rowsGap(2)
      .columnsTemplate("1fr 1fr 1fr")
      .layoutWeight(1)
      Row() {
         Button("取消")
           .onClick(() => {
             this.close()
           })
           .backgroundColor(Color.Gray)
         Text(`可选${this.maxSelectNumber}张`)
        Button("完成")
      }
      .justifyContent(FlexAlign.SpaceBetween)
      .padding({
        left: 10,
        right: 10
      })
      .height(50)
      .width('100%')

    }
    .justifyContent(FlexAlign.SpaceBetween)
    .width('100%')
    .height('100%')
    .backgroundColor(Color.White)
    .position({
      x: 0,
      y: 0
    })
  }
}
```

![](https://cdn.nlark.com/yuque/0/2024/png/8435673/1709818801927-b455875b-e692-4cb9-9c20-818c82a22964.png)

- 相册选择

```typescript
import { GoodItem } from '../04/models'
import { promptAction } from '@kit.ArkUI'

@Entry
@Component
struct PropBigCase {
  @State
  showAlbum: boolean = false
  @State
  list: GoodItem[] = [
    {
      "id": 1,
      "goods_name": "班俏BANQIAO超火ins潮卫衣女士2020秋季新款韩版宽松慵懒风薄款外套带帽上衣",
      "goods_img": "assets/1.webp",
      "goods_price": 108,
      "goods_count": 1,
    },
    {
      "id": 2,
      "goods_name": "嘉叶希连帽卫衣女春秋薄款2020新款宽松bf韩版字母印花中长款外套ins潮",
      "goods_img": "assets/2.webp",
      "goods_price": 129,
      "goods_count": 1,
    },
    {
      "id": 3,
      "goods_name": "思蜜怡2020休闲运动套装女春秋季新款时尚大码宽松长袖卫衣两件套",
      "goods_img": "assets/3.webp",
      "goods_price": 198,
      "goods_count": 1,
    },
    {
      "id": 4,
      "goods_name": "思蜜怡卫衣女加绒加厚2020秋冬装新款韩版宽松上衣连帽中长款外套",
      "goods_img": "assets/4.webp",
      "goods_price": 99,
      "goods_count": 1,
    },
    {
      "id": 5,
      "goods_name": "幂凝早秋季卫衣女春秋装韩版宽松中长款假两件上衣薄款ins盐系外套潮",
      "goods_img": "assets/5.webp",
      "goods_price": 156,
      "goods_count": 1,
    },
    {
      "id": 6,
      "goods_name": "ME&CITY女装冬季新款针织抽绳休闲连帽卫衣女",
      "goods_img": "assets/6.webp",
      "goods_price": 142.8,
      "goods_count": 1,
    },
    {
      "id": 7,
      "goods_name": "幂凝假两件女士卫衣秋冬女装2020年新款韩版宽松春秋季薄款ins潮外套",
      "goods_img": "assets/7.webp",
      "goods_price": 219,
      "goods_count": 2,
    },
    {
      "id": 8,
      "goods_name": "依魅人2020休闲运动衣套装女秋季新款秋季韩版宽松卫衣 时尚两件套",
      "goods_img": "assets/8.webp",
      "goods_price": 178,
      "goods_count": 1,
    },
    {
      "id": 9,
      "goods_name": "芷臻(zhizhen)加厚卫衣2020春秋季女长袖韩版宽松短款加绒春秋装连帽开衫外套冬",
      "goods_img": "assets/9.webp",
      "goods_price": 128,
      "goods_count": 1,
    },
    {
      "id": 10,
      "goods_name": "Semir森马卫衣女冬装2019新款可爱甜美大撞色小清新连帽薄绒女士套头衫",
      "goods_img": "assets/10.webp",
      "goods_price": 153,
      "goods_count": 1,
    }
  ]

  build() {
     Column() {
       Button("选择图片")
         .onClick(() => {
           this.showAlbum = true
         })
       if(this.showAlbum) {
         PhotoAlbum({
           list: this.list,
           maxSelectNumber: 9,
           close: () => {
             this.showAlbum = false
           }
         })
       }
     }
    .width('100%')
    .height('100%')
    .padding(2)
  }
}

@Component
struct PhotoAlbum {
  @Prop
  maxSelectNumber: number = 9 // 设置可选择的图片的张数
  @Prop
  list: GoodItem[] = []
  @State
  selectPhotos: SelectPhoto[] = []
  close: () => void = () => {}

  // 用来选中或者取消选中图片
  selectImage (item: GoodItem) {
    // 通过一个标记 能够知道当前的图片到底是选中还是没选中 如果选中 取消选中
    // 如果没选中 则选中 可以设置最多选择9张图片
    const index = this.selectPhotos.findIndex(obj => obj.imgId === item.id)
    if(index > -1) {
      // 表示已经选择了 选中的化需要移除
      // 数组移除
      // 先找索引
      // 再通过吧splice进行移除
      this.selectPhotos.splice(index, 1) // 移除一个内容
      promptAction.showToast({ message: '执行移除' })
    }
    else {
      // 当选择张数小于最大张数时才可以继续
      if(this.selectPhotos.length < this.maxSelectNumber) {
        // 没有选中
        this.selectPhotos.push({ imgUrl: item.goods_img, imgId: item.id })
      }
    }
  }
  // 获取是否显示对号
  getShowSelect (item: GoodItem) {
   return  this.selectPhotos.findIndex(obj => obj.imgId === item.id) > -1
  }
  build() {
    Column() {
      Grid() {
        // 数据的
        ForEach(this.list, (item: GoodItem) => {
          GridItem() {
            Stack({ alignContent: Alignment.BottomEnd }) {
              Image(item.goods_img)
                .aspectRatio(1)

                if(this.getShowSelect(item)) {
                  // 需要展示对号
                  Image($r("app.media.select"))
                    .width(60)
                    .height(60)
                    .fillColor(Color.Orange)
                }
            }
            .onClick(() => {
              // 通过一个标记 能够知道当前的图片到底是选中还是没选中 如果选中 取消选中
              // 如果没选中 则选中 可以设置最多选择9张图片
              this.selectImage(item)
            })
          }
        })
      }
      .columnsGap(2)
      .rowsGap(2)
      .columnsTemplate("1fr 1fr 1fr")
      .layoutWeight(1)
      Row() {
         Button("取消")
           .onClick(() => {
             this.close()
           })
           .backgroundColor(Color.Gray)
         Text(`已选${this.selectPhotos.length}/ 可选${this.maxSelectNumber}张`)
        Button("完成")
      }
      .justifyContent(FlexAlign.SpaceBetween)
      .padding({
        left: 10,
        right: 10
      })
      .height(50)
      .width('100%')

    }
    .justifyContent(FlexAlign.SpaceBetween)
    .width('100%')
    .height('100%')
    .backgroundColor(Color.White)
    .position({
      x: 0,
      y: 0
    })
  }
}

interface SelectPhoto {
  imgUrl: string | ResourceColor
  imgId: number
}
```

![](https://cdn.nlark.com/yuque/0/2024/png/8435673/1709820323157-b6eacd69-cbd4-4e19-8f50-0395e1e0b6bf.png)

#### 新的诉求

- 希望点击图片-完成图片的预览 需要使用弹出层- 两种的使用方式- dialog - sheet
- <font style="color:#DF2A3F;">弹窗 UI 的第一种方式 CustomDialog</font>
- struct 这个结构体只能被 Component 和 CustomDialog 修饰
- 必须被 CustomDialog 修饰
- 组件中必须有一个属性 它的类型是 CustomDialogController,名字其实无所谓
- Component 的修饰符可以没有，有的话意味着它可以作为组件使用
