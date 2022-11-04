---
title: Python的观察者模式，PyQt 信号和插槽 机制的理解和应用
categories:
- Languages
tags:
- Design Pattern
- Python
- Python3
- PyQt
- 观察者模式
date: 2017-05-18T08:06:03+08:00
year: 2017
week: 20
updated: 2017-06-22T16:04:03+08:00
---

介绍一下Python语言下的观察者模式和其在PyQt运用
 <!-- more --> 

## 内容介绍
* [x] 提供信号插槽的python实现方式（更大的灵活性）
- [ ] 修改PyQt版的信号和插槽机制使slot可以接受到关键字参数


## 信号与插槽的 Python 实现

### 介绍
* 因为实现机制的是Python代码不是PyQt的c++，效率可能会更低
* 更贴合Python的风格，more pythonic，限制更少

#### 1. **不能用实例属性替换`pyqtSignal`的类属性**

..
虽然 `pySignal` 起源于 `pyqtSignal` 而且直接实例化 `pySignal` 类的各种使用方式和 `pyqtSignal`相同（毕竟设计的时候就接口相同），但是有个容易让人困惑的点是

```
# 代码示例，pySignal 在实例化后的类中替换 pyqtSignal
class Window(QWidget):
	same_signal = pyqtSignal()
	def __init__(self, parent=None):
		super(Window, self).__init__(parent)
		self.same_signal = Signal()
```

常理来说 `pyqtSignal()` 应该被 `pySignal` 替换掉了，但如早先提到的 `pyqtSignal` 在所在类实例化后摇身一变成实例化的`PyQt5.QtCore.pyqtBoundSignal`对象。

这代表我们先赋值的类属性已经变成了实例化类的属性了

这样造成的影响是，如果我们给某类多个实例，它们将有自己的signal而且互不干扰（emit 也不干扰）

我们实现的类没有这样的特性，类属性一直是类属性

这意味着给某类赋值类属性，其实例也会遗传相同的类属性（不独立）

解决方案是，pySignal在最开始就应该是实例属性而不是类属性（这样能保证独立）

### pySignal
```python
class Signal:
    def __init__(self):
        self.__subscribers = []
      
    def emit(self, *args, **kwargs):
        for subs in self.__subscribers:
            subs(*args, **kwargs)

    def connect(self, func):
        self.__subscribers.append(func)  
      
    def disconnect(self, func):  
        try:  
            self.__subscribers.remove(func)  
        except ValueError:  
            print('Warning: function %s not removed '
                  'from signal %s'%(func,self))
```

## PyQt signal&slot 
### 介绍
#### 1. signal只能作为类属性 Only works with class attributes
Signals are not class attributes. `PyQt5.QtCore.pyqtSignal()` is merely a vessel for a future instance variable containing a PyQt*.QtCore.pyqtBoundSignal instance. When you instantiate your class, pyqtSignal goes to work and injects itself as an instance variable and adds itself to the QMetaObject of the class.

QMetaObject? It comes with useful methods such as .className(), superClass(), methodCount() which returns the name of the class, its superclasses and number of methods respectively.

In C++ these are probably very useful, however a Python programmer might not be very impressed. It’s something we’ve had access to all along via any instances’ __class__, __bases__ and __dict__attributes.

#### 2.不能在已经实例化的类中声明 Cannot be used in an already instantiated class
这是最让人头痛的特性 Now here’s the kicker. 

If you’re doing any sort of base- or abstract class work with Qt widgets, you’ll quickly realise that you can’t inherit signals.

Other than that, if try and bypass inheritance and have a builder spit out widgets for you, you’ll also notice how Dependency Injection isn’t going to work with signals. They have to be created as class attributes and they can only be created using pyqtSignal(). Please correct me if I’m wrong.

#### 3.必须在声明signal时指明传输参数的类型 Must be pre-specified with any data-types you wish to emit
类似于强类型语言的类型声明，这完全不是Python的风格嘛。
In other languages, this is referred to as static typing. Python however doesn’t do any of that.

```python
# 这是演示代码（伪代码），实际上得作为类属性声明
# normally have to be run via a class' class attribute.
signal = pyqtSignal(int, str)
signal.emit(my_number, my_string)

signal.emit(my_string, my_number)
# 报错 TypeError
signal.emit(not_enough_args)
# 报错 TypeError

```


#### 4. 不支持关键字参数 Does not support keyword arguments
> TypeError: emit() takes no keyword arguments

Keyword arguments are quite useful as a means of self-documenting code.

`signal.emit(5)`
本应该写做
`signal.emit(velocity=5)`
这样不仅增强可读性，还可以加强机制使其可以携带相同关键字参数 it can also be used to enforce signals and slots to carry an identical argument signature.

