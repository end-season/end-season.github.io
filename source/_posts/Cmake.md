---
title: Cmake
data: 2026-3-4
tags: CMake
categories: CMake
cover: /img_post/image.png
---

# Cmake



> 本文章均来源于B站up 双笙子佯谬 的CMake教学。[视频链接](https://www.bilibili.com/video/BV1fa411r7zp)  [github仓库](https://github.com/parallel101/course)




* 简短cmake示例：

```
MyProject/
├── CMakeLists.txt
├── main.cpp
└── mylib/
    ├── CMakeLists.txt
    ├── mylib.cpp
    └── mylib.h
```

```cmake
cmake_minimum_required(VERSION 3.12)
project(MyProject LANGUAGES CXX)

add_subdirectory(mylib)

add_executable(myapp main.cpp)
target_link_libraries(myapp PRIVATE mylib)
target_compile_features(myapp PRIVATE cxx_std_11)
```

```cmake
# mylib/CMakeLists.txt
add_library(mylib STATIC mylib.cpp)
target_include_directories(mylib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
```





## 库（library）

### 为什么需要库?

* 有时候我们会有多个可执行文件，他们之间用到的某些功能是相同的，我们想把这些共用的功能做成一个**库**,方便大家一起共享。

* 库中函数可以被可执行文件调用，也可以被其他库文件调用。

* 库文件又分为**静态库文件**和**动态库文件**。

* 其中静态库相当于直接把代码插入到生成的可执行文件中，会导致体积变大，但只需要一个文件即可运行。

* 而动态库则只在生成的可执行文件中生成“插桩”函数，当可执行文件被加载时会读取指定目录中的.dll文件，加载到内存的空闲位置，并且替换相应的“插桩”指定的地址为加载后的地址，这个过程称为**重定向**。这样以后函数被调用就会跳转到动态加载的地址去。

  > - Windows：可执行文件同目录，其次是环境变量%PATH%
  > - Linux：ELF格式可执行文件的RPATH，其次是/usr/lib等

### Cmake中的静态库与动态库

cmake除了add_executable可以生成可执行文件外，话可以通过add_library生成库文件：

生成静态库libtest.a：

```cmake
add_library(test STATIC source1.cpp source2.cpp)
```

生成动态库libtest.so：

```cmake
add_library(test SHARED source1.cpp source2.cpp)
```

创建库以后，要在某个**可执行文件**中使用该库，只需要：

```cmake
target_link_libraries(myexec PUBLIC test) #为mtexec链接刚刚制作的库libtest.a
```

示例：

```cmake
cmake_minimum_required(VERSION 3.12)
project(hellocmake LANGUAGES CXX)

add_library(hellolib STATIC hello.cpp)
add_executable(a.out main.cpp)
target_link_libraries(a.out PUBLIC hellolib)
```



## Cmake中的子模块

* 复杂的工程中，我们需要划分子模块，通常一个库一个目录，比如：

  ```
  hellocmake/                
  │
  ├── CMakeLists.txt          
  │
  ├── main.cpp                
  │
  └── hellolib/               
      │
      ├── CMakeLists.txt      
      │
      └── hello.cpp           
  ```

  

* 这里我们把`hellolib`库的东西移到`hellolib`文件夹下了，里面的`CMakeLists.txt`定义了`hellolib`的生成规则：  

  ```cmake
  add_library(hellolib STATIC hello.cpp)
  ```

  * 要在根目录使用它，可以用Cmake的`add_subdirectory`添加子目录，子目录也包含一个`CMakeLists.txt`，其中定义的库在`add_subdirectory`之后就可以在外面使用。

    > 子目录的`CmakeLists.txt`里路径名（比如`hello.cpp`）都是相对路径，这也是很方便的一点。

  ```cmake
  cmake_minimum_required(VERSION 3.12)
  project(hellocmake LANGUAGES CXX)
  
  add_subdirectory(hellolib)
  
  add_executable(a.out main.cpp)
  target_link_libraries(a.out PUBLIC hellolib)
  ```

### 子模块的头文件如何处理

依旧是刚刚的目录：

```
hellocmake/                
│
├── CMakeLists.txt         
│
├── main.cpp                
│
└── hellolib/               
    │
    ├── CMakeLists.txt     
    │
    └── hello.cpp           
```

* 因为`hello.h`被移到了`hellolub`子文件夹里，因此`main.cpp`中要使用`hello()`，需添加：

  ```c
  #include <hellolib/hello.h>
  ```

  如果要避免这样修改，可以通过`target_include_directoriess`指定`a.out`的头文件搜索目录：

  ```cmake
  target_include_directoryies(a.out PUBLIC hellolib)
  ```

  这样甚至可以用`#include<hello.h>`来引用头文件，因为通过`target_include_directories`指定的路径会被视为与系统路径等价。              

* 但这样如果另一个`b.out`也需要`hellolib`这个库，也需要指定一遍搜索路径。

  **解决方法**是：我们只需要定义`hellolib`的头文件搜索路径，引用它的可执行文件`Cmake`会自动添加这个路径：

  ```cmake
  add_library(hellolib STATIC hello.cpp)
  target_include_directories(hellolib PUBLIC .)
  ```

  这里用了`.`表示当前路径，因为子目录的路径是相对路径，类似还有`..`表示上一层目录。

* 此外，如果不希望让引用`hellolib`的可执行文件自动添加这个路径，把`PUBLIC`改成`PRIVATE`即可。这就是他们的用途：决定一个属性要不要在被`link`的时候传播。

## 目标的一些其他选项

* 除了头文件搜索目录以外，还有这些选项，`PUBLIC`和`PRIVATE`对它们同理：

  * 添加头文件搜索目录：

    ```cmake
    target_include_directories(myapp PUBLIC /usr/include/eigen3)
    ```

  * 添加要链接的库：

    ```cmake
    target_link_libraries(myapp PUBLIC hellolib)
    ```

  * 添加一个宏定义

    ```cmake
    target_add_definitions(myapp PUBLIC MY_MACRO=1)
    ```

  * 添加编译器命令行选项：

    ```cmake
    target_compile_options(myapp PUBLIC -fopenmp)
    ```

  * 添加要编译的源文件：

    ```cmake
    target_sources(myapp PUBLIC hello.cpp other.cpp)
    ```

* 以及可以通过下列指令（**不推荐使用**），把选项加到所有接下来的目标去：

  * 添加头文件搜索目录：

    ```cmake
    include_directories(/opt/cuda/include)
    ```

  * 添加库文件的搜索路径

    ```cmake
    link_directories(/opt/cuda)
    ```

  * 添加一个宏定义

    ```cmake
    add_definations(MY_MACRO=1)
    ```

  * 添加编译器命令行选项：

    ```cmake
    add_compile_options(-fopenmp)
    ```



## 一些纯头文件库

1. `nothings/stb`大名鼎鼎的`stb_image`系列，涵盖图像，声音，字体等。
2. `Neargye/magic_enum` 枚举类型的反射，如枚举转字符串等。
3. `g-truc/glm` 模仿`GLSL`语法的数学矢量/矩阵库（附带一些常用函数，随机数生成等）
4. `Tencent/rapidjson` 单纯的`JSON`库，甚至没依赖`STL`（可定制性高，工程美学经典）
5. `ericniebler/range-v3` `c++20 ranges`库就是受他启发。
6. `fmtlib/fmt` 格式化库，提供`std::format`的替代品（需要`-DFMT_HEADER_ONLY`）
7. `gabime/spdlog` 能适配控制台，安卓等多后端的日志库（和`fmt`冲突！）

* 只需要把他们的`include`目录或头文件下载下来，然后`include_directories(spdlog/include)`即可。
* 缺点：函数直接实现在头文件里，没有提前编译，从而需要重复编译同样内容，编译时间长。

### 使用glm(纯头文件)

首先克隆仓库：

```
git clone https://github.com/g-truc/glm.git --depth=1
```

`CMakeLists.txt`：

```cmake
cmake_minimum_required(VERSION 3.12)
project(hellocmake LANGUAGES CXX)

add_executable(a.out main.cpp)
target_include_directories(a.out PUBLIC glm/glm)
```

`main.cpp`:

```c++
#include <vec3.hpp>
#include <iostream>

int main()
{
    glm::vec3 v(1, 2, 3);
    v += 1;
    std::cout << v.x << ' ' << v.y << ' ' << v.z << std::endl;
    return 0;
}
```



## 作为子模块引入第三方库

* 这些库能够很好地支持作为子模块引入：
  1. `fmtlib/fmt` 格式化库，提供`std::format`的替代品。
  2. `gabime/spblog` 能适配控制台，安卓等多后端的日志库。
  3. `ericniebler/range-v3` `c++20 ranges`库收它启发。
  4. `g-truc/glm` 模仿`GLSL`语法的数学矢量/矩阵库。
  5. `abseil/absek-cpp` 旨在补充标准库没有的常用功能。
  6. `bombela/backward-cpp` 实现了`c++`的堆栈回溯便于调试。
  7. `google/googletest` 谷歌单元测试框架。
  8. `google/benchmark` 谷歌性能评估框架。
  9. `glfw/glfw` `OpenGL`窗口和上下文管理。
  10. `libigl/libigl` 各种图形学算法大合集。

### 使用fmt（作为子模块引入）

克隆仓库：

```
git clone https://github.com/fmtlib/fmt.git --depth=1
```

`CMakeLists`：

```cmake
cmake_minimum_required(VERSION 3.12)
project(hellocmake LANGUAGES CXX)

add_subdirectory(fmt)

add_executable(a.out main.cpp)
target_link_libraries(a.out PUBLIC fmt)
```

`main.cpp`

```c++
#include <fmt/core.h>
#include <iostream>

int main()
{
    fmt::print("The answer is {}.\n", 42);
    return 0;
}
```



## 引用系统中预安装的第三方库

* 可以通过`find_package`命令寻找系统中的包/库：

  ```cmake
  find_package(fmt REQUIRED)
  target_link_libraries(myexec PUBLIC fmt::fmt)
  ```

  * 为什么是`fmt::fmt`而不是简单的`fmt`？

    现代`CMake`认为一个**包**（package）可以提供多个**库**，又称多个**组件**（components），比如`TBB`这个包。就包含了`tbb`,`tbbmalloc`,`tbbmalloc_proxy`这三个组件

* 你可以指定要用哪几个组件：

  ```cmake
  find_package(TBB REQUIRED COMPONENTS tbb tbbmalloc REQUIRED)
  target_link_libraries(myexec PUBLIC TBB::tbb TBB::tbbmalloc)
  ```

* 常用的`package`l列表

  1. `fmt::fmt`
  2. `spblog::spblog`
  3. `range-v3::range-v3`
  4. `TBB::tbb`
  5. `OpenVDB::openvdb`
  6. `Boost::iostreams`
  7. `Eigen3::Eigen`
  8. `OpenMP::OpenMP_CXX`

  * 不同的包之间常常有着依赖关系，而包管理器的作者为`find_package`编写的脚本（例如`/usr/lib/cmake/TBB/TBBConfig.cmake`）能够自动查找所有依赖，并利用`PUBLIC PRIVATE`正确处理依赖项，比如你引用了`OpenVDB::openvdb`那么`TBB::tbb`也会被自动引用。

## 包管理器

* `linux`可以用系统自带的包管理器（如`apt`）安装`c++`包。

  > pacman -S fmt

* `windows`没有自带的包管理器。因此可以使用跨平台的`vcpkg`：

  > https://github.com/microsoft/vcpkg

  * 使用方法：下载`vcpkg`的源码，放到项目根目录，接着：

    ```cmd
    cd vcpkg
    
    ./bootstrap-vcpkg.bat
    
    ./vcpkg integrate install
    
    ./vcpkg install fmt:x64-windows
    
    cd ..
    
    cmake -B build -DCMAKE_TOOLCHAIN_FILE="%CD%/vcpkg/scripts/buildsystems/vcpkg.cmake"
    ```

    



## 为什么c++需要声明？

```c
//hello.cpp
#include <cstdio>

void hello()
{
    printf("hello world\n");
}
```

```c
//main.cpp
#include <cstdio>

void hello();

int main()
{
    hello();
    return 0;
}
```



1. 因为需要知道函数的参数和返回值类型：这样才能支持**重载**，**隐式类型转换**等特征。例如show(3)，如果声明了void show(float x)，那么编译器知道把3转换成3.0f才能调用。
2. 让编译器知道hello这个名字是一个**函数**，而不是一个**变量**或者**类**的名字：这样当我写下hello()的时候，它知道我想调用的是hello这个函数。

- 其实，c++是一种强烈依赖**上下文**的信息的编程语言：

  ```c++
  vector<MyClass> a; //声明一个由MyClass组成的数组
  ```

  如果编译器不知道vector是个模板类，那他完全可以把vector看作一个变量名，把<解释为小于号，从而理解为vector这个变量是否小于MyClass这个变量的值。

  > 正因如此，我们常常可以在c++代码中看见这样的写法：typename decay<T>::type
  >
  > 因为T是不确定的，导致编译器无法确定decay<T>的type是一个**类型**，还是一个**值**。因此用typename修饰来让编译器确信这是一个类型名。