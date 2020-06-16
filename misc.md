
<p align="center">
  <a href="index.html">Programming</a> |
  <a href="unreal.html">Unreal</a> |
  <a href="misc.html">Misc</a> |
  <a href="soon.html">Coming Soon</a>
</p>

* * *
## Miscellaneous

Niche stuff, mostly related to non-coding workflow like building projects, cmake, etc.
 
## Article List
* [Cross-Compile for Android Using CMake on Windows](#cross-compile-for-android-using-cmake)


### Cross Compile For Android Using CMake
CMake can be used to compile Android libraries on a windows machine. This process is a bit painful so this article will address possible roadblocks. The example covered here will be a walkthrough of building google protobuf static libraries for android.

Firstly, NDK must be installed on the windows machine. NDK will come with a toolchain file located in build/cmake directory. After hitting configure for the first time in the CMake GUI, this toolchain file must be specified. Additionally NVIDIA Tegra might need to be installed if using Visual Studio Generators.

Sometimes the CMakeFile will build a simple project to verify everything is working. This might be problematic for cross-compilation, so if errors are encountered, the following flags can help ignore this step.

```
CMAKE_CXX_COMPILER_WORKS 1
CMAKE_C_COMPILER_WORKS 1
```

If there are further errors complaining about a threading library not being found, adding the following lines to the project CMake file might help:
```
set(CMAKE_THREAD_LIBS_INIT "-lpthread")
set(CMAKE_HAVE_THREADS_LIBRARY 1)
set(CMAKE_USE_WIN32_THREADS_INIT 0)
set(CMAKE_USE_PTHREADS_INIT 1)
set(THREADS_PREFER_PTHREAD_FLAG ON)
```

Hopefully configure + generate should work at this point. Then open up the generated visual studio solution, and try to build the project. For the protobuf library, EnableRuntimeInformation must be set to true in C/C++ / Language tab in project properties, otherwise the project won't compile. 

