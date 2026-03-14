---
title: QML_3
data: 2026-3-10
tags: QML
categories: QML
cover: /img_post/qml_cover.png 
---



# 动画效果制作

本文章来源于B站up 洛雨薄青衫 的qml的教学视频 [视频链接](https://www.bilibili.com/video/BV1dS4y1u7vN/)



* `states`

  状态设置，可以在`states`中设置多种状态，并用`name:`标识每个状态。可以通过`state:`调用，看以下示例：

  ```qml
  Rectangle {
          id: root
          height: 100; width: 200
          state: "red_color"
          states:[
              State {
                  name: "red_color"
                  PropertyChanges {
                      target: root
                      color: "red"
                  }
              },
              State {
                  name: "blue_color"
                  PropertyChanges {
                      target: root
                      color: "blue"
                  }
              }
          ]
      }
  ```

  我们为这个矩形创建了两个状态：`red_color`和`blue_color`,并在矩形的`state`中设置了`red_color`，`PropertyChanges`可以用来改变属性，其中需要指定目标。当我们运行这个程序时，会在窗口中生成一个红色的矩形，若是将`state: "red_color"`改为`state: "bluw_color"`，则会生成蓝色的矩形。

  再看以下示例：

  ```qml
  Rectangle {
          id: root
          height: 100; width: 200
          state: "normal"
          states:[
              State {
                name: "normal"
                PropertyChanges {
                    target: root
                    color: "black"
                }
              },
  
              State {
                  name: "red_color"
                  PropertyChanges {
                      target: root
                      color: "red"
                  }
              },
              State {
                  name: "blue_color"
                  PropertyChanges {
                      target: root
                      color: "blue"
                  }
              }
          ]
      }
  
      MouseArea {
          anchors.fill: parent
          onPressed: {
              root.state = "red_color"
          }
          onReleased: {
              root.state = "blue_color"
          }
  
      }
  ```

  我们又为矩形添加了一个`normal`状态，并将它设置到矩形中去，接着我们又创建了一个鼠标事件，矩形默认为黑色，当我们在窗口中按下鼠标时，矩形变为红色，当我们松开鼠标时，矩形变为蓝色。

* `Aninmation`

  可以通过`PropertyAnimation`与`NumberAnimation`制作动画效果：

  ```qml
  Rectangle {
          id: flashingblob
          width: 75; height: 75
          color: "blue"
          opacity: 1.0
  
          MouseArea {
              anchors.fill: parent
              onClicked: {
                  animateColor.start()
                  animateOpacity.start()
                  animateWidth.start()
              }
          }
  
          PropertyAnimation {
              id: animateColor
              target: flashingblob
              properties: "color"
              to: "green"
              duration: 1000
          }
  
          NumberAnimation {
              id: animateOpacity
              target: flashingblob
              properties: "opacity"
              from: 0.1
              to: 1.0
              duration: 2000
         }
  
          NumberAnimation {
              id: animateWidth
              target: flashingblob
              properties: "width"
              from: 75
              to: 200
              duration: 2000
         }
  
      }
  ```

  在这个示例中，我们创建除了一个矩形，并为这个矩形创建了鼠标事件，当矩形区域被点击时，将启动三个动画：

  * 第一个动画将矩形颜色变为绿色，持续一秒。
  * 第二个动画将矩形透明度从0.1变为1.0，持续时间两秒。
  * 第三个动画将矩形的宽度从75变到200，持续时间两秒。

  还有如下的简洁用法：

  ```qml
  Rectangle {
          height: 100; width: 100
          color: "red"
  
          PropertyAnimation on x {
              to: 100
              duration: 1000
          }
  
          PropertyAnimation on y {
              to: 100
              duration: 1000
          }
      }
  ```

  此代码会生成一个红色矩形，并逐渐像坐标（100，100）移动。这样写可以不用指定目标，也不用手动开启。

* `SequnticialAnimation`:

  分组动画，可以写多个对同一属性的动画：

  ```qml
  Rectangle {
          height: 100; width: 100
          color: "red"
          SequentialAnimation on color{
  
              ColorAnimation {
                  to: "yellow"
                  duration: 1000
              }
  
  
              ColorAnimation {
                  to: "blue"
                  duration: 1000
              }
          }
      }
  ```

  运行结果会创建一个红色矩形，并逐渐变为黄色，再逐渐变为蓝色。

* `transitions`:

  通过`transitions`可以实现状态切换之间的动画效果，请看以下示例：

  ```qml
  Rectangle {
          id: button
          height: 75; width: 75
          state: "RELEASED"
          MouseArea {
              anchors.fill: parent
              onPressed: button.state = "PRESSED"
              onReleased: button.state = "RELEASED"
          }
  
          states: [
              State {
                  name: "PRESSED"
                  PropertyChanges {
                      target: button
                      color: "red"
                  }
              },
  
              State {
                  name: "RELEASED"
                  PropertyChanges {
                      target: button
                      color: "blue"
                  }
              }
          ]
  
          transitions: [
              Transition {
                  from: "PRESSED"
                  to: "RELEASED"
  
                  ColorAnimation {
                      target: button
                      duration: 1000
                  }
              },
  
              Transition {
                  from: "RELEASED"
                  to: "PRESSED"
  
  
                  ColorAnimation {
                      target: button
                      duration: 1000
                  }
              }
          ]
      }
  ```

  此代码为矩形创建了两种状态：鼠标按下时颜色为红色，松开时为蓝色。加入`transitions`后，鼠标在松下和按下之间切换时，矩形颜色的变化会更加平缓，上述代码给了一秒的持续时间，也就是有一秒钟时间进行颜色切换。

  > 对于简单的转换，可以只设置`Tansition`中的`to`属性为”*“，这样就适配所有类型的转换。

* `behavior`:

  在某一属性改变时，会触发的动作。请看以下示例：

  ```qml
  Rectangle {
          id: ball
          height: 75; width:75
          color: "black"
          radius: width
  
          MouseArea {
              anchors.fill: parent
              onClicked: {
                  ball.x +=50
                  ball.y +=40
              }
          }
  
          Behavior on x {
              NumberAnimation {
                  id: bouncebehavior
                  easing {
                      type: Easing.OutElastic
                      amplitude: 1.0
                      period: 0.5
                  }
              }
          }
  
          Behavior on y {
              animation: bouncebehavior
          }
      }
  ```

  上述代码创建了一个黑色的圆形，用鼠标点击时，圆形的x，y坐标会发生变化，此使会触发`Behavior on x`与`Behavior on y`。小球会有类似”弹跳“的动画，至于这个动作画的实现，先不做讨论。