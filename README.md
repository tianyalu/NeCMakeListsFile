# NeCMakeListsFile CMakeLists.txt文件详解 

## 一、概念
### 1.1 CMakeLists.txt简析
使用Android Studio3.4 创建一个C/C++Support的项目，默认在app/src/main目录下会生成cpp目录，里面包含CMakeLists.txt和native-lib.cpp.下方代码为CMakeLists.txt去掉英文注释格式化后的内容。  
```bash
# 指定cmake最低支持的版本（可选，但是在CMakeLists文件中使用高版本特有的命令时必选）
cmake_minimum_required(VERSION 3.4.1)

add_library(
	narive-lib
	SHARED
	native-lib.cpp)

find_library(
	log-lib
	log)

target_link_libraries(
	native-lib
	${log-lib})
```
### 1.2 常用命令
#### 1.2.1 `cmake_minimum_required` 
指定cmake最低支持的版本（可选，但是在CMakeLists文件中使用高版本特有的命令时必选）  
```bash
cmake_minimum_required(VERSION 3.4.1)
```

#### 1.2.2 `aux_source_directory`
查找当前目录所有源文件，并将源文件名称列表保存到 DIR_SRCS 变量。注：该命令不能查找子目录。  
```bash
aux_source_directory(. DIR_SRCS)
```

#### 1.2.3 `add_library`
* 添加一个库  
> 添加 一个库文件，名为`<name>`.  
> 指定STATIC，SHARED，MODULE参数来指定库的类型。STATIC：静态库； SHARED：动态库； MODULE：在使用dyld的系统有效，若不支持dyld,等同于SHARED。  
> EXCLUDE_FROM_ALL: 表示该库不会被默认构建。  
> source1 source2...sourceN: 用来指定库的源文件。  
```bash
add_library(<name> [STATIC | SHARED | MODULE] [EXCLUDE_FROM_ALL] source1 source2 ... sourceN)
```
* 导入预编译库 
> 添加一个已存在的预编译库，名为`<name>`。  
> 一般配合`set_target_properties`使用。
```bash
add_library(<name> <SHARED | STATIC | MODULE | UNKNOWN> IMPORTED)
# 比如 
add_library(test SHARED IMPORTED)
set_target_properties(
	test #指明目标库名
	PROPERTIES IMPORTED_LOCATION #指明要设置的参数
	库路径/${ANDROID_ABI}/libtest.so #导入库的路径
	)
```

#### 1.2.4 `set`
设置CMake变量  
```bash
# 设置可执行文件的输出路径（EXECUTABLE_OUTPUT_PATH是全局变量）
set(EXECUTABLE_OUTPUT_PATH [output_path])

# 设置库文件的输出路径（LIBRARY_OUTPUT_PATH是全局变量）
set(LIBRARY_OUTPUT_PATH [output_path]) 

# 设置C++编译参数（CMAKE_CXX_FLAGS是全局变量）
set(CMAKE_CXX_FLAGS "-Wall std=c++11")

# 设置源文件集合（SOURCE_FILES是本地变量即自定义变量）
set(SOURCE_FILES main.cpp test.cpp ...)
```

#### 1.2.5 `include_directories`
设置头文件目录，相当于g++选项中的-l参数  
```bash
# 可以用相对或绝对路径，也可以用自定义的变量值
include_directories(./include ${MY_INCLUDE})
```

#### 1.2.6 `add_executable`  
添加可执行文件  
```bash
add_executable(<name> ${SRC_LIST})
```

#### 1.2.7 `target_link_libraries`  
将若干库链接到目标库文件，链接的顺序应当符合gcc链接顺序规则，被链接的库在依赖它的库后面，即如果上面的命令中，lib1依赖于lib2,lib2又依赖于lib3,则在上面的命令中必需严格按照lib1 lib2 lib3的顺序排列，否则会报错。  
```bash
target_link_libraries(<name> lib1 lib2 lib3)
# 如果出现相互依赖的静态库，CMake会允许依赖库中包含循环依赖，如：  
add_library(A STATIC a.c)
add_library(B STATIC b.c) 
target_link_libraries(A B)
target_link_libraries(B A)
add_executable(main main.c)
target_link_libraries(main A)
```

