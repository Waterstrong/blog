---
title: "DirectShow获取摄像头图像"
category: Graphics
tags: [图形图像, DirectX, C++, OpenCV]
---

#### 在Windows下配置DirectX Aug09 DShow获取摄像头图像:

1、首先带例子安装DXSDK_Aug09.exe，可在[官网](msdn.microsoft.com/directx/)下载[DirectX August 2009](http://download.microsoft.com/download/4/C/F/4CFED5F5-B11C-4159-9ADC-E133B7E42E5C/DXSDK_Aug09.exe)。假设安装路径为：D:\Program Files\Microsoft DirectX SDK (August 2009)。

2、将strmbasd+&+strmbase文件夹中的两个dll文件拷到D:\Program Files\Microsoft DirectX SDK (August 2009)\Lib\x86下(64位机到x64下)。

3、将DShow文件拷到D:\Program Files\Microsoft DirectX SDK (August 2009)\sample\C++下。

4、Visual Studio中包含文件D:\Program Files\Microsoft DirectX SDK (August 2009)\Include和D:\Program Files\Microsoft DirectX SDK (August 2009)\Samples\C++\DirectShow\BaseClasses。

5、添加库文件D:\Program Files\Microsoft DirectX SDK (August 2009)\Lib\x86到最顶端。

6、添加strmbase.lib strmbasd.lib到lib链接器中。

7、将ARFrameGrabber文件夹中的ARFrameGrabber.h 和 ARFrameGrabber.cpp 拷贝到到自己工程中（自己修改一下，加了些逻辑判断），再添加进来。测试一下：

<!-- more -->

```c_cpp
#include<iostream>  
#include <highgui.h>  
#include "ARFrameGrabber.h"  
#include <Windows.h>  
using namespace std;  
int _tmain(int argc, _TCHAR* argv[])  
{  
    static ARFrameGrabber frameGrabber;  
    IplImage ds_frame;  
    int stride;  
    BYTE* myBuffer;  
    frameGrabber.Init(0, true); //设置支持directshow的设备编号，从0开始  
    frameGrabber.SetFlippedImage(true); //图像是否翻转  
    IplImage* frame=NULL;  
    cvNamedWindow("test",0);  
    while (true)  
    {  
        frame = NULL;  
        frameGrabber.GrabByteFrame(); //获取一帧  
        myBuffer = frameGrabber.GetByteBuffer(); //得到图像的缓冲  
        while(!myBuffer)  
        {  
            UINT nRet=MessageBox(0,"警告：\n\n摄像头正被其他程序占用，请关闭可能使用摄像头的程序后重试！","启动出错",MB_RETRYCANCEL|MB_ICONEXCLAMATION);  
            if(nRet==IDRETRY)  
            {  
                frameGrabber.GrabByteFrame(); //获取一帧  
                myBuffer = frameGrabber.GetByteBuffer(); //得到图像的缓冲  
            }  
            else  
            {  
                cvDestroyAllWindows();  
                exit(0);  
            }  
        }  
        int width = frameGrabber.GetWidth();  
        int height = frameGrabber.GetHeight();  
        stride  = (width * sizeof( RGBTRIPLE ) + 3) & -4;//图像每行所占的字节数，4的倍数，对齐  
        cvInitImageHeader( &ds_frame, cvSize(width, height), 8, 3,IPL_ORIGIN_BL, 4 ); //创建IplImage  
        ds_frame.widthStep = stride;  
        cvSetData( &ds_frame, myBuffer, stride ); //copy数据  
        frame = &ds_frame;  
        cvShowImage("test",frame);  
        if (cvWaitKey(3)==27)  
        {  
            break;  
        }  
    }  
    cvDestroyAllWindows();  
    return 0;  
}
```

----

#### 遇到问题一：

```c_cpp
如果报错: "dxtrans.h": No such file or directory
```

在qedit.h 中添加如下

```c_cpp
#define __IDxtCompositor_INTERFACE_DEFINED__   
#define __IDxtAlphaSetter_INTERFACE_DEFINED__   
#define __IDxtJpeg_INTERFACE_DEFINED__   
#define __IDxtKey_INTERFACE_DEFINED__  
```

再修改qedit.h 中引用 dxtrans.h 的部分，要求注释掉

```c_cpp
#include "oaidl.h"  
#include "ocidl.h"  
//#include "dxtrans.h"  
#include "amstream.h"  
```

----

#### 遇到问题二：

```c_cpp
//#include <qedit.h> 出现问题，或者找不到了什么基类可以直接用下面方式  
#define __IDxtCompositor_INTERFACE_DEFINED__   
#define __IDxtAlphaSetter_INTERFACE_DEFINED__   
#define __IDxtJpeg_INTERFACE_DEFINED__   
#define __IDxtKey_INTERFACE_DEFINED__   
#include <qedit.h>
```

---
*注：以上除DXSDK_Aug09.exe安装文件外(553MB)，其他都可在资源《[配置DX_Aug09_DShow获取摄像头图像](http://download.csdn.net/download/waterstrong/3981334)》中下载。*
