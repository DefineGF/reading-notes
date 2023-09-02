##### 实例1 - 一个main.c 文件

- main.c
- CMakeLists.txt



CMakeLists.txt：

```cmake
cmake_minimum_required(VERSION 3.1)
project(demo)
add_executable(main main.c)
```



先后在 CMakeLists.txt 所在文件目录 执行命令：

1. cmake .
2. make

生成文件如下：

```shell
├── CMakeCache.txt
├── CMakeFiles
│   ├── 3.10.2
│   │   ├── CMakeCCompiler.cmake
│   │   ├── CMakeCXXCompiler.cmake
│   │   ├── CMakeDetermineCompilerABI_C.bin
│   │   ├── CMakeDetermineCompilerABI_CXX.bin
│   │   ├── CMakeSystem.cmake
│   │   ├── CompilerIdC
│   │   │   ├── CMakeCCompilerId.c
│   │   │   ├── a.out
│   │   │   └── tmp
│   │   └── CompilerIdCXX
│   │       ├── CMakeCXXCompilerId.cpp
│   │       ├── a.out
│   │       └── tmp
│   ├── CMakeDirectoryInformation.cmake
│   ├── CMakeOutput.log
│   ├── CMakeTmp
│   ├── Makefile.cmake
│   ├── Makefile2
│   ├── TargetDirectories.txt
│   ├── cmake.check_cache
│   ├── feature_tests.bin
│   ├── feature_tests.c
│   ├── feature_tests.cxx
│   ├── main.dir
│   │   ├── C.includecache
│   │   ├── DependInfo.cmake
│   │   ├── build.make
│   │   ├── cmake_clean.cmake
│   │   ├── depend.internal
│   │   ├── depend.make
│   │   ├── flags.make
│   │   ├── link.txt
│   │   ├── main.c.o
│   │   └── progress.make
│   └── progress.marks
├── CMakeLists.txt
├── Makefile
├── cmake_install.cmake
├── main
└── main.c
```



#### 实例2 - 多个 .c 文件

- func.h
- func.c
- main.c // include "func.h"



CMakeLists.txt:

```cmake
cmake_minimum_required(VERSION 3.1)

project(demo)

add_executable(main main.c func.c)
```

同样步骤：

1. cmake . 
2. make 



如果当前目录下有多个 c 文件，避免重复写多个 .c 文件：

```cmake
cmake_minimum_required(VERSION 3.1)

project(demo)

aux_source_directory(. SRC_LIST)

add_executable(main ${SRC_LIST})
```

#####  **aux_source_directory(dir var)**：

将 dir 指定目录下的源文件，存放到 SRC_LIST 变量中！

弊端：
aux_source_directory 会将目录下所有源文件添加进来，可使用以下方式避免：

```cmake
set (SRC_LIST
	./main.c
	./func1.c
	./func2.c)
```



##### 多个目录

同样的，如果有多个源文件目录，多添加几个变量即可：

```cmake
cmake_minimum_required(VERSION 3.1)

project(demo)

include_directories(add sub)
aux_source_directory(add ADD_DIR)
aux_source_directory(sub SUB_DIR)

add_executable(main  main.c ${ADD_DIR} ${SUB_DIR})
```

include_directories：添加头文件所在路径；



##### 改进

文件目录格式如下：

├── CMakeLists.txt
├── bin
├── build
├── include
│   ├── add.h
│   └── sub.h
└── src
    ├── CMakeLists.txt
    ├── add.c
    ├── main.c
    └── sub.c

主目录下CMake 文件：

```cmake
cmake_minimum_required(VERSION 3.1)

project(demo)

add_subdirectory(src)
```

- add_subdirectory： 添加源文件的目录

源文件中的CMake 文件：

```cmake
include_directories(../include)

aux_source_directory(. SRC_LIST)
add_executable(main ${SRC_LIST})

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
```



当然也可以把两个文件合并成一个：

```cmake
cmake_minimum_required(VERSION 3.1)
project(demo)

include_directories(include)

aux_source_directory(src SRC_LIST)
add_executable(main ${SRC_LIST})

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
```





