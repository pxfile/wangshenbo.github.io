App启动流程
===

用户点击Home上的一个App图标, 启动一个应用时:

![app launch](https://upload-images.jianshu.io/upload_images/851999-a9c2c456c9f91596.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/683)

## app launch

Click事件会调用startActivity(Intent), 会通过Binder IPC机制, 最终调用到ActivityManagerService. 该Service会执行如下操作:

*   第一步通过PackageManager的resolveIntent()收集这个intent对象的指向信息.
*   指向信息被存储在一个intent对象中.
*   下面重要的一步是通过grantUriPermissionLocked()方法来验证用户是否有足够的权限去调用该intent对象指向的Activity.
*   如果有权限, ActivityManagerService会检查并在新的task中启动目标activity.
*   现在, 是时候检查这个进程的ProcessRecord是否存在了.

如果ProcessRecord是null, ActivityManagerService会创建新的进程来实例化目标activity.

## 创建进程

ActivityManagerService调用startProcessLocked()方法来创建新的进程, 该方法会通过前面讲到的socket通道传递参数给Zygote进程. Zygote孵化自身, 并调用ZygoteInit.main()方法来实例化ActivityThread对象并最终返回新进程的pid.

ActivityThread随后依次调用Looper.prepareLoop()和Looper.loop()来开启消息循环.

![创建进程](https://upload-images.jianshu.io/upload_images/851999-b6b5dacf9d1488f9.jpg?imageMogr2/auto-orient/)

## 绑定Application

接下来要做的就是将进程和指定的Application绑定起来. 这个是通过上节的ActivityThread对象中调用bindApplication()方法完成的. 该方法发送一个BIND_APPLICATION的消息到消息队列中, 最终通过handleBindApplication()方法处理该消息. 然后调用makeApplication()方法来加载App的classes到内存中.

![绑定Application
](https://upload-images.jianshu.io/upload_images/851999-32893aaf343caeac.jpg?imageMogr2/auto-orient/)

## 启动Activity

经过前两个步骤之后, 系统已经拥有了该application的进程. 后面的调用顺序就是普通的从一个已经存在的进程中启动一个新进程的activity了.

实际调用方法是realStartActivity(), 它会调用application线程对象中的sheduleLaunchActivity()发送一个LAUNCH_ACTIVITY消息到消息队列中, 通过 handleLaunchActivity()来处理该消息.

![启动Activity](https://upload-images.jianshu.io/upload_images/851999-9f76d2f18051881c.jpg?imageMogr2/auto-orient/)

* Launcher通过Binder进程间通信机制通知ActivityManagerService，它要启动一个Activity；

* ActivityManagerService通过Binder进程间通信机制通知Launcher进入Paused状态；

* Launcher通过Binder进程间通信机制通知ActivityManagerService，它已经准备就绪进入Paused状态，于是ActivityManagerService就创建一个新的进程，用来启动一个ActivityThread实例，即将要启动的Activity就是在这个ActivityThread实例中运行；

* ActivityThread通过Binder进程间通信机制将一个ApplicationThread类型的Binder对象传递给ActivityManagerService，以便以后ActivityManagerService能够通过这个Binder对象和它进行通信；

* ActivityManagerService通过Binder进程间通信机制通知ActivityThread，现在一切准备就绪，它可以真正执行Activity的启动操作了。

## [Android Application启动流程分析](https://www.jianshu.com/p/a5532ecc8377)

## [Android应用启动流程分析](http://solart.cc/2016/08/20/launch_app/)