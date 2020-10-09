# DPI Scaling

```C++
// 设置程序根据分辨率自适应缩放
QGuiApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
// 设置缩放比例精确到小数点不需要四舍五入
QGuiApplication::setHighDpiScaleFactorRoundingPolicy(Qt::HighDpiScaleFactorRoundingPolicy::PassThrough);
```
