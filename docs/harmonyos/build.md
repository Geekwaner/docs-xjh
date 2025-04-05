**官网推荐**： [官方编译构建文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-build-V5)  

### 一、Release与Debug模式核心差异
#### 1.1 编译特性对比
| 特性 | Debug模式 | Release模式 |
| --- | --- | --- |
| 代码优化 | -O0（无优化） | -O3（最高优化） |
| 符号表 | 完整调试符号 | 剥离符号表 |
| 断言检查 | 启用 | 禁用 |
| 日志级别 | DEBUG及以上 | WARN及以上 |
| 资源压缩 | 保留原始资源 | 启用ProGuard资源混淆 |
| 性能特征 | 低性能（约60%） | 高性能（100%） |
| 热修复支持 | 支持 | 受限 |


---

### 二、useNormalizedOHMUrl属性解析
#### 属性作用机制
+ 开启严格模式之后，<font style="color:rgb(104, 153, 50);background-color:rgb(241, 243, 245);">byteCodeHar </font>默认会为true , 此时编译出来的包里面为字解码文件，此时无法看到里面的代码，通常项目创建会默认带上该`"useNormalizedOHMUrl": true`属性
+ ![](https://cdn.nlark.com/yuque/0/2025/png/32778948/1742998575843-67036916-9909-4893-8773-978afd489ce6.png)![](https://cdn.nlark.com/yuque/0/2025/png/32778948/1743000095286-059ecc29-78ef-4414-a540-9d3040bdd1c0.png)		

```javascript
// build-profile.json
"buildOption": {
  "strictMode": {
    "caseSensitiveCheck": true,
    "useNormalizedOHMUrl": true  // 设置严格模式
  }
}
```

+ 如果是在集成的har包中，不想包代码编译成字节码，
    - hap包中<font style="color:rgb(36, 39, 40);">build-profile.json5 中，关闭严格模式，设置"useNormalizedOHMUrl": false</font>
    - 此时可<font style="color:rgb(36, 39, 40);">在HAR模块的build-profile.json5中，将byteCodeHar设置为true。</font>

```javascript
{
  "buildOption": {
    "arkOptions": {
      "byteCodeHar": true
    }
  }
}
```

#### 编译处理差异：
| 状态 | 资源映射策略 | 哈希处理 |
| --- | --- | --- |
| true | 生成全局资源ID（如0x1000001） | 启用文件名哈希 |
| false | 保留原始路径（如"img/logo"） | 仅大小写规范化 |


---

### 三、Release模式下的ImageSource陷阱
#### 问题现象
+ release包有可能会引起获取 资源文件内容内容失败
+ 相关概念
+ 通常我们在编译sdk包的时候会有几种编译模式，分为release和debug包，两种编译方式会造成包内容的一些区别，release会严格遵循配置文件，当前sdk配置文件中配置了混淆文件，因此会release里的文件会被混淆
+ 而debug包则不会混淆，无论是否配置混淆，源文件是什么样的打出来的包就是什么样的，而混淆的包会将代码里的资源变成id=-1带.d就表示文件被混淆了

![](https://cdn.nlark.com/yuque/0/2025/png/32778948/1743001540842-c7d86aaf-50b0-4ae7-965d-56d7aead5d94.png)

#### 影响以及解决方案
+ 会造成release包不能使用id去获取imageSource,如果获取到Resource文件后，调用getMediaContent接口，传入i值，此时会传入-1，自然获取不到资源内容，进而会影响功能

```arkts
// 通过传递资源id来
const res = ResourceUtil.iconMap.get(src) as Resource;
const resourceMgr = context.resourceManager;
const imageBuffer = await resourceMgr.getMediaContent(res.id);    
```

+ 使用getMediaByNameSync 接口，通过传入name名称来获取资源内容

```arkts
const res = ResourceUtil.iconMap.get(src) as Resource;
const ResourceName = res.params?.[0]?.split('.')[2] as string; // ResourceName ic_copy_abilitykitsdk
const resourceMgr = context.resourceManager;
const imageBuffer = resourceMgr.getMediaByNameSync(ResourceName);    
```



