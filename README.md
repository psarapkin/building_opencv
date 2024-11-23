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

