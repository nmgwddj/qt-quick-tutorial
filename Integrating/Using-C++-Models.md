# Using C++ Models with Qt Quick Views

在 QML 中使用视图容器时，前面的示例中我们是使用 JavaScript 来生成数据模型插入到 Model 中的。如果沿用这种方式，我们需要建立一种机制，将 C++ 中一些原始数据通过信号或者属性等方式传递给 QML，再使用 JavaScript 遍历的方式将数据插入到 Model 中。对于内容较大的数据，这绝对会严重影响程序性能。

## 使用自定义数据模型

Qt 提供了几种 C++ 数据模型的抽象类来让我们从 C++ 这一层维护数据，配合与 QML 之间通信的机制传递一个实例让 QML 可以访问这些数据作为模型。下表是 Qt 提供的一些预定义好的数据模型抽象，你可以继承他们来实现自己的数据模型。

| 类名 | 作用 |
| -- | -- |
| QAbstractListModel | 一个列表模型的抽象类 |
| QAbstractProxyModel | 一个代理模型的抽象类，可以执行排序、过滤或其他数据处理任务 |
| QAbstractTableModel | 一个表格模型的抽象类 |
| QConcatenateTablesProxyModel | 表格模型代理抽象类，同样可以执行排序、过滤等功能 |
| QDirModel | 本地文件系统目录数据模型 |
| QFileSystemModel | 本地文件系统文件数据模型 |
| QStandardItemModel | 一个储存自定数据的通用数据模型 |

Qt 官方文档中使用 QAbstractListModel 为例制作了一个视频教程，教大家如何使用自定义数据模型，如果你尚未了解如何注册自定义类型请先阅读前面的文章，否则您可以无法理解视频中的全部内容。

文档地址：https://doc.qt.io/qt-5/qtquick-modelviewsdata-cppmodels.html

[![Using C++ Models with Qt Quick Views](https://res.cloudinary.com/marcomontalbano/image/upload/v1601989723/video_to_markdown/images/youtube--9BcAYDlpuT8-c05b58ac6eb4c4700831b2b3070cd403.jpg)](https://www.youtube.com/watch?v=9BcAYDlpuT8&feature=emb_logo "Using C++ Models with Qt Quick Views")
