+++
date = '2022-06-01T12:00:00+08:00'
draft = true
title = 'Qt5.12学习记录（已废弃）'
tags = ['coding', 'qt']
categories = ['coding']
+++

# Qt学习笔记

lamaper  2022/6/1

参考网站[Qt编程（1） - 子卿の小站 (baiziqing.cn)](http://www.baiziqing.cn/index.php/archives/26/)

参考教程https://www.bilibili.com/video/BV1t64y1f7d1

## 第一章 Qt的基本使用

### 1、QtCreator快捷键（1）

选中某一主类 F1 查看开发文档，F2查看源文件，

进入.h文件 F4 切换至对应的.cpp文件。

### 2、 基础知识和QPushButton

```c++
#include "widget.h"

#include <QApplication>

int main(int argc, char *argv[])
{
    //应用程序类
    QApplication a(argc, argv);//每个Qt程序只有一个
    Widget w;//窗口类，创建后默认不显示
    w.show();
    return a.exec();
}

```

Qt的基本框架（.pro）

```c++
# 在项目文件中, 注释需要使用 井号(#)
# 项目编译的时候需要加载哪些底层模块
QT       += core gui 

# 如果当前Qt版本大于4, 会添加一个额外的模块: widgets
# Qt 5中对gui模块进行了拆分, 将 widgets 独立出来了
greaterThan(QT_MAJOR_VERSION, 4): QT += widgets
   
# 使用c++11新特性
CONFIG += c++11

#如果在项目中调用了废弃的函数, 项目编译的时候会有警告的提示  
DEFINES += QT_DEPRECATED_WARNINGS

# 项目中的源文件
SOURCES += \
        main.cpp \
        mainwindow.cpp
  
# 项目中的头文件
HEADERS += \
        mainwindow.h
  
# 项目中的窗口界面文件
FORMS += \
        mainwindow.ui
```

以Qwidget为例。

使用QPushButton首先需要在主窗口头文件的头文件中导入相应头文件：

```c++
#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>
#include <QPushButton>

#endif // WIDGET_H
```

#### QtPushButton相应的方法（1）

```c++
QPushButton q1;
QPushButton *q2;
q2 = new QPushButton(const QIcon &icon, const QString &text, QWidget *parent = nullptr);//(按钮图标, 按钮上显示的文字, QWidget类型的父类-表示q2依附于某类)

q2->show();//在父类上显示该控件
q1.show();

q2->setParent(this);//设置父类
q1.setParent(this);//this表示当前父类

q2->move(const &int，const &int);//窗口的坐标系，原点在左上角，X轴向右递增，Y轴向下递增，理论上不存在负轴
q1.move();

q1.resize(const &int，const &int);//设置按钮的大小,父类是Qweiget
```

#### Qt存在垃圾自动回收机制，会自动回收：

1. QObject的派生类或自己；
2. 指定父类，先析构子类再析构父类；

#### Qt新建一个Button类

![image-20220601202337528](E:\lamaper\QtNote\image-20220601202337528.png)

右键工程文件夹，选择Add New...

![image-20220601202509169](E:\lamaper\QtNote\image-20220601202509169.png)

![image-20220601202550062](E:\lamaper\QtNote\image-20220601202550062.png)

因为Qt选项中没有QPushButton作为继承选项，所以选择widget现行代替，之后修改头文件中继承的类；

![image-20220601202703879](E:\lamaper\QtNote\image-20220601202703879.png)

然后修改源文件中的继承类；

![image-20220601202728943](E:\lamaper\QtNote\image-20220601202728943.png)

#### QDebug的使用（1）

```c++
#include <QDebug>
qDebug() << "helloworld ;"//类似cout的标准输出
```

#### QWidget相应的方法（1）

```c++
#include "widget.h"
#include "ui_widget.h"

Widget::Widget(QWidget *parent)
    : QWidget(parent)
    , ui(new Ui::Widget)
{
    ui->setupUi(this);
    this->setWindowTitle(cosnt &string)//设置窗口标题
    this->resize();//窗口大小
    this->setFixedSize();//设置不可变更的窗口大小
    this->setWindowIcon(QIcon(<绝对路径>));//设置窗口图标
    
}

```

### 3、信号和槽

#### 标准信号和槽

```c++
//connect(信号发出者，发出的信号，信号接受者，处理信号的槽函数);
connect(const &provider ,const &信号发出者类的名字::信号的名字 , const &saver , &处理信号者类的名字::槽的名字);
```

需要注意的是，connect函数中四个参数均为指针，必须对对象进行取址。

#### 自定义槽函数

1. 槽函数在Qt5中可以是任意成员函数、全局函数、静态函数、lambda表达式；

2. 槽函数要与信号相对应；

   ```c++
   void mysign(int ,double ,Qstrting);
   int mysolt(int ,double ,Qstring);
   //上下形参一一对应，形参是为了接收信号数据
   //槽函数形参个数应小于等于信号的形参个数
   ```

3. 信号没有返回值，槽函数拥有返回值；

**!注意<font color="#dd0000"> 信号和槽虽然是函数，但不能携带括号和形参值，否则会报错！</font> **

#### 自定义信号函数

```c++
class MyButton : public QPushButton
{
    Q_OBJECT
public:
    explicit MyButton(QWidget *parent = nullptr);

signals://声明信号
    void tteessstt();//信号函数
};
```

信号函数可以被重载，可以有形成，返回值为void;

发送信号 `emit tteesstt;`

