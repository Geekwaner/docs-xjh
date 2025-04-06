# 鸿蒙开发之简易操作沙箱文件

在鸿蒙应用开发中，文件操作是很常见的需求，但需要注意鸿蒙系统采用了沙箱机制，应用只能在自己的沙箱目录中进行文件操作。本文介绍如何在鸿蒙系统中进行基本的文件操作。

## 目录

- [文件系统基础](#文件系统基础)
- [基本文件操作](#基本文件操作)
- [读写文本文件](#读写文本文件)
- [读写二进制文件](#读写二进制文件)
- [完整示例](#完整示例)

## 文件系统基础

鸿蒙系统提供了 `fs` 模块用于文件系统操作，应用可以在沙箱目录内进行文件和目录的创建、读取、写入、删除等操作。

### 常用沙箱目录

```typescript
import fs from "@ohos.file.fs";
import common from "@ohos.app.ability.common";

// 获取上下文
const context = getContext(this) as common.UIAbilityContext;

// 沙箱目录类型
const sandboxDirs = {
  // 应用文件目录，用于存储应用数据文件
  filesDir: context.filesDir,

  // 应用缓存目录，用于存储临时文件
  cacheDir: context.cacheDir,

  // 应用偏好目录，用于存储少量的结构化数据
  preferencesDir: context.preferencesDir,

  // 应用临时目录，应用退出后可能被清除
  tempDir: context.tempDir,

  // 应用数据库目录，用于存储数据库文件
  databaseDir: context.databaseDir,
};

// 打印各目录路径
console.info(`Files Dir: ${sandboxDirs.filesDir}`);
console.info(`Cache Dir: ${sandboxDirs.cacheDir}`);
console.info(`Preferences Dir: ${sandboxDirs.preferencesDir}`);
console.info(`Temp Dir: ${sandboxDirs.tempDir}`);
console.info(`Database Dir: ${sandboxDirs.databaseDir}`);
```

## 基本文件操作

以下是常见的文件操作示例：

### 创建目录

```typescript
async function createDirectory(dirPath: string): Promise<void> {
  try {
    // 检查目录是否存在
    const isExist = await fs.access(dirPath);
    if (isExist) {
      console.info(`Directory already exists: ${dirPath}`);
      return;
    }
  } catch (err) {
    // 目录不存在，创建目录
    try {
      await fs.mkdir(dirPath);
      console.info(`Directory created: ${dirPath}`);
    } catch (mkdirErr) {
      console.error(`Failed to create directory: ${mkdirErr.message}`);
      throw mkdirErr;
    }
  }
}
```

### 列出目录内容

```typescript
async function listDirectory(dirPath: string): Promise<string[]> {
  try {
    const files = await fs.listFile(dirPath);
    console.info(`Files in directory ${dirPath}: ${files.join(", ")}`);
    return files;
  } catch (err) {
    console.error(`Failed to list directory: ${err.message}`);
    throw err;
  }
}
```

### 检查文件/目录是否存在

```typescript
async function checkFileExistence(path: string): Promise<boolean> {
  try {
    await fs.access(path);
    console.info(`Path exists: ${path}`);
    return true;
  } catch (err) {
    console.info(`Path does not exist: ${path}`);
    return false;
  }
}
```

### 获取文件信息

```typescript
async function getFileInfo(filePath: string): Promise<fs.Stat> {
  try {
    const fileInfo = await fs.stat(filePath);
    console.info(`File info for ${filePath}:`);
    console.info(`Size: ${fileInfo.size} bytes`);
    console.info(`Last modified: ${new Date(fileInfo.mtime).toLocaleString()}`);
    console.info(`Is directory: ${fileInfo.isDirectory()}`);
    console.info(`Is file: ${fileInfo.isFile()}`);
    return fileInfo;
  } catch (err) {
    console.error(`Failed to get file info: ${err.message}`);
    throw err;
  }
}
```

### 删除文件

```typescript
async function deleteFile(filePath: string): Promise<void> {
  try {
    await fs.unlink(filePath);
    console.info(`File deleted: ${filePath}`);
  } catch (err) {
    console.error(`Failed to delete file: ${err.message}`);
    throw err;
  }
}
```

### 删除目录

```typescript
async function deleteDirectory(
  dirPath: string,
  recursive: boolean = false
): Promise<void> {
  try {
    if (recursive) {
      await fs.rmdir(dirPath, { recursive: true });
    } else {
      await fs.rmdir(dirPath);
    }
    console.info(`Directory deleted: ${dirPath}`);
  } catch (err) {
    console.error(`Failed to delete directory: ${err.message}`);
    throw err;
  }
}
```

## 读写文本文件

### 写入文本文件

```typescript
async function writeTextFile(filePath: string, content: string): Promise<void> {
  try {
    // 打开文件，如果不存在则创建
    const file = await fs.open(
      filePath,
      fs.OpenMode.CREATE | fs.OpenMode.WRITE
    );

    // 写入内容
    await fs.write(file.fd, content);

    // 关闭文件
    await fs.close(file.fd);

    console.info(`Content written to file: ${filePath}`);
  } catch (err) {
    console.error(`Failed to write to file: ${err.message}`);
    throw err;
  }
}
```

### 读取文本文件

```typescript
async function readTextFile(filePath: string): Promise<string> {
  try {
    // 打开文件
    const file = await fs.open(filePath, fs.OpenMode.READ_ONLY);

    // 获取文件信息
    const fileInfo = await fs.stat(filePath);

    // 创建缓冲区
    const buffer = new ArrayBuffer(fileInfo.size);

    // 读取内容
    const readLen = await fs.read(file.fd, buffer);

    // 关闭文件
    await fs.close(file.fd);

    // 转换为字符串
    const decoder = new TextDecoder();
    const content = decoder.decode(buffer);

    console.info(`Content read from file: ${filePath}`);
    console.info(`Read ${readLen} bytes`);

    return content;
  } catch (err) {
    console.error(`Failed to read file: ${err.message}`);
    throw err;
  }
}
```

## 读写二进制文件

### 写入二进制文件

```typescript
async function writeBinaryFile(
  filePath: string,
  data: ArrayBuffer
): Promise<void> {
  try {
    // 打开文件，如果不存在则创建
    const file = await fs.open(
      filePath,
      fs.OpenMode.CREATE | fs.OpenMode.WRITE
    );

    // 写入数据
    await fs.write(file.fd, data);

    // 关闭文件
    await fs.close(file.fd);

    console.info(`Binary data written to file: ${filePath}`);
  } catch (err) {
    console.error(`Failed to write binary data: ${err.message}`);
    throw err;
  }
}
```

### 读取二进制文件

```typescript
async function readBinaryFile(filePath: string): Promise<ArrayBuffer> {
  try {
    // 打开文件
    const file = await fs.open(filePath, fs.OpenMode.READ_ONLY);

    // 获取文件信息
    const fileInfo = await fs.stat(filePath);

    // 创建缓冲区
    const buffer = new ArrayBuffer(fileInfo.size);

    // 读取内容
    const readLen = await fs.read(file.fd, buffer);

    // 关闭文件
    await fs.close(file.fd);

    console.info(`Binary data read from file: ${filePath}`);
    console.info(`Read ${readLen} bytes`);

    return buffer;
  } catch (err) {
    console.error(`Failed to read binary file: ${err.message}`);
    throw err;
  }
}
```

## 完整示例

下面是一个完整的文件操作示例：

```typescript
import fs from '@ohos.file.fs';
import common from '@ohos.app.ability.common';

@Entry
@Component
struct FileOperationDemo {
  private context: common.UIAbilityContext = getContext(this) as common.UIAbilityContext;
  @State fileContent: string = '';
  @State directoryList: string[] = [];
  @State operationResult: string = '';

  build() {
    Column({ space: 20 }) {
      Text('文件操作示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Scroll() {
        Column({ space: 10 }) {
          Button('创建目录')
            .onClick(() => {
              this.createTestDirectory();
            })

          Button('写入文本文件')
            .onClick(() => {
              this.writeTestFile();
            })

          Button('读取文本文件')
            .onClick(() => {
              this.readTestFile();
            })

          Button('列出目录内容')
            .onClick(() => {
              this.listTestDirectory();
            })

          Button('删除文件')
            .onClick(() => {
              this.deleteTestFile();
            })

          Button('删除目录')
            .onClick(() => {
              this.deleteTestDirectory();
            })

          if (this.fileContent) {
            Text('文件内容:')
              .fontSize(16)
              .fontWeight(FontWeight.Bold)

            Text(this.fileContent)
              .fontSize(14)
              .backgroundColor('#f0f0f0')
              .padding(10)
              .width('100%')
          }

          if (this.directoryList.length > 0) {
            Text('目录内容:')
              .fontSize(16)
              .fontWeight(FontWeight.Bold)

            ForEach(this.directoryList, (item) => {
              Text(item)
                .fontSize(14)
            })
          }

          if (this.operationResult) {
            Text('操作结果:')
              .fontSize(16)
              .fontWeight(FontWeight.Bold)

            Text(this.operationResult)
              .fontSize(14)
              .fontColor(this.operationResult.includes('Error') ? Color.Red : Color.Green)
          }
        }
        .width('100%')
        .padding(16)
      }
      .width('100%')
      .height('80%')
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }

  async createTestDirectory() {
    try {
      const dir = `${this.context.filesDir}/testDir`;
      await this.createDirectory(dir);
      this.operationResult = `目录创建成功: ${dir}`;
    } catch (err) {
      this.operationResult = `Error: 创建目录失败 - ${err.message}`;
    }
  }

  async writeTestFile() {
    try {
      const filePath = `${this.context.filesDir}/testDir/test.txt`;
      const content = `这是一个测试文件\n创建时间: ${new Date().toLocaleString()}`;
      await this.writeTextFile(filePath, content);
      this.operationResult = `文件写入成功: ${filePath}`;
    } catch (err) {
      this.operationResult = `Error: 写入文件失败 - ${err.message}`;
    }
  }

  async readTestFile() {
    try {
      const filePath = `${this.context.filesDir}/testDir/test.txt`;
      this.fileContent = await this.readTextFile(filePath);
      this.operationResult = `文件读取成功: ${filePath}`;
    } catch (err) {
      this.operationResult = `Error: 读取文件失败 - ${err.message}`;
      this.fileContent = '';
    }
  }

  async listTestDirectory() {
    try {
      const dir = `${this.context.filesDir}/testDir`;
      this.directoryList = await this.listDirectory(dir);
      this.operationResult = `目录列表读取成功: ${dir}`;
    } catch (err) {
      this.operationResult = `Error: 列出目录内容失败 - ${err.message}`;
      this.directoryList = [];
    }
  }

  async deleteTestFile() {
    try {
      const filePath = `${this.context.filesDir}/testDir/test.txt`;
      await this.deleteFile(filePath);
      this.operationResult = `文件删除成功: ${filePath}`;
      this.fileContent = '';
    } catch (err) {
      this.operationResult = `Error: 删除文件失败 - ${err.message}`;
    }
  }

  async deleteTestDirectory() {
    try {
      const dir = `${this.context.filesDir}/testDir`;
      await this.deleteDirectory(dir, true);
      this.operationResult = `目录删除成功: ${dir}`;
      this.directoryList = [];
    } catch (err) {
      this.operationResult = `Error: 删除目录失败 - ${err.message}`;
    }
  }

  // 创建目录
  async createDirectory(dirPath: string): Promise<void> {
    try {
      try {
        await fs.access(dirPath);
        console.info(`Directory already exists: ${dirPath}`);
      } catch (err) {
        await fs.mkdir(dirPath);
        console.info(`Directory created: ${dirPath}`);
      }
    } catch (err) {
      console.error(`Failed to create directory: ${err.message}`);
      throw err;
    }
  }

  // 列出目录内容
  async listDirectory(dirPath: string): Promise<string[]> {
    try {
      const files = await fs.listFile(dirPath);
      console.info(`Files in directory ${dirPath}: ${files.join(', ')}`);
      return files;
    } catch (err) {
      console.error(`Failed to list directory: ${err.message}`);
      throw err;
    }
  }

  // 写入文本文件
  async writeTextFile(filePath: string, content: string): Promise<void> {
    try {
      const file = await fs.open(filePath, fs.OpenMode.CREATE | fs.OpenMode.WRITE);
      await fs.write(file.fd, content);
      await fs.close(file.fd);
      console.info(`Content written to file: ${filePath}`);
    } catch (err) {
      console.error(`Failed to write to file: ${err.message}`);
      throw err;
    }
  }

  // 读取文本文件
  async readTextFile(filePath: string): Promise<string> {
    try {
      const file = await fs.open(filePath, fs.OpenMode.READ_ONLY);
      const fileInfo = await fs.stat(filePath);
      const buffer = new ArrayBuffer(fileInfo.size);
      await fs.read(file.fd, buffer);
      await fs.close(file.fd);
      const decoder = new TextDecoder();
      const content = decoder.decode(buffer);
      console.info(`Content read from file: ${filePath}`);
      return content;
    } catch (err) {
      console.error(`Failed to read file: ${err.message}`);
      throw err;
    }
  }

  // 删除文件
  async deleteFile(filePath: string): Promise<void> {
    try {
      await fs.unlink(filePath);
      console.info(`File deleted: ${filePath}`);
    } catch (err) {
      console.error(`Failed to delete file: ${err.message}`);
      throw err;
    }
  }

  // 删除目录
  async deleteDirectory(dirPath: string, recursive: boolean = false): Promise<void> {
    try {
      if (recursive) {
        await fs.rmdir(dirPath, { recursive: true });
      } else {
        await fs.rmdir(dirPath);
      }
      console.info(`Directory deleted: ${dirPath}`);
    } catch (err) {
      console.error(`Failed to delete directory: ${err.message}`);
      throw err;
    }
  }
}
```
