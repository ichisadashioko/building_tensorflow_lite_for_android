<!-- https://www.tensorflow.org/lite/guide/build_android -->

# Build TensorFlow Lite for Android

This document describes how to build TensorFlow Lite Android library on your own. Normally, you do not need to locally build TensorFlow Lite Android library. If you just want to use it, the easiest way is using the TensorFlow Lite AAR hosted at JCenter. See Android quickstart for more details on how to use them in your Android projects.

## Build TensorFlow Lite locally

In some cases, you might wish to use a local build of TensorFlow Lite. For example, you may be building a custom binary that includes operations selected from TensorFlow, or you may wish to make local changes to TensorFlow Lite.

### Set up build environment using Docker

- Download the Docker file. By downloading the Docker file, you agree that the following terms of service govern your use thereof:

_By clicking to accept, you hereby agree that all use of the Android Studio and Android Native Development Kit will be governed by the Android Software Development Kit License Agreement available at https://developer.android.com/studio/terms (such URL may be updated or changed by Google from time to time).

You can download the Docker file here

> original Docker file content

```
FROM tensorflow/tensorflow:devel

ENV ANDROID_DEV_HOME /android
RUN mkdir -p ${ANDROID_DEV_HOME}

# Install Android SDK.
ENV ANDROID_SDK_FILENAME tools_r25.2.5-linux.zip
ENV ANDROID_SDK_URL https://dl.google.com/android/repository/${ANDROID_SDK_FILENAME}
ENV ANDROID_API_LEVEL 23
ENV ANDROID_NDK_API_LEVEL 18
# Build Tools Version liable to change.
ENV ANDROID_BUILD_TOOLS_VERSION 28.0.0
ENV ANDROID_SDK_HOME ${ANDROID_DEV_HOME}/sdk
ENV PATH ${PATH}:${ANDROID_SDK_HOME}/tools:${ANDROID_SDK_HOME}/platform-tools
RUN cd ${ANDROID_DEV_HOME} && \
    wget -q ${ANDROID_SDK_URL} && \
    unzip ${ANDROID_SDK_FILENAME} -d android-sdk-linux && \
    rm ${ANDROID_SDK_FILENAME} && \
    bash -c "ln -s ${ANDROID_DEV_HOME}/android-sdk-* ${ANDROID_SDK_HOME}"

# Install Android NDK.
ENV ANDROID_NDK_FILENAME android-ndk-r18b-linux-x86_64.zip
ENV ANDROID_NDK_URL https://dl.google.com/android/repository/${ANDROID_NDK_FILENAME}
ENV ANDROID_NDK_HOME ${ANDROID_DEV_HOME}/ndk
ENV PATH ${PATH}:${ANDROID_NDK_HOME}
RUN cd ${ANDROID_DEV_HOME} && \
    wget -q ${ANDROID_NDK_URL} && \
    unzip ${ANDROID_NDK_FILENAME} -d ${ANDROID_DEV_HOME} && \
    rm ${ANDROID_NDK_FILENAME} && \
    bash -c "ln -s ${ANDROID_DEV_HOME}/android-ndk-* ${ANDROID_NDK_HOME}"

# Make android ndk executable to all users.
RUN chmod -R go=u ${ANDROID_DEV_HOME}
```

- You can optionally change the Android SDK or NDK version. Put the downloaded Docker file in an empty folder and build your docker image by running:

```sh
docker build . -t tflite-builder -f tflite-android.Dockerfile
```

- Start the docker container interactively by mounting your current folder to `/tmp` inside the container (note that `/tensorflow_src` is the TensorFlow repository inside the container):

```sh
docker run -it -v $PWD:/tmp tflite-builder bash
```

If you use PowerShell on Windows, replace `$PWD` with `pwd`.

If you would like to use a TensorFlow repository on the host, mount that host directory instead (`-v hostDir:/tmp`).

- Once you are inside the container, you can run the following to download additional Android tools and libraries (note that you may need to accept the license):

```sh
android update sdk --no-ui -a --filter tools,platform-tools,android-${ANDROID_API_LEVEL},build-tools-${ANDROID_BUILD_TOOLS_VERSION}
```

