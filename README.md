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

## Launcher 中重要模块的功能实现

## 各应用 Widget 的数据存储与更新


