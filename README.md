# [Transitted] Common errors seen in C++
【转】C++常见错误大全，转自 https://www.cnblogs.com/mlv5/p/3739429.html

### C++常见错误大全

#### 0. XXXX "is not a class or namespace"错误
最诡异的错误，提示意思很明显，说你写的名字既不是一个类也不是一个命名空间，虽然我C＋＋水平不是很高，但再愚笨也不至于连类的格式`class MyClass{....}`;也写不明白吧，报此错误原因显然跟它没关系，那又是怎么回事呢？  
答案是：`#include   "stdafx.h"`没放在代码最开头！！！  
`stdafx.h`知识简单说一下：  
所谓头文件预编译，就是把一个工程(Project)中使用的一些MFC 标准头文件(如Windows.H、Afxwin.H)预先编译，以后该工程编译时，不再编译这部分头文件，仅仅使用预编译的结果。这样可以加快编译速度，节省时间。   
预编译头文件通过编译`stdafx.cpp` 生成，以工程名命名，由于预编译的头文件的后缀是“pch”，所以编译结果文件是`projectname.pch`。  
没把：`#include   "stdafx.h"`放在代码最开头会发生什么问题呢？编译器认为，所有在指令`#include "stdafx.h"`前的代码都是预编译的，它会跳过`#include"stdafx. h"`之前的指令，使用projectname.pch 编译这条指令之后的所有代码。因此，当然会出现找不到类的错误了。  
因此，所有的CPP 实现文件第一条语句都应该是：`#include "stdafx.h"`。  

#### 一、LINK2001错误：

此类错误VS给出提示但双击不会定位到出错地点，提示也莫名其妙，要究其原因排错

(1)`error LNK2001: unresolved external symbol "void __cdecl GameShutdown(void)" ()`

原因：头文件(如`main.h`)声明了方法`GameShutdown()`，但`main.cpp`里没有实现它。

排错：在工程中搜“`GameShutdown`”，定位在哪个.h文件，再在对应的.cpp文件中添加实现方法。

 (2)子类未实现父类纯虚函数错误：

`D3DRenderer.obj : error LNK2001: unresolved external symbol "public: virtual void __thiscall CD3DRenderer::SetMultiTexture(void)" ()
Debug/GameProject4.exe : fatal error LNK1120: 1 unresolved externals`

原因：同上，在`D3DRenderer.cpp(D3DRenderer.obj`对应相应的.cpp)中未实现虚方法`SetMultiTexture（）`,这个虚方法是在类的CD3DRenderer的父类中声明，虚方法一定要实现，否则会出现unresolved externals错误

排错：简单，在`类CD3DRenderer（D3DRenderer.cpp）`中实现这个虚方法既可。

    C++ 子类没有实现父类的纯虚函数，则子类也变成抽象类，子类也不能实例化对象
    *在设计基类的时候不好确定将来的行为具体应该表现什么行为，但是必须的。
    含有纯虚函数的类叫抽象类。抽象类不能实例化它的对象，只能为它的派生类服务。
    如果子类没有实现父类的纯虚函数，则子类也变成抽象类，它也不能实例化对象。
    

#### 二、LINK2019 无法解析外部符号"`__declspec(dllimport)`"的错误

错误 1 error LNK2019: 无法解析的外部符号 "__declspec(dllimport) public: __thiscall CEGUI::OgreCEGUIRenderer::OgreCEGUIRenderer(class Ogre::RenderWindow *,unsigned char,bool,unsigned int,class Ogre::SceneManager *)" ()，该符号在函数 "protected: virtual void __thiscall MouseQueryApplication::createScene(void)" () 中被引用 SampleAPP.obj 

经典的找不到dll错误，程序调用的方法在dll中定义，解决方法有两步：

1.找到此dll文件，还有对应的同名lib(如果是静态链接库)，根据上述错误为OgreCEGUIRenderer开头的dll文件中，找到后放到你的工作空间(大概是\bin)目录下。

2.如果还是没有解决，那么一定是项目里没有引用相应的lib文件，因为别人的程序代码没有问题但是程序设置里添加了lib文件你可能没有设置，你可以在项目属性-》链接器-》输入-》附加依赖项里添加相应的lib，或者一劳永逸的方法是在程序代码的开头（#include下面）指定：

#pragma comment(lib,"CEGUIBase_d.lib")

这与在项目属性设置是一样的效果，但是会更醒目，第三方用户拷入你的代码不会再出现找不到dll的错误

#### 三、程序中读取磁盘遇到找不到图片，资源文件引起的错误，或调用某个复杂SDK函数失败返回

    此类错误vs中不会给出提示，排错很困难，在编程时在可能出错的地方要养成多MessageBox的好习惯

   不良写法 if(!m_Device) return; // 如果m_Device创建失败就返回，可是下面的代码不执行，这不是我们想要的

  正确写法 
  ```
  if(!m_Device) {
     Messagebox(0, "!m_Device", 0, 0);  return;  //这里可知道程序无故返回是因为m_Device创建失败造成的
  }
  ```