编译细节：

- 进入 build 目录
- cmake .. 

这样生成的 cmake 中间文件都会存在build 中，不会污染源文件；





#### 动态 、 静态链接库

##### 生成动态|静态链接库

├── CMakeLists.txt
├── build
├── lib
└── lib_src
    ├── add.c
    └── add.h

CMakeLists 文件内容

```cmake
cmake_minimum_required(VERSION 3.1)
project(demo)

aux_source_directory(lib_src SRC_LIST)

add_library(add_shared SHARED ${SRC_LIST})
add_library(add_static STATIC ${SRC_LIST})


set_target_properties(add_shared PROPERTIES OUTPUT_NAME "add")
set_target_properties(add_static PROPERTIES OUTPUT_NAME "add")

set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
```

build之后，在lib文件夹下生成文件：

- libadd.a
- libadd.so



##### 链接库

src/main.c 使用刚刚编译成功的静态、动态链接库；

主目录下 CMakeLists.txt：

```cmake
cmake_minimum_required(VERSION 3.1)
project(add_demo)

add_subdirectory(src)
```



src 目录下：

```cmake
aux_source_directory(. SRC_LIST)

include_directories(../../test5/lib_src) # 头文件
link_directories(../../test5/lib)		 # 非标准的共享库搜索路径

add_executable(main ${SRC_LIST})
# target_link_libraries(main add) # 默认使用动态链接库，可以指定静态链接库：libadd.a
target_link_libraries(main libadd.a)	 # 将目标与共享库进行链接

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
```



#### 其他

##### 添加编译选项

add_compile_options(-std=c++11 -Wall)



##### 添加控制选项

外层CMake：

```cmake
option(MYDEBUG "enable debug compilation" OFF) # 第二个参数用以描述options， 第三个参数：默认OFF
```

src目录下：

```cmake
if (MYDEBUG)
    add_executable(main2 main2.c)
else()
    message(STATUS "Currently is not in debug mode")    
endif()
```



#### 总结

```cmake
add_executable(main main.c) # 指定可执行文件的名字 和 源文件

aux_source_directory(. SRC_LIST) # 将当前文件夹内的所有 源文件 存放到 SRC_LIST 变量中
add_executable(main ${SRC_LIST}) # 选择变量内容作为可执行文件的输入

# aux_source_directory 会将所有源文件添加进去
set(SRC_LIST ./main.c ./func1.c ./func2.c) # 选择指定的源文件
add_executable(main , ${SRC_LIST})

add_subdirectory (src) # 一般 src 目录下还会有 CMakeLists.txt 文件

include_directories (../include) 						# 指定 头文件 目录
set (EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)  # 设置可执行文件输出路径

# 生成：静态 | 动态链接库
add_library (动态链接库变量名 SHARED ${SRC_LIST})
add_library (静态链接库变量名 STATIC ${SRC_LIST})

set_target_properties (动态链接库变量名 PROPERTIES OUTPUT_NAME "add") # 将动态链接库输出文件名设置为:add, 实际上输出 libadd.so
set_target_properties (静态链接库变量名 PROPERTIES OUTPUT_NAME "add") # 实际上输出  libadd.a
set (LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)                # 设置链接库输出路径

# 使用：静态、动态 链接库
include_directories(../../test5/lib_src) # 共享库头文件路径
link_directories(../../test5/lib)        # 共享库文件所在路径
target_link_libraries(main add)          # 默认使用动态链接库，可以指定静态链接库：target_link_libraries(main libadd.a)

# 编译选项
add_compile_options(-std=c++11 -Wall) 
# 控制选项
option(MYDEBUG "enable debug compilation" OFF)
if (MYDEBUG)
    add_executable(main2 main2.c)
else()
    message(STATUS "Currently is not in debug mode")    
endif()
```



预定义变量：

- EXECUTABLE_OUTPUT_PATH：可执行文件存放路径；
- PROJECT_SOURCE_DIR：工程根目录；

