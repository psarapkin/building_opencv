# Building OpenCV (with Cuda)

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
