# 自动打包
![image](http://upload-images.jianshu.io/upload_images/1253942-f218f61e611122fb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

## 应用构建

### 构建过程
- 配置（编译器确定当前系统环境）
- 确定标准库和头文件的位置
- 确定依赖关系 
- 头文件预编译(precompilation) 
- 预处理(preprocessing) 
- 编译(compilation) 
- 连接(Linking) 
- 打包

### 包的组成
- 一个`Mach-O`格式的二进制可执行文件(签名的数据就在这个二进制文件中)
- 一些资源文件。

### Mach-O可执行文件

Mach是一种操作系统内核。在Mach上，一种可执行的文件格是就是`Mach-O（Mach Object file format）`。iOS是从OS X演变而来，所以同样支持Mach-O格式的可执行文件。

`ipa`包实际上就是一个`zip`压缩包，解压之后会有一个`Payload`文件夹，其中有个`XXX.app`文件，它里面除了有个各种资源、图片等，有个和包名相同的文件——就是`二进制可执行文件`。
`file`命令查看文件类型：
```bash
file iXiao
iXiao: Mach-O universal binary with 2 architectures: [arm_v7:Mach-O executable arm_v7] [arm64]
iXiao (for architecture armv7):	Mach-O executable arm_v7
iXiao (for architecture arm64):	Mach-O 64-bit executable arm64
```
从上面看是支持arm7和arm64两种处理器架构的通用程序包，里面的格式是Mach-O。用Sublime打开部分如下：
```
cafe babe 0000 0002 0000 000c 0000 0009
0000 4000 0109 2df0 0000 000e 0100 000c
0000 0000 0109 8000 012a 4970 0000 000e
0000 0000 0000 0000 0000 0000 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000
``` 

开头的4个字节是`cafebabe`，这被称为“魔数”，反映文件的类型。
OS X上几个标识：

- `cafebabe`:就是跨处理器架构的通用格式
- `feedface`和`feedfacf`则分别是某一处理器架构下的Mach-O格式
- 脚本的就很常见了，比如`#!/bin/bash`开头的shell脚本。

### CodeResources文件

为了达到为所有文件设置签名，签名的过程中会在程序包中新建一个叫做 `_CodeSignatue/CodeResources` 的`plist`文件，文件中存储了被签名的程序包中所有文件的签名。

- 文件中不光包含了文件和它们的签名的列表、一系列规则，这些规则决定了哪些资源文件应当被设置签名。
- 10.9.5之后在 `CodeResources` 文件中会有4个不同区域，
  - `rules` 和 `files` 是老版本准备的，
  - `files2` 和 `rules2`是新版的代码签名准备的

  - 新版本中所有的代码文件和资源文件都必须设置签名。
  - 老版本程序包添加一个名为 `ResourceRules.plist` 的文件，在检查时被忽略。

更详细的介绍参见[《代码签名探析》](http://objccn.io/issue-17-2/)

## 打包

- 苹果自带的`xcodebuild`命令行工具
- [xctool](https://github.com/facebook/xctool)
  1. 相比较xcodebuild输出的log杂乱，xctool更有结构
  2. xctool有人性化的颜色输出
  3. facebook声称xctool更快，据说能快2、3倍
  4. 完全用Ojbective-C实现

### xctool

xctool是可以使用[homebrew](http://brew.sh/)安装的，或者下[源码](https://github.com/facebook/xctool)然后运行 `xctool.sh`脚本，homebrew安装命令如下：

```
brew install xctool
```
### PlistBuddy
脚本的开头有`PlistBuddy`命令，它是Mac下一个用来读写plist文件的工具，在/usr/libexec/下。

