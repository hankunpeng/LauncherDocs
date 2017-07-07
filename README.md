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


## Launcher 中重要类概览

* **Launcher**
    主界面 Activity，最核心且唯一的 Activity。

* **LauncherModel**
    数据处理类，保存桌面运行时的状态信息，提供读写数据库的 API，内部类 LoaderTask 用来初始化桌面。

* **LauncherAppState**
    单例对象，构造方法中初始化对象、注册应用安装、卸载、更新，配置变化等广播。
    这些广播用来实时更新桌面图标等，其 Receiver 的实现在 LauncherModel 类中，LauncherModel 也在这里初始化。

* **LauncherStateTransitionAnimation**
    各类动画总管处理执行类，负责各种情况下的各种动画效果处理。

* **LauncherProvider**
    数据库类，Launcher3 使用 SQLite，数据库文件保存在 /data/data/your_package_name/databases/launcher.db 里。

* **BubblTextView**
    继承自 TextView，桌面图标都是基于它。

* **CellLayout**
    mCountX 和 mCountY 分别表示 “x 方向 icon 的个数” 和 “y 方向 icon 的个数”
    mOccupied 二维数组用来标记每个cell是否被占用，内部类CellInfo为静态类，其对象用于存储cell的基本信息。

* **DeviceProfile**
    设置各元素布局的 padding 。修改 workspace 的 padding 使用 getWorkspacePadding 方法。

* **InvariantDeviceProfile**
    一些不变的设备相关参数管理类，landscapeProfile 和 portraitProfile 为横竖屏模式的 DeviceProfile。getPredefinedDeviceProfiles 方法负责加载在不同设备上不同的布局，和图标大小等。

* **WidgetPreviewLoader**
    存储 Widget 信息的数据库，内部创建了数据库 widgetpreviews.db。

* **LauncherAppsCompat**
    获取已安装App列表信息的兼容抽象基类，子类依据不同版本 API 进行兼容性处理。

* **AppWidgetManagerCompat**
    获取 AppWidget 列表的兼容抽象基类，子类依据不同版本 API 进行兼容性处理。

* **IconCache**
    图标缓存类，应用程序 icon 和 title 的缓存，内部类创建了数据库 app_icons.db。

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
    每一个 ItemInfo 对象都对应着数据库中的一条记录。
    Launcher3 源码里有很多以 ***Info*** 结尾的类，这些类都是 **ItemInfo** 的子类，具体代表了桌面上的某个项目。
    譬如 FolderIcon 和 ***FolderInfo*** 是对应的，BubbleTextView 和 ***ShortcutInfo*** 是对应的，AppWidgetHostView 和 ***LauncherAppWidgetInfo*** 是对应的。有了对应关系，可以这样通过 View 获取 ItemInfo 对象：
    ```java
    ItemInfo info = (ItemInfo)bubbletextview.getTag();
    // 上面的 info 其实是个 ShortcutInfo 对象
    ```

## Launcher3 的启动流程

Launcher3 运行时维护着许多信息，而这些信息都需要在开机的时候加载完，下面我们来看一下 Launcher3 是怎样一步一步启动的。

1 **LauncherApplication.java**
    在启动 Launcher 这个主 Activity 之前先运行 LauncherApplication 里的代码。
    在 LauncherApplication 里获取 LauncherAppState 实例与应用上下文。
    LauncherAppState 用于存储全局变量，比如缓存(各种cache)，维护内存数据的类(LauncherModel)，下面是 LauncherAppState 的类结构
    ```java
    private final AppFilter mAppFilter;
    private final BuildInfo mBuildInfo;
    private final LauncherModel mModel;
    private final IconCache mIconCache;
    private WidgetPreviewLoader.CacheDb mWidgetPreviewCacheDb;
    private static WeakReference<LauncherProvider> sLauncherProvider;
    ```
    mAppFilter 用于存储 App 文件夹的一些信息
    mModel 用于维护 Launcher 在内存中的数据，比如 App 信息列表和 widget 信息列表，同时提供了更新数据库的操作。
    mIconCache 应用程序 Icon 和 Title 的缓存
    mWidgetPreviewCacheDb 存储 Widget 预览信息的数据库
    sLauncherProvider 是 App 和 Widget 的 ContentProvider，用数据库存储信息。
    LauncherAppState.getInstance() 方法实例化了以上的数据，同时对 Launcher 中使用到的 Receiver 和 Observer 进行了注册。
    这里监听的广播有应用的安装、卸载和更新，SD 卡上应用的可用或不可用，地区变化和配置变化等等。接收应用安装更新的广播，都是为了方便实时更新桌面上的图标。