#### 1.2.8 `add_definitions`  
为当前路径以及子目录的源文件加入由-D引入的define flag (通常用来添加编译参数)
```bash
add_definitions(-DFOO -DDEBUG ...)
```

#### 1.2.9 `add_subdirectory`
如果当前目录下还有子目录时可以使用`add_subdirectory`, 子目录中也需要包含有CMakeLists.txt  
```bash
# sub_dir指定包含CMakeLists.txt和源文件的子目录位置
# binary_dir是输出路径，一般可以不指定
add_subdirectory(sub_dir [binary_dir])
```

#### 1.2.10 `file`
文件操作命令  
```bash
# 将message写入finename文件中，会覆盖文件原有内容
file(WRITE filename "message")
# 将message写入filename文件中，会追加在文件末尾
file(APPEND filename "message")
# 从filename文件中读取内容并存储到var变量中，如果指定了numBytes和offset，则从offset出开始最多读numBytes个字节，
# 另外如果指定了HEX参数，则内容会以十六进制形式存储在var变量中
file(READ filename var [LIMIT numBytes] [OFFSET offset] [HEX])
# 重命名文件
file(RENAME <oldname> <newname>)
# 删除文件，等于rm命令
file(REMOVE [file1 ...])
# 递归地执行删除文件命令，等于 rm -r
file(REMOVE_RECURSE [file1 ...])
# 根据指定的url下载文件
# timeout超时时间；下载的状态会保存到status中；下载日志会被保存到log；sum指定所下载的文件预期的MD5值，如果指定会自动进行比对，
# 如果不一致，则返回一个错误；SHOW_PROGRESS，进度信息会以状态信息的形式被打印出来
file(DOWNLOAD url file [TIMEOUT timeout] [STATUS status] [LOG log] [EXPECTED_MD5 sum] [SHOW_PROGRESS])
# 创建目录
file(MAKE_DIRECTORY [dir1 dir2 ...])
# 会把path转换为以unix的/开头的cmake风格路径，保存在result中
file(TO_CMAKE_PATH path result)
# 它会把cmake风格的路径转换为本地路径风格：Windows下用”\“,而unix下用"/"
file(TO_NATIVE_PATH path result)
# 将会为所有匹配查询表达式的文件生成一个文件list，并将该list存储进变量variable里，如果一个表达式指定了RELATIVE，返回的结果
# 将会是相对于给定路径的相对路径，查询表达式例子：*.cxx，*.vt?
# note：按照官方文档的说法，不建议使用file的GLOB指令来收集工程的源文件
file(GLOB variable [RELATIVE path] [globbing expressions] ...)
```

#### 1.2.11 `set_directory_properties`
设置某个路径的一种属性  
```bash
# prop1,prop2代表属性，取值为：INCLUDE_DIRECTORIES LINK_DIRECTORIES INCLUDE_REGULAR_EXPRESSION ADDITIONAL_MAKE_CLEAN_FILES
set_directory_properties(PROPERTIES prop1 value1 prop2 value2)
```

#### 1.2.12 `set_property`
> 在给定的作用域内设置一个命名的属性  
> PROPERTY参数是必须的  
> 第一个参数决定了属性可以影响的作用域  
	* GLOBAL：全局作用域  
	* DIRECTORY：默认当前路径，也可以用[dir]指定路径  
	* TARGET：目标作用域，可以是0个或者多个已有目标
	* SOURCE：源文件作用域，可以是0个或多个源文件  (源文件属性只对同目录下的CMakeLists的目标可见)
	* TEST：测试作用域，可以是0个或多个已有的测试  
	* CACHE：必须指定0个或多个cache中已有的条目  
