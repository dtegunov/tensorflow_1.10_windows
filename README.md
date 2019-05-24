# About this modified version:
This is a slightly modified copy of TensorFlow  r1.10. The changes allow full GPU memory deallocation, and enable flawless (at least on my machine) compilation with CMake, MSVC++ 2017 and CUDA 10.0 on Windows.

To compile the static libraries successfully, I used:
* Visual Studio 2017 (updated to latest version as of November 2018)
* CUDA 10.0
* CUDNN 7.4
* CMake 3.12.3
* swigwin 3.0.12
* Python 3.5

## Compile static libraries

* Open "x64 Native Tools Command Prompt for VS 2017" to get a command line properly configured for MSVC 2017.
* Navigate to /tensorflow/contrib/cmake.
* If you want to compile the code for a CUDA CC other than 5.2, 6.1, 7.0 or 7.5, modify the CMakeLists.txt file accordingly (just search for one of those numbers; CC and the corresponding flags are defined in several places in the file).
* `mkdir build`
* `cd build`
* Modify the paths accordingly and execute:
```
cmake .. -DCMAKE_GENERATOR="Visual Studio 15 2017 Win64" ^
-DCMAKE_BUILD_TYPE=Release ^
-DSWIG_EXECUTABLE="C:\Program Files\swigwin-3.0.12\swig.exe" ^
-DPYTHON_EXECUTABLE="C:\Program Files\Python35\python.exe" ^
-DPYTHON_LIBRARIES="C:\Program Files\Python35\libs\python35.lib" ^
-Dtensorflow_ENABLE_GPU=ON ^
-DCUDNN_HOME="C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v10.0" ^
-Dtensorflow_WIN_CPU_SIMD_OPTIONS=/arch:AVX2 ^
-Dtensorflow_BUILD_PYTHON_BINDINGS=OFF ^
-Dtensorflow_ENABLE_GRPC_SUPPORT=ON ^
-Dtensorflow_BUILD_SHARED_LIB=ON ^
-Dtensorflow_CUDA_VERSION="10.0" ^
-Dtensorflow_CUDNN_VERSION="7.5" ^
-DCUDA_HOST_COMPILER="C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.15.26726/bin/Hostx64/x64/cl.exe"
```
* This should result in successful creation of all VS solution and project files inside the 'build' folder.
* To start the actual compilation process, execute:
```
MSBuild /m:4 /p:CL_MPCount=4 /p:Configuration=Release /p:Platform=x64 /p:PreferredToolArchitecture=x64 ALL_BUILD.vcxproj
```
* The compilation will take several hours.


## Use static libraries in a MSVC++ project

