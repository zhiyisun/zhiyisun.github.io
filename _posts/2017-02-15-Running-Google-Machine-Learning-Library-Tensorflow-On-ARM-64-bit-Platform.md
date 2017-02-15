---
layout: post
title: Running Google Machine Learning Library Tensorflow On ARM 64-bit Platform
---

TensorFlow is an open source software library for machine learning which was developed by Google and open source to community. It supports various kinds of fundamental operations for Machine learning. Tensorflow can be deployed on single server or cloud and supports both CPU and GPU devices. It is used for both research and production. It can even be deployed on phones! That makes it unique to other machine learning library, like Theano, Caffe and Torch.

Tensorflow supports x86-64, GPU and ARM 32-bit (Android and Raspberry Pi) platform officially. This article will introduce to install Tensorflow on ARM 64-bit CPU platform. (Of course, Tensorflow also works on ARM 64-bit CPU + GPU platform.) Below work is based on many prior efforts which make Tensorflow running on [Raspberry Pi](https://github.com/samjabrahams/tensorflow-on-raspberry-pi) and [ODROID-C2](https://github.com/neo-titans/odroid). And it fixes some issues I met when I follow these guides.

#Build Bazel

Before build Tensorflow, Bazel binary for ARM64 should be built first. Bazel is used to build the majority of Google's software. It is the build tool for Tensorflow.

According to the Bazel [Installation Guide](https://bazel.build/versions/master/docs/install.html), we need to install JDK 8 first. For Ubuntu 15.10 or higher, OpenJDK 8 can be installed. For Ubuntu 14.04 or lower, OpenJDK 8 is not available, so please install Oracle JDK 8.

```shell
$ sudo apt-get install openjdk-8-jdk
```

Other tools and libs are also needed for building Bazel.

```shell
$ sudo apt-get install pkg-config zip g++ zlib1g-dev unzip
```

Download [Bazel release package 0.4.4](https://github.com/bazelbuild/bazel/releases/download/0.4.4/bazel-0.4.4-dist.zip). Many online articles point to [Bazel 0.4.3](https://github.com/bazelbuild/bazel/releases/download/0.4.3/bazel-0.4.3-dist.zip). But that version has a bug and will cause [Tensorflow configure failed](https://github.com/bazelbuild/bazel/issues/2423).

Using Bazel release package is much easier to get Bazel ready for use. Otherwise, protobuf and grpc-java are needed for compiling Bazel from source code.

```shell
$ wget https://github.com/bazelbuild/bazel/releases/download/0.4.4/bazel-0.4.4-dist.zip
$ mkdir bazel-0.4.4
$ unzip bazel-0.4.4-dist.zip -d bazel-0.4.4
$ cd bazel-0.4.4
``` 

Then modify bazel source file to compatible with aarch64.

```diff
diff --git a/scripts/bootstrap/buildenv.sh b/scripts/bootstrap/buildenv.sh
index 502f2c1..a2ab4dc 100755
--- a/scripts/bootstrap/buildenv.sh
+++ b/scripts/bootstrap/buildenv.sh
@@ -40,7 +40,7 @@ PLATFORM="$(uname -s | tr 'A-Z' 'a-z')"

 MACHINE_TYPE="$(uname -m)"
 MACHINE_IS_64BIT='no'
-if [ "${MACHINE_TYPE}" = 'amd64' -o "${MACHINE_TYPE}" = 'x86_64' -o "${MACHINE_TYPE}" = 's390x' ]; then
+if [ "${MACHINE_TYPE}" = 'amd64' -o "${MACHINE_TYPE}" = 'x86_64' -o "${MACHINE_TYPE}" = 's390x'  -o "${MACHINE_TYPE}" = 'aarch64' ]; then
   MACHINE_IS_64BIT='yes'
 fi

diff --git a/src/main/java/com/google/devtools/build/lib/analysis/config/BuildConfiguration.java b/src/main/java/com/google/devtools/build/lib/analysis/config/BuildConfiguration.jav
a
index 553c88c..90c8392 100755
--- a/src/main/java/com/google/devtools/build/lib/analysis/config/BuildConfiguration.java
+++ b/src/main/java/com/google/devtools/build/lib/analysis/config/BuildConfiguration.java
@@ -406,6 +406,8 @@ public final class BuildConfiguration {
                 return "arm";
               case S390X:
                 return "s390x";
+             case AARCH64:
+               return "aarch64";
               default:
                 return "unknown";
             }
diff --git a/src/main/java/com/google/devtools/build/lib/util/CPU.java b/src/main/java/com/google/devtools/build/lib/util/CPU.java
index 7a85c29..e5f3eae 100755
--- a/src/main/java/com/google/devtools/build/lib/util/CPU.java
+++ b/src/main/java/com/google/devtools/build/lib/util/CPU.java
@@ -26,6 +26,7 @@ public enum CPU {
   X86_64("x86_64", ImmutableSet.of("amd64", "x86_64", "x64")),
   PPC("ppc", ImmutableSet.of("ppc", "ppc64", "ppc64le")),
   ARM("arm", ImmutableSet.of("arm", "armv7l")),
+  AARCH64("aarch64", ImmutableSet.of("aarch64")),
   S390X("s390x", ImmutableSet.of("s390x", "s390")),
   UNKNOWN("unknown", ImmutableSet.<String>of());
 
diff --git a/third_party/BUILD b/third_party/BUILD
index 9cd2fac..f1cd14c 100755
--- a/third_party/BUILD
+++ b/third_party/BUILD
@@ -583,6 +583,11 @@ config_setting(
 )

 config_setting(
+    name = "aarch64",
+    values = {"host_cpu": "aarch64"},
+)
+
+config_setting(
     name = "freebsd",
     values = {"host_cpu": "freebsd"},
 )
diff --git a/tools/cpp/cc_configure.bzl b/tools/cpp/cc_configure.bzl
index 0ae4483..975908b 100755
--- a/tools/cpp/cc_configure.bzl
+++ b/tools/cpp/cc_configure.bzl
@@ -142,8 +142,10 @@ def _get_cpu_value(repository_ctx):
   result = repository_ctx.execute(["uname", "-m"])
   if result.stdout.strip() in ["power", "ppc64le", "ppc"]:
     return "ppc"
-  if result.stdout.strip() in ["arm", "armv7l", "aarch64"]:
+  if result.stdout.strip() in ["arm", "armv7l"]:
     return "arm"
+  if result.stdout.strip() in ["aarch64"]:
+    return "aarch64"
   return "k8" if result.stdout.strip() in ["amd64", "x86_64", "x64"] else "piii"
```

Then compile Bazel

```shell
$ ./compile.sh
#Copy bazel to $PATH
$ sudo cp output/bazel /usr/local/bin/
```

Bazel binary is ready for use now.

#Build Tensorflow

```shell
$ git clone git@github.com:tensorflow/tensorflow.git
```

Modify Tensorflow to compatible with aarch64.

```diff
diff --git a/tensorflow/core/platform/platform.h b/tensorflow/core/platform/platform.h
index 55d7954..ccebaf3 100644
--- a/tensorflow/core/platform/platform.h
+++ b/tensorflow/core/platform/platform.h
@@ -45,7 +45,7 @@ limitations under the License.

 // Since there's no macro for the Raspberry Pi, assume we're on a mobile
 // platform if we're compiling for the ARM CPU.
-#define IS_MOBILE_PLATFORM
+//#define IS_MOBILE_PLATFORM

 #else
 // If no platform specified, use:
```

```shell
$ ./configure
Please specify the location of python. [Default is /usr/bin/python]:
Please specify optimization flags to use during compilation [Default is -march=native]: 
Do you wish to use jemalloc as the malloc implementation? (Linux only) [Y/n]
jemalloc enabled on Linux
Do you wish to build TensorFlow with Google Cloud Platform support? [y/N]
No Google Cloud Platform support will be enabled for TensorFlow
Do you wish to build TensorFlow with Hadoop File System support? [y/N]
No Hadoop File System support will be enabled for TensorFlow
Do you wish to build TensorFlow with the XLA just-in-time compiler (experimental)? [y/N]
No XLA support will be enabled for TensorFlow
Found possible Python library paths:
  /usr/local/lib/python2.7/dist-packages
  /usr/lib/python2.7/dist-packages
Please input the desired Python library path to use.  Default is [/usr/local/lib/python2.7/dist-packages]

Using python library path: /usr/local/lib/python2.7/dist-packages
Do you wish to build TensorFlow with OpenCL support? [y/N]
No OpenCL support will be enabled for TensorFlow
Do you wish to build TensorFlow with CUDA support? [y/N]
No CUDA support will be enabled for TensorFlow
Configuration finished
Extracting Bazel installation...
.....................
INFO: Starting clean (this may take a while). Consider using --expunge_async if the clean takes more than several minutes.
.....................
INFO: Downloading http://pkgs.fedoraproject.org/repo/pkgs/gmock/gmock-1.7.0.zip/073b984d8798ea1594f5e44d85b20d66/gmock-1.7.0.zip: 297,424 bytes
```

Use a few toolchain optimization flags for build.
```shell
$ bazel build -c opt --copt="-funsafe-math-optimizations" --copt="-ftree-vectorize" --copt="-fomit-frame-pointer" --verbose_failures tensorflow/tools/pip_package:build_pip_package

#Build pip package
$ bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg

#Install tensorflow pip package
$ sudo pip install /tmp/tensorflow_pkg/tensorflow-1.0.0rc2-cp27-cp27mu-linux_aarch64.whl
```

Done!

#Try convolutional model

Then train machine learning hello world model by using Tensorflow. 

```shell
$ git clone git@github.com:tensorflow/models.git
$ cd models/tutorials/image/mnist
$ python convolutional.py
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting data/train-images-idx3-ubyte.gz
Extracting data/train-labels-idx1-ubyte.gz
Extracting data/t10k-images-idx3-ubyte.gz
Extracting data/t10k-labels-idx1-ubyte.gz
Initialized!
Epoch 0.00
Minibatch loss: 12.054, learning rate: 0.010000
Minibatch error: 90.6%
Validation error: 84.6%
Epoch 0.12
Minibatch loss: 3.285, learning rate: 0.010000
Minibatch error: 6.2%
Validation error: 7.0%
...
...
```

Have fun!

Reference

 1. [Building TensorFlow for Raspberry Pi: a Step-By-Step Guide](https://github.com/samjabrahams/tensorflow-on-raspberry-pi)
 2. [Installing TensorFlow on ODROID-C2](https://github.com/neo-titans/odroid)
