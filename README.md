# LauncherDocs
Launcher 文档，记录一些要点，以便团队成员快速熟悉代码。


## Launcher 在 Android 系统中的位置
一图胜千言，Launcher 已用橘色标记。
![Android Stack](./LauncherDocs-Figure001.png?raw=true "Android Stack")
细心的同学发现上图中 SystemUI 也用橘色标记了，这样做是有原因的，跟实际业务需求有关，以后有机会细说。


## Launcher 的界面构成要素
网上找的图，对于小组研究 Launcher3 已经够用了。当然，以后有时间我会重绘一张更精美的。
![Launcher Layout](./LauncherDocs-Figure002.png?raw=true "Launcher Layout")
  * SearchDropTargetBar 是屏幕最上方的搜索框，在我们拖动图标的时候，搜索框会替换成“删除”。
  * Workspace 是屏幕上左右滑的好几屏幕的容器。
  * CellLayout 是 Workspace 里面可以滑动的单独一屏，它负责图标和小部件的显示和整齐摆放。
  * BubbleTextView 是图中显示的一个个小方格，显示各应用 Widget 用的。
  * PageIndicator 是滑动时屏幕下方的小圆点指示器。
  * Hotseat 用来放置比较常用的应用，比如拨号，短信，相机等。
  还有一个看不见的 DragLayer，与 Widget 拖动相关的操作都在这个 View 里处理。


## Launcher 中重要类

* **CellLayout**
  mCountX 和 mCountY 分别表示 “x 方向 icon 的个数” 和 “y 方向 icon 的个数”
  mOccupied 二维数组用来标记每个cell是否被占用，内部类CellInfo为静态类，其对象用于存储cell的基本信息。

* **DeviceProfile**
  设置各元素布局的 padding 。修改 workspace 的 padding 使用 getWorkspacePadding 方法。

* **Launcher**
  主界面 Activity，最核心且唯一的 Activity。

* **LauncherAppState**
  单例对象，构造方法中初始化对象、注册应用安装、卸载、更新，配置变化等广播。这些广播用来实时更新桌面图标等，其 receiver 的实现在 LauncherModel 类中，LauncherModel 也在这里初始化。

* **LauncherModel**
  数据处理类，保存桌面状态，提供读写数据库的 API，内部类 LoaderTask 用来初始化桌面。

* **InvariantDeviceProfile**
  一些不变的设备相关参数管理类，landscapeProfile 和 portraitProfile 为横竖屏模式的 DeviceProfile。getPredefinedDeviceProfiles 方法负责加载在不同设备上不同的布局，和图标大小等。

* **WidgetPreviewLoader**
  存储 Widget 信息的数据库，内部创建了数据库 widgetpreviews.db。

* **LauncherAppsCompat**
  获取已安装App列表信息的兼容抽象基类，子类依据不同版本 API 进行兼容性处理。

* **AppWidgetManagerCompat**
  获取 AppWidget 列表的兼容抽象基类，子类依据不同版本 API 进行兼容性处理。

* **LauncherStateTransitionAnimation**
  各类动画总管处理执行类，负责各种情况下的各种动画效果处理。

* **IconCache**
  图标缓存类，应用程序 icon 和 title 的缓存，内部类创建了数据库 app_icons.db。

* **LauncherProvider**
  核心数据库类，负责 launcher.db 的创建与维护。

* **LauncherAppWidgetHost**
  AppWidgetHost 子类，是桌面 Widget 宿主，为了方便托拽等才继承处理的。

* **LauncherAppWidgetHostView**
  AppWidgetHostView 子类，配合 LauncherAppWidgetHost 得到 HostView。

* **LauncherRootView**
  竖屏模式下根布局，继承了 InsettableFrameLayout，控制是否显示在状态栏等下面。

* **DragLayer**
  一个用来负责分发事件的 ViewGroup。

* **DragController**
  DragLayer 只是一个 ViewGroup，具体的拖拽的处理都放到了 DragController 中。

* **BubblTextView**
  图标都基于它，继承自 TextView。

* **DragView**
  拖动图标时跟随手指移动的 View。

* **Folder**
  打开文件夹展示的 View。

* **FolderIcon**
  文件夹图标。

* **DragSource/DropTarget**
  拖拽接口，DragSource 表示图标从哪开始拖，DropTarget 表示图标被拖到哪去。

* **ItemInfo**
  桌面上每个 Item 的信息数据结构，包括在第几屏、第几行、第几列、宽高等信息。
  该对象与数据库中记录一一对应。
  该类有多个子类，譬如 FolderIcon 的 FolderInfo，BubbleTextView 的 ShortcutInfo 等。


## Auto Launcher 对 Launcher3 的修改

* 隐藏 SearchDropTargetBar。可以在 Launcher.xml 里注释掉 layout：

  ```xml
  <!--
          <include
              android:id="@+id/search_drop_target_bar"
              layout="@layout/search_drop_target_bar" />
  -->
  ```

  或设置 SearchDropTargetBar 的高度为 0 ：

  ```java
  // DeviceProfile.java
  padding.set(desiredWorkspaceLeftRightMarginPx - defaultWidgetPadding.left,
          0, // searchBarBounds.bottom
          desiredWorkspaceLeftRightMarginPx - defaultWidgetPadding.right,
          hotseatBarHeightPx + pageIndicatorHeightPx);
  ```

* 隐藏了 **Folder** 相关的内容
  一期产品规划里不涉及图标文件夹。


## Launcher 中涉及的关键技术点



## 各应用 Widget 的数据存储与更新