You can now proceed to the "Build and Install" section. After you are finished building the libraries, you can copy them to `/tmp` inside the container so that you can access them on the host.

### Set up build environment without Docker

#### Install Bazel and Android Prerequisites

Bazel is the primary build system for TensorFlow. To build with it, you must have it and the Android NDK and SDK installed on your system.

1. Install the latest version of the Bazel build system.
2. The Android NDK is required to build the native (C/C++) TensorFlow Lite code. The current recommended version is `17c`, which may be found here.
3. The Android SDK and build tools may be obtained here, or alternatively as part of Android Studio. Build tools API `>= 23` is the recommended version for building TensorFlow Lite.

#### Configure `WORKSPACE` and `.bazelrc`

Run the `./configure` script in the root TensorFlow checkout directory, and answer "Yes" when the script asks to interactively configure the `./WORKSPACE` for Android builds. The script will attempt to configure settings using the following environment variables:

- `ANDROID_SDK_HOME`
- `ANDROID_SDK_API_LEVEL`
- `ANDROID_NDK_HOME`
- `ANDROID_NDK_API_LEVEL`

If these variables aren't set, they must be provided interactively in the script prompt. Successful configuration should yield entries similar to the following in the `.tf_configure.bazelrc` file in the root folder:

```sh
build --action_env ANDROID_NDK_HOME="/usr/local/android/android-ndk-r18b"
build --action_env ANDROID_NDK_API_LEVEL="21"
build --action_env ANDROID_BUILD_TOOLS_VERSION="28.0.3"
build --action_env ANDROID_SDK_API_LEVEL="23"
build --action_env ANDROID_SDK_HOME="/usr/local/android/android-sdk-linux"
```

### Build and install

Once Bazel is properly configured, you can build the TensorFlow Lite AAR from the root checkout directory as follows:

```sh
bazel build -c opt --fat_apk_cpu=x86,x86_64,arm64-v8a,armeabi-v7a \
  --host_crosstool_top=@bazel_tools//tools/cpp:toolchain \
  //tensorflow/lite/java:tensorflow-lite
```

This will generate an ARR file in `bazel-bin/tensorflow/lite/java/`. Note that this builds a "fat" AAR with several different architectures; if you don't need all of them, use the subset appropriate for your deployment environment.

> __Caution:__ Following feature is experimental and only available at HEAD. You can buld smaller AAR files targeting only a set of models as follows:

```
bash tensorflow/lite/tools/build_aar.sh \
  --input_models=mdoel1,model2 \
  --target_archs=x86,x86_64,arm64-v8a,armeabi-v7a
```

Above script will generate the `tensorflow-lite.aar` file and optionally the `tensorflow-lite-select-tf-ops.aar` file if one of the models is using Tensorflow ops. For more detail, please see the Reduce TensorFlow Lite binary size section.

#### Add AAR directly to project

Move the `tensorflow-lite.aar` file into a directory called `libs` in your project. Modify your app's `build.gradle` file to reference the new directory and replace the existing TensorFlow Lite dependency with the new local library, e.g.:

```build.gradle
allprojects {
    repositories {
        jcenter()
        flatDir {
            dirs 'libs'
        }
    }
}

dependencies {
    compile(name:'tensorflow-lite', ext:'aar')
}
```

#### Install AAR to local Maven repository

Execute the following command from your root checkout directory:

```sh
mvn install:install-file \
  -Dfile=bazel-bin/tensorflow/lite/java/tensorflow-lite.aar \
  -DgroupId=org.tensorflow \
  -DartifactId=tensorflow-lite -Dversion=0.1.100 -Dpackaging=aar
```

In your app's `build.gradle`, ensure you have the `mavenLocal()` dependency and replace the standard TensorFlow Lite dependency with the one that has support for select TensorFlow ops:

```build.gradle
allprojects {
    repositories {
        jcenter()
        mavenLocal()
    }
}

dependencies {
    implementation 'org.tensorflow:tensorflow-lite:0.1.100'
}
```

Note that the `0.1.100` version here is purely for the sake of testing/development. With the local AAR installed, you can use the standard TensorFlow Lite Java interence APIs in your app code.