* Create an environment variable 'TENSORFLOW_LIBS' that points to {TF folder}/build
* Open Visual Studio and create a new C++ project.
* Add this to the include directories:
```
$(TENSORFLOW_LIBS)\external\eigen_archive
$(TENSORFLOW_LIBS)\..
$(TENSORFLOW_LIBS)\..\third_party\eigen3
$(TENSORFLOW_LIBS)\protobuf\src\protobuf\src
$(TENSORFLOW_LIBS)\external\nsync\public
$(TENSORFLOW_LIBS)
$(TENSORFLOW_LIBS)\external\zlib_archive
$(TENSORFLOW_LIBS)\external\gif_archive\giflib-5.1.4
$(TENSORFLOW_LIBS)\external\png_archive
$(TENSORFLOW_LIBS)\external\jpeg_archive
$(TENSORFLOW_LIBS)\external\lmdb
$(TENSORFLOW_LIBS)\gemmlowp\src\gemmlowp
$(TENSORFLOW_LIBS)\jsoncpp\src\jsoncpp
$(TENSORFLOW_LIBS)\external\farmhash_archive
$(TENSORFLOW_LIBS)\external\farmhash_archive\util
$(TENSORFLOW_LIBS)\external\highwayhash
$(TENSORFLOW_LIBS)\cub\src\cub
$(TENSORFLOW_LIBS)\re2\install\include
$(TENSORFLOW_LIBS)\external\sqlite
$(TENSORFLOW_LIBS)\grpc\src\grpc\include
$(TENSORFLOW_LIBS)\snappy\src\snappy
```
* Add this (and everything else you need, e. g. CUDA) to the linked libraries:
```
$(TENSORFLOW_LIBS)\tf_c.dir\Release\tf_c.lib
$(TENSORFLOW_LIBS)\tf_cc.dir\Release\tf_cc.lib
$(TENSORFLOW_LIBS)\tf_cc_ops.dir\Release\tf_cc_ops.lib
$(TENSORFLOW_LIBS)\tf_cc_framework.dir\Release\tf_cc_framework.lib
$(TENSORFLOW_LIBS)\tf_core_cpu.dir\Release\tf_core_cpu.lib
$(TENSORFLOW_LIBS)\tf_core_direct_session.dir\Release\tf_core_direct_session.lib
$(TENSORFLOW_LIBS)\tf_core_framework.dir\Release\tf_core_framework.lib
$(TENSORFLOW_LIBS)\tf_core_kernels.dir\Release\tf_core_kernels.lib
$(TENSORFLOW_LIBS)\tf_core_lib.dir\Release\tf_core_lib.lib
$(TENSORFLOW_LIBS)\tf_core_ops.dir\Release\tf_core_ops.lib
$(TENSORFLOW_LIBS)\tf_cc_while_loop.dir\Release\tf_cc_while_loop.lib
$(TENSORFLOW_LIBS)\tf_stream_executor.dir\Release\tf_stream_executor.lib
$(TENSORFLOW_LIBS)\Release\tf_protos_cc.lib
$(TENSORFLOW_LIBS)\Release\tf_core_gpu_kernels.lib
$(TENSORFLOW_LIBS)\zlib\install\lib\zlibstatic.lib
$(TENSORFLOW_LIBS)\gif\install\lib\giflib.lib
$(TENSORFLOW_LIBS)\png\install\lib\libpng16_static.lib
$(TENSORFLOW_LIBS)\jpeg\install\lib\libjpeg.lib
$(TENSORFLOW_LIBS)\lmdb\install\lib\lmdb.lib
$(TENSORFLOW_LIBS)\jsoncpp\src\jsoncpp\src\lib_json\Release\jsoncpp.lib
$(TENSORFLOW_LIBS)\farmhash\install\lib\farmhash.lib
$(TENSORFLOW_LIBS)\fft2d\\src\lib\fft2d.lib
$(TENSORFLOW_LIBS)\highwayhash\install\lib\highwayhash.lib
$(TENSORFLOW_LIBS)\nsync\install\lib\nsync.lib
$(TENSORFLOW_LIBS)\protobuf\src\protobuf\Release\libprotobuf.lib
$(TENSORFLOW_LIBS)\re2\src\re2\Release\re2.lib
$(TENSORFLOW_LIBS)\sqlite\install\lib\sqlite.lib
$(TENSORFLOW_LIBS)\snappy\src\snappy\Release\snappy.lib
$(TENSORFLOW_LIBS)\double_conversion\src\double_conversion\Release\double-conversion.lib
```
* My preprocessor definitions look like this:
```
COMPILER_MSVC
NOMINMAX
PLATFORM_WINDOWS
_SCL_SECURE_NO_WARNINGS
_WINDLL
EIGEN_STRONG_INLINE=inline
SQLITE_OMIT_LOAD_EXTENSION
EIGEN_AVOID_STL_ARRAY
_WIN32_WINNT=0x0A00
LANG_CXX11
OS_WIN
_MBCS
WIN64
WIN32_LEAN_AND_MEAN
NOGDI
TENSORFLOW_USE_EIGEN_THREADPOOL
EIGEN_HAS_C99_MATH
TF_COMPILE_LIBRARY
GRPC_ARES=0
TF_USE_SNAPPY
```
* Other options that deviate from defaults in my projects are:
```
/permissive-
/sdl
```