```python
def callback(name, address):
	print("Name=%s and address=%s" % (name, address))

signal = Signal()
signal.connect(callback)

# Mistake the first argument for a tuple.
signal.emit(names=('marcus', 'ottosson'), address='earth')
# TypeError: callback() got an unexpected keyword argument 'names'

# When actually, its a single string value.
signal.emit(name='marcus ottosson', address='earth')
# Name=marcus ottosson and address=earth

# Of course, non-keyword arguments works too.
signal.emit('marcus ottosson', 'earth')

```

#### 5. Cannot be modified after instantiation
Python对象，很自然的支持 代码动态修改（monkey-patch）,但`pyqtSignals`非常特别的不支持
As a Python object, you would expect the ability to monkey-patch, but pyqtSignals are special enough to not let you do any of that.

I’ll provide an example of monkey-patching for you below.

## 理解 Understanding Signals and Slots

Qt 提供的 信号和槽 机制基于一个(行为型)设计模式 ->观察者模式 ([Observer pattern](http://en.wikipedia.org/wiki/Observer_pattern)) (TODO 另开一文详细分析 Python的观察者模式)

### 跨线程

上述实现很直接的介绍了基本原理但缺少了很多应该考虑到的应用场景比如跨线程。

在使用pySignal时你可能已经遇到slot所在的线程**很容易**崩溃（无论是在 `QThread` 或者 Python的多线程模块），这“怪罪于”多线程之美，两个线程可能*同时*尝试或访问同一资源.

### `QObject.sender()`

简而言之, [`QObject.sender()`](http://doc.qt.io/qt-5/qobject.html#sender) 可以让接收端可以获得发送端资源。

```python
def callback(self, message):
	# 在 pySignal 和 pyqtSignal 中都支持slot中查询来源signal
	source_of_signal = self.sender()
	# 这在当slot可能收到多个来源的signal且需要区分他们的时候非常有用
```
**API参考资料中警告说这种方式可能会破坏面向对象程序的模块性，建议尽量避免**

在这里我不准备提供全方位的测评, 但py版相比qt版 `pyqtSignal()`, 如果是obj1 -> obj2 -> obj3 这种链式调用的话，在obj3的slot中查询sender的话会返回obj1

```python

# 在 obj3.listen 中调用sender()会返回 obj1而不是obj2，虽然obj2是调用obj3的signal
obj1.signal.connect(obj2.emit)
obj2.signal.connect(obj3.listen)
obj1.signal.emit()
# 而在 pyqtSignal, 将会返回 obj2.
```

## 应用
### Example – 监视类属性的状态
It can sometimes be useful to monitor an attribute of a class.

```python
class Listener(object):
	def __init__(self):
		self.container = Container()
		self.container.value_changed.connect(self.value_changed_event)

	def value_changed_event(self, previous, current):
		print("%r says: Value was changed from %s to %s" % 
		     (self.__class__.__name__, previous, current))

class Container(object):
	def __init__(self):
		self.__value = None

		# 当`value` 改变了我们就发送信号
		self.value_changed = core.Signal()

	@property
	def value(self):
		return self.__value

	@value.setter
	def value(self, value):
		self.value_changed.emit(previous=self.__value, 
					current=value)
		self.__value = value


if __name__ == '__main__':
	list = Listener()
        
	# 改变value值便会触发

	list.container.value = 5
	# 'Listening' 中打印: Value was changed from None to 5

	list.container.value = 6
	# 'Listening' 中打印: Value was changed from 5 to 6

```


## 总结

1. 在某些复杂的场景下，即使功能强大的`pyqtSignal`也会触短板,这个时候就可以扩展或自己实现
*  但选择不站在巨人肩膀上而自己实现某一特性的话，有些情况我们会在遇到时发现（像先提到的线程间的安全性）。这个时候就不得不挑起自己动手实现新特性的担子。
*  虽然两种signal机制的实现目的是一样的，但它们不一定需要用谁来替换谁。可能最好的方式是两个在代码中同时使用，各取所长，在各自适合的应用场景使用。  
例如，`QThreads` 可以使用 `pyqtSignal` 而我们自定义的组件的基类和制造器(TODO builders)就可以使用 `pySignal`

### 相关资源

* [来源: 英文 多样化信号](http://blog.abstractfactory.io/dynamic-signals-in-pyqt/)
* [资料: PyQt5 官方文档](http://pyqt.sourceforge.net/Docs/PyQt5/signals_slots.html)
* [书籍: Design Patterns](https://www.amazon.co.uk/Design-patterns-elements-reusable-object-oriented/dp/0201633612)  
Is an excellent summary and reference of many very useful patterns.
* [书籍: Head First Design Patterns](https://www.amazon.co.uk/Head-First-Design-Patterns-Freeman/dp/0596007124)  
Provides a more gentle and explanatory view of many of the same patterns.
* [讨论: SOF Custom pyqtSignal implementation](http://stackoverflow.com/questions/21101500/custom-pyqtsignal-implementation)