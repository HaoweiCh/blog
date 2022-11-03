---
title: PyQT5 与 PyQT4 的思考和应用
description: PyQT5 与 PyQT4 的思考和应用
tags:
- Python
- PyQt
- 观察者模式
categories:
- Languages
date: 2017-06-14T07:06:03+08:00
year: 2017
week: 24
updated: 2017-06-14T07:06:03+08:00
---

文章大部分为官网原文的翻译和整理

 <!-- more --> 

>描述能力有限，不确定的翻译会在旁边注释英文
>更新时间：2017年05月12日
>[官网链接](http://pyqt.sourceforge.net/Docs/PyQt5/pyqt4_differences.html)

### 不与 PyQt4 兼容
虽然实际上升级PyQt4写的项目不是那么糟

### 不再对Python老版本提供支持（Python 2.6 之前）

### 不再实现PyQt4不推荐的API接口 
PyQt5 不支持任何在PyQt4版本中标记为不推荐或舍弃的Qt API（如果有就会当Bug处理）

### 不再提供多版本API接口
PyQt4 支持多版本的API（如`QString `，`QVariant `等）
PyQt5 只支持最新的API版本(除`QVariant`外)
    `QVariant`的改变是去掉了 `QPyNullVariant` （在`QVariant`的帮助文档里也有显示）

### 信号和插槽（Signals and Slots）机制更新

```python
# 下面所列出来的调用方式不再支持
QObject.connect()
QObject.emit()
SIGNAL()
SLOT()
```
所有含有以SIGNAL()或SLOT()返回结果为参数的方法不再支持，转而提供可调用方法（函数）或已捆绑的信号（a bound signal）
#### 风格对比（代码）
```python
# PyQt5
combo = QtWidgets.QComboBox(self)
combo.activated.connect(self.onActivated)
# PyQt4
combo = QtWidgets.QComboBox(self)
self.connect(combo, QtCore.pyqtSignal('activated(QString)'), self.onActivated)
```    
    
####  `QObject.disconnect()` 调用无参数，作用断掉所有信号和插槽的连接

#### TODO New-style Signals and Slots
Qt implements signals with an optional argument as two separate signals, one with the argument and one without it. PyQt4 exposed both of these allowing you to connect to each of them. However, when emitting the signal, you had to use the signal appropriate to the number of arguments being emitted.

PyQt5 exposes only the signal where all arguments are specified. However it allows any optional arguments to be omitted when emitting the signal.

Unlike PyQt4, PyQt5 supports the definition of properties, signals and slots in classes not sub-classed from QObject (i.e. in mixins).

### 新增 `QtQml` `QtQuick` 模块并支持从QML创建Python对象
不再支持`QtDeclarative`, `QtScript`, `QtScriptTools`模块
以上模块被 `QtQml` 和 `QtQuick` 替换。
支持从QML创建Python对象

### `QtGui` 模块更新
QtGui模块被拆分了为QtGui, QtPrintSupport 和QtWidgets三大模块
```python
from PyQt5 import QtGui, QtPrintSupport, QtWidgets
```
### `QtOpenGL` 模块更新
PyQt5的`QtOpenGL`模块只提供`QGLContext` `QGLFormat` 和 `QGLWidget`类

### `QtWebKit` 模块更新
PyQt4的`QtWebKit`在PyQt5中分成了`QtWebKit`和`QtWebKitWidgets`模块

### 扩展性增强
不再支持`pyqtconfig`模块
[The PyQt5 Extension API](http://pyqt.sourceforge.net/Docs/PyQt5/extension_api.html#ref-build-system)
PyQt5 支持第三方包直接基于PyQt5开发(如[QScintilla](https://www.riverbankcomputing.com/software/qscintilla/intro))

### `dbus.mainloop.qt` 模块更名
```python
#dbus.mainloop.qt
dbus.mainloop.pyqt5 # 相同功能只更名
```

### `QDataStream` 明显数值的参数以数值处理和返回
`readUint8(); readInt8(); writeUInt8(); writeInt8()` 方法在PyQt5中以数值类型写入和返回（PyQt4中是以数值文本）

### `QFileDialog`  文件操作接口更新

| PyQt5 | PyQt4 | 备注 |
| --- | --- | --- |
| getOpenFileName() | getOpenFileNameAndFilter()  | |
| getOpenFileNames() | getOpenFileNamesAndFilter() | |
| getSaveFileNameAndFilter() | getSaveFileName() | |

**PyQt5 舍弃了 PyQt4 同名的方法**

### `QMatrix` 方法不再支持
PyQt5 中已经不再支持 PyQt4种不推荐方法 `QMatrix` 
PyQt5 中可以考虑使用 `QPropertyAnimation`

### `QGraphicsItemAnimation` 方法不再支持
PyQt5 中已经不再支持 PyQt4种不推荐方法 `QGraphicsItemAnimation` 
PyQt5 中可以考虑使用 `QTransform`

### `QPyTextObject` 被舍弃
PyQt4 implements the QPyTextObject as a workaround for the inability to define a Python class that is sub-classed from more than one Qt class. PyQt5 does support the ability to define a Python class that is sub-classed from more than one Qt class so long as all but one of the Qt classes are interfaces, i.e. they have been declared in C++ as such using Q_DECLARE_INTERFACE. Therefore QPyTextObject is not implemented in PyQt5.

### QSet 在PyQt5中完全用 集合 形式实现

### pyuic5 不再提供 `--pyqt3-wrapper`参数
pyuic5 does not support the --pyqt3-wrapper flag of pyuic4.

### pyrcc5 不再提供 `-py2` 或 `-py3` 参数
`pyrcc5` cli工具不再支持 `-py2` 或者 `-py3` 参数。因为输出的代码兼容所有Python版本（v2.6以后的所有版本）

### PyQt5 优化 合作性多重继承 （Cooperative Multi-inheritance）的初始化方式

```python
# PyQt5 调用父类的`__init__`方法.
def __init(self, **kwargs):
    super().__init__(**kwargs)
```

### PyQt5 自动释放GIL，而不是PyQt4的强制释放

### PyQt5 退出时自动调用`sip.setdestroyonexit()`以禁用自动析构
Python解释器退出PyQt4应用程序时会默认调用C++析构器处理所有它拥有的线程（这通常是以随机的顺序，因此可能会导致解析器崩溃），通过调用 sip.setdestroyonexit() 函数可以禁用。
PyQt5 总会自动调用 `sip.setdestroyonexit()` 函数.