#### 四、程序中编译通过运行时却出错退出，警告框“某exe 在XXXXXX处出现未处理的异常”


      这是最难调的BUG，很多错误只有运行时才暴露，这时需要借助VS中的堆栈监控来查错。


         例：运行时弹出 “couldn't add <id, 4002201> twice, Continue?”
         点“否”让程序中断，VS弹出 "Client.exe 在 XXXXXX处出现未处理的异常，是否中断？"点击中断。
  这时打开“调用堆栈”视窗，看到绿色箭头停在哪儿就是哪地方出现的错误，但是一般它停的地方都是WIN API的低层，你是看不出来哪儿有问题，就向堆栈底部找，就是向列表下面找，看是谁调用了出错的方法函数，一步一步找。这时我找到：
  VarContain.Add(XX, XX) 看来是这儿加载时出错，某一关键字段加载了两次，但仍看不出哪儿重复加载，这时再向堆栈底部找。
  SkillProtoData->LoadFromFile(strFilename, .......) 看来是这儿加载文件有问题，用debug监视查看strFileName的文件名和路径，是“skillProtoName.xml”文件，去打开它，果然，文件里“id = 4002201”项有两条，删去一个重复项问题就解决了。

  结论：当出现“某exe 在XXXXXX处出现未处理的异常”这样恐怖错误时，堆栈的帮助很重要！！！

 

#### 五.error LNK2019: 无法解析的外部符号 _WinMain@16，该符号在函数 ___tmainCR...

一，问题描述
MSVCRTD.lib(crtexew.obj) : error LNK2019: 无法解析的外部符号 _WinMain@16，该符号在函数 ___tmainCRTStartup 中被引用 
Debug\jk.exe : fatal error LNK1120: 1 个无法解析的外部命令

error LNK2001: unresolved external symbol _WinMain@16
debug/main.exe:fatal error LNK 1120:1 unresolved externals 
error executing link.exe;

二，原因及解决办法
产生这个问题的真正原因是c语言运行时找不到适当的程序入口函数，

一般情况下，如果是windows程序，那么WinMain是入口函数，在VS2008中新建项目为“win32项目”

如果是dos控制台程序，那么main是入口函数，在VS2008中新建项目为“win32控制台应用程序”

而如果入口函数指定不当，很显然c语言运行时找不到配合函数，它就会报告错误。

修改设置适应你的需求

如果是windows程序：

1.菜单中选择 Project->Properties, 弹出Property Pages窗口

2.在左边栏中依次选择：Configuration Properties->C/C++->Preprocessor,然后在右边栏的Preprocessor Definitions对应的项中删除_CONSOLE, 添加_WINDOWS.

3.在左边栏中依次选择：Configuration Properties->Linker->System,然后在右边栏的SubSystem对应的项改为Windows(/SUBSYSTEM:WINDOWS)


如果是控制台程序：

1.菜单中选择 Project->Properties, 弹出Property Pages窗口

2.在左边栏中依次选择：Configuration Properties->C/C++->Preprocessor,然后在右边栏的Preprocessor Definitions对应的项中删除_WINDOWS, 添加_CONSOLE.

3.在左边栏中依次选择：Configuration Properties->Linker->System,然后在右边栏的SubSystem对应的项改为CONSOLE(/SUBSYSTEM:CONSOLE)

五、**.exe 中的 0x111552a1 处未处理的异常: 0xC0000005: 读取位置 0x00000018 时发生访问冲突

在Ogre::ResourceGroupManager::getSingleton().addResourceLocation( "resource\gui.zip", "Zip", "GUI"); 处程序中断

        提示又是像外星文天书一般晦涩，看来排错成功希望渺茫。上网查查看人家说，0xC0000005一般表示空指针，加这里涉及到读取资源可以猜测可能路径不对。果然相对路径"resource\gui.zip" 改成绝对的"d:\\gui.zip",就正常了，但是放在工程目录下还是有问题 ，那这个相对路径是相对哪儿的呢？工作空间难道设置的有问题吗？

        查“项目属性”->“调试”->"工作空间" 设置是“..\bin\Debug”， 退回上级目录进入此目录一看，果然没有“resource”文件夹。在此新建一个“resource”文件夹，再放一个“gui.zip”文件放进去，再编译，还是报错，仔细检查，改成“resource\\gui.zip”，结果终于编译成功了!

心得： 所谓工作空间就是相当于传统意义的工程文件夹，所以相对路径都是相对此目录的，而不是你想当然的程序目录。还有程序代码写路径时一定要双划线，切记！

