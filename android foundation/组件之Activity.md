# 组件之Activity生命周期和注意点
##引言
说到Activity,就会想到它的声明周期,面试官也经常会问。所以我们来说明一下。
##声明周期
![activity_lifecyle](activity_lifecycle.png)

除了上图还有四个回调方法,如下:

* onContentChanged
* onPostCreate
* onPostResume
* onConfigurationChanged

综合起来一共11个方法，我们按照触发顺序说明:

* onCreate()
  初始化Activity数据，同时setConentView(view),设置好布局。
* onContentChanged
  当Activity布局变动的时候触发,setContentView 和 addContentView 方法执行完毕后调用该方法,onCreate方法中有setContentView，所以会触发此方法。
* onPostCreate()
  当onCreate()方法执行完毕后触发。
* onStart()
  调用这个方法的时候，界面被用户所看见，但是不能交互即点击啊什么的。
* onResume()
  可见且可以进行交互,执行这个方法后,Activity就会处于running状态。
* onPause()
  整个窗口被半遮盖或者半透明的时候会执行这个方法,即当你想离开这个Activity,或者想进入下一个Activity,处于半遮盖或者半透明状态就会被调用。
* onPostPause()
  和onPostCreate()类似，也是当onPause()执行完毕后调用此方法彻底执行完毕的回调。
* onStop()
  当Activity整个窗口都被遮盖的时候会被触发。当然完全被遮盖之前肯定会经历半遮盖的过程，所以onPause()方法肯定先被调用。
* onDestory()
  在这里做资源的回收与销毁,引用在这时候会被自动回收且销毁，但是线程不会被自动回收，所以你需要手动在这个方法中清楚开启的线程，或者其他资源。
  
##注意点

* 当处于半透明，半遮盖状态的时候，又回到Activity,就会重回界面，即会执行onResume()方法。
* 当处于全遮盖的时候，即执行完onStop()方法后，又重回界面，这时候就会调用onRestart()方法，再执行onStart()方法，onResume()重新回到界面。
* 还有ANR问题,当处于onPause(),onStop即全遮盖或者半遮盖状态的时候，内存占用过多，Activity就有可能被回收，回收后重新回到Activity,就会执行onCreate()方法执行Activity的声明周期。

##总结

* 灵活运用声明周期，可以做到很舒服的用户体验。
 
  
  

