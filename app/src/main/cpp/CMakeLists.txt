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