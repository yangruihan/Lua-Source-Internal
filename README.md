# Lua 设计与实现

本库原地址：https://github.com/lichuang/Lua-Source-Internal

## 目录

**注：以下内容将修订为以 Lua 5.3.5 源代码为基础**

*	第一章 概论

*	第二章 基础数据结构
	*	[Lua中的数据类型](doc/ch02-Lua%E4%B8%AD%E7%9A%84%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.md) ✓
	*	[Lua字符串](doc/ch02-Lua%E5%AD%97%E7%AC%A6%E4%B8%B2%E7%B1%BB%E5%9E%8B.md) ✓
	*	[Lua表](doc/ch02-Lua%E8%A1%A8.md)

*	第三章 Lua虚拟机
	*	[Lua虚拟机栈结构及相关数据结构](doc/ch03-Lua%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%A0%88%E7%BB%93%E6%9E%84%E5%8F%8A%E7%9B%B8%E5%85%B3%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.md)
	*	[Lua指令执行过程](doc/ch03-Lua%E6%8C%87%E4%BB%A4%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.md)
	*	[Lua虚拟机概述](doc/ch03-Lua%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%A6%82%E8%BF%B0.md)
	*	[Lua栈](doc/ch03-Lua%E6%A0%88.md)
	*	[Hello World](doc/ch03-HelloWorld.md)
	*	[Lua虚拟机指令格式](doc/ch03-lua%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%8C%87%E4%BB%A4%E6%A0%BC%E5%BC%8F.md)
	*	[ChunkSpy使用说明](doc/ch03-ChunkSpy%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.md)

*	第四章 Lua虚拟机指令
	* 	[Lua词法](doc/ch04-Lua%E8%AF%8D%E6%B3%95.md)
	* 	[赋值类指令](doc/ch04-%E8%B5%8B%E5%80%BC%E7%B1%BB%E6%8C%87%E4%BB%A4.md)
	*	[表相关操作指令](doc/ch04-%E8%A1%A8%E7%9B%B8%E5%85%B3%E6%93%8D%E4%BD%9C%E6%8C%87%E4%BB%A4.md)
	*	[函数相关类指令](doc/ch04-%E5%87%BD%E6%95%B0%E7%B1%BB%E6%8C%87%E4%BB%A4.md)
	*	[计算类指令](doc/ch04-%E8%AE%A1%E7%AE%97%E7%B1%BB%E6%8C%87%E4%BB%A4.md)
	*	[字符串操作类指令](doc/ch04-%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%93%8D%E4%BD%9C%E6%8C%87%E4%BB%A4.md)
	*	[关系逻辑类指令](doc/ch04-%E9%80%BB%E8%BE%91%E5%85%B3%E7%B3%BB%E7%B1%BB%E6%8C%87%E4%BB%A4.md)
	*	[循环类指令](doc/ch04-循环类指令.md)

* 第五章 表
	*	[元表及面向对象的实现](doc/ch05-%E5%85%83%E8%A1%A8.md)

* 第六章 环境与模块
	* 	[环境相关的变量](doc/ch06-%E7%8E%AF%E5%A2%83%E7%9B%B8%E5%85%B3%E7%9A%84%E5%8F%98%E9%87%8F.md)
	* 	[模块](doc/ch06-%E6%A8%A1%E5%9D%97.md)
	* 	[Lua模块热更新原理](doc/ch06-%E7%83%AD%E6%9B%B4%E6%96%B0.md)
	*	[闭包]

* 第七章 [Lua调试器](doc/ch07-%E8%B0%83%E8%AF%95%E5%99%A8.md)(100%)

* 第八章 [Lua GC](doc/ch08-GC.md)

* 第九章 杂项
	* [异常处理](doc/ch09-%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86.md)(100%)
	* [协程](doc/ch09-%E5%8D%8F%E7%A8%8B.md) (100%)

## Lua 语言特性

- **可移植性**：使用 clean C 编写的解释器，可以在 Mac、Windows、Linux、Unix 等多个平台轻松编译

- **良好的嵌入性**：Lua 提供了非常丰富的 API，可供宿主程序与 Lua 脚本之间进行通信和交换数据

- **非常小的尺寸**：Lua 5.3 版本的源代码压缩包仅297KB

- **Lua 的效率很高，是速度最快的脚本语言之一**：为了提高 Lua 的性能，作者们将最初使用 Lex、Yacc 等工具自动生成的代码都改成了自己手写的词法分析器和解析器

## Lua 源代码结构

### 虚拟机核心相关文件列表

|文件名|作用|对外接口前缀|
|:---|:---|:---|
|lapi.c|C语言接口|lua_|
|lcode.c|源码生成器|luaK_|
|ldebug.c|调试库|luaG_|
|ldo.c|函数调用及栈管理|luaD_|
|ldump.c|序列化预编译的Lua字节码||
|lfunc.c|提供操作函数原型及闭包的辅助函数|luaF_|
|lgc.c|GC|luaC_|
|llex.c|词法分析|luaX_|
|lmem.c|内存管理|luaM_|
|lobject.c|对象管理|luaO_|
|lopcodes.c|字节码操作|luaP_|
|lparser.c|分析器|luaY_|
|lstate.c|全局状态机|luaE_|
|lstring.c|字符串操作|luaS_|
|ltable.c|表操作|luaH_|
|lundump.c|加载预编译字节码|luaU_|
|ltm.c|tag方法|luaT_|
|lzio.c|缓存流接口|luaZ_|

### 内嵌库相关文件列表

|文件名|作用|
|:---|:---|
|lauxlib.c|库编写时需要用到的辅助函数库|
|lbaselib.c|基础库|
|ldblib.c|调试库|
|liolib.c|IO 库|
|lmathlib.c|数学库|
|loslib.c|OS 库|
|ltablib.c|表操作库|
|lstrlib.c|字符串操作库|
|loadlib.c|动态扩展库加载器|
|linit.c|负责内嵌库的初始化|

### 解析器、字节码编译相关文件列表

|文件名|作用|
|:---|:---|
|lua.c|解释器|
|luac.c|字节码编译器|

