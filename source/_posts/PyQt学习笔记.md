title: PyQt学习笔记
author: Jinyun
tags:
  - PyQt
  - Python
categories:
  - Python
date: 2020-11-26 13:54:00
---
## 1. 环境搭建安装和配置

### 1.安装

我这里使用的环境是ubuntu20.04 + python3.8.5 + pycharm2017

使用pyenv创建虚拟环境
```
pyenv virtualenv Qt5

pyenv activate Qt5

pip install PyQt5 PyQt5-tools
```
安装QtCreator
```
sudo apt-get install qtcreator pyqt5-dev-tools
```

### 2. 配置pytharm

1. 在　｀File ->　Settings -> Tools -> External Tools｀　中创建扩展菜单.

![](http://mediaqn.meitranslation.com/blog/exttools-qt.png)

添加三个子菜单
1. QtDesigner
```
Name: QtDesigner
Group: Qt
Program: /usr/bin/designer
Arguments: $FileName$
working directory: $ProjectFileDir$
```

2. PyUIC
```
Name: PyUIC
Group: Qt
Program: python
Arguments: -m PyQt5.uic.pyuic $FileName$ -o $FileNameWithoutExtension$.py
working directory: $FileDir$
```
3. Rcc2Py
```
Name: Rcc2Py
Group: Qt
Program: python
Arguments: -m PyQt5.pyrcc_main $FileName$ -o $FileNameWithoutExtension$_rc.py
working directory: $FileDir$
```

### 3. 测试

在PyCharm中创建python项目HelloQt，选择pyenv 创建的虚拟环境Qt5
在项目上右键打开`Qt -> QtDesigner`,打开界面创建一个测试界面,`template/forms`选择`Dialog with Buttons Bottom`,Create , 保存成`HelloQtDialog.ui`

关闭QtDesigner 回到pycharm,在HelloQtDialog.ui右键选在`Qt -> PyUIC`,直接编译生成对应的python文件

创建一个启动的Main.py文件.
```python
import sys

from PyQt5.QtWidgets import QApplication, QDialog

import HelloQtDialog

if __name__ == '__main__':
    app = QApplication(sys.argv)
    dlg = QDialog()

    myUI = HelloQtDialog.Ui_Dialog()

    myUI.setupUi(dlg)
    dlg.show()
    sys.exit(app.exec_())
```
启动这个main文件的main方法,可以直接看到界面.

到此位置,开发环境就算搭建好了.



## 2. Qt Designer界面介绍

### 创建界面

1. Diglog模板，基于QDialog类的窗体，具有一般对话框的特性，如可以模态显示，具有返回值等．
2. Main Window模板, 基于QMainWindow类的窗口，具体有主窗口的特性，窗口上有主菜单栏，工具栏和状态栏等．
3. Widget模板，基于QWidget类的窗体．QWidget 类是所有界面的基类，比如QLable,QPushButton等界面组件都是从QWidget类继承而来．QWidget类也是QDialog和QMainWindow的父类，基于QWidget类创建的窗体可以作为独立的窗口运行，也可以嵌入到其他界面组件内显示．



![image-20201202173818469](http://mediaqn.meitranslation.com/blog/image-20201202173818469.png )



## 3. 界面与逻辑分离的设计方法

### 创建UI文件

在pycharm 项目中，右键打开`Qt -> QtDesigner`, 创建`Widget`, 添加`QLabel`,`QPushButton`,保存`HelloQtForm.ui`. 在pycharm中`HelloQtForm.ui`右键选在`Qt -> PyUIC`．生成对应的py文件．

### 多继承方法

```py
import sys
from PyQt5.QtWidgets import QWidget, QApplication

from HelloQtForm import Ui_Form

# 窗体的业务逻辑类
class QMyWidget(QWidget,Ui_Form):
    def __init__(self,parent=None):
    	# 函数super()获取父类，并执行父类的构造函数，代码是super().__init__(parent)，在多继承时，使用super()得到的是第一个基类，在这里就是QWidget。所以，执行这条语句后，self就是一个QWidget对象。
        super().__init__(parent)

        self.Lab = "多重继承的QMyWidget"
        # 因为QmyWidget的基类包括Ui_Form类，所以可以调用Ui_Form类的setupUi()函数。同时，经过前面调用父类的构造函数，self是一个QWidget对象，可以作为参数传递给setupUi()函数，正好作为各组件的窗体容器。
        self.setupUi(self)
        self.label.setText(self.Lab)

if __name__ == '__main__':
    app = QApplication(sys.argv)
    myWidget = QMyWidget()
    myWidget.show()
    myWidget.btnClose.setText("不关闭了")
    sys.exit(app.exec_())
```



优缺点：

1. 界面上的组件都成为窗体业务逻辑类QMyWidget的公共属性，外界可以直接访问．优点是方便访问，缺点是过于开放，不符合面向对象的严格封装的思想
2. 界面上的组件与包装组件QMyWidget类中的新定义属性混合到一起，不便于区分，组件较多时，属性是定义在界面类还是逻辑类，不方便区分，这样不利于界面和业务逻辑分离．

### 单继承与界面独立封装方法

```py
import sys
from PyQt5.QtWidgets import QWidget, QApplication

from HelloQtForm import Ui_Form


class QMyWidget(QWidget):
    def __init__(self,parent=None):
        super().__init__(parent)
        # 显式地创建了一个Ui_Form类的私有属性self.__ui，即self.__ui = Ui_Form。私有属性self.__ui包含了可视化设计的UI窗体上的所有组件，所以，只有通过self.__ui才可以访问窗体上的组件，包括调用其创建界面组件的setupUi()函数
        self.__ui = Ui_Form()
        self.__ui.setupUi(self)

        self.Lab = "单继承的QMyWidget"
        self.__ui.label.setText(self.Lab)

    def set_btn_text(self,a_text):
        self.__ui.btnClose.setText(a_text)

if __name__ == '__main__':
    app = QApplication(sys.argv)
    myWidget = QMyWidget()
    myWidget.show()
    myWidget.set_btn_text("不关闭了")
    sys.exit(app.exec_())
```



可视化设计的窗体对象被定义为QmyWidget类的一个私有属性self.__ui，在QmyWidget类的内部对窗体上的组件的访问都通过这个属性实现，而外部无法直接访问窗体上的对象，这更符合面向对象封装隔离的设计思想。

窗体上的组件不会与QmyWidget里定义的属性混淆，例如self.Lab和self.__ui.label，有利于界面与业务逻辑的分离。

也可以定义界面对象为公共属性，即创建界面对象时用下面的语句：self.ui = Ui_Form()，这里的ui就是个公共属性，在类的外部也可以通过属性ui直接访问界面上的组件

### 小总结

多继承和单继承，其实就是类的组成**`继承`**和**`组合`**之间的关系．`组合关系`更加灵活和易扩展和拆分．



## 4.信号与槽的使用

信号(Signal)：就是在特定情况下发送的一种消息,比如一个PushButton常见的信号就是点击发送的clicked()信号等.GUI程序主要内容就是对界面上的各种组件发射的各种信号进行响应,只需要知道什么情况下发射了那些信号,然后合理的处理这些信号就可以

槽(Slot):就是对应信号响应的函数,槽实质也是一个函数,可以被直接调用. 槽函数可以与一个信号关联,当信号被发射的时候,关联的槽函数会被自动调用.Qt会有一些內建(build-in)的槽函数,比如QWidget有一个槽函数close(),其功能是关闭窗口,如果把PushButton按钮的的click()信号和窗体的close()槽函数关联,点击按钮就会关闭窗口.

### 创建一个UI

在QtDesigner中创建ui文件.

![](http://mediaqn.meitranslation.com/blog/image-20201203124605172.png)



1. 创建基于`Dialog without Buttons`模板的窗口

2. 添加多个`Horizontal Layout` ,只有添加了组件之后,Dialog组件的布局才能调整, 直接选择`Vertival Layout`

3. 添加组件构成如下图

   ![](http://mediaqn.meitranslation.com/blog/image-20201203130646725.png)

4. 配置属性

   |    对象名    |     类名称     |                 属性设置                  |                   功能                   |
   | :----------: | :------------: | :---------------------------------------: | :--------------------------------------: |
   |    Dialog    |    QDialog     |          windowTitle="信号与槽"           | 窗体的类名称是Dialog，objectName不要修改 |
   |   textEdit   | QPlainTextEdit | plainText="PyQt5 编程指南\nPython 和 Qt." |           用于显示文字，可编辑           |
   | chkBoxUnder  |   QCheckBox    |             Text="Underline"              |           设置字体的下划线特定           |
   | chkBoxItalix |   QCheckBox    |               Text="Italic"               |            设置字体的斜体特性            |
   |  chkBoxBold  |   QCheckBox    |                Text="Bold"                |            设置字体的粗体特性            |
   |  radioBlack  |  QRadioButton  |               Text="Black"                |            设置字体颜色为黑色            |
   |   radioRed   |  QRadioButton  |                Text="Red"                 |            设置字体颜色为红色            |
   |  radioBlue   |  QRadioButton  |                Text="Blue"                |            设置字体颜色为蓝色            |
   |   btnClear   |  QPushButton   |                Text="清空"                |              清空文本框内容              |
   |    btnOK     |  QPushButton   |                Text="确定"                |           返回确定，并关闭窗口           |
   |   btnClose   |  QPushButton   |                Text="退出"                |                 退出程序                 |

   每个布局还有layoutTopMargin、layoutBottomMargin、layoutLeftMargin、layoutRightMargin这四个属性用于调整布局边框与内部组件之间的上、下、左、右的边距大小。

5. 工具栏

   ![](http://mediaqn.meitranslation.com/blog/image-20201203131825243.png)

   

   在设计窗体的上方有一个工具栏，用于使界面进入不同的设计状态，以及进行布局设计，工具栏商各按钮的功能如下：

   | 按钮以及快捷键         | 功能                                                         |
   | ---------------------- | ------------------------------------------------------------ |
   | Edit Widget(F3)        | 界面设计进入编辑状态，也就是正常的设计状态                   |
   | Edit Signals/Slots(F4) | 进入信号与槽的可视化设计状态                                 |
   | Edit Buddies           | 进入伙伴关系编辑状态，可以设置一个Lable与一个组件成为伙伴关系 |
   | Edit Tab Order         | 进入Tab顺序编辑状态，Tab顺序是指在键盘上按Tab键时，输入焦点在界面各组件之间跳动的顺序 |

6. 组件的信号与内建槽函数的关联

   点击`Edit Signals/Slots(F4)`,窗体进入信号与槽函数编辑状态。鼠标点选“确定”按钮，在按住鼠标左键拖动到窗体的空白区域后释放左键，这时出现关联设置对话框

   ![](http://mediaqn.meitranslation.com/blog/image-20201203132323263.png)

   btnOK中的clicked()连接到Dialog的accept()槽函数,点击OK按钮.同样的将btnClose的clicked()信号与Dialog的close()槽函数关联，值得注意的是，如果没有看到close()槽函数，可以将下方的“Show signals and slots inherited from QWidget”打勾。

### 把ui文件转换

1. 把转换ui文件转换成py文件

2. 创建窗体业务逻辑类

   ```python
   import sys
   from PyQt5.QtWidgets import QDialog, QApplication
   
   import HelloQtDialogSignalSlot
   
   class QMyDialogSignalSlot(QDialog):
       def __init__(self,parent=None):
           super().__init__(parent)
           self.ui = HelloQtDialogSignalSlot.Ui_Dialog()
           self.ui.setupUi(self)
   
   if __name__ == '__main__':
       app = QApplication(sys.argv)
       form = QMyDialogSignalSlot()
       form.show()
       sys.exit(app.exec_())
   ```

3. 创建主程序文件main.py

   ```python
   import sys
   from PyQt5.QtWidgets import QApplication
   from myDialog import QmyDialog
   
   app = QApplication(sys.argv)  # 创建GUI应用程序
   mainform = QmyDialog()  # 创建主窗体
   mainform.show()  # 显示主窗体
   sys.exit(app.exec_())
   ```

### 总结

把前几天的代码整理了一下如下:

![](http://mediaqn.meitranslation.com/blog/image-20201203133312351.png)

发现import 同目录文件的时候pycharm提示错误,这是因为pycharm没有把目录作为`源文件目录`

## 5.为组件的內建信号编写槽函数

1. 为`清空`按钮添加槽函数,在`__init__()`方法中给`清空`按钮绑定一个函数 `self.ui.btnClear.connect(self.clear_text)`

2. 在业务逻辑类中添加`clear_text()`函数

   ```python
   class QMyDialogSignalSlot(QDialog):
       def __init__(self,parent=None):
           super().__init__(parent)
           self.ui = HelloQtDialogSignalSlot.Ui_Dialog()
           self.ui.setupUi(self)
   
           self.ui.btnClear.connect(self.clear_text) # btnClear绑定clear_text槽函数
   
       def clear_text(self):
           self.ui.textEdit.clear()
   ```

   如果函数按照`on_<object name>_<signal name>(<signal parameters>)`命名的话，那么就可以省略绑定函数那一步。

   在于HelloQtDialogSignalSlot.py文件中的Ui_Dialog.setupUi()函数的最后一行语句 QtCore.QMeteObject.connectSlotsByName(Dialog)，使用了Qt的元对象（QMetaObject），它会搜索Dialog窗体上的所有从属组件，将匹配的信号和槽函数关联起来。比如清空按钮槽函数取名为on_btnClear_clicked()。

3. 更新代码如下

   ```python
   import sys
   
   from PyQt5.QtCore import Qt
   from PyQt5.QtGui import QPalette
   from PyQt5.QtWidgets import QDialog, QApplication
   
   import HelloQtDialogSignalSlot
   
   class QMyDialogSignalSlot(QDialog):
       def __init__(self,parent=None):
           super().__init__(parent)
           self.ui = HelloQtDialogSignalSlot.Ui_Dialog()
           self.ui.setupUi(self)
   
           # ///////////// 自行添加的部分 /////////////
   
           """
           这部分如果是按照槽函数命名规则命名的，可以注释掉，如果是自己命名的，那么需要加上。
           命名规则：on_<object name>_<signal name>(<signal parameters>)
           如清空按钮就是：on_btnClear_clicked
           """
           # ==================================================================
           # # 添加下划线
           # self.ui.chkBoxUnder.clicked.connect(self.on_chkBoxUnder_clicked)
           #
           # # 修改编辑框中字体为斜体
           # self.ui.chkBoxItalix.clicked.connect(self.on_chkBoxItalix_clicked)
           #
           # # # 修改编辑框中字体为斜体
           # # self.ui.chkBoxItalix.toggled.connect(self.on_chkBoxItalix_toggled)
           #
           # # 添加加粗效果
           # self.ui.chkBoxBold.toggled.connect(self.on_chkBoxBold_toggled)
           #
           # # 添加clear效果
           # self.ui.btnClear.clicked.connect(self.on_btnClear_clicked)
           # ==================================================================
   
           # 设置颜色关联函数
           self.ui.radioBlack.clicked.connect(self.do_setTextColor)
           self.ui.radioRed.clicked.connect(self.do_setTextColor)
           self.ui.radioBlue.clicked.connect(self.do_setTextColor)
   
       # 设置文本颜色
       def do_setTextColor(self):
           plet = self.ui.textEdit.palette()  # 获取palette
           if (self.ui.radioBlack.isChecked()):
               plet.setColor(QPalette.Text, Qt.black)
           elif (self.ui.radioRed.isChecked()):
               plet.setColor(QPalette.Text, Qt.red)
           elif (self.ui.radioBlue.isChecked()):
               plet.setColor(QPalette.Text, Qt.blue)
           self.ui.textEdit.setPalette(plet)
   
       # 将编辑框里面的文字添加下划线
       def on_chkBoxUnder_clicked(self):
           checked = self.ui.chkBoxUnder.isChecked()  # 读取勾选状态
           font = self.ui.textEdit.font()
           font.setUnderline(checked)
           self.ui.textEdit.setFont(font)
   
       # @pyqtSlot(bool)
       # # 将编辑框里面的文字变为斜体
       # def on_chkBoxItalic_clicked(self, checked):
       #     font = self.ui.textEdit.font()
       #     font.setItalic(checked)
       #     self.ui.textEdit.setFont(font)
       #
       # # 将编辑框里面的文字变为斜体
       # def on_chkBoxItalic_clicked(self):
       #     checked = self.ui.chkBoxItalix.isChecked()
       #     font = self.ui.textEdit.font()
       #     font.setItalic(checked)
       #     self.ui.textEdit.setFont(font)
   
       # 将编辑框里面的文字变为斜体
       def on_chkBoxItalix_toggled(self, checked):
           font = self.ui.textEdit.font()
           font.setItalic(checked)
           self.ui.textEdit.setFont(font)
   
       # 将编辑框里面的内容加粗
       def on_chkBoxBold_toggled(self, checked):
           font = self.ui.textEdit.font()
           font.setBold(checked)  # 参数checked表示勾选状态
           self.ui.textEdit.setFont(font)
   
       # 清除编辑框里面的内容函数
       def on_btnClear_clicked(self):
           self.ui.textEdit.clear()
   
   if __name__ == '__main__':
       app = QApplication(sys.argv)
       form = QMyDialogSignalSlot()
       form.show()
       sys.exit(app.exec_())
   ```

   关于斜体部分，举了3种方式，都可以实现其效果，其中有一种是@pyqtSlot(bool)，这是overload型信号.

   overload型信号的处理，在QCheckBox类组件中，有两个名称为clicked的信号.

   一个是不带参数的clicked()信号，使用这个信号生成槽函数是可以自动关联的；

   另一个是带参数的clicked(bool)信号，它将复选框的当前勾选状态作为参数传递给槽函数。

   这种名称相同但参数个数或类型不同的信号就是overload型信号。

4. connectSlotsByName()函数进行信号与槽函数的关联时会使用一个默认的信号，对QCheckBox来说，默认使用的是不带参数的clicked()信号，而如果定义on_chkBoxItalic_clicked(self,checked)是需要传递进来一个参数的，那么如果想要使用这个槽函数，就需要使用@pyqtSlot修饰符，用这个修饰符将函数的参数类型声明清楚。这样，connectSlotsByName()函数就会自动使用clicked(bool)信号与这个槽函数关联，运行就没问题。

## 6. 自定义型号与槽使用

1. 例子代码`human.py`

   ```python
   from PyQt5.QtCore import QObject, pyqtSignal
   
   
   class Human(QObject):
   
       nameChanged = pyqtSignal(str)
   
       ageChanged = pyqtSignal([int],[str])
   
       def __init__(self,name="Mike",age =10,parent=None):
           super().__init__(parent)
           self.setAge(age)
           self.setName(name)
   
       def setAge(self,age):
           self.__age = age
           self.ageChanged.emit(self.__age)
   
           if age <= 18:
               age_info = "你是 少年"
           elif 18 < age <= 35:
               age_info = "你是 年轻人"
           elif 35 < age <= 55:
               age_info = "你是 中年人"
           elif 55 < age <= 80:
               age_info = "您是 老人"
           else:
               age_info = "您是 寿星啊"
           self.ageChanged[str].emit(age_info)
   
       def setName(self,name):
           self.__name = name
           self.nameChanged.emit(self.__name)
   
   class Responsor(QObject):
       # @pyqtSlot(int)
       def do_ageChanged_int(self,age):
           print("您的年龄是：" + str(age))
   
       # @pyqtSlot(str)
       def do_ageChanged_str(self,ageInfo):
           print(ageInfo)
   
       def do_nameChanged(self,name):
           print("Hello, " + name)
   
   if __name__ == '__main__':
       print("**创建对象时**")
       boy = Human("Boy",16)
       resp = Responsor()
       boy.nameChanged.connect(resp.do_nameChanged)
   
       boy.ageChanged.connect(resp.do_ageChanged_int)
       boy.ageChanged[str].connect(resp.do_ageChanged_str)
   
       print("\n **建立关联后**")
       boy.setAge(35)
       boy.setName("Jack")
   
       boy.ageChanged[str].disconnect(resp.do_ageChanged_str)
       print("\n **断开ageChanged[str关联后**")
       boy.setAge(10)
   
   ```

2. 信号的定义

   定义的类Human是从QObject继承而来的，它定义了两个信号，两个信号都需要定义为类的属性。

   nameChanged信号是带有一个str类型参数的信号，定义为nameChanged=pyqtSignal(str)

   ageChanged信号是具有两种类型参数的overload型的信号，信号的参数类型可以是int，也可以是str。ageChanged信号定义为：ageChanged=pyqtSignal([int], [str])

3. 信号的发射

   通过信号的emit()函数发射信号。在类的某个状态发生变化，需要通知外部发生了这种变化时，发射相应的信号。如果信号关联了一个槽函数，就会执行槽函数，如果信号没有关联槽函数，就不会产生任何动作。

4. 信号与槽的关联

   另外定义的一个类Responsor也是从QObject继承而来的，它定义了三个函数，分别用于与Human类实例对象的信号建立关联。因为信号ageChanged有两种参数类型，要与两种参数的ageChanged信号建立关联，两个槽函数的名称必须不同，所以定义的两个槽函数名称分别是do_ageChanged_int和do_ageChanged_str。需要在创建类的具体实例后在进行信号与槽的关联，所以程序在测试部分先创建两个具体的对象。

   boy=Human("Boy", 16)

   resp=Responsor()

   如果一个信号的名称是唯一的，即不是overload型信号，那么关联时无须列出信号的参数，例如，nameChanged信号的连接为boy.nameChanged.connect(resp.do_nameChanged)

   对于overload型的信号，定义信号时的第一个位置的参数是默认参数。例如，ageChanged信号的定义是：ageChanged=pyqtSignal([int], [str])，所以，ageChanged信号的默认参数就是Int型。默认参数的信号关联无须标明参数类型，所以有：

   boy.ageChanged.connect(resp.do_ageChanged_int)

   但是对于另外一个非默认参数，必须在信号关联时在信号中注明参数，即：

   boy.ageChanged[str].connect(resp.do_ageChanged_str)

5. 断开信号与槽的关联

   在程序中可以使用disconnect()函数断开信号与槽的关联，例如：boy.ageChanged[str].disconnect(resp.do_ageChanged_str)

## 7. 常用界面组件的使用(QSlider和QProgressBar)

1. 创建ui

   ![](http://mediaqn.meitranslation.com/blog/image-20201203155713732.png)

2. 编译成py文件

   ```python
   # -*- coding: utf-8 -*-
   
   # Form implementation generated from reading ui file 'Form.ui'
   #
   # Created by: PyQt5 UI code generator 5.10.1
   #
   # WARNING! All changes made in this file will be lost!
   
   from PyQt5 import QtCore, QtGui, QtWidgets
   
   class Ui_Form(object):
       def setupUi(self, Form):
           Form.setObjectName("Form")
           Form.resize(726, 468)
           self.verticalLayout = QtWidgets.QVBoxLayout(Form)
           self.verticalLayout.setObjectName("verticalLayout")
           self.groupBox = QtWidgets.QGroupBox(Form)
           self.groupBox.setTitle("")
           self.groupBox.setObjectName("groupBox")
           self.verticalLayout_2 = QtWidgets.QVBoxLayout(self.groupBox)
           self.verticalLayout_2.setObjectName("verticalLayout_2")
           self.horizontalLayout_3 = QtWidgets.QHBoxLayout()
           self.horizontalLayout_3.setSizeConstraint(QtWidgets.QLayout.SetDefaultConstraint)
           self.horizontalLayout_3.setObjectName("horizontalLayout_3")
           self.label = QtWidgets.QLabel(self.groupBox)
           sizePolicy = QtWidgets.QSizePolicy(QtWidgets.QSizePolicy.Ignored, QtWidgets.QSizePolicy.Preferred)
           sizePolicy.setHorizontalStretch(0)
           sizePolicy.setVerticalStretch(0)
           sizePolicy.setHeightForWidth(self.label.sizePolicy().hasHeightForWidth())
           self.label.setSizePolicy(sizePolicy)
           self.label.setAlignment(QtCore.Qt.AlignCenter)
           self.label.setIndent(-1)
           self.label.setObjectName("label")
           self.horizontalLayout_3.addWidget(self.label)
           self.horizontalSlider = QtWidgets.QSlider(self.groupBox)
           sizePolicy = QtWidgets.QSizePolicy(QtWidgets.QSizePolicy.Ignored, QtWidgets.QSizePolicy.Fixed)
           sizePolicy.setHorizontalStretch(0)
           sizePolicy.setVerticalStretch(0)
           sizePolicy.setHeightForWidth(self.horizontalSlider.sizePolicy().hasHeightForWidth())
           self.horizontalSlider.setSizePolicy(sizePolicy)
           self.horizontalSlider.setOrientation(QtCore.Qt.Horizontal)
           self.horizontalSlider.setObjectName("horizontalSlider")
           self.horizontalLayout_3.addWidget(self.horizontalSlider)
           self.verticalLayout_2.addLayout(self.horizontalLayout_3)
           self.horizontalLayout_2 = QtWidgets.QHBoxLayout()
           self.horizontalLayout_2.setObjectName("horizontalLayout_2")
           self.label_2 = QtWidgets.QLabel(self.groupBox)
           sizePolicy = QtWidgets.QSizePolicy(QtWidgets.QSizePolicy.Ignored, QtWidgets.QSizePolicy.Preferred)
           sizePolicy.setHorizontalStretch(0)
           sizePolicy.setVerticalStretch(0)
           sizePolicy.setHeightForWidth(self.label_2.sizePolicy().hasHeightForWidth())
           self.label_2.setSizePolicy(sizePolicy)
           self.label_2.setAlignment(QtCore.Qt.AlignCenter)
           self.label_2.setObjectName("label_2")
           self.horizontalLayout_2.addWidget(self.label_2)
           self.horizontalScrollBar = QtWidgets.QScrollBar(self.groupBox)
           sizePolicy = QtWidgets.QSizePolicy(QtWidgets.QSizePolicy.Ignored, QtWidgets.QSizePolicy.Fixed)
           sizePolicy.setHorizontalStretch(0)
           sizePolicy.setVerticalStretch(0)
           sizePolicy.setHeightForWidth(self.horizontalScrollBar.sizePolicy().hasHeightForWidth())
           self.horizontalScrollBar.setSizePolicy(sizePolicy)
           self.horizontalScrollBar.setOrientation(QtCore.Qt.Horizontal)
           self.horizontalScrollBar.setObjectName("horizontalScrollBar")
           self.horizontalLayout_2.addWidget(self.horizontalScrollBar)
           self.verticalLayout_2.addLayout(self.horizontalLayout_2)
           self.horizontalLayout = QtWidgets.QHBoxLayout()
           self.horizontalLayout.setObjectName("horizontalLayout")
           self.label_3 = QtWidgets.QLabel(self.groupBox)
           sizePolicy = QtWidgets.QSizePolicy(QtWidgets.QSizePolicy.Ignored, QtWidgets.QSizePolicy.Preferred)
           sizePolicy.setHorizontalStretch(0)
           sizePolicy.setVerticalStretch(0)
           sizePolicy.setHeightForWidth(self.label_3.sizePolicy().hasHeightForWidth())
           self.label_3.setSizePolicy(sizePolicy)
           self.label_3.setAlignment(QtCore.Qt.AlignCenter)
           self.label_3.setObjectName("label_3")
           self.horizontalLayout.addWidget(self.label_3)
           self.progressBar = QtWidgets.QProgressBar(self.groupBox)
           sizePolicy = QtWidgets.QSizePolicy(QtWidgets.QSizePolicy.Ignored, QtWidgets.QSizePolicy.Fixed)
           sizePolicy.setHorizontalStretch(0)
           sizePolicy.setVerticalStretch(0)
           sizePolicy.setHeightForWidth(self.progressBar.sizePolicy().hasHeightForWidth())
           self.progressBar.setSizePolicy(sizePolicy)
           self.progressBar.setProperty("value", 24)
           self.progressBar.setObjectName("progressBar")
           self.horizontalLayout.addWidget(self.progressBar)
           self.verticalLayout_2.addLayout(self.horizontalLayout)
           self.verticalLayout.addWidget(self.groupBox)
           self.groupBoxSettings = QtWidgets.QGroupBox(Form)
           self.groupBoxSettings.setObjectName("groupBoxSettings")
           self.gridLayout = QtWidgets.QGridLayout(self.groupBoxSettings)
           self.gridLayout.setObjectName("gridLayout")
           self.checkBox = QtWidgets.QCheckBox(self.groupBoxSettings)
           self.checkBox.setObjectName("checkBox")
           self.gridLayout.addWidget(self.checkBox, 0, 0, 1, 1)
           self.checkBox_2 = QtWidgets.QCheckBox(self.groupBoxSettings)
           self.checkBox_2.setObjectName("checkBox_2")
           self.gridLayout.addWidget(self.checkBox_2, 0, 1, 1, 1)
           self.radioButton = QtWidgets.QRadioButton(self.groupBoxSettings)
           self.radioButton.setObjectName("radioButton")
           self.gridLayout.addWidget(self.radioButton, 1, 0, 1, 1)
           self.radioButton_2 = QtWidgets.QRadioButton(self.groupBoxSettings)
           self.radioButton_2.setObjectName("radioButton_2")
           self.gridLayout.addWidget(self.radioButton_2, 1, 1, 1, 1)
           self.verticalLayout.addWidget(self.groupBoxSettings)
   
           self.retranslateUi(Form)
           QtCore.QMetaObject.connectSlotsByName(Form)
   
       def retranslateUi(self, Form):
           _translate = QtCore.QCoreApplication.translate
           Form.setWindowTitle(_translate("Form", "Form"))
           self.label.setText(_translate("Form", "Slider"))
           self.label_2.setText(_translate("Form", "ScollBar"))
           self.label_3.setText(_translate("Form", "ProcessBar"))
           self.groupBoxSettings.setTitle(_translate("Form", "ProcessBar设置"))
           self.checkBox.setText(_translate("Form", "textVisible"))
           self.checkBox_2.setText(_translate("Form", "InvertedAppearance"))
           self.radioButton.setText(_translate("Form", "显示格式--百分比"))
           self.radioButton_2.setText(_translate("Form", "显示格式--当前值"))
   
   
   ```

   

3. 创建逻辑类,定义自定义信号和槽

   ```python
   import sys
   from PyQt5.QtCore import pyqtSlot
   from PyQt5.QtWidgets import QWidget, QApplication
   
   import Form
   
   class QMyWidget(QWidget):
       def __init__(self,parent=None):
           super().__init__(parent)
           self.ui = Form.Ui_Form()
           self.ui.setupUi(self)
   
           self.ui.horizontalSlider.setMaximum(200)
           self.ui.horizontalScrollBar.setMaximum(200)
           self.ui.progressBar.setMaximum(200)
   
           # self.ui.progressBar.setStyleSheet("QProgressBar::chunk { background-color: rgb(255, 0, 0)}")
           self.ui.progressBar.setStyleSheet("QProgressBar::chunk { background-color: rgb(255, 0, 0)}"
                                             "QProgressBar{text-align: center}")
   
           # self.ui.progressBar.setStyleSheet("QProgressBar{text-align: center;}")
           self.ui.horizontalSlider.valueChanged.connect(self.do_valueChanged)
           self.ui.horizontalScrollBar.valueChanged.connect(self.do_valueChanged)
   
       def on_radioButton_clicked(self):
           self.ui.progressBar.setFormat("%p%")
   
       def on_radioButton_2_clicked(self):
           self.ui.progressBar.setFormat("%v")
   
       @pyqtSlot(bool)
       def on_checkBox_clicked(self,checked):
           self.ui.progressBar.setTextVisible(checked)
   
       @pyqtSlot(bool)
       def on_checkBox_2_clicked(self,checked):
           self.ui.progressBar.setInvertedAppearance(checked)
   
       #自定义槽函数
       def do_valueChanged(self,value):
           self.ui.progressBar.setValue(value)
   
   if __name__ == '__main__':
       app = QApplication(sys.argv)
       form = QMyWidget()
       form.show()
       sys.exit(app.exec_())
   ```

4. QSlider和QScrollerBar

   都是从QAbstractSlider类继承来的，拥有一些相同的属性

   1. minimum 和 maximum：输入范围的最小值和最大值
   2. singleStep：单步长，拖动标尺上的滑块，或按下左/右键时的最小变化数值。
   3. pageStep：输入焦点在组件上时，按PgUp或PgDn键时变化的数值。
   4. value：组件的当前值，拖动滑块时自动改变此值，并限定在minimum和maximum定义的范围之类。
   5. sliderPosition：滑块的位置，若tracking属性设置为True，sliderPosition就等于value。
   6. tracking：sliderPosition是否等同于value，如果tracking设置为True，改变value时也同时改变sliderPosition。
   7. orientation：Slider或ScollBar的方向，可以设置为水平或垂直。方向参数是枚举类型Qt.Orientation，其值包括Qt.Orientation(水平方向)，Qt.Vertical(垂直方向)。
   8. invertedAppearance：显示方式是否反向，若invertedAppearance设置为False，水平的Slider由左向右数值逐渐增大，否则反过来。
   9. invertedControls：反向按键控制，若invertedControls设置为True，则按下PgUp或PgDn键时调整数值的方向相反。

5. Slider的专有属性有两个

   1. tickPosition 标尺刻度的显示位置，使用枚举类型QSlider.TickPosition
      1. QSlider.NoTicks（不显示刻度）
      2. QSlider.TicksBothSides（标尺两侧都显示刻度）
      3. QSlider.TicksAbove（标尺上方显示刻度）
      4. QSlider.TicksBelow（标尺下方显示刻度）
      5. QSlider.TicksLeft（标尺左侧显示刻度）
      6. QSlider.TicksRight（标尺右侧显示刻度）
   2. tickInterval 标尺刻度的间隔值，若设置为0，会在singleStep和pageStep之间自动选择。

6. QSlider和QScollBar最常用的一个信号是valueChanged(int)，在拖动滑块改变当前值时就会发射这个信号。

7. QProcessBar的父类是QWidget，一般用于进度显示，常用的属性有以下几个：

   1. minimum和maximum：最小值和最大值

   2. value：当前值

   3. textVisible：是否显示文字，文字一般是百分比表示的进度

   4. orientation：可以设置为水平或垂直方向。

   5. format：显示文字的格式，“%p%”显示百分比，“%v”显示当前值，“%m”显示总步数。默认是“%p%”

   6. 关于进度条颜色和百分比位置的部分：

      ```python
      self.ui.progressBar.setStyleSheet("QProgressBar::chunk { background-color: rgb(255, 0, 0)}"
                                                "QProgressBar{text-align: center}")
      ```

      

## 8.日期和时间数据

1. 创建ui

   ![](http://mediaqn.meitranslation.com/blog/image-20201203173121144.png)

2. ui转换py

   ```python
   # -*- coding: utf-8 -*-
   
   # Form implementation generated from reading ui file 'DateForm.ui'
   #
   # Created by: PyQt5 UI code generator 5.10.1
   #
   # WARNING! All changes made in this file will be lost!
   
   from PyQt5 import QtCore, QtGui, QtWidgets
   
   class Ui_Form(object):
       def setupUi(self, Form):
           Form.setObjectName("Form")
           Form.resize(800, 410)
           self.horizontalLayout = QtWidgets.QHBoxLayout(Form)
           self.horizontalLayout.setObjectName("horizontalLayout")
           self.groupBox = QtWidgets.QGroupBox(Form)
           self.groupBox.setObjectName("groupBox")
           self.verticalLayout_2 = QtWidgets.QVBoxLayout(self.groupBox)
           self.verticalLayout_2.setObjectName("verticalLayout_2")
           self.btnGetTime = QtWidgets.QPushButton(self.groupBox)
           self.btnGetTime.setObjectName("btnGetTime")
           self.verticalLayout_2.addWidget(self.btnGetTime)
           self.splitter = QtWidgets.QSplitter(self.groupBox)
           self.splitter.setOrientation(QtCore.Qt.Horizontal)
           self.splitter.setObjectName("splitter")
           self.label_2 = QtWidgets.QLabel(self.splitter)
           self.label_2.setObjectName("label_2")
           self.timeEdit = QtWidgets.QTimeEdit(self.splitter)
           self.timeEdit.setObjectName("timeEdit")
           self.editTime = QtWidgets.QLineEdit(self.splitter)
           self.editTime.setObjectName("editTime")
           self.verticalLayout_2.addWidget(self.splitter)
           self.btnSetTime = QtWidgets.QPushButton(self.groupBox)
           self.btnSetTime.setObjectName("btnSetTime")
           self.verticalLayout_2.addWidget(self.btnSetTime)
           self.splitter_2 = QtWidgets.QSplitter(self.groupBox)
           self.splitter_2.setOrientation(QtCore.Qt.Horizontal)
           self.splitter_2.setObjectName("splitter_2")
           self.label_3 = QtWidgets.QLabel(self.splitter_2)
           self.label_3.setObjectName("label_3")
           self.dateEdit = QtWidgets.QDateEdit(self.splitter_2)
           self.dateEdit.setObjectName("dateEdit")
           self.editDate = QtWidgets.QLineEdit(self.splitter_2)
           self.editDate.setObjectName("editDate")
           self.verticalLayout_2.addWidget(self.splitter_2)
           self.btnSetDate = QtWidgets.QPushButton(self.groupBox)
           self.btnSetDate.setObjectName("btnSetDate")
           self.verticalLayout_2.addWidget(self.btnSetDate)
           self.splitter_3 = QtWidgets.QSplitter(self.groupBox)
           self.splitter_3.setOrientation(QtCore.Qt.Horizontal)
           self.splitter_3.setObjectName("splitter_3")
           self.label_4 = QtWidgets.QLabel(self.splitter_3)
           self.label_4.setObjectName("label_4")
           self.dateTimeEdit = QtWidgets.QDateTimeEdit(self.splitter_3)
           self.dateTimeEdit.setObjectName("dateTimeEdit")
           self.editDateTime = QtWidgets.QLineEdit(self.splitter_3)
           self.editDateTime.setObjectName("editDateTime")
           self.verticalLayout_2.addWidget(self.splitter_3)
           self.btnSetDateTime = QtWidgets.QPushButton(self.groupBox)
           self.btnSetDateTime.setObjectName("btnSetDateTime")
           self.verticalLayout_2.addWidget(self.btnSetDateTime)
           self.horizontalLayout.addWidget(self.groupBox)
           self.groupBox_2 = QtWidgets.QGroupBox(Form)
           self.groupBox_2.setObjectName("groupBox_2")
           self.verticalLayout = QtWidgets.QVBoxLayout(self.groupBox_2)
           self.verticalLayout.setObjectName("verticalLayout")
           self.horizontalLayout_2 = QtWidgets.QHBoxLayout()
           self.horizontalLayout_2.setObjectName("horizontalLayout_2")
           self.label = QtWidgets.QLabel(self.groupBox_2)
           self.label.setObjectName("label")
           self.horizontalLayout_2.addWidget(self.label)
           self.editCalendar = QtWidgets.QLineEdit(self.groupBox_2)
           self.editCalendar.setInputMask("")
           self.editCalendar.setObjectName("editCalendar")
           self.horizontalLayout_2.addWidget(self.editCalendar)
           self.verticalLayout.addLayout(self.horizontalLayout_2)
           self.calendarWidget = QtWidgets.QCalendarWidget(self.groupBox_2)
           self.calendarWidget.setObjectName("calendarWidget")
           self.verticalLayout.addWidget(self.calendarWidget)
           self.horizontalLayout.addWidget(self.groupBox_2)
   
           self.retranslateUi(Form)
           QtCore.QMetaObject.connectSlotsByName(Form)
   
       def retranslateUi(self, Form):
           _translate = QtCore.QCoreApplication.translate
           Form.setWindowTitle(_translate("Form", "Form"))
           self.groupBox.setTitle(_translate("Form", "日期时间"))
           self.btnGetTime.setText(_translate("Form", "读取当前日期时间"))
           self.label_2.setText(_translate("Form", "时间"))
           self.timeEdit.setDisplayFormat(_translate("Form", "H:mm:ss"))
           self.editTime.setInputMask(_translate("Form", "99:99:99"))
           self.btnSetTime.setText(_translate("Form", "设置时间"))
           self.label_3.setText(_translate("Form", "日期"))
           self.dateEdit.setDisplayFormat(_translate("Form", "yyyy年M月d日"))
           self.editDate.setInputMask(_translate("Form", "9999-99-99"))
           self.btnSetDate.setText(_translate("Form", "设置日期"))
           self.label_4.setText(_translate("Form", "日期时间"))
           self.dateTimeEdit.setDisplayFormat(_translate("Form", "yyyy-M-d H:mm:ss"))
           self.editDateTime.setInputMask(_translate("Form", "9999-99-99 99:99:99"))
           self.btnSetDateTime.setText(_translate("Form", "设置日期时间"))
           self.groupBox_2.setTitle(_translate("Form", "日历选择"))
           self.label.setText(_translate("Form", "选择的日期："))
   
   ```

3. 业务逻辑类

   ```python
   import sys
   
   from PyQt5.QtCore import QDateTime, QTime, QDate
   from PyQt5.QtWidgets import QWidget, QApplication
   
   import DateForm
   
   class QMyWidget(QWidget):
       def __init__(self,parent=None):
           super().__init__(parent)
           self.ui = DateForm.Ui_Form()
           self.ui.setupUi(self)
   
       # # ============由connectSlotByName() 自动关联的槽函数 ======================
       def on_btnGetTime_clicked(self):
           curDateTime = QDateTime.currentDateTime()
           self.ui.timeEdit.setTime(curDateTime.time())
           self.ui.editTime.setText(curDateTime.toString("hh:mm:ss"))
           self.ui.dateEdit.setDate(curDateTime.date())
           self.ui.editDate.setText(curDateTime.toString("yyyy-MM-dd"))
           self.ui.dateTimeEdit.setDateTime(curDateTime)
           self.ui.editDateTime.setText(curDateTime.toString("yyyy-MM-dd hh:mm:ss"))
   
       def on_calendarWidget_selectionChanged(self):
           date = self.ui.calendarWidget.selectedDate()
           self.ui.editCalendar.setText(date.toString("yyyy年M月d日"))
   
       def on_btnSetTime_clicked(self):  # 设置时间按钮
           tmStr = self.ui.editTime.text()
           tm = QTime.fromString(tmStr, "hh:mm:ss")
           self.ui.timeEdit.setTime(tm)
   
       def on_btnSetDate_clicked(self):  # 设置日期按钮
           dtStr = self.ui.editDate.text()
           dt = QDate.fromString(dtStr, "yyyy-MM-dd")
           self.ui.dateEdit.setDate(dt)
   
       def on_btnSetDateTime_clicked(self):
           dttmStr = self.ui.editDateTime.text()
           dttm = QDateTime.fromString(dttmStr, "yyyy-MM-dd hh:mm:ss")
           self.ui.dateTimeEdit.setDateTime(dttm)
   
   if __name__ == "__main__":
       app = QApplication(sys.argv)  # 创建app，用QApplication类
       form = QMyWidget()
       form.show()
       sys.exit(app.exec_())
   ```

4. 类解析

   1. QTime：时间数据类型，仅表示时间。displayFormat设置为H:mm:ss。如15:21:13
   2. QDate：日期数据类型，仅表示日期。如2018-5-6
   3. QDateTime：日期时间数据类型，表示日期和时间，如2018-05-23 09:12:43

5. 组件

   1. QTimeEdit：编辑和显示时间的组件

   2. QDateEdit：编辑和显示日期的组件。若calendarPopup属性设置为True，运行时右侧按钮变成下拉按钮，单击按钮时出现一个日历选择框，用于在日历上选择日期。

   3. QDateTimeEdit：编辑和显示日期时间的组件，也有calendarPopup属性。

   4. QCalendarWidget：一个用日历形式选择日期的组件，在日历上点击日期发生变化时发射信号selectionChanged()，可响应此信号读取选择的日期。

   5. 为了限定QLineEdit的输入符合某些格式，可以设置其inputMask属性

      | 组件         | input Mask          | 意义                              |
      | ------------ | ------------------- | --------------------------------- |
      | editTime     | 99:99:99            | 只能输入0到9的数字，空格用“_”显示 |
      | editDate     | 9999-99-99          | 只能输入0到9的数字                |
      | editDateTime | 9999-99-99 99:99:99 | 只能输入0到9的数字                |

      


## 9. 定时器和计时器

1. 创建ui

   ![image-20201204114703071](http://mediaqn.meitranslation.com/blog/image-20201204114703071.png)

2. 把ui文件转换成py文件

   ```python
   # -*- coding: utf-8 -*-
   
   # Form implementation generated from reading ui file 'Timer.ui'
   #
   # Created by: PyQt5 UI code generator 5.10.1
   #
   # WARNING! All changes made in this file will be lost!
   
   from PyQt5 import QtCore, QtGui, QtWidgets
   
   class Ui_Form(object):
       def setupUi(self, Form):
           Form.setObjectName("Form")
           Form.resize(400, 300)
           self.verticalLayout = QtWidgets.QVBoxLayout(Form)
           self.verticalLayout.setObjectName("verticalLayout")
           self.groupBox = QtWidgets.QGroupBox(Form)
           self.groupBox.setObjectName("groupBox")
           self.gridLayout = QtWidgets.QGridLayout(self.groupBox)
           self.gridLayout.setObjectName("gridLayout")
           self.btnStart = QtWidgets.QPushButton(self.groupBox)
           self.btnStart.setObjectName("btnStart")
           self.gridLayout.addWidget(self.btnStart, 0, 0, 1, 1)
           self.btnStop = QtWidgets.QPushButton(self.groupBox)
           self.btnStop.setObjectName("btnStop")
           self.gridLayout.addWidget(self.btnStop, 0, 1, 1, 1)
           self.btnSetIntv = QtWidgets.QPushButton(self.groupBox)
           self.btnSetIntv.setObjectName("btnSetIntv")
           self.gridLayout.addWidget(self.btnSetIntv, 1, 0, 1, 1)
           self.spinBoxIntv = QtWidgets.QSpinBox(self.groupBox)
           self.spinBoxIntv.setMinimum(1000)
           self.spinBoxIntv.setMaximum(10000)
           self.spinBoxIntv.setSingleStep(1000)
           self.spinBoxIntv.setProperty("value", 1000)
           self.spinBoxIntv.setObjectName("spinBoxIntv")
           self.gridLayout.addWidget(self.spinBoxIntv, 1, 1, 1, 1)
           self.verticalLayout.addWidget(self.groupBox)
           self.groupBox_2 = QtWidgets.QGroupBox(Form)
           self.groupBox_2.setObjectName("groupBox_2")
           self.horizontalLayout = QtWidgets.QHBoxLayout(self.groupBox_2)
           self.horizontalLayout.setObjectName("horizontalLayout")
           self.LCDHour = QtWidgets.QLCDNumber(self.groupBox_2)
           font = QtGui.QFont()
           font.setPointSize(20)
           self.LCDHour.setFont(font)
           self.LCDHour.setDigitCount(2)
           self.LCDHour.setObjectName("LCDHour")
           self.horizontalLayout.addWidget(self.LCDHour)
           self.LCDMin = QtWidgets.QLCDNumber(self.groupBox_2)
           self.LCDMin.setDigitCount(2)
           self.LCDMin.setObjectName("LCDMin")
           self.horizontalLayout.addWidget(self.LCDMin)
           self.LCDSec = QtWidgets.QLCDNumber(self.groupBox_2)
           self.LCDSec.setSmallDecimalPoint(False)
           self.LCDSec.setDigitCount(2)
           self.LCDSec.setObjectName("LCDSec")
           self.horizontalLayout.addWidget(self.LCDSec)
           self.verticalLayout.addWidget(self.groupBox_2)
           self.LabElapsedTime = QtWidgets.QLabel(Form)
           sizePolicy = QtWidgets.QSizePolicy(QtWidgets.QSizePolicy.Preferred, QtWidgets.QSizePolicy.Fixed)
           sizePolicy.setHorizontalStretch(0)
           sizePolicy.setVerticalStretch(0)
           sizePolicy.setHeightForWidth(self.LabElapsedTime.sizePolicy().hasHeightForWidth())
           self.LabElapsedTime.setSizePolicy(sizePolicy)
           self.LabElapsedTime.setText("")
           self.LabElapsedTime.setObjectName("LabElapsedTime")
           self.verticalLayout.addWidget(self.LabElapsedTime)
   
           self.retranslateUi(Form)
           QtCore.QMetaObject.connectSlotsByName(Form)
   
       def retranslateUi(self, Form):
           _translate = QtCore.QCoreApplication.translate
           Form.setWindowTitle(_translate("Form", "Form"))
           self.groupBox.setTitle(_translate("Form", "定时器"))
           self.btnStart.setText(_translate("Form", "开始"))
           self.btnStop.setText(_translate("Form", "停止"))
           self.btnSetIntv.setText(_translate("Form", "设置周期"))
           self.spinBoxIntv.setSuffix(_translate("Form", "ms"))
           self.groupBox_2.setTitle(_translate("Form", "当前时间"))
   
   ```

3. 创建逻辑类

   ```python
   import sys
   from PyQt5.QtWidgets import QApplication, QWidget
   from PyQt5.QtCore import QTime, QTimer
   
   import Timer
   
   
   class QMyWidget(QWidget):
       def __init__(self, parent=None):
           super().__init__(parent)
           self.ui = Timer.Ui_Form()
           self.ui.setupUi(self)
   
           self.timer = QTimer()  # 创建定时器
           self.timer.stop()  # 停止
           self.timer.setInterval(1000)  # 定时周期1000ms
           self.timer.timeout.connect(self.do_timer_timeout)
           self.counter = QTime()  # 创建计时器
   
       def on_btnStart_clicked(self):
           self.timer.start()  # 开始定时
           self.counter.start()
           self.ui.btnStart.setEnabled(False)
           self.ui.btnStop.setEnabled(True)
           self.ui.btnSetIntv.setEnabled(False)
   
       def on_btnSetIntv_clicked(self):  # #设置定时器的周期
           self.timer.setInterval(self.ui.spinBoxIntv.value())
   
       def on_btnStop_clicked(self):
           self.timer.stop()  # 定时器停止
           tmMs = self.counter.elapsed()  # 计时器经过的毫秒数, 这个时候counter其实还在运行,重新start之后从0开始计时
           ms = tmMs % 1000  # 取余数，毫秒
           sec = tmMs/1000  # 整秒
           timeStr = "经过的时间：%d 秒，%d 毫秒" % (sec, ms)
           self.ui.LabElapsedTime.setText(timeStr)
           self.ui.btnStart.setEnabled(True)
           self.ui.btnStop.setEnabled(False)
           self.ui.btnSetIntv.setEnabled(True)
   
       def do_timer_timeout(self):  # 定时中断响应
           curTime = QTime.currentTime()  # 获取当前时间
           self.ui.LCDHour.display(curTime.hour())
           self.ui.LCDMin.display(curTime.minute())
           self.ui.LCDSec.display(curTime.second())
   
   
   if __name__ == "__main__":
       app = QApplication(sys.argv)  # 创建app，用QApplication类
       form = QMyWidget()
       form.show()
       sys.exit(app.exec_())
   ```

4. PyQt5中的定时器类是QTimer。QTimer不是一个可见的界面组件，在Qt Designer的组件面板里找不到它。

   - QTimer主要的属性是interval，是定时中断的周期，单位是毫秒。
   - QTimer主要的信号是timeout()，在定时中断时发射此信号，若要在定时中断里作出响应，就需要编写与timeout()信号关联的槽函数。

5. 代码分析

   1. 在构造函数中创建了定时器对象self.timer并立刻停止。设置定时周期为1000ms，并为其timeout()信号关联了自定义槽函数do_timer_timeout()。还创建了一个计时器对象self.counter，这是一个QTime类的实例，用于在开始与停止之间计算经过的时间。
   2. 定时器开始运行,点击“开始”按钮后，定时器开始运行，计时器也开始运行。定时器的定时周期到了之后发射timeout()信号，触发关联的自定义槽函数do_timer_timeout()执行，此槽函数的功能通过类函数QTime.currentTime()读取当前时间，然后将时、分、秒显示在三个LCD组件上。
   3. 定时器停止运行,点击“停止”按钮时，定时器停止运行。计时器通过调用elapsed()函数可以获得上次执行start()之后经过的毫秒数。



## 10. 下拉列表框

1. 创建ui

   ![image-20201204183434189](http://mediaqn.meitranslation.com/blog/image-20201204183434189.png )

2. ui转换py文件

   ```python
   # -*- coding: utf-8 -*-
   
   # Form implementation generated from reading ui file 'QComboBox.ui'
   #
   # Created by: PyQt5 UI code generator 5.10.1
   #
   # WARNING! All changes made in this file will be lost!
   
   from PyQt5 import QtCore, QtGui, QtWidgets
   
   class Ui_Form(object):
       def setupUi(self, Form):
           Form.setObjectName("Form")
           Form.resize(564, 334)
           self.verticalLayout = QtWidgets.QVBoxLayout(Form)
           self.verticalLayout.setObjectName("verticalLayout")
           self.groupBox = QtWidgets.QGroupBox(Form)
           self.groupBox.setObjectName("groupBox")
           self.verticalLayout_2 = QtWidgets.QVBoxLayout(self.groupBox)
           self.verticalLayout_2.setObjectName("verticalLayout_2")
           self.lineEdit = QtWidgets.QLineEdit(self.groupBox)
           self.lineEdit.setObjectName("lineEdit")
           self.verticalLayout_2.addWidget(self.lineEdit)
           self.verticalLayout.addWidget(self.groupBox)
           self.frame = QtWidgets.QFrame(Form)
           self.frame.setFrameShape(QtWidgets.QFrame.StyledPanel)
           self.frame.setFrameShadow(QtWidgets.QFrame.Raised)
           self.frame.setObjectName("frame")
           self.horizontalLayout = QtWidgets.QHBoxLayout(self.frame)
           self.horizontalLayout.setObjectName("horizontalLayout")
           self.groupBox_2 = QtWidgets.QGroupBox(self.frame)
           self.groupBox_2.setObjectName("groupBox_2")
           self.verticalLayout_3 = QtWidgets.QVBoxLayout(self.groupBox_2)
           self.verticalLayout_3.setObjectName("verticalLayout_3")
           self.splitter = QtWidgets.QSplitter(self.groupBox_2)
           self.splitter.setOrientation(QtCore.Qt.Horizontal)
           self.splitter.setObjectName("splitter")
           self.btnIniItems = QtWidgets.QPushButton(self.splitter)
           self.btnIniItems.setObjectName("btnIniItems")
           self.pushButton_2 = QtWidgets.QPushButton(self.splitter)
           self.pushButton_2.setObjectName("pushButton_2")
           self.chkBoxEditable = QtWidgets.QCheckBox(self.splitter)
           self.chkBoxEditable.setObjectName("chkBoxEditable")
           self.verticalLayout_3.addWidget(self.splitter)
           self.comboBox = QtWidgets.QComboBox(self.groupBox_2)
           self.comboBox.setObjectName("comboBox")
           self.verticalLayout_3.addWidget(self.comboBox)
           self.horizontalLayout.addWidget(self.groupBox_2)
           self.groupBox_3 = QtWidgets.QGroupBox(self.frame)
           self.groupBox_3.setObjectName("groupBox_3")
           self.verticalLayout_4 = QtWidgets.QVBoxLayout(self.groupBox_3)
           self.verticalLayout_4.setObjectName("verticalLayout_4")
           self.btnIni2 = QtWidgets.QPushButton(self.groupBox_3)
           self.btnIni2.setObjectName("btnIni2")
           self.verticalLayout_4.addWidget(self.btnIni2)
           self.comboBox_2 = QtWidgets.QComboBox(self.groupBox_3)
           self.comboBox_2.setObjectName("comboBox_2")
           self.verticalLayout_4.addWidget(self.comboBox_2)
           self.horizontalLayout.addWidget(self.groupBox_3)
           self.verticalLayout.addWidget(self.frame)
   
           self.retranslateUi(Form)
           QtCore.QMetaObject.connectSlotsByName(Form)
   
       def retranslateUi(self, Form):
           _translate = QtCore.QCoreApplication.translate
           Form.setWindowTitle(_translate("Form", "Form"))
           self.groupBox.setTitle(_translate("Form", "选择的内容"))
           self.groupBox_2.setTitle(_translate("Form", "简单的ComboBox"))
           self.btnIniItems.setText(_translate("Form", "初始化列表"))
           self.pushButton_2.setText(_translate("Form", "清除列表"))
           self.chkBoxEditable.setText(_translate("Form", "可编辑"))
           self.groupBox_3.setTitle(_translate("Form", "有用户数据的ComboBox"))
           self.btnIni2.setText(_translate("Form", "初始化城市+区号"))
   
   
   ```

3. 创建逻辑类

   ```python
   import sys
   from PyQt5.QtCore import pyqtSlot
   from PyQt5.QtGui import QIcon
   from PyQt5.QtWidgets import QWidget, QApplication
   
   import QComboBox
   
   class QMyWidget(QWidget):
       def __init__(self,parent=None):
           super().__init__(parent)
           self.ui = QComboBox.Ui_Form()
           self.ui.setupUi(self)
   
           self.ui.btnIni2.clicked.connect(self.on_btn_ini2_clicked)
   
           self.ui.comboBox_2.currentIndexChanged[str].connect(self.combo_box_current_index_change)
   
       def on_btnIniItems_clicked(self):
           icon = QIcon("./icons/images/icon.png")
           self.ui.comboBox.clear()
           provinces = ["山东", "河北", "河南", "湖北", "湖南", "广东"]  # 列表数据
           # self.ui.comboBox.addItems(provinces)  # 直接添加列表，但无法加图标
           for i in range(len(provinces)):
               self.ui.comboBox.addItem(icon, provinces[i])
   
       @pyqtSlot(bool)
       def on_chkBoxEditable_clicked(self,checked):
           self.ui.comboBox.setEditable(checked)
   
       @pyqtSlot(str)
       def on_comboBox_currentIndexChanged(self,curText):
           self.ui.lineEdit.setText(curText)
   
       def on_btn_ini2_clicked(self):
           icon = QIcon("./icons/images/icon.png")
           self.ui.comboBox_2.clear()
           cities =  {"北京": 10, "上海": 21, "天津": 22, "徐州": 516, "福州": 591, "青岛": 532}  # 字典数据
           for k in cities:
               self.ui.comboBox_2.addItem(icon,k,cities[k])
   
   
       def combo_box_current_index_change(self,curText):
           self.ui.lineEdit.setText(curText)
           zone = self.ui.comboBox_2.currentData()
           if zone is not None:
               self.ui.lineEdit.setText(curText + ":区号=%d" % zone)
   
   
   if __name__ == '__main__':
       app = QApplication(sys.argv)
       form = QMyWidget()
       form.show()
       sys.exit(app.exec_())
      
   ```

### QComboBox常用函数总结

QComboBox存储的项是一个列表，但是QComboBox不提供整个列表用于访问，而可以通过索引访问某个项。访问项的一些函数主要有以下几个。

1. currentIndex()：返回当前项的序号，第一项的序号为0。
2. currentText()：返回当前项的文字。
3. currentData(role)：返回当前项的关联数据，参数role表示数据角色，角色role的默认值为Qt.UserRole。可以为一个项定义多个角色的用户数据，更多自定义角色的编号从Qt.UserRole开始增加，如Qt.UserRole+1、Qt.UserRole+2.
4. itemText(index)：返回索引号为index的项的文字。
5. itemData(index, role)：返回索引号为index的项的角色为role的关联数据，角色role的默认值为Qt.UserRole。
6. count()：返回项的个数。



## 11. WebView使用

1. 创建ui

   ![image-20201204192413199](../../../../.config/Typora/typora-user-images/image-20201204192413199.png)

2. 转换ui文件到py

   ```python
   # -*- coding: utf-8 -*-
   
   # Form implementation generated from reading ui file 'WebBrowser.ui'
   #
   # Created by: PyQt5 UI code generator 5.10.1
   #
   # WARNING! All changes made in this file will be lost!
   
   from PyQt5 import QtCore, QtGui, QtWidgets
   
   class Ui_MainWindow(object):
       def setupUi(self, MainWindow):
           MainWindow.setObjectName("MainWindow")
           MainWindow.resize(800, 600)
           self.centralwidget = QtWidgets.QWidget(MainWindow)
           self.centralwidget.setObjectName("centralwidget")
           self.verticalLayout = QtWidgets.QVBoxLayout(self.centralwidget)
           self.verticalLayout.setObjectName("verticalLayout")
           self.webView = QWebEngineView(self.centralwidget) ## update to QWebEngineView
           self.webView.setUrl(QtCore.QUrl("https://www.baidu.com/"))
           self.webView.setObjectName("webView")
           self.verticalLayout.addWidget(self.webView)
           MainWindow.setCentralWidget(self.centralwidget)
           self.menubar = QtWidgets.QMenuBar(MainWindow)
           self.menubar.setGeometry(QtCore.QRect(0, 0, 800, 30))
           self.menubar.setObjectName("menubar")
           MainWindow.setMenuBar(self.menubar)
           self.statusbar = QtWidgets.QStatusBar(MainWindow)
           self.statusbar.setObjectName("statusbar")
           MainWindow.setStatusBar(self.statusbar)
   
           self.retranslateUi(MainWindow)
           QtCore.QMetaObject.connectSlotsByName(MainWindow)
   
       def retranslateUi(self, MainWindow):
           _translate = QtCore.QCoreApplication.translate
           MainWindow.setWindowTitle(_translate("MainWindow", "MainWindow"))
   
   from PyQt5.QtWebEngineWidgets import QWebEngineView ## update to QWebEngineView
   
   ```

   当前使用的版本是5.15 默认不包含webEngine模块需要单独安装 `pip install PyQtWebEngine`

   安装之后我的pip list 

   ```bash
   Package          Version
   ---------------- ------------
   click            7.1.2
   pip              20.2.2
   PyQt5            5.15.1
   pyqt5-plugins    5.15.1.2.0.1
   PyQt5-sip        12.8.1
   pyqt5-tools      5.15.1.3
   PyQtWebEngine    5.15.2
   python-dotenv    0.15.0
   qt5-applications 5.15.1.2
   qt5-tools        5.15.1.1.0.1
   setuptools       50.2.0
   wheel            0.35.1
   ```

3. 创建业务逻辑类

   ```python
   import sys
   from PyQt5 import QtCore
   
   from PyQt5.QtWidgets import QWidget, QApplication, QMainWindow
   
   import WebBrowser
   
   class QMyWidget(QMainWindow):
       def __init__(self,parent=None):
           super().__init__(parent)
           self.ui = WebBrowser.Ui_MainWindow()
           self.ui.setupUi(self)
   
   
   if __name__ == '__main__':
       app = QApplication(sys.argv)
       win = QMyWidget()
       win.ui.webView.load(QtCore.QUrl("http://jinyun.github.io"))
       win.show()
   
       sys.exit(app.exec_())
   ```

   

