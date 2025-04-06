# 鸿蒙开发之存储数据

本文介绍鸿蒙开发中如何进行数据存储的相关操作，包括轻量级存储、首选项和分布式数据库等内容。

## 目录

- [轻量级存储](#轻量级存储)
- [首选项](#首选项)
- [分布式数据库](#分布式数据库)

## 轻量级存储

鸿蒙系统提供了轻量级的数据存储机制，适合存储少量的简单数据。

```typescript
import storage from "@ohos.data.storage";
import fs from "@ohos.file.fs";

// 使用示例
async function storageExample() {
  // 获取应用上下文
  const context = getContext(this);

  // 获取存储路径
  const DIR = context.filesDir;

  // 创建storage实例
  const storage = await storage.getStorage(DIR + "/mystore");

  // 存储数据
  storage.put("name", "HarmonyOS");
  storage.put("version", "3.0");

  // 将数据从内存中持久化到文件中
  await storage.flush();

  // 读取数据
  const name = storage.get("name", "default");
  const version = storage.get("version", "1.0");

  console.log(`Name: ${name}, Version: ${version}`);

  // 删除数据
  storage.delete("version");

  // 清除存储
  storage.clear();
}
```

## 首选项

首选项（Preferences）是鸿蒙提供的轻量级存储，可用于存储应用的配置信息。

```typescript
import preferences from "@ohos.data.preferences";

// 使用示例
async function preferencesExample() {
  // 获取应用上下文
  const context = getContext(this);

  // 创建preferences实例
  const prefs = await preferences.getPreferences(context, "myPrefs");

  // 存储各种类型的数据
  await prefs.put("username", "geekwaner");
  await prefs.put("isLogged", true);
  await prefs.put("loginCount", 10);

  // 持久化数据
  await prefs.flush();

  // 读取数据
  const username = await prefs.get("username", "guest");
  const isLogged = await prefs.get("isLogged", false);
  const loginCount = await prefs.get("loginCount", 0);

  console.log(`User: ${username}, Logged: ${isLogged}, Count: ${loginCount}`);

  // 删除数据
  await prefs.delete("loginCount");

  // 清除所有数据
  await prefs.clear();
}
```

## 分布式数据库

鸿蒙的分布式数据库（KVStore）提供了设备间数据共享与同步的能力。

```typescript
import kvStore from "@ohos.data.distributedKVStore";

// 使用示例
async function distributedDBExample() {
  // 创建KV管理器配置
  const kvConfig = {
    bundleName: "com.example.myapp",
    userInfo: {
      userId: "0",
      userType: kvStore.UserType.SAME_USER_ID,
    },
  };

  // 创建KV管理器
  const kvManager = await kvStore.createKVManager(kvConfig);

  // 创建KV存储配置
  const storeConfig = {
    storeId: "myStore",
    securityLevel: kvStore.SecurityLevel.S1,
    syncable: true, // 支持分布式同步
    encrypt: false,
  };

  // 获取KV存储
  const store = await kvManager.getKVStore(storeConfig);

  // 存储数据
  await store.put("key1", "value1");
  await store.put("key2", 123);

  // 读取数据
  const entry1 = await store.get("key1");
  const entry2 = await store.get("key2");

  console.log(`Key1: ${entry1.value}, Key2: ${entry2.value}`);

  // 订阅数据变化
  store.on("dataChange", 1, (data) => {
    console.log("Data changed:", data);
  });

  // 删除数据
  await store.delete("key1");
}
```

## 实际应用场景

在实际应用中，可以根据不同场景选择合适的存储方式：

1. 轻量级存储：适合存储应用内部简单配置
2. 首选项：适合存储用户设置、应用配置
3. 分布式数据库：适合需要跨设备同步的场景，如多设备信息共享

```typescript
// 保存用户偏好设置示例
async function saveUserPreferences(theme: string, fontSize: number) {
  const context = getContext(this);
  const prefs = await preferences.getPreferences(context, "userSettings");

  await prefs.put("theme", theme);
  await prefs.put("fontSize", fontSize);
  await prefs.flush();

  console.log("用户设置已保存");
}

// 加载用户偏好设置
async function loadUserPreferences() {
  const context = getContext(this);
  const prefs = await preferences.getPreferences(context, "userSettings");

  const theme = await prefs.get("theme", "light");
  const fontSize = await prefs.get("fontSize", 16);

  return { theme, fontSize };
}
```
