## View框架
![View框架](View框架.png)

- Activity内部有个Window成员
- Window是基类，系统默认生成子类PhoneWindow
- Window面向Activity，表示“UI外框”，而“框内具体内容”由Window的子类（如PhoneWindow）来规划
- Window要与WMS通信，而一个应用中又可能存在多个Window，借助WindowManager进行统一管理
- WindowManager是接口，真正实现是WindowManagerImpl，是整个应用中所有Window的管理者
- WindowManagerGlobal中3个数组变量，分别表示view树根节点、ViewRootImpl和Window属性，统一管理ViewRootImpl和view树
- ViewRootImpl可以理解为view树的管理者，它有一个mView成员、指向它所管理的view树的根
- ViewRootImpl的核心任务就是和WMS通信
- ViewRootImpl中变量IWindowSession是利用WMS的openSession得到的，用于ViewRootImpl到WMS的连接
- ViewRootImpl又通过IWindowSession提供IWindow对象，从而使WMS通过它与ViewRootImpl进行通信

![Activity、WindowManagerGlobal、WMS等关系图](Activity、WindowManagerGlobal、WMS等关系图.png)
## View树创建过程
![ViewTree建立流程图](ViewTree建立流程图.png)

1. ActivityThread负责处理各种核心事件，如“AMS通知App启动一个Activity”，将转化为其管理的消息、并调用对应的处理方法
2. performLaunchActivity主要任务是生成一个Activity对象，并调用它的attach方法，然后间接调用它的onCreate方法
3. attach方法为Activity内部变量赋值，其中最重要就是mWindow（PhoneWindow对象），每个Activity有且仅有一个Window实例
4. PhoneWindow的setContentView方法生成DecorView（继承FrameLayout）和mContentParent（ViewGroup对象，容纳页面内容）
5. WindowManager继承自ViewManager，真正实现为WindowManagerImpl，它间接调用WindowManagerGlobal的addView，添加新的ViewRootImpl，并将DecorView记录到ViewRootImpl中
## 向WMS注册窗口
PhoneWindow是App端对于“窗口”的描述，WindowState则是WMS中对“窗口”的描述。
IWindowSession和IWindow都是匿名Binder Server，需要借助一定方式才能提供服务。
![](WMS注册窗口流程.png)
1. ViewRootImpl构造方法中，利用WMS的openSession打开一条通道
2. WindowManagerImpl继承自WindowManager，存储App内部用于窗口管理的事务，完全属于本地；IWindowManager是WMS在App进程中的本地代理
3. 向WMS申请注册一个窗口，同时将IWindow对象作为参数传递给WMS
## ViewRootImpl基本工作方式
ViewRootImpl将和WMS进行一系列的通信，主要的触发源有两种：
- View树内部请求，比如某个子View需要更新UI，它通过invalidate或其他方式发起请求，然后该请求就沿着View树层层向上传递，最终到达ViewRootImpl
- 外部状态更新，比如WMS回调ViewRootImpl通知界面大小改变等
无论内部还是外部请求，ViewRootImpl都会把对应消息入队、之后依次处理（ViewRootHandler）。
 ![ViewRoot工作流程](ViewRoot工作流程.png)
## View树遍历（traversal）
系统综合考量各个view“请求”的过程，当过程结束后，各个view就能得到系统的最终分配，包括view的大小、位置以及自身显示内容（UI显示三要素）。
### 遍历时机
- Activity第一次显示：ViewRootImpl的setView中调用requestLayout触发第一次遍历请求
- 外部事件，比如用户产生触摸事件
- 内部事件：
	- View.requestLayout
	- View.setLayoutParams
	- View.invalidate

遍历流程的入口为scheduleTraversals方法，其中调用Choreographer的postCallback，一旦VSYNC信号到来，mTraversalRunnable被回调，进而调用doTraversal方法，进一步调用performTraversals方法。
![ViewTree遍历时机](ViewTree遍历时机.png)
### 遍历流程
遍历主体是performTraversals方法，它的整体逻辑按照3个步骤进行：
1. performMeasure，用于计算View在界面上的绘制区域大小（width、height）
2. performLayout，用于计算View在界面上的绘制位置（left、top）
3. performDraw，View绘制显示内容（canvas、paint）
![](performTraversals实现主体.png)
## View和ViewGroup
View属性：
- id
- tag
- size
- position
- padding
- visibility
- background
ViewGroup属性：
- margin
- layout

![](View_ViewGroup_ViewParent关系.png)
- View定义了一个视图所需具有的属性和基本操作
- ViewGroup的content由若干View组成
- view树的根（DecorView）的ViewParent是ViewRootImpl，其他子view的ViewParent是ViewGroup