```bash
set_property(<GLOBAL |
	DIRECTORY [dir] | 
	TARGET [target ...] |
	SOURCE [src1 ...] |
	TEST [test1 ...] | 
	CACHE [entry1 ...]>
	  [APPEND]
	  PROPERTY <name> [value ...])
```

### 1.3 常见场景
#### 1.3.1 多个源文件处理
如果源文件很多，把所有文件一个一个加入会很麻烦，可以使用`aux_source_directory`命令或file命令，会查找指定目录下的所有源文件，然后将结果存进指定变量名。
```bash
cmake_minimum_required(VERSION 3.4.1)
# 查找当前目录所有源文件，并将名称保存到 DIR_SRCS 变量
# 不能查找子目录
aux_source_directory(. DIR_SRCS)
# 也可以使用
# file(GLOB DIR_SRCS *.c *.cpp)

add_library(native-lib
	SHARED
	${DIR_SRCS})
```

#### 1.3.2 多目录多个源文件处理
> 主目录中的CMakeLists.txt中添加`add_subdirectory(child)`命令，指明本项目包含一个子目录child。并在`target_link_libraries`指明本项目需要的链接。  
> 子目录child中创建CMakeLists.txt，这里child编译为共享库。
```bash
cmake_minimum_required(VERSION 3.4.1)
aux_source_directory(. DIR_SRCS)
# 添加 child 子目录下的cmakelist
add_subdirectory(child)

add_library(
	native-lib
	SHARED
	${DIR_SRCS})
target_link_libraries(native-lib child)
----------------------------------------

# child目录下的CMakeLists.txt:
cmake_minimum_required(VERSION 3.4.1)
aux_source_directory(. DIR_LIB_SRCS)
add_library(
	child
	SHARED
	${DIR_LIB_SRCS})
```

#### 1.3.3 添加预编译库（Android6.0版本以前）
> 假设我们本地项目引用了`libimported-lib.so`。  
> 添加`add_library`命令，第一个参数是模块名，第二个参数SHARED表示动态库，STATIC表示静态库，第三参数IMPORTED表示以导入的响声添加。  
> 添加`set_target_properties`命令设置导入路径属性。  
> 将import-lib添加到`target_link_libraries`命令参数中，表示native-lib需要链接imported-lib模块。  
```bash
cmake_minimum_required(VERSION 3.4.1)
# 使用 IMPORTED 标志告知 CMake 只希望将库导入到项目中
# 如果是静态库则将shared改为static
add_library(
	imported-lib
	SHARED
	IMPORTED)
# 参数分包为：库、属性、导入地址、库所在地址
set_target_properties(
	imported-lib
	PROPERTIES
	IMPORTED_LOCATION
	<路径>/libimported-lib.so)
aux_source_directory(. DIR_SRCS)
add_library(
	native-lib
	SHARED
	${DIR_SRCS})
target_link_libraries(native-lib imported-lib)
```

#### 1.3.4 添加预编译库（Android6.0版本以后）
在Android6.0及以上版本，如果使用1.3.3所述方法添加预编译动态库的话会有问题，我们可以使用以下方式来配置：  
```bash
# set命令定义一个变量
# CMAKE_C_FLAGS: c的参数，会传递给编译器
# 如果是c++文件，需要用CMAKE_CXX_FLAGS
# -L: 库的查找路径
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -L[SO所在目录]")
```

#### 1.3.5 添加头文件目录
为了确保CMake可以在编译时定位头文件，使用`include_directories`，相当于g++选项中的-l参数。这样就可以使用`#include<xx.h>`，否则需要使用`#include "path/xx.h"`
```bash
cmake_minimum_required(VERSION 3.4.1)
# 设置头文件目录
include_directories(<文件目录>)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -L[SO所在目录]")
aux_source_directory(. DIR_SRCS)
add_library(
	native-lib
	SHARED
	${DIR_SRCS})
target_link_libraries(native-lib imported-lib)
```

