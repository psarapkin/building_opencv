# Building OpenCV (with CUDA)

## Introduction

At this page I would like to gather information about building OpenCV lib and pitfalls that I met and how I solved it (or maybe not, and may be smbd help me). I build OpenCV libs only for Linux and MacOS. When I started writing this README, I used Fedora 41. Information on the internet about building OpenCV and how to solve problems is scattered, so, I had to collect it bit by bit (several times). And after some times I decided to write about it, mostly for myself, but I would be glad if it would be useful for somebody else.

So, if we talk about spherical horse in the vacuum, I would like to build OpenCV with

*  QT support
*  CUDA support
*  OpenGL support
*  Vulkan support
*  and other libs like (gstreamer, ffmpeg, openblas, atlas, vtk... and so on)

But the real life is harsh, and I could not to build all this in one (if somebody could - please write me). 

I think, the main problem here is CUDA. CUDA doesn't work with new versions of GCC. For example, now I'm on Fedora 41, it has GCC version 14, but the newer CUDA requires gcc with version <= 13.2 (https://docs.nvidia.com/cuda/cuda-installation-guide-linux/). So, we have to build GCC version 13.2 if we use newer version of Fedora (13.2 is for Fedora 39). Thanks to INTTF - https://www.if-not-true-then-false.com/2023/fedora-build-gcc/. After building gcc we can install CUDA. Of course, there is an option "--allow-unsupported-compiler" for nvcc, but I wouldn't do that. 

So, what does it means for us. We can divide building OpenCV for two big parts:

1.  Without CUDA
2.  With CUDA (but without QT and VTK)

P.S. VTK gave me error, I had to remove it from system. I think the problem is that VTK built with GCC 14... May be I should build VTK by myself and use GCC-13.2 for it.

## Building OpenCV without CUDA

The start point for us of course is OpenCV documentation - https://docs.opencv.org/5.x/d7/d9f/tutorial_linux_install.html . 
We think that you downloaded opencv source code and opencv_contrib modules.

So, the first command that you should run is this. 
```
cmake -DOPENCV_EXTRA_MODULES_PATH=../opencv_contrib-5.x/modules -DCMAKE_INSTALL_PREFIX=<path to where you want to install opencv> -DWITH_QT=ON -DWITH_OPENGL=ON -DWITH_CUDA=OFF ../opencv-5.x
```

And the first problem that I see, is that I don't have VTK support:

This is the result of cmake. 

```
VTK support:                 NO
```

But I would like to have VTK support. Of course the simplest way is to install it from package manager, in my case - dnf install vtk. But this VTK is build with GCC-14 and it will be the cause of problems. So, I have to build VTK by myself. 

And this is my command to build VTK
```
cmake -DCMAKE_INSTALL_PREFIX=<path to where you want to install VTK> ../VTK-9.4.0.rc3/
```

Well, of course the upper command was the command to configure build. And the next command is for build:

```
cmake --build . --parallel $(nproc)
```

And after that we need to run **make install** to install VTK to destination folder. But, for building OpenCV we need VTK build folder (not install). And that folder we need to pass via VTK_DIR parameter for CMake. So, the final command to configure OpenCV with VTK is: 

```
cmake -DOPENCV_EXTRA_MODULES_PATH=../opencv_contrib-5.x/modules -DCMAKE_INSTALL_PREFIX=<path to install folder for opencv> -DWITH_QT=ON -DWITH_OPENGL=ON -DWITH_CUDA=OFF -DVTK_DIR=<VTK build dir> ../opencv-5.x
```

And after that we run **make install**, and finally we have OpenCV with QT, VTK, FFMPEG and so on.

P.S. We have built VTK without CUDA, cause I couldn't pass to VTK build system the path for gcc previous version (13). May be I solve this later, or may be somebody would help me. 


## Building OpenCV with CUDA

Well, I've already built OpenCV with CUDA, and now I know a plenty of pitfalls (may be not all), and that's why now I think that it's simple. But to get to know what to do I had to read a lot of pages, I had to read CMake files (that I have never do), and so on. But the root of all evil is that the CUDA doesn't work with newer versions of GCC. And everything is connected with it. No, no, not like that - EVERYTHING. So, in that case you have following problems. 

*  First of all, you have two options - if you want to work with CUDA - you should forget about system updates. Or if you want newer version of system, but in that case you have to solve problems)).  Well, I use Fedora, and as said earlier, Fedora has releases twice a year. May be Debian is the solution, or may be Linux Mint Debian Edition (LMDE). I've just checked, and I see that LMDE Faye 6 has gcc 12.2 version (and Debian too), but Fedora 41 has gcc-14.3 version. But I have some reasons to use Fedora.
*  If you use CUDA with gcc 13.3, on the system with gcc 14.3, then you will have problems with some libraries from repositories, for example - Qt, VTK... Qt and VTK for Fedora 41 it seems is built on gcc 14.3. In my case, I couldn't build OpenCV with QT and VTK.

So, to build OpenCV with CUDA you have to specify:
* path to GCC compiler version 13.2 - for C building objects
* path to G++ compiler version 13.2 - for CXX building objects
* CUDA detection flags

Now we know everything we need, so let's do it. 

First of all, we need to specify path to GCC. And we'll do that via environment variable:
```
export CC=gcc-13.2
```

Also, we have to specify CXX compiler, let's do that. We'll use g++. 

```
export CXX=g++-13.2
```

Then we need to specify CUDA detection flags - this is a parameter for cmake, but let's see it. 

```
-DOPENCV_CUDA_DETECTION_NVCC_FLAGS="-ccbin;/usr/bin/gcc-13.2"
```

Now, let's put this parameter under microscope, what is going on here. So the ccbin is the parameter for nvcc for specifying C compiler. Don't see at semicolon, when this parameter passing to nvcc, semicolon disapear. So, it seems like "-ccbin /usr/bin/gcc-13.2". If you don't pass this parameter, the cuda compiler (nvcc) will use gcc version 14, and it will lead to an error. Of course, you can pass "--allow-unsupported-compiler" to nvcc, but it will lead to error too. 

So, the final command to configure cmake build is: 

```
cmake -DOPENCV_EXTRA_MODULES_PATH=../opencv_contrib-5.x/modules -DCMAKE_INSTALL_PREFIX=<path to install OpenCV> -DWITH_QT=OFF -DWITH_OPENGL=ON -DWITH_CUDA=ON -DOPENCV_CMAKE_CUDA_DEBUG=1 -DOPENCV_CUDA_DETECTION_NVCC_FLAGS="-ccbin;/usr/bin/gcc-13.2" -DCUDA_VERBOSE_BUILD=ON ../opencv-5.x 
```

Look at this command. I switched off Qt, and I don't have path to VTK_DIR, cause I couldn't build it with CUDA. 

After that we run
```
cmake --build . --parallel $(nproc)
```


Finally, I almost have built OpenCV with CUDA support, but I ran **cmake --build** twice because of error:

```
nvcc error   : '"$CICC_PATH/cicc"' died due to signal 11 (Invalid memory reference)
nvcc error   : '"$CICC_PATH/cicc"' core dumped

```

It seems the process doesn't have enough memory... But, I have 128Gb... It looks ridiculous but it took place)) 