2 **Launcher.java**
    注意：Android 6.0+ 的源码里已经没有 LauncherApplication 了，原先 LauncherApplication 里面的内容放到 Launcher.java 里执行。
    我们着重看 Launcher.onCreate() 里的内容：
    ```java
    mSharedPrefs = getSharedPreferences(LauncherAppState.getSharedPreferencesKey(), Context.MODE_PRIVATE);
    ```
    之前我们说过，桌面在第一次启动的时候会有一个指导界面，告诉用户怎么使用，在以后的启动中，就不会再出现了。控制这个逻辑就是通过SharedPreference，他以key-value的形式把信息存储到一个xml文件当中。

    ```java
    mModel = app.setLauncher(this);
    ```
    LauncherModel 的对象，通过 LauncherAppState.setLauncher(..) 方法获得的，在其中执行了 LauncherModel.initialize(..) 方法，他给 LaucherModel 传递了一个CallBacks对象(在Launcher.java中定义)。为什么要传这个对象呢，因为 LauncherModel 主要是负责数据层的东西，界面方面的操作都交给了 Launcher 来做，那当 LauncherModel 需要更新界面的时候怎么操作呢，就是通过调用这些个回调函数来完成的。

    ```java
    mDragController = new DragController(this);
    ```
    mDragController 是用来控制拖拽对象的东西，到后面会和 DragLayer 结合起来。

    ```java
    mInflater = getLayoutInflater();
    ```
    在图标加载的时候，会一个个去生成 BubbleTextView，而这些都是通过 mInflater.inflate(..) 方法从 xml 生成出来的，详情参考 Launcher.creatShortcut 方法。

    ```java
    mAppWidgetManager = AppWidgetManagerCompat.getInstance(this);
    mAppWidgetHost = new LauncherAppWidgetHost(this, APPWIDGET_HOST_ID);
    ```
    mAppWidgetManager 用来管理小工具，mAppWidgetHost 用来生成小工具的 View 对象并更新。

    ```java
    setContentView(R.layout.launcher);
    ```
    Launcher 在做完一系列初始化操作之后，才进行 `setContentView(R.layout.launcher);`，目的应该是方便界面上的 View 来获取初始化的这些对象。

    ```java
    setupViews();
    ```
    有了界面之后通过 setupViews() 方法把 View 对象和之前初始化过的 DragController 等东西结合起来。
    跳进函数里会看到一溜 findViewById 用来获取 UI 对象，并通过 setup 方法和 DragController 联系起来，然后设置 onClickListener 和 onLongClickListener。

    ```java
    mModel.startLoader(true, mWorkspace.getRestorePage());
    ```
    这句话表示开始加载数据模型了，接下来整个过程都跳转到 LauncherModel 里面。
    在这里会读取数据库，确定哪些东西要加载到桌面上，加载的顺序等等，并通过 Launcher里面的 Callbacks 最终把 ItemInfo 显示到 UI 上。


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
  注意：如果仅仅是让 Launcher.java 里的 `getQsbBar()` 方法返回 `null`，那么谷歌搜索那一栏虽然看不见但还会占着位置。

* 隐藏了 **Folder** 相关的内容
  一期产品规划里不涉及图标文件夹。

* 隐藏了 **overview_panel** 相关的内容
  一期产品规划里不涉及 Launcher 长按显示的 overview_panel 界面。

* 隐藏了 **FocusIndicatorView** 相关的内容

* 双层结构变单层
  设置 `LauncherAppState.isDisableAllApps()` 返回 `true`

* 增加了最左页相关内容



