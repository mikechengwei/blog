# Service 基础及注意点

##引言
文档内容主要翻译自google文档，也有个人补充。
##Service
>A Service is an application component that can perform long-running operations in the background and does not provide a user interface. Another application component can start a service and it will continue to run in the background even if the user switches to another application. Additionally, a component can bind to a service to interact with it and even perform interprocess communication (IPC). For example, a service might handle network transactions, play music, perform file I/O, or interact with a content provider, all from the background.
>>Service是一个可以在后台执行长时间运行且不提供用户交互操作的应用组件.其他应用组件(activity,fragment)可以在后台运行一个service即使用户跳转到其他应用。除此之外，一个组件可以和service绑定进行交互,例如IPC机制(AIDL)。举个例子，service可以在后台处理网络传输，播放音乐，执行文件I/O，或者和content provider交互。

##使用形式
Service有两种形式:
 
 * Started  启动型<br>
 当应用组件(例如activity)启动service当调用startService(),Service处于started状态。一旦启动，service会不确定的运行在后台，即使组件启动后销毁。比较常见的，一个启动的service会执行单一的操作后并不返回结果给调用者。举例，可能发生在下载或者上传文件超时。当这种执行接收不到结果，应该停止这个service.
 * Bound<br> 绑定型
 当应用组件通过调用bindService()绑定service，service处于bound的状态。一个绑定的service提供客户端-服务端交互的行为，允许组件和service进行交互，发送请求，得到结果，这个就是IPC中的ALDL,只有当其他应用组件和service绑定后，绑定型service才会运行。多个部分的组件可以立即绑定service,但是当他们都解除绑定，service就会被销毁。
 * 虽然讨论了这两种形式，但是官方建议你的Service可以兼顾这两种形式，启动和绑定都可以运行。

##函数介绍
如果你想要创建一个service,你必须创建Service的子类，实现并覆盖某些方法。<br>
我们应该覆盖的方法有:

* onStartCommand()<br>
//系统在其它组件比如activity通过调用startService()请求service启动时调用这个方法．一旦这个方法执行，service就启动并且在后台长期运行．如果你实现了它，你需要负责在service完成任务时停止它，通过调用stopSelf()或stopService()．(如果你只想提供绑定，你不需实现此方法)．
* onBind()<br>
//当组件调用bindService()想要绑定到service时(比如想要执行进程间通讯)系统调用此方法．在你的实现中，你必须提供一个返回一个IBinder来以使客户端能够使用它与service通讯，你必须总是实现这个方法，但是如果你不允许绑定，那么你应返回null．
* onCreate()
//系统在service第一次创建时执行此方法，来执行只运行一次的初始化工作(在调用它方法如onStartCommand()或onBind()之前)．如果service已经运行，这个方法不会被调用．
* onDestroy()
//系统在service不再被使用并要销毁时调用此方法．你的service应在此方法中释放资源，比如线程，已注册的侦听器，接收器等等．这是service收到的最后一个调用

##声明周期
![service lifestyle](service_lifecycle.png)

生命周期根据两种service的两种形式分为两种:
 
 * onCreate()->onStartCommand()->onDestory()
 * onBind->onUnBind->onDestory()

启动后的service在不用的时候调用stopService来停止它，系统会自动销毁.<br>
绑定后的service在不用的时候调用unbindService来停止它,系统会自动销毁.<br>

##简述用Service通信的情况
###Activity和Service通信
Activity和Service进行通信会通过绑定的形式，在onBind method中返回Binder实例，然后Binder中实现当前Service的引用，这样Activity和Service 就建立起通信。
   
    /** 
     * 返回一个Binder对象 
     */  
    @Override  
    public IBinder onBind(Intent intent) {  
        return new MsgBinder();  
    }  
      
    public class MsgBinder extends Binder{  
        /** 
         * 获取当前Service的实例 
         * @return 
         */  
        public MsgService getService(){  
            return MsgService.this;  
        }  
    }  

###AIDL
* AIDL是android为了方便开发者进行进程间通信，定义的一种语言。
* Android有一个跨进程间通信的工具Messenger，那为什么还设计AIDL呢？原因是AIDL可以处理多线程、多客户端并发访问的，而Messenger只能是单线程处理。

AIDL里面通过service来当做server端。

##总结
* Service的生命期比较简单，但是还是要注意销毁的问题，不要在不用的时候不进行Stop.
* Service可以做成兼顾启动和绑定两种模式。