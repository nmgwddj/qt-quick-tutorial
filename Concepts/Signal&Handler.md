# Signal and Handler Event System

Qt Quick 像传统 Qt Widgets 一样将信号与槽的能力沿用了下来，虽然我们可以这样理解，但是实现机制是不同的。Qt Quick 中使用 QML 描述 UI 对象和一些非可见的组件，当它们需要响应不同的事件时同样是触发一个信号，而响应信号的位置我们通常叫做信号处理程序（类似槽函数）

## 使用处理程序接收信号

通常情况下，一个按钮会有单击信号，这个信号的名字叫做 clicked，当我们需要响应这个信号的时候可以通过在信号名称前 + on 并在后面紧跟首字母大写的信号名，即可响应。如下所示：

```QML
import QtQuick 2.15
import QtQuick.Controls 2.15

Rectangle {
    id: rect
    width: 250; height: 250

    Button {
        anchors.bottom: parent.bottom
        anchors.horizontalCenter: parent.horizontalCenter
        text: "Change color!"
        onClicked: {
            rect.color = Qt.rgba(Math.random(), Math.random(), Math.random(), 1);
        }
    }
}
```

这个例子中当按钮被点击，我们将其父容器的矩形颜色设置了一个随机值。

### 属性变更信号处理程序

除了传统的信号以外，QML 中所有 Item 子项都具有自己的属性，每一个属性的变更我们都可以通过 on + Property + Changed 关键字来响应属性的变更，下面的例子中描述了当 TapHandler 的 pressed 属性变更时我们打印一个日志到控制台。

```QML
import QtQuick 2.15

Rectangle {
    id: rect
    width: 100; height: 100

    TapHandler {
        onPressedChanged: console.log("taphandler pressed?", pressed)
    }
}
```

### 使用 Connections 处理信号

同样的，我们可以在外部响应一个 Item 子项的信号或者属性变更，使用 Connections 关键字，设置其 target 目标即可关注其各种属性或信号的变更，将需要关注的变更以 on + Signal or Property + Changed 关键字响应即可，如下所示：

```QML
import QtQuick 2.15
import QtQuick.Controls 2.15

Rectangle {
    id: rect
    width: 250; height: 250

    Button {
        id: button
        anchors.bottom: parent.bottom
        anchors.horizontalCenter: parent.horizontalCenter
        text: "Change color!"
    }

    Connections {
        target: button
        function onClicked(): {
            rect.color = Qt.rgba(Math.random(), Math.random(), Math.random(), 1);
        }
    }
}
```

### 附加信号处理程序

除了控件自有信号和属性的变更通知，每一个组件都有自己的加载完成和销毁的通知，Qt 文档中以 Attached signal 表示他们。他们以 Component.on* 开头，如下所示：

```QML
import QtQuick 2.15

Rectangle {
    width: 200; height: 200
    color: Qt.rgba(Qt.random(), Qt.random(), Qt.random(), 1)

    Component.onCompleted: {
        console.log("The rectangle's color is", color)
    }
}
```

## 给自定义组件添加信号

通常情况下一个 Qt Quick 自带的组件已经包含了丰富的信号和属性，响应他们已经满足大部分场景的需求，但当我们自己封装的组件需要一个信号让外接根据变更连接时，我们可能需要自己来自定义信号。通过 `signal` 关键字来声明自己的信号，需要触发信号时只需要像调用一个函数一样直接调用信号名称即可。信号可以携带参数，下面的例子中携带了两个 x 和 y 的位置信息参数供外部获取：

```QML
// SquareButton.qml
import QtQuick 2.15

Rectangle {
    id: root

    signal activated(real xPosition, real yPosition)
    property point mouseXY
    property int side: 100
    width: side; height: side

    TapHandler {
        id: handler
        onTapped: root.activated(mouseXY.x, mouseXY.y)
        onPressedChanged: mouseXY = handler.point.position
    }
}
```

当外部使用这个组件的时候，我们可以像处理组件自带信号一样处理我们添加的自定义信号，同样格式也是 on + SignalName

```QML
// myapplication.qml
SquareButton {
    onActivated: console.log("Activated at " + xPosition + "," + yPosition)
}
```

## 连接一个信号到指定的方法

有时我们可能无法对一个动态创建的组件去写信号连接的处理程序，因为他们还没有创建出来，我们还不知道组件的名称，该如何链接？

其实除了信号处理程序以外，我们还可以将信号与一个函数进行连接，通过信号自带的 `connect` 方法我们可以连接该信号到一个函数上，当信号触发时该函数自动执行。这种方法在动态创建组件时非常实用。如下所示：

```QML
import QtQuick 2.15

Rectangle {
    id: relay

    signal messageReceived(string person, string notice)

    Component.onCompleted: {
        relay.messageReceived.connect(sendToPost)
        relay.messageReceived.connect(sendToTelegraph)
        relay.messageReceived.connect(sendToEmail)
        relay.messageReceived("Tom", "Happy Birthday")
    }

    function sendToPost(person, notice) {
        console.log("Sending to post: " + person + ", " + notice)
    }
    function sendToTelegraph(person, notice) {
        console.log("Sending to telegraph: " + person + ", " + notice)
    }
    function sendToEmail(person, notice) {
        console.log("Sending to email: " + person + ", " + notice)
    }
}
```

当我们不在需要这些函数响应信号时，你需要将信号与函数断开链接，使用 `disconnect` 关键字来断开信号与方法的连接：

```QML
Rectangle {
    id: relay
    //...

    function removeTelegraphSignal() {
        relay.messageReceived.disconnect(sendToTelegraph)
    }
}
```

### 信号与信号之间连接

信号可以连接一个信号处理程序（槽函数）或一个具体的方法（JavaScript function），像 Qt Widgets 一样，我们也可以将两个信号连接到一起，让信号 A 触发信号 B，这种场景不是很常用，以下为具体的示例：

```QML
import QtQuick 2.15

Rectangle {
    id: forwarder
    width: 100; height: 100

    signal send()
    onSend: console.log("Send clicked")

    TapHandler {
        id: mousearea
        anchors.fill: parent
        onTapped: console.log("Mouse clicked")
    }

    Component.onCompleted: {
        mousearea.tapped.connect(send)
    }
}
```

我们将 TapHandler 组件的 tapped 信号与父节点 Rectangle 的 send 信号进行了连接，这样当 tapped 信号触发，也同时回触发 send 信号。

