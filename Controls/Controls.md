## Qt Quick Controls

Qt Quick Controls 是目前来看提供的基础控件能力最完整的集合。它不像一些 React、Vue 这种前端 UI 框架仅仅提供基础能力，而由外部团队去完成交互控件的设计和实现（如 Ant Design 等类似视觉交互语言）而是将一些常用的基础交互能力的控件全部抽象出来提供开发者使用。

当我们使用 Duilib 或者 MFC 等类似 UI 框架时，如果需要实现一个 Loading 状态的图标，我们需要自己组织图片、使用基础控件能力去加载一个 gif 图像来实现类似功能。而 Qt Quick Controls 则直接提供了一个 `BusyIndicator Control` 提供开发者快速引入到项目中，并且它的样式你是可以高度自定义的。如下所示：

<img src="../images/Controls/qtquickcontrols2-busyindicator.png">