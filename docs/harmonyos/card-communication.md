# 卡片双向通信

## 项目前置准备

- [创建一个 ArkTS 卡片-ArkTS 卡片开发指导-开发基于 ArkTS UI 的卡片-服务卡片开发指导（Stage 模型）-Form Kit（卡片开发服务）-应用框架 - 华为 HarmonyOS 开发者](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-ui-widget-creation-V5) 文档指导
- 创建一个卡片项目
- <img src="https://cdn.nlark.com/yuque/0/2024/png/32778948/1729427337058-6f375700-8c08-4835-9fd8-74a363965ae0.png" alt="卡片项目" />

## 卡片双向通信实现

- 现在我们通过一个加减器来实现卡片的双向通信
- ![](https://cdn.nlark.com/yuque/0/2024/png/32778948/1729692035098-a2c96ebe-d862-4491-9425-db3b2f83c582.png)
- 卡片页面和应用页面的 ui 我们保持一致

```arkts
@Entry
@Component
struct Index {
  @State
  num: number = 0
  build() {
    Column() {
      Row({ space: 10 }) {
        Button("-")
          .onClick(() => {
            if (this.num) {
              this.num--
            }
          })
        Text(this.num.toString())
        Button("+")
          .onClick(() => {
            this.num++
          })
      }
      .justifyContent(FlexAlign.Center)
      .width("100%")
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
  }
}
```

### 创建卡片时应用数据同步到卡片

- 创建卡片时会触发卡片生命周期`onAddForm`函数
- 卡片框架会通过 want 传给我们一个创建好的 formId 字符串
- 此时我们在生命周期接收到 formId 返回给卡片页面

```arkts
import { formBindingData, FormExtensionAbility, formInfo } from '@kit.FormKit';
import { Want } from '@kit.AbilityKit';

export default class EntryFormAbility extends FormExtensionAbility {
  onAddForm(want: Want) {
    // Called to return a FormBindingData object.
    // 添加卡片事件中，获取卡片id推送到卡片半身
    return formBindingData.createFormBindingData({
      formId: want.parameters!["ohos.extra.param.key.form_identity"] as string
    });
  }
}
```

- 接着我们到卡片页面去接收 formId
- 此时 formId 监听到变化，触发 updateFormId 事件
- 在 updateFormId 事件中调用`postCardAction`方法 备注：`postCardAction`方法<font style="color:rgba(0, 0, 0, 0.9);">能够快速拉起卡片提供方应用的指定 UIAbility，从而将数据从卡片==>应用</font>

![](https://cdn.nlark.com/yuque/0/2024/png/32778948/1729429615872-f4f9d781-d1d6-4ce4-9951-4e9cb4366a79.png)

```arkts
@Entry
@Component
struct Count {
  @LocalStorageProp("formId") // 接收formId
  @Watch("updateFormId") //formId触发监听事件
  formId: string = ""

  @LocalStorageProp("num")
  num: number = 0

  updateFormId() {
     // 卡片 => 应用
    postCardAction(this, {
      action: 'call',
      abilityName: 'EntryAbility', // 只能跳转到当前应用下的UIAbility
      params: {
        // 使用call方式 需要第二个参数
        method: 'updateFormId', // 名字必须叫method method的值是在ability中监听的方法名
        formId: this.formId
      }
    })
  }
    .....
}
```

- <font style="color:rgb(38, 38, 38);">请注意，如果使用 call 方式传递调用，需要开启一个后台权限-保持应用在后台</font>

```arkts
"requestPermissions": [{
    "name": "ohos.permission.KEEP_BACKGROUND_RUNNING"
  }],
```

- 此时在应用卡片中设置 call 函数，可以拿到 formId 数据

```arkts
// 设置返回类型
class Params implements rpc.Parcelable {
  marshalling(messageSequence: rpc.MessageSequence): boolean {
    return true;
  }
  unmarshalling(messageSequence: rpc.MessageSequence): boolean {
    return true;
  }
}
class CardParams {
  num: number = 0
  formId: string = ""
}
this.callee.on("updateFormId", (data) => {
    const res = JSON.parse(data.readString()) as CardParams
     ....
    return new Params() // 只是为了不报错
  })
```

- 此时接收到 formId 我们需要存入首选项中，因为需要存储多个 formId 来推送给不同的卡片, 所以我们可以封装一个类去管理首选项

```arkts
import { preferences } from '@kit.ArkData'
import { Context } from '@kit.AbilityKit'

export default class FormIdManager {
  static context: Context

  // 获取store
  static getStore() {
    return preferences.getPreferencesSync(FormIdManager.context || getContext(), {
      name: "num_formId" // 文件名
    })
  }

  // 存储卡片ID
  static async addFormId(formId: string) {
    const store = FormIdManager.getStore()
    const list = FormIdManager.getFormIdList()
    if (!list.some(id => id === formId)) {
      list.push(formId)
    }
    store.putSync("num_formId_key", JSON.stringify(list))
    await store.flush()
  }

  // 获取卡片ID数组
  static getFormIdList() {
    const store = FormIdManager.getStore()
    return JSON.parse(store.getSync("num_formId_key", '[]') as string) as string[]
  }

  // 删除卡片ID
  static async delFormId(formId: string) {
    const store = FormIdManager.getStore()
    const list = FormIdManager.getFormIdList()
    const index = list.findIndex(id => id === formId)
    list.splice(index, 1)
    store.putSync("num_formId_key", JSON.stringify(list))
    await store.flush()
  }
}
```

- 设置完毕后，在 onCreate 中调用

```arkts
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    this.callee.on("updateFormId", (data) => {
      const res = JSON.parse(data.readString()) as CardParams
      FormIdManager.addFormId(res.formId) // 添加formId到首选项中
      // 添加卡片的时候就把数据推送过去
      formProvider.updateForm(res.formId, formBindingData.createFormBindingData({
        num: AppStorage.get("num") // 取应用中的num数据
      }))
      return new Params() // 只是为了不报错
    })
  }
```

- 此时点击添加卡片，卡片中的数据和应用的 num 保持一致
- ![](https://cdn.nlark.com/yuque/0/2024/png/32778948/1729436471381-530bdc06-d8b6-4e58-a772-f4be69bed16c.png)

### 卡片改变数据触发应用刷新

- 在卡片的 onClick 事件中点击加减触发 postCardAction

```arkts
Button("-")
  .onClick(() => {
    if (this.num) {
      this.num--
    }
    postCardAction(this, {
      action: 'call',
      abilityName: 'EntryAbility', // 只能跳转到当前应用下的UIAbility
      params: {
        // 使用call方式 需要第二个参数
        method: 'updateNum', // 名字必须叫method method的值是在ability中监听的方法名
        num: this.num
      }
    })
  })
```

- 在 onCreate 方法接收数据并存储到全局数据, 接着在应用页面中接收
- 将@state 改成 @StorageLink("num")

```arkts
onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
  this.callee.on("updateNum", (data) => {
    const res = JSON.parse(data.readString()) as CardParams // json
    AppStorage.setOrCreate("num", res.num)
    return new Params()
  })
  this.callee.on("updateFormId", (data) => {
      ....
  })
}

@Entry
@Component
struct Index {
  @StorageLink("num")
  @Watch("pushCard")
  num: number = 0
  ....
}
```

### 应用改变数据触发卡片刷新

- 在应用页面中监听 num 数据的变换，当数据变化时触发更新卡片函数

```arkts
@Entry
@Component
struct Index {
  @StorageLink("num")
  @Watch("pushCard")
  num: number = 0

  // 推送卡片数据
  pushCard() {
    const store = preferences.getPreferencesSync(getContext(), {
      name: 'formIdList'
    })
    const list = FormIdManager.getFormIdList()
    // 卡片id的列表
    list.forEach(formId => {
      formProvider.updateForm(formId, formBindingData.createFormBindingData({
        num: this.num
      }))
    })
  }
  ....
}
```

- 卡片接收,num 设置成 ` @LocalStorageProp("num")`

```arkts
@Entry
@Component
struct Count {
  @LocalStorageProp("num") // 直接从推送的数据中获取
  num: number = 0
    ....
}
```

## 源代码

- 仓库地址：[card: 卡片通信以及 logger 输出](https://gitee.com/geekwaner/card)
- 应用页面代码

```arkts
import { preferences } from '@kit.ArkData'
import { formBindingData, formProvider } from '@kit.FormKit'
import FormIdManager from '../utils/FormIdManager'

@Entry
@Component
struct Index {
  @StorageLink("num")
  @Watch("pushCard")
  num: number = 0

  // 推送卡片数据
  pushCard() {
    const store = preferences.getPreferencesSync(getContext(), {
      name: 'formIdList'
    })
    const list = FormIdManager.getFormIdList()
    // 卡片id的列表
    list.forEach(formId => {
      formProvider.updateForm(formId, formBindingData.createFormBindingData({
        num: this.num
      }))
    })
  }

  build() {
    Column() {
      Row({ space: 10 }) {
        Button("-")
          .onClick(() => {
            if (this.num) {
              this.num--
            }
          })
        Text(this.num.toString())
        Button("+")
          .onClick(() => {
            this.num++
          })
      }
      .justifyContent(FlexAlign.Center)
      .width("100%")
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
  }
}
```

- 卡片页面代码

```arkts
@Entry
@Component
struct Count {
  @LocalStorageProp("num") // 直接从推送的数据中获取
  num: number = 0
  @LocalStorageProp("formId")
  @Watch("updateFormId")
  formId: string = ""

  // 说明拿到id
  updateFormId() {
    console.log('formId')
    // 应用 => 卡片
    postCardAction(this, {
      action: 'call',
      abilityName: 'EntryAbility', // 只能跳转到当前应用下的UIAbility
      params: {
        // 使用call方式 需要第二个参数
        method: 'updateFormId', // 名字必须叫method method的值是在ability中监听的方法名
        formId: this.formId
      }
    })
  }

  build() {
    Row({ space: 10 }) {
      Button("-")
        .onClick(() => {
          if (this.num) {
            this.num--
          }
          postCardAction(this, {
            action: 'call',
            abilityName: 'EntryAbility', // 只能跳转到当前应用下的UIAbility
            params: {
              // 使用call方式 需要第二个参数
              method: 'updateNum', // 名字必须叫method method的值是在ability中监听的方法名
              num: this.num
            }
          })
        })
      Text(this.num.toString())
        .fontSize(20)
      Button("+")
        .onClick(() => {
          this.num++
          postCardAction(this, {
            action: 'call',
            abilityName: 'EntryAbility', // 只能跳转到当前应用下的UIAbility
            params: {
              method: 'updateNum', // 名字必须叫method method的值是在ability中监听的方法名
              num: this.num
            }
          })
        })
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
    .alignItems(VerticalAlign.Center)
    .onClick(() => {
      postCardAction(this, {
        action: 'router',
        abilityName: 'EntryAbility'
      })
    })
  }
}
```

- EntryAbility.ets 代码

```arkts
import { AbilityConstant, UIAbility, Want } from '@kit.AbilityKit';
import { hilog } from '@kit.PerformanceAnalysisKit';
import { window } from '@kit.ArkUI';
import rpc from '@ohos.rpc';
import { preferences } from '@kit.ArkData';
import { formBindingData, formProvider } from '@kit.FormKit';
import { Logger } from '../utils/Logger';
import FormIdManager from '../utils/FormIdManager';

class Params implements rpc.Parcelable {
  marshalling(messageSequence: rpc.MessageSequence): boolean {
    return true;
  }

  unmarshalling(messageSequence: rpc.MessageSequence): boolean {
    return true;
  }
}

class CardParams {
  num: number = 0
  formId: string = ""
}

export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    this.callee.on("updateNum", (data) => {
      const res = JSON.parse(data.readString()) as CardParams // json
      AppStorage.setOrCreate("num", res.num)
      return new Params()
    })
    this.callee.on("updateFormId", (data) => {
      const res = JSON.parse(data.readString()) as CardParams
      FormIdManager.addFormId(res.formId) // 添加formId到首选项中
      // 添加卡片的时候就把数据推送过去
      formProvider.updateForm(res.formId, formBindingData.createFormBindingData({
        num: AppStorage.get("num") // 取应用中的num数据
      }))
      return new Params() // 只是为了不报错
    })
  }

  onDestroy(): void {
    this.callee.off("updateNum")
    hilog.info(0x0000, 'testTag', '%{public}s', 'Ability onDestroy');
  }

  onWindowStageCreate(windowStage: window.WindowStage): void {
    // Main window is created, set main page for this ability
    hilog.info(0x0000, 'testTag', '%{public}s', 'Ability onWindowStageCreate');

    windowStage.loadContent('pages/Index', (err) => {
      if (err.code) {
        hilog.error(0x0000, 'testTag', 'Failed to load the content. Cause: %{public}s', JSON.stringify(err) ?? '');
        return;
      }
      hilog.info(0x0000, 'testTag', 'Succeeded in loading the content.');
    });
  }
}
```