## To deallocate GPU memory completely after destroying all tensor objects and sessions
```c_cpp
tensorflow::GPUProcessState::singleton()->~GPUProcessState();
```

# Original README below:

<div align="center">
  <img src="https://www.tensorflow.org/images/tf_logo_transp.png"><br><br>
</div>

-----------------


| **`Documentation`** |
|-----------------|
| [![Documentation](https://img.shields.io/badge/api-reference-blue.svg)](https://www.tensorflow.org/api_docs/) |

**TensorFlow** is an open source software library for numerical computation using
data flow graphs.  The graph nodes represent mathematical operations, while
the graph edges represent the multidimensional data arrays (tensors) that flow
between them.  This flexible architecture enables you to deploy computation to one
or more CPUs or GPUs in a desktop, server, or mobile device without rewriting
code.  TensorFlow also includes [TensorBoard](https://www.tensorflow.org/guide/summaries_and_tensorboard), a data visualization toolkit.

TensorFlow was originally developed by researchers and engineers
working on the Google Brain team within Google's Machine Intelligence Research
organization for the purposes of conducting machine learning and deep neural
networks research.  The system is general enough to be applicable in a wide
variety of other domains, as well.

Keep up to date with release announcements and security updates by
subscribing to
[announce@tensorflow.org](https://groups.google.com/a/tensorflow.org/forum/#!forum/announce).

## Installation
*See [Installing TensorFlow](https://www.tensorflow.org/get_started/os_setup.html) for instructions on how to install our release binaries or how to build from source.*

People who are a little more adventurous can also try our nightly binaries:

**Nightly pip packages**
* We are pleased to announce that TensorFlow now offers nightly pip packages
under the [tf-nightly](https://pypi.python.org/pypi/tf-nightly) and
[tf-nightly-gpu](https://pypi.python.org/pypi/tf-nightly-gpu) project on pypi.
Simply run `pip install tf-nightly` or `pip install tf-nightly-gpu` in a clean
environment to install the nightly TensorFlow build. We support CPU and GPU
packages on Linux, Mac, and Windows.


#### *Try your first TensorFlow program*
```shell
$ python
```
```python
>>> import tensorflow as tf
>>> hello = tf.constant('Hello, TensorFlow!')
>>> sess = tf.Session()
>>> sess.run(hello)
'Hello, TensorFlow!'
>>> a = tf.constant(10)
>>> b = tf.constant(32)
>>> sess.run(a + b)
42
>>> sess.close()
```
Learn more examples about how to do specific tasks in TensorFlow at the [tutorials page of tensorflow.org](https://www.tensorflow.org/tutorials/).

## Contribution guidelines

**If you want to contribute to TensorFlow, be sure to review the [contribution
guidelines](CONTRIBUTING.md). This project adheres to TensorFlow's
[code of conduct](CODE_OF_CONDUCT.md). By participating, you are expected to
uphold this code.**

**We use [GitHub issues](https://github.com/tensorflow/tensorflow/issues) for
tracking requests and bugs. So please see
[TensorFlow Discuss](https://groups.google.com/a/tensorflow.org/forum/#!forum/discuss) for general questions
and discussion, and please direct specific questions to [Stack Overflow](https://stackoverflow.com/questions/tagged/tensorflow).**

The TensorFlow project strives to abide by generally accepted best practices in open-source software development:

[![CII Best Practices](https://bestpractices.coreinfrastructure.org/projects/1486/badge)](https://bestpractices.coreinfrastructure.org/projects/1486)


## Continuous build status

### Official Builds

| Build Type      | Status | Artifacts |
| ---             | ---    | ---       |
| **Linux CPU**   | ![Status](https://storage.googleapis.com/tensorflow-kokoro-build-badges/ubuntu-cc.png) | [pypi](https://pypi.org/project/tf-nightly/) |
| **Linux GPU**   | ![Status](https://storage.googleapis.com/tensorflow-kokoro-build-badges/ubuntu-gpu-cc.png) | [pypi](https://pypi.org/project/tf-nightly-gpu/) |
| **Linux XLA**   | TBA | TBA |
| **MacOS**       | ![Status](https://storage.googleapis.com/tensorflow-kokoro-build-badges/macos-py2-cc.png) | [pypi](https://pypi.org/project/tf-nightly/) |
| **Windows CPU** | [![Status](https://ci.tensorflow.org/buildStatus/icon?job=tensorflow-master-win-cmake-py)](https://ci.tensorflow.org/job/tensorflow-master-win-cmake-py) | [pypi](https://pypi.org/project/tf-nightly/) |
| **Windows GPU** | [![Status](http://ci.tensorflow.org/job/tf-master-win-gpu-cmake/badge/icon)](http://ci.tensorflow.org/job/tf-master-win-gpu-cmake/) | [pypi](https://pypi.org/project/tf-nightly-gpu/) |
| **Android**     | [![Status](https://ci.tensorflow.org/buildStatus/icon?job=tensorflow-master-android)](https://ci.tensorflow.org/job/tensorflow-master-android) | [![Download](https://api.bintray.com/packages/google/tensorflow/tensorflow/images/download.svg)](https://bintray.com/google/tensorflow/tensorflow/_latestVersion) [demo APK](https://ci.tensorflow.org/view/Nightly/job/nightly-android/lastSuccessfulBuild/artifact/out/tensorflow_demo.apk), [native libs](https://ci.tensorflow.org/view/Nightly/job/nightly-android/lastSuccessfulBuild/artifact/out/native/) [build history](https://ci.tensorflow.org/view/Nightly/job/nightly-android/) |


### Community Supported Builds

| Build Type      | Status | Artifacts |
| ---             | ---    | ---       |
| **IBM s390x**       | [![Build Status](http://ibmz-ci.osuosl.org/job/TensorFlow_IBMZ_CI/badge/icon)](http://ibmz-ci.osuosl.org/job/TensorFlow_IBMZ_CI/) | TBA |
| **IBM ppc64le CPU** | [![Build Status](http://powerci.osuosl.org/job/TensorFlow_Ubuntu_16.04_CPU/badge/icon)](http://powerci.osuosl.org/job/TensorFlow_Ubuntu_16.04_CPU/) | TBA |
| **IBM ppc64le GPU** | [![Build Status](http://powerci.osuosl.org/job/TensorFlow_Ubuntu_16.04_PPC64LE_GPU/badge/icon)](http://powerci.osuosl.org/job/TensorFlow_Ubuntu_16.04_PPC64LE_GPU/) | TBA |
| **Linux CPU with Intel® MKL-DNN®** | [![Build Status](https://tensorflow-ci.intel.com/job/tensorflow-mkl-linux-cpu/badge/icon)](https://tensorflow-ci.intel.com/job/tensorflow-mkl-linux-cpu/) | TBA |


## For more information

* [TensorFlow Website](https://www.tensorflow.org)
* [TensorFlow White Papers](https://www.tensorflow.org/about/bib)
* [TensorFlow YouTube Channel](https://www.youtube.com/channel/UC0rqucBdTuFTjJiefW5t-IQ)
* [TensorFlow Model Zoo](https://github.com/tensorflow/models)
* [TensorFlow MOOC on Udacity](https://www.udacity.com/course/deep-learning--ud730)
* [TensorFlow Course at Stanford](https://web.stanford.edu/class/cs20si)

Learn more about the TensorFlow community at the [community page of tensorflow.org](https://www.tensorflow.org/community) for a few ways to participate.

## License

[Apache License 2.0](LICENSE)
