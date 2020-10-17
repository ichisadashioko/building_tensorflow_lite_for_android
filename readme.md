- install docker (Ubuntu)
- download docker file for building docker image
- modify docker file
- build the docker image

```sh
mkdir docker_build
cd docker_build
sudo docker build . -t tflite-builder -f ../tflite-android.Dockerfile.txt
```

```sh
docker run -it -v $PWD:/tmp tflite-builder bash
```

```sh
android update sdk --no-ui -a --filter tools,platform-tools,android-${ANDROID_API_LEVEL},build-tools-${ANDROID_BUILD_TOOLS_VERSION}
```

```sh
cd tensorflow_src
git checkout tags/v2.3.1
```

```sh
bazel build -c opt --fat_apk_cpu=x86,x86_64,arm64-v8a,armeabi-v7a \
  --host_crosstool_top=@bazel_tools//tools/cpp:toolchain \
  //tensorflow/lite/java:tensorflow-lite
```

```
FAILED: Build did NOT complete successfully (46 packages loaded, 821 targets configured)
Internal error thrown during build. Printing stack trace: java.lang.RuntimeException: Unrecoverable error while evaluating node '@bazel_tools//src/tools/android/java/com/google/devtools/build/android/incrementaldeployment:incremental_split_stub_application BuildConfigurationValue.Key[b6c0f68614aefb61c49685ba6348794d44e10cfae2a002fe64ee70c671a75afd] false' (requested by nodes '@bazel_tools//tools/android:incremental_split_stub_application BuildConfigurationValue.Key[b6c0f68614aefb61c49685ba6348794d44e10cfae2a002fe64ee70c671a75afd] false')
        at com.google.devtools.build.skyframe.AbstractParallelEvaluator$Evaluate.run(AbstractParallelEvaluator.java:515)
        at com.google.devtools.build.lib.concurrent.AbstractQueueVisitor$WrappedRunnable.run(AbstractQueueVisitor.java:399)
        at java.base/java.util.concurrent.ForkJoinTask$AdaptedRunnableAction.exec(Unknown Source)
        at java.base/java.util.concurrent.ForkJoinTask.doExec(Unknown Source)
        at java.base/java.util.concurrent.ForkJoinPool$WorkQueue.topLevelExec(Unknown Source)
        at java.base/java.util.concurrent.ForkJoinPool.scan(Unknown Source)
        at java.base/java.util.concurrent.ForkJoinPool.runWorker(Unknown Source)
        at java.base/java.util.concurrent.ForkJoinWorkerThread.run(Unknown Source)
Caused by: java.lang.NullPointerException
        at com.google.devtools.build.lib.rules.android.BusyBoxActionBuilder.addAapt(BusyBoxActionBuilder.java:315)
        at com.google.devtools.build.lib.rules.android.AndroidResourcesProcessorBuilder.createAapt2ApkAction(AndroidResourcesProcessorBuilder.java:279)
        at com.google.devtools.build.lib.rules.android.AndroidResourcesProcessorBuilder.build(AndroidResourcesProcessorBuilder.java:208)
        at com.google.devtools.build.lib.rules.android.AndroidResourcesProcessorBuilder.buildWithoutLocalResources(AndroidResourcesProcessorBuilder.java:179)
        at com.google.devtools.build.lib.rules.android.ResourceApk.processFromTransitiveLibraryData(ResourceApk.java:328)
        at com.google.devtools.build.lib.rules.android.AndroidLibrary.create(AndroidLibrary.java:178)
        at com.google.devtools.build.lib.rules.android.AndroidLibrary.create(AndroidLibrary.java:42)
        at com.google.devtools.build.lib.analysis.ConfiguredTargetFactory.createRule(ConfiguredTargetFactory.java:484)
        at com.google.devtools.build.lib.analysis.ConfiguredTargetFactory.createConfiguredTarget(ConfiguredTargetFactory.java:190)
        at com.google.devtools.build.lib.skyframe.SkyframeBuildView.createConfiguredTarget(SkyframeBuildView.java:888)
        at com.google.devtools.build.lib.skyframe.ConfiguredTargetFunction.createConfiguredTarget(ConfiguredTargetFunction.java:899)
        at com.google.devtools.build.lib.skyframe.ConfiguredTargetFunction.compute(ConfiguredTargetFunction.java:338)
        at com.google.devtools.build.skyframe.AbstractParallelEvaluator$Evaluate.run(AbstractParallelEvaluator.java:438)
        ... 7 more
java.lang.RuntimeException: Unrecoverable error while evaluating node '@bazel_tools//src/tools/android/java/com/google/devtools/build/android/incrementaldeployment:incremental_split_stub_application BuildConfigurationValue.Key[b6c0f68614aefb61c49685ba6348794d44e10cfae2a002fe64ee70c671a75afd] false' (requested by nodes '@bazel_tools//tools/android:incremental_split_stub_application BuildConfigurationValue.Key[b6c0f68614aefb61c49685ba6348794d44e10cfae2a002fe64ee70c671a75afd] false')
        at com.google.devtools.build.skyframe.AbstractParallelEvaluator$Evaluate.run(AbstractParallelEvaluator.java:515)
        at com.google.devtools.build.lib.concurrent.AbstractQueueVisitor$WrappedRunnable.run(AbstractQueueVisitor.java:399)
        at java.base/java.util.concurrent.ForkJoinTask$AdaptedRunnableAction.exec(Unknown Source)
        at java.base/java.util.concurrent.ForkJoinTask.doExec(Unknown Source)
        at java.base/java.util.concurrent.ForkJoinPool$WorkQueue.topLevelExec(Unknown Source)
        at java.base/java.util.concurrent.ForkJoinPool.scan(Unknown Source)
        at java.base/java.util.concurrent.ForkJoinPool.runWorker(Unknown Source)
        at java.base/java.util.concurrent.ForkJoinWorkerThread.run(Unknown Source)
Caused by: java.lang.NullPointerException
        at com.google.devtools.build.lib.rules.android.BusyBoxActionBuilder.addAapt(BusyBoxActionBuilder.java:315)
        at com.google.devtools.build.lib.rules.android.AndroidResourcesProcessorBuilder.createAapt2ApkAction(AndroidResourcesProcessorBuilder.java:279)
        at com.google.devtools.build.lib.rules.android.AndroidResourcesProcessorBuilder.build(AndroidResourcesProcessorBuilder.java:208)
        at com.google.devtools.build.lib.rules.android.AndroidResourcesProcessorBuilder.buildWithoutLocalResources(AndroidResourcesProcessorBuilder.java:179)
        at com.google.devtools.build.lib.rules.android.ResourceApk.processFromTransitiveLibraryData(ResourceApk.java:328)
        at com.google.devtools.build.lib.rules.android.AndroidLibrary.create(AndroidLibrary.java:178)
        at com.google.devtools.build.lib.rules.android.AndroidLibrary.create(AndroidLibrary.java:42)
        at com.google.devtools.build.lib.analysis.ConfiguredTargetFactory.createRule(ConfiguredTargetFactory.java:484)
        at com.google.devtools.build.lib.analysis.ConfiguredTargetFactory.createConfiguredTarget(ConfiguredTargetFactory.java:190)
        at com.google.devtools.build.lib.skyframe.SkyframeBuildView.createConfiguredTarget(SkyframeBuildView.java:888)
        at com.google.devtools.build.lib.skyframe.ConfiguredTargetFunction.createConfiguredTarget(ConfiguredTargetFunction.java:899)
        at com.google.devtools.build.lib.skyframe.ConfiguredTargetFunction.compute(ConfiguredTargetFunction.java:338)
        at com.google.devtools.build.skyframe.AbstractParallelEvaluator$Evaluate.run(AbstractParallelEvaluator.java:438)
FAILED: Build did NOT complete successfully (46 packages loaded, 821 targets configured)
```
