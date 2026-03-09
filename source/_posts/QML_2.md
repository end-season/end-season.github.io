---
title: QML_2
data: 2026-3-6
tags: QML
categories: QML
cover: /img_post/qml_cover.png 
---

# `item`与`Rectangle`

本文章来源于B站up 洛雨薄青衫 的qml的教学视频 [视频链接](https://www.bilibili.com/video/BV1dS4y1u7vN/)



## `Rectangle`

`rectangle`是继承自`item`的，以下先介绍`item`的一些常用属性:

* `rectangle`可以绘制出一个带有可选边框的填充矩形。

  ```qml
  Window {
      width: 640
      height: 480
      visible: true
      title: qsTr("Hello World")
  
      Rectangle{
          x: 100
          y: 100
          width: 100
          height: 50
          color: "black"
      }
  }
  ```

  运行结果：

  ![](/img_post/qml_2.1.png)

  当我们再创建一个矩形和原来的矩形有部分重合的时候：

  ```qml
  Rectangle{
          x: 120
          y: 120
          width: 100
          height: 50
          color: "blue"
      }
  ```

  会发现新创建的蓝色的矩形会将原来的黑色的矩形部分覆盖：

  ![](/img_post/qml_2.2.png)

  也就是说，后来创建的矩形会比原先创建的矩形**“离屏幕更近”**

  我们可以通过设置`rectangle`的`z`属性来调整位置：

  > z的默认值为0。

  ```qml
  Rectangle{
          x: 100
          y: 100
          z: 1
          width: 100
          height: 50
          color: "black"
      }
  
      Rectangle{
          x: 120
          y: 120
          width: 100
          height: 50
          color: "blue"
      }
  ```

  

  ![](/img_post/qml_2.3.png)

  * `focus`:

    `regtangle`同样有`focus`属性，我们看下面这些代码：

    ```qml
    Rectangle{
            x: 100
            y: 100
            z: 1
            width: 100
            height: 50
            color: "black"
    
            focus: true
    
            MouseArea{
                anchors.fill: parent
                onClicked: {
                    console.log("on clicked")
                }
            }
            
            Keys.onReturnPressed: {
                console.log("return pressed")
            }
        }
    ```

    这段代码的意思是：当我们按回车键或者鼠标点击了矩形后，打印日志信息。

    但是当我们将focus去掉或改为false后：

    ```qml
    Rectangle{
            x: 100
            y: 100
            z: 1
            width: 100
            height: 50
            color: "black"
    
            MouseArea{
                anchors.fill: parent
                onClicked: {
                    console.log("on clicked")
                }
            }
            
            Keys.onReturnPressed: {
                console.log("return pressed")
            }
        }
    ```

    运行程序后按回车键会发现没有对应日志信息，这是因为焦点不在当前矩形上。

* `anchors`:

  当我们创建两个矩形并且不为矩形设置坐标时，矩形会默认生成在左上角，也就是说两个矩形会重合。

  如果想要两个矩形不重合，可以为其设置坐标，但是这样的话，设置的坐标就会有些绝对，我们可以使用`anchors`：

  ```qml
  Rectangle{
          id: rec1
          width: 100
          height: 50
          color: "black"
      }
  
      Rectangle{
          id: rec2
          anchors.left: rec1.right
          width: 100
          height: 50
          color: "black"
      }
  ```

  * 运行结果：

    ![](/img_post/qml_2.4.png)

  ​	我们将`rec2`的左边设置为`rec1`的右边，这样两个矩形就不会重合了，而我们要想让两个矩形间隔开，可以为`rec2`加上margin:

  ```qml
   Rectangle{
          id: rec1
          width: 100
          height: 50
          color: "black"
      }
  
      Rectangle{
          id: rec2
          anchors.left: rec1.right
          anchors.margins: 20
          width: 100
          height: 50
          color: "black"
      }
  ```

  * `anchors.centerIn`:

    将控件居中到父窗口：

    ```qml
    Rectangle {
            height: 50
            width: 100
            color: "black"
            anchors.centerIn: parent
        }
    ```
  
    > 会在窗口中间生成一个黑色矩形
  
  * `anchors.horizontalCenter`、`anchors.verticalCenter`
  
    水平居中与垂直居中：
  
    ```qml
    Rectangle {
            height: 50
            width: 100
            color: "black"
            anchors.horizontalCenter: parent.horizontalCenter
        }
    
        Rectangle {
            height: 50
            width: 100
            color: "blue"
            anchors.verticalCenter: parent.verticalCenter
        }
    ```
  
    > 在父窗口顶部中间位置生成黑色矩形，在父窗口左侧中间位置生成蓝色矩形。
  
* `rotation`:

  旋转操作：

  ```qml
  Rectangle {
          width: 100
          height: 50
          color: "black"
          rotation: 90
      }
  ```

  > 创建一个矩形并旋转90°。

* `scale`:

  放缩操作：

  ```qml
   Rectangle {
          width: 100
          height: 50
          color: "black"
          scale: 2
      }
  ```

  > 创建出一个宽100，高50的黑色矩形，并放大两倍，即宽200，高100

  

接下来是`rectangle`的一些属性：

* `antialiasing`:

  抗锯齿

  ```qml
  Rectangle {
          id: rec1
          width: 100
          height: 50
          color: "black"
          antialiasing: false
          rotation: 40
      }
  
      Rectangle {
          width: 100
          height: 50
          color: "black"
          anchors.left: rec1.right
          antialiasing: true
          rotation: 40
      }
  ```

  我们创建两个矩形来对比看一下区别:

  ![](/img_post/qml_2.5.png)

  可以明显看出第一个矩形边缘有锯齿形。

* `border`

  边框，可以为边框设置范围和颜色：

  ```qml
  Rectangle {
          width: 100
          height:50
          color: "black"
          border.width: 20
          border.color: "blue"
      }
  ```

  这里我们为边框宽度设置为20，颜色设置为蓝色。需要注意的是，边框是向内收缩的，也就是说，在边框为20的情况下，黑色矩形部分就剩下宽度60，高度10了。

* `radius`:

  ```qml
   Rectangle {
          width: 100
          height:50
          color: "black"
          radius: 10
      }
  ```

  这样便会创建一个圆角矩形，`radius`的数值越大，圆角就会越明显。

* `gradient`

  控制渐变颜色，以下是官方文档中的示例：

  ![](/img_post/qml_2.6.png)

  ```qml
  Rectangle {
          width: 80; height: 80
          color: "lightsteelblue"
      }
     
      Rectangle {
          width: 80; height: 80
          gradient: Gradient {
              GradientStop { position: 0.0; color: "lightsteelblue" }
              GradientStop { position: 1.0; color: "blue" }
          }
      }
     
      Rectangle {
          width: 80; height: 80
          gradient: Gradient {
              orientation: Gradient.Horizontal
              GradientStop { position: 0.0; color: "lightsteelblue" }
              GradientStop { position: 1.0; color: "blue" }
          }
      }
     
      Rectangle {
          width: 80; height: 80
          rotation: 90
          gradient: Gradient {
              GradientStop { position: 0.0; color: "lightsteelblue" }
              GradientStop { position: 1.0; color: "blue" }
          }
      }
  ```

  

## 实现自定义边框的`rectangle`

首先创建一个新的`QML File`，名字为`Myrectangle`，我们在这里实现自定义边框的`rectangle`:

```qml
Rectangle {
    id: border_rect
    property int top_margin: 0
    property int bottom_margin: 0
    property int left_margin: 0
    property int right_margin: 0
    color: "black"
    Rectangle {
        id: inner_rect
        color: "blue"
        anchors.fill: parent
        anchors.topMargin: top_margin
        anchors.bottomMargin: bottom_margin
        anchors.leftMargin: left_margin
        anchors.rightMargin: right_margin
    }
}
```

先创建的矩形是边框，我们为它设置id和颜色，并定义了四个属性，代表四个边框的大小，这四个属性将作为接口暴露出去。

接着在内层再创建一个矩形，这个矩形就是内部的矩形，设置了id以及颜色后，将当前矩形填充到父窗口中，也就是边框矩形中，再为内部的矩形设置`margin`，这样便可以将边框显示出来，而`margin`的大小又可以在暴露的接口中进行调整，这样就可以实现自定义边框的矩形。

我们回到`Main.qml`中使用这个自定义边框矩形：

```qml
 Myrectangle {
        width: 100
        height: 50
        top_margin: 5
        bottom_margin: 5
    }
```

运行结果如下：

![](/img_post/qml_2.7.png)