#### 六、error C2360: initialization of 'c' is skipped by 'case' label

void   func(   void   )   
{   
        int   x;   
        switch   (   x   )   
        {   
        case   0   :   
              int   i   =   1;               //   error,   skipped   by   case   1       
              {   int   j   =   1;   }       //   ok,   initialized   in   enclosing   block   
        case   1   :   
              int   k   =   1;               //   ok,   initialization   not   skipped   
        }   
}  
在switch语句内定义一个变量的时候，如果不在一个语句块内，它是直到遇到switch的"}"才结束的。

所以，如果有在case内定义新变量，最好将该条case内的语句加上{}构成语句块，避免出错
要么就不在case内定义变量，要定义整个case加上{}

#### 七、“UINT WM_MY_REGISTERED_MSG” 已经在“XXX.obj”中定义
#pragma once不能解决头文件重复定义变量

咋一看，分明是UINT WM_MY_REGISTERED_MSG变量在XXX.cpp中重复定义了，可是搜遍整个工程，只发现 WM_MY_REGISTERED_MSG只在userMsg.h头文件中定义一处：UINT WM_MY_REGISTERED_MSG; 别处再无定义，这是怎么回事呢？
想起以前有位恩师反复告诉过我，头文件中只适合#define 常量和声明类和结构体的结构，不适合定义变量，要定义变量都要在.cpp中定义。那我试试把UINT WM_MY_REGISTERED_MSG放在主文件userMsg.cpp中，果然没错了。

看来是头文件多处包含惹的祸，虽然头文件已经写了#pragma once能解决头文件重复包含，但不能解决重复定义变量。在一个头文件中定义了一个变量，哪怕是再不起眼的int n;只要这个头文件被多个cpp #include，那这个int n就算是重复定义，就会报XXX 已经在 XXX.obj中定义的怪现象。

结论：
头文件中只能声明结构，万万不可以定义变量！！！！

#### 八、VS2005编译DLL错误 error C2491: "XXXXXX": definition of dllimport function not allowed

dll头件：dll_object.h
```
#ifdef DLL_OBJECT_EXPORT
#define DLL_OBJECT_API __declspec(dllexport)
#else 
#define DLL_OBJECT_API __declspec(dllimport)
#endif
DLL_OBJECT_API void FuncInDll(void);
extern DLL_OBJECT_API int g_nDll;
class DLL_OBJECT_API CDll_Object
{
public:
 CDll_Object(void);
 void show(void);
};
```

dll_object.cpp:
```
#define DLL_OBJECT_EXPORTS
#include <Windows.h>
#include <iostream>
#include "dll_object.h"
using namespace std;

DLL_OBJECT_API void FuncInDll(void)
{
   cout << "This is FuncInDll"<<endl;
}

DLL_OBJECT_API int g_nDll = 9;

CDll_Object::CDll_Object()
{
 cout << "class of CDll_Object" << endl;
}

void CDll_Object::show()
{
 cout<<"function show in class CDll_Object"<<endl;
}
```
这样写完全没问题，但是老是报error C2491: "XXXXXX": definition of dllimport function not allowed 错，解决方案：
改变VS2005的编译器生成的预定义宏。
我建立的项目名字是MyClass,系统就自定义生成了MyClass_EXPORTS，改成程序中用到的DLL_OBJECT_EXPORTS就能编译通过了

#### 九、A buffer overrun has occured
缓冲区溢出，可能是数组越界，范围太小而存的东西太多升造成

 
#### 十、fatal error CVT1100: duplicate resource. type:MANIFEST, name:1, language:0x0409
解决办法：在工程的rc文件中，搜索类似 RT_MANIFEST             "res//TestApp.manifest"，然后删除掉就OK了

 

#### 十一、头文件相互包含导致的错误
例，有一个状态类
```
#include "Player.h"
class State
{
public:
 virtual void execute(Player* mutou);
};
```
Player类
```
#include "State.h"
class Player
{
 public:
   void changeState(State* curState);

}
```
编译时Player始终报Sate符号错误，说明类Sate有问题，可是看类State就是看不出什么毛病，看了半天才知道，原来State需要用到Player类，而Player又要用到State类，两者相互包含导致一个无解的死循环，所以报错
那怎么办，就真的无解了吗？肯定不是，可以用指针+类声明解决, State类把include "Player.h"去掉，改成这样
class Player;  //要用到的类事先声明
class State ...//下面的和以前一样

然后在State.cpp里再包含 Player.h就可以了，这样就解决了头文件互相包含的问题， Player.h也作相应处理

注意此法只能适应头文件里包含的类是指针形式，如void execute(Player* mutou);，如果参数不是用指针而是实例如 void execute(Player player)就不行了
总体原则上头文件尽量在.cpp里包含，当然避免不了的就采用这种class XX事先声明+指针的形式来解决
