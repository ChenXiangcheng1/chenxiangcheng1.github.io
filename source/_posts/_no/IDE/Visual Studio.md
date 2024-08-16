

# Visual Studio202217.10.4

[官方文档](https://learn.microsoft.com/zh-cn/visualstudio/ide/synchronized-settings-in-visual-studio?view=vs-2022)



## 界面

项目  配置属性(输出目录 Debug/Release 平台)
	配置属性-常规 : Windows SDK版本、平台工具集、语言标准
	配置属性-C/C++-优化 = 启动(Release默认) / 禁用(Debug默认)
	配置属性-C/C++-代码生成-运行库-多线程调试 DLL =  MD(默认 程序小 依赖系统MS运行时DLL库) / **MT**(程序更大 但是不依赖系统DLL)
	配置属性-C/C++-常规-附加包含目录：设置**头文件相对路径**
	配置属性-C/C++-预编译头：使用(编译选项/Yu)(大项目) / **不使用预编译头**(Precompiled Header .pch)(小项目、跨语言项目)(推荐)
	配置属性-C/C++-链接器-常规-附加库目录：${SolutionDir}/x64/Release **lib库所在path**
	配置属性-C/C++-链接器-输入-附加依赖项：xx.lib **lib库名**
	配置属性-调试：工作目录、命令

生成  配置管理器
调试
测试
分析
工具  所有设置
	命令行-开发者命令提示



### 快捷键

| 快捷键            | 释义                                                         |      |
| ----------------- | ------------------------------------------------------------ | ---- |
| ctrl+l+d          | 格式化代码                                                   |      |
| Ctrl+Shift+空格键 | 查看重载提示信息<br />光标在括号中间，。上下键查看即可。或者，写有重载的函数名后，直接打上shift+(，上下键查看即可。 |      |
| Shift+F12         | 查看方法所有引用                                             |      |
| ///+enter         | 方法注释                                                     |      |



## 项目

推荐：sln和项目分离

解决方案SolutionDir
	启动项目：粗体为启动项目
		引用：右键添加引用 = 生成依赖项-项目依赖项
		外部依赖项
		头文件
		源文件
		资源文件
	静态库项目
	动态链接库项目(删除framework.h pch.h dllmain.cpp pch.cpp以编写适配Linux的)
		dumpbin EXPORTS xxx.dll 查看DLL中导出的函数列表



组织项目：
过滤器+配置属性-C/C++-常规-附加包含目录
lib静态库
dll动态库静态导入
	配置属性-C/C++-链接器-输入-附加依赖项：xx.lib
dll动态库动态加载



导出dll：

```c++
# nn_api.h
#ifdef __cplusplus
extern "C" {
#endif
    
_declspec(dllexport) int __stdcall fun1(const chat* param);
_declspec(dllexport) int __stdcall fun2(const chat* param);
    
#ifdef __cplusplus
}
#endif
```

\__declspec 导出dll
__stddcall 解决报错问题

给别人 xx.dll xx.lib x_api.h



静态导入：

```c++
# nn_api.h
#ifdef __cplusplus
extern "C" {
#endif
    
_declspec(dllimport) int __stdcall fun1(const chat* param);
_declspec(dllimport) int __stdcall fun2(const chat* param);
    
#ifdef __cplusplus
}
#endif
```



动态加载：

```c++
int __stdcall fun1(const chat* param);
typedef int(__stdcall* Proc_fun1)(const char* param);  # 用于提供给GetProcAddress后类型强转
# using Proc_fun1 = int(_stdcall*)(const char* param)  # C++11
# _stdcall*类型的函数指针 参数const char* 返回int
# 函数指针定义：返回值(函数指针名)(参数类型) 

int __stdcall fun2(const chat* param);
```



1. 库的加载，LoadLibraray、dlopen
2. 获取函数地址 GetProcAddress、dlsym
3. 调用某个函数完成功能
4. 卸载dll FreeLibrary、dlclose

```c++
#include "nn/nn_api"

#include "iostream"
#include <windows.h>

static const char* filename = "libnn.dll";

int main()
{
    /* 1. 加载DLL到当前地址空间 */
    HMODELE handle = LoadLibrarayA(filename);
    if (!handle)
    {
        std::cout << "failed to load libraray:" << filename << std::endl;
        return -1;
    }
    
    /* 获取dll导出的函数地址 */
    auto func_fun1 = (Proc_fun1)GetProcAddress(handle, "fun1");
    
    /* 3. 调用dll*/
    func_fun1();
    
    /* 4. 释放dll */
    FreeLibrary(handle)
    
}

```



### 添加lib

1：通过vs工程配置

配置属性-C/C++-链接器-常规-附加库目录：${SolutionDir}/x64/Release **lib库所在path**
配置属性-C/C++-链接器-输入-附加依赖项：xx.lib **lib库名**



2：通过宏

```c++
#ifdef _DEBUG
#pragma comment(lib,"..\\debug\\LedCtrlBoard.lib")
#else
#pragma comment(lib,"..\\release\\LedCtrlBoard.lib")
#endif
```



3：直接添加现有项，选择lib文件(推荐)





## 文件

X86  # 生成的解决方案(可删除重新生成)  # \$(SolutionDir)$(Platform)\$(Configuration)\
	Debug
		projectname.exe
		projectname.pdb
src
	projectame.cpp
bin
dll动态库
lib静态库
resprojectname.vcxproj
projectname.vsxproj.filters
projectname.vsxproj.user	