```Java
public class View implements Drawable.Callback, KeyEvent.Callback,  
        AccessibilityEventSource
```
Drawable表达了希望在屏幕上绘制的图像，还封装了相关的操作函数，当图像有变化或需要重绘时，通过Callback接口通知View。
## Canvas
- Bitmap：存储“图像”数据的地方
- Canvas：将数据写入Bitmap的工具集
- drawing primitive：Rect、Path、text等基本元素
- Paint：绘制不同风格时用到的颜色、样式等

与Surface协作：
- Java层的Canvas和Surface在native层都有相对应的类——SkCanvas和Surface
- Surface的职责是管理SurfaceFlinger分配的用于存储UI数据的内存块
- ViewRootImpl中Surface.lockCanvas获取Canvas
- ViewRootImpl中Surface.unlockCanvasAndPost告知Surface绘图操作已经完成
![](lockCanvas元素关系.png)
## draw
View的子类只重载onDraw、而不是整个draw，后者具有共性，其中的绘制顺序：
1. 绘制背景
2. （如果需要）保存canvas的layers，以备后续fading
3. 绘制内容
4. 绘制子View（如果有）
5. 绘制fading，restore步骤2保存的layers（如果有）
6. 绘制decorations（主要为scrollbars）
7. 步骤2、5可选
## 触摸事件分发、处理
```
+-------------------+      +-----------------+      +-------------------+
|   用户触摸/按键   |------>|   硬件/驱动     |------>| InputManagerService |
|     (原始事件)    |      |                 |      |    (系统服务)     |
+-------------------+      +-----------------+      +-------------------+
          |                                                   |
          |                                                   |  (InputDispatcher)
          V                                                   V
+-------------------+      +-----------------------+      +------------------+
|   InputReader     |------>| InputDispatcher       |------>| InputChannel     |
|   (系统层)        |      | (决定目标应用窗口)    |      | (IPC管道)        |
+-------------------+      +-----------------------+      +------------------+
                                                                     |
                                                                     | (跨进程通信)
                                                                     V
+--------------------------------------------------------------------------------+
|                             应用进程 (UI 线程)                                 |
+--------------------------------------------------------------------------------+
          |
          V
+-------------------+
|  ViewRootImpl     |
| (接收系统事件)    |
| .deliverInputEvent()|
+-------------------+
          |
          | 调用
          V
+-------------------+
|  DecorView        |
| (窗口根视图)      |
| .dispatchTouchEvent()|
+-------------------+
          |
          | 调用 (通过 Window)
          V
+-------------------+
|  Activity         |
| .dispatchTouchEvent()|
| (开发者常见起点)  |
+-------------------+
          |  (如果 Activity 未消费或调用 super)
          V
+-------------------+
|  ViewGroup        |
| .dispatchTouchEvent()|
| (例如 LinearLayout)|
+-------------------+
          |
          | 1. .onInterceptTouchEvent() (是否拦截)
          | 2. 遍历子 View，找到目标
          | 3. 调用目标子 View 的 .dispatchTouchEvent()
          V
+-------------------+
|  子 View          |
| .dispatchTouchEvent()|
| (例如 Button)     |
+-------------------+
          |
          | 1. 调用 OnTouchListener.onTouch() (如果设置)
          | 2. 调用 .onTouchEvent() (如果 OnTouchListener 未消费)
          | 3. (在 onTouchEvent 内部) 触发 OnClickListener.onClick() 等
          V
+-------------------+
|  事件被消费/处理   |
| (返回 true 停止分发) |
+-------------------+
          |
          | (如果未消费，返回 false)
          V
+-------------------+
|  事件回传或被丢弃   |
+-------------------+
```
### View中
1. dispatchTouchEvent
2. OnTouchListener
3. onTouchEvent
4. ACTION_DOWN：事件序列起点，setPressed、checkForLongClick
5. ACTION_MOVE：pointInView
6. ACTION_UP：performClick
7. ACTION_CANCEL：做好清理工作
### ViewGroup中
1. dispatchTouchEvent
2. onInterceptTouchEvent
3. mFirstTouchTarget
4. child.canReceivePointerEvents()、isTransformedTouchPointInView
## 动画
- 属性动画：以对象属性为动画契机（灵活）
- View动画：提供缩放、平移、旋转、透明度动画效果
- 帧动画：通过连续放映图片产生动画效果

动画的本质是时间和图像的关系。根据视觉原理，人体对于一张图的”存储时间“是0.34秒，只要以特定速率来切换图片，人眼就会感觉整个画面是流畅的、没有卡顿现象。

- 时长
- 起点、终点状态
- 中间过程变化速率
