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
