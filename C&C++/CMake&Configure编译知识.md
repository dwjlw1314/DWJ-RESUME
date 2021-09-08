<font color=#FF0000 size=4> <p align="center">opencv编译命令</p></font>
```
root@gjsy:~# cmake
-D CMAKE_INSTALL_PREFIX=/opt
-DBUILD_SHARED_LIBS=ON
-D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib/modules
-D WITH_CUDA=ON
-D WITH_TBB=ON
-D ENABLE_FAST_MATH=1
-D CUDA_FAST_MATH=1
-D WITH_CUBLAS=1
-D WITH_QT=OFF
-D WITH_FFMPEG=ON
-DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda/
-D WITH_NVCUVID=ON
-D BUILD_opencv_cudacodec=ON
-D WITH_CUBLAS=ON
-D WITH_CUFFT=ON
-D WITH_OPENMP=ON
-D WITH_OPENGL=ON ..
```

<font color=#FF0000 size=4> <p align="center">ffmpeg编译命令</p></font>
```
root@gjsy:~# ./configure
--enable-nonfree
--enable-pic
--enable-shared
--enable-cuda-nvcc
--enable-nvenc
--enable-nvdec
--enable-cuvid
--enable-ffnvcodec
--enable-libnpp
--extra-cflags=-I/usr/local/cuda/include
--extra-ldflags=-L/usr/local/cuda/lib64
```

<font color=#FF0000 size=4> <p align="center">cmake find_package原理及使用说明</p></font>
```
cmake它仅仅是按照优先级顺序在指定的搜索路径进行查找Findxxx.cmake文件和xxxConfig.cmake文件(其中xxx代表库的名字，特别注意的是有大小写之分)，这两个文件大体上是没有区别的，这两个文件中的任何一个都能成功使用该库，也就是可以用库的内置好了Cmake变量。包含了库的头文件和库文件的路径信息，当执行cmake ..命令之后，Cmake会读取执行CMakeLists.txt中的代码，当执行find_package()这条命令后,Cmake 就会从某些路径中找这Findxxx.cmake文件或者xxxConfig.cmake文件，Cmake找到任意一个之后就会执行这个文件，然后这个文件执行后就会设置好一些Cmake变量

比如下面的变量（NAME表示库的名字 比如可以用Opencv 代表Opencv库）:

<NAME>_FOUND
<NAME>_INCLUDE_DIRS or <NAME>_INCLUDES
<NAME>_LIBRARIES or <NAME>_LIBRARIES or <NAME>_LIBS
<NAME>_DEFINITIONS
一般常用的就是xxx_FOUND 、xxx_INCLUDE_DIRS、xxx_LIBS，分别代表是否找到库的标志、库的头文件路径、库文件路径

find_package()有两种模式:Module模式和Config模式，分别对应上面的Findxxx.cmake 和xxxConfig.cmake两个文件

cmake默认优先Module模式，而Config模式是备选项

Module模式(仅仅查找Findxxx.cmake文件):
Cmake会优先搜索CMAKE_MODULE_PATH指定的路径,如果在CMakeLists.txt中没有设置CMAKE_MODULE_PATH路径，也就是说没有下面的指令：
set(CMAKE_MODULE_PATH "Findxxx.cmake文件所在的路径")
那么Cmake不会搜索CMAKE_MODULE_PATH指定的路径，此时Cmake会搜索第二优先级的路径，也就是<CMAKE_ROOT>/share/cmake-x.y/Mdodules
（注意:x.y表示版本号，eg：opencv3.4.9）。其中CMAKE_ROOT是你在安装Cmake的时候的系统路径
如果没有指定安装路径，就使用系统默认的路径，ubuntu18.04系统的默认路径是/usr/loacl
如果你在安装的过程中使用了cmake -DCMAKE_INSTALL_PREFIX="自己dir路径" ，那么此时CMAKE_ROOT就代表那个你写入的路径
如果Cmake在两个路径下都没有找到Findxxx.cmake文件。那么Cmake就会进入Config模式

Config模式（仅仅查找xxxConfig.cmake文件）：
Cmake会优先搜索xxx_DIR 指定的路径。如果在CMakeLists.txt中没有设置这个cmake变量。也就是说没有下面的指令:
set(xxx_DIR "xxxConfig.cmake文件所在的路径")
那么Cmake就不会搜索xxx_DIR指定的路径，此时Cmake 就会自动到第二优先级的路径下搜索
/usr/local/lib/cmake/xxx/中的xxxConfig.cmake文件

如果Cmake在两种模式提供的路径中没有找到对应的Findxxx.cmake和xxxConfig.cmake文件，此时系统就会提示下面的错误信息
CMake Error at CMakeLists.txt:40 (find_package):
  By not providing "FindOpenCV.cmake" in CMAKE_MODULE_PATH this project has
  asked CMake to find a package configuration file provided by "OpenCV", but
  CMake did not find one.

  Could not find a package configuration file provided by "OpenCV" with any
  of the following names:

    OpenCVConfig.cmake
    opencv-config.cmake

  Add the installation prefix of "OpenCV" to CMAKE_PREFIX_PATH or set
  "OpenCV_DIR" to a directory containing one of the above files.  If "OpenCV"
  provides a separate development package or SDK, be sure it has been
  installed.
```
