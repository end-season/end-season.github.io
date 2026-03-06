---
title: QML_1
data: 2026-3-5
tags: QML
categories: QML
cover: /img_post/qml_cover.png
---

# QML-Window介绍

> 本文章来源于B站up 洛雨薄青衫 的qml的教学视频 [视频链接](https://www.bilibili.com/video/BV1dS4y1u7vN/)

* 首先创建出一个QML工程，并打开`main.qml`文件。

  > 使用Qt6.8，Qt Creator 18.0.2创建Qt Quick Application

  ![1.1](/img_post/qml_1.1.png)
  
* 点击运行后发现生成了一个宽度640，高度480，标题为`Hello World`的窗口，此窗口对应的属性，也就是qml文件中`Window`的属性。

  ![1.2](/img_post/qml_1.2.png)

>  打开帮助页面，查找Window可以查到Window的其他信息。

## Window的常用属性介绍

* `width` `height` `visible` `title`:

  `width`与`heigth`控制窗口大小，`visible`控制窗口是否可见（若改为`false`则运行时不会显示此窗口），`title`控制窗口标题。

* `x` `y`:

  `x`与`y`控制窗口相对于**父窗口**的显示位置，均以左上角为（0，0）坐标。

  > x， y可以为负数。

* `minimumHeight` `minimumWidth` `maximumHeight` `maximumWidth`:

  在默认的窗口中，窗口的大小可以被随意拉伸，从而改变大小。这四个属性可以设置窗口被拉伸的最大与最小大小。

  > 若将最大窗口大小与最小窗口大小设置为相同的数值，则窗口大小固定，不可被拉伸。

* `opacity`:

  设置窗口的透明度，大小为0-1的实数，默认为1。

* `color`:

  设置当前窗口的背景颜色。

## 信号和槽

* qml依然存在信号和槽，自定义信号和槽的方式如下：

  ```qml
  Window {
      width: 640
      height: 480
      visible: true
      title: qsTr("QML_1")
      
      signal my_signal()
      
      onMy_signal: {
          
      }
  }
  ```

  > 与信号对应的槽的名字为on+信号名，信号名的首字母需要大写。

* 在window的属性被修改时，也会触发对应的槽函数，以`width`为例：

  ```qml
  Window {
      width: 200
      height: 480
      visible: true
      title: qsTr("QML_1")
      
      onWidthChanged: {
          console.log("width", width)
      }
  }
  ```

  这样，当我们在拖动窗口改变`width`的值的时候，控制台会输出相应的日志信息。

  > 每一个属性都有对应的槽函数。

  自定义的变量也会自动生成对应的changed槽函数：

  ![1.3](/img_post/qml_1.3.png)

* `activeFocusItem`:

  在窗口中有多个控件时，我们会经常弄不清楚当前焦点在哪个控件上。当焦点切换时，可以用`onActiveFocusItemChanged`这个槽，请看以下示例：

  ```qml
  Button{
          //设置id（用于标识，使之可以被别的控件访问）
          id: btn1
  
          //设置objectname
          objectName: "button1"
  
          //设置高度和宽度
          width: 50
          height: 50
  
          //默认焦点在btn1上
          focus: true
  
          //设置按钮边框在被聚焦时为蓝色，否则为黑色。
          background: Rectangle{
              border.color: btn1.focus ? "blue" : "black"
          }
  
          //当按钮被点击时打印日志
          onClicked: {
              console.log("btn1 clicked")
          }
     
          //当键盘右方向键被按下时，切换焦点到btn2
          Keys.onRightPressed: {
              btn2.focus = true
          }
      }
  
      Button{
          //设置id（用于标识，使之可以被别的控件访问）
          id: btn2
  
          //设置objectname
          objectName: "button2"
  
          //设置高度和宽度
          width: 50
          height: 50
  
          //设置坐标
          x: 100
          y: 100
  
          //设置按钮边框在被聚焦时为蓝色，否则为黑色。
          background: Rectangle{
              border.color: btn2.focus ? "blue" : "black"
          }
  
          //当按钮被点击时打印日志
          onClicked: {
              console.log("btn2 clicked")
          }
  
          //当键盘左方向键被按下时，切换焦点到btn1
          Keys.onLeftPressed: {
              btn1.focus = true
          }
      }
  
      //当焦点切换时，打印日志信息并打印获得焦点的item以及item的objectname
      onActiveFocusItemChanged: {
          console.log("active focus item changed ", activeFocusItem,"object name:", activeFocusItem.objectName)
      }
  
  ```

  >示例中的代码只是Window中的一部分

  在这个示例中，我们创建了两个按钮，分别为两个按钮设置了`id` `objectName`以及大小，并设置了btn2的位置，防止与btn1重合。之后，我们为了方便看出焦点在哪个按钮上，为按钮设置了按钮边框在非聚焦与聚焦时的颜色。接着，我们设置了当按钮被点击时的槽。最后设置了键盘左右键与按钮焦点切换的对应关系：左键焦点在btn1，右键在btn2。

  之后，我们设置了`onActiveFocusItemChanged`，这个槽会在焦点切换时被触发，并可以用`activeFocusItem`与`activeFocusItem.objectName`获取当前焦点在哪个控件上。

  这样，我们就能通过`onActiveFocusItemChanged`这个槽来知道当前的焦点在哪个控件上。