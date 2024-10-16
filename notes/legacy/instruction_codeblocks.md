# Code Blocks 使用指南

Code Blocks 算是比较进阶的 IDE 了，它跨平台（Windows，Mac，Linux），拥有比 Dev-C++ 更好的界面与代码补全功能。综合来看，Code Blocks 是一个完全可以代替 Dev-C++ 而又不丧失其方便、易学、好用特性的 IDE。并且在 Win7 这类平台上（点名批评清北学堂、学校机房一&四的电脑），VS Code 并不总是很好地工作，在这时，Code Blocks 就成了我们的首选。<s>当然，唯一的缺点是不支持中文</s>

下面我们主要以 Windows 7，Code Blocks 20.04 为例，讲解它如何使用。

## 下载安装

### Windows

在 Windows 下，可以去[官网](https://www.codeblocks.org)寻找链接下载其安装包安装，它托管在 SourceForge。这里推荐选择不带 MinGW 的版本，以便自行配置编译器。当然，Code Blocks 目前自带的编译器 MinGW-w64 GCC 8.1.0 是较新的，如果不想另外下载可以选择带 MinGW 的版本，可以开箱即用，省去后续配置路径的麻烦。

![](https://s2.loli.net/2024/09/30/TrbjGBWZ8Jw1zke.png )

![](https://s2.loli.net/2024/09/30/R8qIzvGfe42agAN.png )

### Ubuntu

在 Ubuntu 下<s>只要你使用的是新版本</s> Code Blocks 应当在 `universe` 源中。可通过如下命令安装。

```shell
sudo apt install codeblocks
```

如遇到安装失败的问题，可尝试更换镜像源，或者更新组件后再试，这里不再赘述。

## 下载完后的配置

### 字体设置

首先打开 Code Blocks，你或许会发现字体非常的丑。因此我们要去修改一下字体设置。

在上方导航栏处依次点击 `Settings -> Editor...` 即可打开编辑器设置页面。首先映入眼帘的就有 `Font` 项，点击旁边的 `Choose` 更改字体。对于 Windows 系统，我推荐使用 `Consolas` 这款字体，这是一款很经典的编程专用字体，且在较低版本 Windows 中也自带。字号设置为 12 或 14 等字号，请不要选择“五号、小五”这类字号，它们有可能会导致显示错误。

![](https://s2.loli.net/2024/09/30/qZFOoLaWtd2QBU3.png )

### 编译器设置

由于我们下载了不自带编译器的版本，因此需要自己下载编译器。这里推荐选择 MinGW-w64 GCC 编译器。可以前往 GitHub [WinLibs-MinGW](https://github.com/brechtsanders/winlibs_mingw) 项目的 Releases 中下载，这个项目由 WinLibs 提供，或者前往 GitHub [MinGW-Builds-Binaries](https://github.com/niXman/mingw-builds-binaries) 项目的 Releases 中下载.这些项目都可以在 [MinGW-w64](https://www.mingw-w64.org/) 上找到，是值得信任的。下载任意 64 位版本，解压到安装目录即可。

[安装完毕后，](https://www.mingw-w64.org/)[打开 Code Blocks。依次点击 `Settings -> Compiler..`](https://github.com/niXman/mingw-builds-binaries/releases)`.` 打开编译器配置页面。首先确保上方的 `Selected Compiler` 项为 `GNU GCC Compiler`，然后在下方的**一级**选项卡列表中选择 `Toolchain Executables` 配置编译器可执行文件路径。请将其修改成你安装的编译器下的 `bin` 目录的**完整**路径。

![](https://s2.loli.net/2024/09/30/eP1mkKBfpsJv6tL.png )

此时返回主界面，点击 `File -> New -> Empty File` 创建一个文件并保存为 `*.cpp`。这时会发现左上快捷栏中 `Compile`、`Run`、`Build and Run`、`Rebuild` 几个键会亮起。我们浅写一个 Hello, World，然后按下 F9 快捷键编译运行。这时出现类似以下界面证明运行成功，编译配置完成。如果代码报错，请查看底边 `Build log`、`Build messages` 栏中有无相关线索。

![](https://s2.loli.net/2024/09/30/YGcBwVRnXQJgIq9.png )

## 如何进行调试

Code Blocks 不知怎么调试的问题困扰了我很久，也成为我之前一段时间无法完全放弃 Dev-C++ 的重要原因。后来经过一番 BFS <s>（Baidu First Search）</s> 终于解决掉了。这里给出调试的做法。

要想调试就必须有项目文件，单文件编辑下 Code Blocks 是无法调试的。因此我们依次点击 `File -> New -> Project... -> Empty Project`，然后单击 `Go`，进入项目创建界面。

![](https://s2.loli.net/2024/09/30/DLethCquoOz1F8J.png )

这里 `Project title` 可自行填写，要重点说的是 `Folder to create project in...` 这一项。该项应填写实际项目目录的父目录，即例如 项目完整路径为 `D:\C++\DebugProject`，则这里应填写 `D:\C++`。其余保持默认即可。

![](https://s2.loli.net/2024/09/30/1r7DN8GSMkHv2Rs.png )

创建好项目后，创建一个 C++ 代码文件（以 `main.cpp` 为例）。我们的调试只能在这个文件中运行（Code Blocks 会将同一项目中的多个文件联合编译，在 OI 中这通常会导致报错）。如果你像我一样没有存代码的习惯就没什么问题，有的话也可以复制出去另外保存。

点击 `Settings -> Debugger...` 并选择 `GDB/CDB Debugger -> Default`，点击 `Executable Path` 右侧 `...`，选择你安装的编译器的 `bin` 目录下的 `gdb.exe` 文件。这是 GCC 自带的调试器，Code Blocks 将利用它实现调试。

![](https://s2.loli.net/2024/09/30/15wtKkLWgC3Envz.png )

在编辑区左侧行号右边的地方点击，就会出现一个红点，表示调试时的断点。按下 F8 启动调试，这时会发现与红点同一列的地方出现了黄色三角，表示程序运行到了何处。

![](https://s2.loli.net/2024/09/30/PhOVNqsXz2BTEjA.png )

在最下方 `Debugger` 栏中会有一行 `Command`，在这里我们可以输入对 gdb 的指令。例如 `p ans` 表示打印 ans 这个变量，`q` 表示立即终止调试。此时按 F7 可以将程序往下执行一行，Shift + F7 可以在程序遇到函数时跳入函数内部执行。

![](https://s2.loli.net/2024/09/30/Wlv8nZ6fgJYD5Rd.png )

关于更多的调试技巧，以及命令行使用 gdb 直接进行调试的方法，可以参考这篇文章：[GDB使用详解](https://zhuanlan.zhihu.com/p/297925056)。

## 结语

现在你应当已经具备利用 Code Blocks 进行 OI 类代码编写的能力了。IDE 作为工具，自然顺手是最好的。Code Blocks 是笔者的推荐，但并非强制使用，你也可以根据自己的情况选择适合自己的 IDE。 <s>（建议 Vim + GCC</s>