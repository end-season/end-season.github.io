---
title: QML_4
data: 2026-3-11
tags: QML
categories: QML
cover: /img_post/qml_cover.png  
---

# `Component`与`Loader`

本文章来源于B站up 洛雨薄青衫 的qml的教学视频 [视频链接](https://www.bilibili.com/video/BV1dS4y1u7vN/)



* 在窗口被创建时，会触发`Component.onCompleted`，当窗口被销毁时，会触发`Component.onDestruction`：

  ```qml
  //当界面被创建时触发
      Component.onCompleted: {
          console.log("onCompleted", color, width, height)
      }
  
      //当界面被销毁时触发
      Component.onDestruction: {
          console.log("onDestruction")
      }
  ```

* 可以实现自定义组件，自定义组件必须内置一个组件，以一个矩形为例：

  ```qml
   Component {
          id: com
          Rectangle {
              height:100; width: 200
              color: "black"
          }
      }
  ```

  组件不会被立刻触发，需要手动触发。也就是说，当运行时，这个矩形并不会被创建出来。

* 可以使用`Loader`触发组件：

  ```qml
  Loader {
          id: loader
          //source: "/Myrectangle.qml"
          sourceComponent: com
      }
  ```

  使用`source:`可以触发外部组件，如在[QML_2](https://end-season.github.io/2026/03/09/QML_2/)中实现的自定义的矩形`Myrectangle`。

  使用`sourceComponent:`可以触发在相同文件中已经创建的组件，比如刚刚创建的`com`。

* 自定义组件有`status`属性，默认为1，在被销毁时为0。看以下示例：

  ```qml
   Component {
          id: com
          Rectangle {
              height:100; width: 200
              color: "black"
          }
      }
  
      Loader {
          id: loader
          //source: "/Myrectangle.qml"
          sourceComponent: com
          onStatusChanged: {
              console.log("status: ", status)
          }
      }
  
      Button {
          height: 50; width:50
          x: 200
          onClicked: {
              loader.sourceComponent = null
          }
      }
  ```

  在`Loader`中，当状态改变时，会打印相应的日志信息。我们又创建了一个新的按钮，当按钮被点击时，使用`loader.sourceComponent`改变加载中的组件的状态，这里我们将其设为`null`，也就是这个组件会被删除。当我们运行时，点击按钮，黑色的矩形就会消失。

* 可以通过`loader.item`访问自定义组件中的`item`的属性（loader为任意`Loader`的`id`），以刚刚的示例为例，可以通过这种方式访问到`Component`中定义的矩形的属性。就像以下示例一样：

  ```qml
  Component {
          id: com
          Rectangle {
              height:100; width: 200
              color: "black"
          }
      }
  
      Loader {
          id: loader
          //source: "/Myrectangle.qml"
          sourceComponent: com
          onStatusChanged: {
              console.log("status: ", status)
          }
      }
  
      Button {
          height: 50; width:50
          x: 200
          onClicked: {
              loader.item.color = "red"
          }
      }
  ```

  当按钮被点击时，黑色的矩形变为红色。

* 当正在加载资源时，`Loader`的`status`会变为2。在刚刚的示例中，在`Loader`中将`asynchronous:`设置为true：

  ```qml
  Component {
          id: com
          Rectangle {
              height:100; width: 200
              color: "black"
          }
      }
  
      Loader {
          id: loader
          asynchronous: true
          //source: "/Myrectangle.qml"
          sourceComponent: com
          onStatusChanged: {
              console.log("status: ", status)
          }
      }
  
      Button {
          height: 50; width:50
          x: 200
          onClicked: {
              loader.item.color = "red"
          }
      }
  ```

  在运行时，就会先打印“status: 2”，后打印“status：1”，表示sourceComponen被加载的过程。

* 可以用以下方式加载图片：

  ```qml
   Component {
          id: com
          Image {
              id: img
              source:"/test.png"
          }
      }
  ```

  之后加载就与之前一样。

  如果所加载的图片是动图，将`Image`替换为`AnimatedImage`即可。

> `Image`,`AnimatedImage`中还有许多属性，可以去官方文档进行查阅。