#### 1.3.6 build.gradle配置
还可以在gradle中使用arguments设置一些配置
```bash
android {
	defaultConfig {
		externalNativeBuild {
			cmake {
				//使用的编译器clang/gcc
				//cmake默认就是 gnustl_static
				arguments "-DANDROID_TOOLCHAIN=clang", "-DANDROID_STL=gnustl_static"
				//指定cflags和cppflags，效果和cmakelist使用一样
				cFlags ""
				cppFlags ""
				//指定需要编译的CPU架构
				abiFilters "armeabi-v7a"
			}
		}
	}
	externalNativeBuild {
		cmake {
			//指定CMakeLists.txt文件相对当前build.gradle的路径
			path "xxx/CMakeLists.txt"
		}
	}
}
```

## 二、实操
Android Studio 新建Native项目。
### 2.1 CMakeLists.txt文件
```cmake
# 指定CMake最小支持版本
cmake_minimum_required(VERSION 3.4.1)
# 设置头文件目录
# 通过这个变量拿到当前cmakeLists.txt文件的目录
include_directories(${CMAKE_SOURCE_DIR}/inc)
# 添加一个库，根据native-lib.cpp源文件编译一个native-lib的动态库
add_library( # Sets the name of the library.
        native-lib
        # Sets the library as a shared library.
        SHARED
        # Provides a relative path to your source file(s).
        native-lib.cpp)
# 设置第三方so库的路径
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -L${CMAKE_SOURCE_DIR}/../jniLibs/${CMAKE_ANDROID_ARCH_ABI}")
# 查找系统库，这里查找的是系统日志库，并赋值给变量log-lib
# 默认会在D:\AndroidDev\AndroidStudio\sdk\ndk-bundle\platforms\android-22\arch-arm\usr\lib 查找系统库
#[[find_library( # Sets the name of the path variable.
        log-lib
        # Specifies the name of the NDK library that
        # you want CMake to locate.
        log)]]

# 设置依赖的库（第一个参数必须为目标模块，顺序不能换）
target_link_libraries( # Specifies the target library.
        native-lib
        fmod
        fmodL
        # Links the target library to the log library
        # included in the NDK.
        log)
```
### 2.2 native-lib.cpp文件
```c++
#include <jni.h>
#include <string>
#include <android/log.h>
#include <fmod.hpp>

using namespace FMOD;

extern "C" JNIEXPORT jstring JNICALL
Java_com_sty_ne_cmakelists_file_MainActivity_stringFromJNI(
        JNIEnv *env,
        jobject /* this */) {
    System *system;
    System_Create(&system);
    unsigned int version;
    system->getVersion(&version);
    __android_log_print(ANDROID_LOG_ERROR, "TEST", "FMOD Version: %08x", version);
    std::string hello = "Hello from C++";
    return env->NewStringUTF(hello.c_str());
}
```

### 2.3 build.gradle 文件
```groovy
apply plugin: 'com.android.application'

android {
    compileSdkVersion 28
    buildToolsVersion "29.0.1"
    defaultConfig {
        applicationId "com.sty.ne.cmakelists.file"
        minSdkVersion 19
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        externalNativeBuild {
            cmake {
                cppFlags ""
                abiFilters "armeabi-v7a" //指定本地库的CPU架构
            }
        }
        ndk {
            abiFilters "armeabi-v7a" //指定第三方库的CPU架构
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    externalNativeBuild {
        cmake {
            path "src/main/cpp/CMakeLists.txt"
            version "3.10.2"
        }
    }
}

dependencies {
    //...
}
```
### 2.4 项目目录结构如下 
![image](https://github.com/tianyalu/NeCMakeListsFile/blob/master/show/directory.png)  

编译生成APK后可以在apk包中的lib目录下发现生成的armeabi-v7a目录，下面是生成的动态库。允许后可以看到控制台输出fmod的版本号。






