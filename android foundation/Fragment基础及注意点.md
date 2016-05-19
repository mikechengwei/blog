#Fragment基础及注意点

##引言
>A Fragment represents a behavior or a portion of user interface in an Activity. You can combine multiple fragments in a single activity to build a multi-pane UI and reuse a fragment in multiple activities. You can think of a fragment as a modular section of an activity, which has its own lifecycle, receives its own input events, and which you can add or remove while the activity is running (sort of like a "sub activity" that you can reuse in different activities).
>>一个碎片代表在时间中的一种行为或者一部分的用户交互，你可以在一个单一的活动(activity)中,组合复杂的碎片形成一个复合UI,并可以重复使用碎片在多个活动中.你可以将碎片想象成活动的一个组件,它有自己的生命周期,获得到它自己的输入事件，并且你可以添加或者移除碎片当活动正在运行(有些类似活动的下级，可以在不同的活动中进行重复使用).

##生命周期
fragment有它自己的生命周期，所以我们先来分析下生命周期.
![Google pic](fragment_lifecycle.png)

* onAttach(Activity) <br>
  called once the fragment is associated with its activity.<br>
  当fragment和activity建立关系的时候调用

* onCreate(Bundle)<br>
  called to do initial creation of the fragment.<br>
  调用初始化创建fragment
* onCreateView(LayoutInflater, ViewGroup, Bundle)<br>
  creates and returns the view hierarchy associated with the fragment.<br>
  创建和返回与fragment相关的布局。
* onActivityCreated(Bundle)<br>
  tells the fragment that its activity has completed its own Activity.onCreate().<br>
  告知fragment activity已经完成它的生命周期中onCreate这一步。
* onViewStateRestored(Bundle)<br>
  tells the fragment that all of the saved state of its view hierarchy has been restored.<br>
  告知fragment 布局的所有存储状态都已经恢复。即这里就是取出存储到Bundle中的数据.
* onStart()<br>
  makes the fragment visible to the user (based on its containing activity being started).<br>
  使得fragment对于用户可见(这是基于包含碎片的activity可见的情况下).在这种情况下用户只能看到视图，但是不能进行交互。
* onResume()<br>
  makes the fragment begin interacting with the user (based on its containing activity being resumed).<br>
  使得碎片开始和用户进行交互(基于包含碎片的activity也开始交互的情况下)
* onPause()<br>
  fragment is no longer interacting with the user either because its activity is being paused or a fragment operation is modifying it in the activity.<br>
  碎片不再和用户交互，可能是activity进入paused状态(activity进入半透明，半遮盖的状态)，或者碎片被移除或者替代。
* onStop()<br>
 fragment is no longer visible to the user either because its activity is being stopped or a fragment operation is modifying it in the activity.<br>
 碎片不再对用户可见，因为activity进stopped状态(activity对用户也不可见)，或者fragment被移除在当前activity中。
* onDestroyView()<br>
  allows the fragment to clean up resources associated with its View.<br>
  fragment清除和视图有关的资源。
* onDestroy()<br>
  called to do final cleanup of the fragment's state.<br>
  对碎片状态的最后清理。
* onDetach()<br>
  called immediately prior to the fragment no longer being associated with its activity.<br>
  通知fragment和当前activity解除关系。
* 大致分为以下过程:
  1. 创建过程：onAttach->onCreate->onCreateView->onActivityCreated
  2. 可见化过程:onStart->onResume
  3. 后台过程:onPause->onStop
  4. 销毁过程:onDestoryView->onDestory->onDetach

按照生命周期的顺序执行。

注意：如上图生命周期图，用户将程序放在后台运行或者进入堆栈，切换到其他视图，就会执行onPause->onStop.这两个操作就是针对可视化到不可视化做出的响应。

##Back Stack
 关于fragment的回退栈可以看我的[这篇文章](http://www.jianshu.com/p/1d0bec0800d2).

##状态的恢复和存储
###方法
* onSaveInstanceState(Bundle outState)<br>
  这是用来储存状态的，比如成员变量等等，但是不需要手动存储视图数据，因为android机制中View自动存储恢复，如果你只想恢复视图只需要实现这个方法。<br>
* onActivityCreated(Bundle)

首先我们要知道fragment中没有和activity 一样的onRestoreInstanceState方法进行状态恢复，所以我们一般在onActivityCreated进行状态恢复,当然onCreate()，onCreateView都可以。

###onSaveInstanceState调用:
* This corresponds to Activity.onSaveInstanceState(Bundle) and most of the discussion there applies here as well. Note however: this method may be called at any time before onDestroy(). There are many situations where a fragment may be mostly torn down (such as when placed on the back stack with no UI showing), but its state will not be saved until its owning activity actually needs to save its state.
* 和activity的onSaveInstanceState(Bundle)相当。这个方法可能被调用在onDestory()之前的许多时机，例如进入后台，但是只有当activity真的需要保存状态的时候才会调用。

* 也就是说当进入后台过程(onPause->onStop->onDestoryView),都有可能调用onSaveInstanceState保存状态，但是只有真的需要保存状态的时候才会调用。我们来列举下不需要调用的情况:
  1. fragmentB返回A，此时不会调用B的onSaveInstanceState方法，因为B不需要被恢复。
  2. 当B在A之后被创建，如果在B的生存期间A并没有被销毁的话，那么系统可能并不会调用A的onSaveInstanceState方法，因为这个时候的A仍然是完整的。

* 当然在旋转和跳转的时候都会进行调用。

注意点：onSaveInstanceState中你只需要保存一些变量，即状态的保存。不需要存储视图资源，因为android View中已经自动实现了存储恢复机制，你只需要声明了onSaveInstanceState方法，View会自动在需要的时候进行存储和恢复。 
##getChildFragmentManager
* fragment中用getChildFragmentManager来管理fragment所嵌套的fragment。
* 所以对fragment进行管理有两种，一种是在activity中对fragment进行管理，一种是在fragment中管理嵌套的fragment.

##总结
* fragment几个重要概念:生命周期，堆栈管理，存储恢复。
* 多实践，理解会更加深刻。
 
  
  
  
   

  
  
