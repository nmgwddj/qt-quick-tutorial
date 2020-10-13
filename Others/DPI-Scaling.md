# DPI Scaling

默认情况下你创建的 QML 应用程序都是开启了根据系统分辨率缩放的能力的，如下所示：

```QML
#include <QGuiApplication>
#include <QQmlApplicationEngine>

int main(int argc, char *argv[])
{
    // 开启 DPI 自动适配能力
    QGuiApplication::setAttribute(Qt::AA_EnableHighDpiScaling); // <--
    QGuiApplication app(argc, argv);
    QQmlApplicationEngine engine;
    const QUrl url(QStringLiteral("qrc:/main.qml"));
    engine.load(url);
    return app.exec();
}
```

你必须在实例化 QGuiApplication 对象之前去设置该能力。很多场景下这一行代码就足够了，但实际 Qt 在对应用程序窗口及内部控件在缩放时，比例并不是严格按着系统的缩放比例来缩放的。我们来看一个示例：

```QML
import QtQuick 2.12
import QtQuick.Window 2.12
import QtQuick.Controls 2.12

Window {
    id: mainWindow
    visible: true
    width: 480
    height: 360
    title: qsTr('Hello world')
    color: '#EFEFEF'
    flags: Qt.Window | Qt.FramelessWindowHint

    Component.onCompleted: {
        console.log('Screen device pixel ratio:', Screen.devicePixelRatio)
        console.log('Window width:', mainWindow.width * Screen.devicePixelRatio)
        console.log('Window height:', mainWindow.height * Screen.devicePixelRatio)
    }

    Label {
        anchors.centerIn: parent
        font.pixelSize: 24
        font.family: 'Microsoft YaHei UI'
        text: qsTr('Windows size: %1 x %2')
            .arg(mainWindow.width * Screen.devicePixelRatio)
            .arg(mainWindow.height * Screen.devicePixelRatio)
    }
}
```

示例在窗口加载完毕后，打印了当前屏幕的 `Screen.devicePixelRatio` 属性，它是实际物理像素和设备无关像素之间的比例，就是我们俗话说的缩放比例。

在屏幕分辨率为 2560x1440 缩放 100% 的时候，我们运行程序，打印如下内容：

```
qml: Screen device pixel ratio: 1
qml: Window width: 480
qml: Window height: 360
```

可以看到信息是正确的，随后我们再设置缩放 125% 的时候，我们运行程序，打印如下内容：

```
qml: Screen device pixel ratio: 1
qml: Window width: 480
qml: Window height: 360
```

好像不太正确？是的，这时 devicePixelRatio 的值应该是 1.25，但这里依然打印了 1。我们再将缩放比例调整为 150% 看一下效果：

```
qml: Screen device pixel ratio: 2
qml: Window width: 960
qml: Window height: 720
```

实际我们看到，devicePixelRatio 的值被四舍五入了。这时 Qt 内部默认的设定，这种四舍五入的计算方式，在某些非正规屏幕如 1920*1080 缩放 150% 的场景下，窗口实际会被放大 200%。原本 1080P 缩放 150% 以后就没有多少空间了，自身窗口再放大 200%，这将导致部分原始窗口较大的窗口严重超出屏幕范围。要解决这个问题可以在 main.cpp 中设置启用缩放后，设置一下 Qt 的缩放策略。如下所示：

```C++
// 设置程序根据分辨率自适应缩放
QGuiApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
// 设置缩放比例精确到小数点不需要四舍五入
QGuiApplication::setHighDpiScaleFactorRoundingPolicy(Qt::HighDpiScaleFactorRoundingPolicy::PassThrough);
```

我们通过 `setHighDpiScaleFactorRoundingPolicy` 将缩放策略设置为 `Qt::HighDpiScaleFactorRoundingPolicy::PassThrough`，该参数目的是告诉 Qt 程序，不对缩放比例四舍五入。

随后我们再测试一下屏幕分辨率为 2560x1440 缩放 150% 时候的效果：

```
qml: Screen device pixel ratio: 1.5
qml: Window width: 720
qml: Window height: 540
```

这时我们看到的缩放比例和实际缩放后的分辨率就没有问题了。

## 总结

在实际开发中我们很少会去修改这些 Qt 默认的设置，但个别场景下应为 UI 适应的需求，我们要根据情况去做一下修改。在不同分辨率适配的场景，除了单独窗口的缩放适应问题，还有窗口、内嵌窗口等适配的问题，您可以根据自己的情况来分别处理不同窗口的缩放范围。
