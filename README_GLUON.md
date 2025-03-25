Gluon version of libsignal
==========================

Introduction
------------

The Gluon fork of libsignal contains experimental components written in Rust that are accessible for
Java applications through JNI. The extra components are the following:

* chat: network communication using protobuf
* grpc: network communication using gRPC
* quic: network communication using the QUIC protocol

The project contains two aspects:

* an implementation of the logic in Rust
* an interface to use the Rust components in Java

The Rust code is compiled into a native shared library for the following platforms:

* linux x64 and aarch64
* mac x64 and aarch64
* windows x64

Build process
-------------

Everything is built from a Gradle project that is located in the `java` directory. Running a build
locally consists of the following steps:

1. Navigate into the `java` directory
2. Run the following gradle command:

    ```
    ./gradlew build publishToMavenLocal -PskipAndroid -x :client:proguard -x :client:diffUnusedProguard
    ```

3. After the build succeeded, two jars should be deployed to the local maven repository:
    1. One that contains the java classes
    2. One that contains the native shared library targeted for the current platform

Producing a new full release
----------------------------

We use Github Actions to create a release by using the workflow `jni_artifacts.yml`. This workflow is
triggered manually. The workflow is split into two jobs. The first job generates the native shared
library for all the platforms except linux x64. Each shared library is uploaded as an artifact. The
second job then builds the native shared library for linux x64, downloads the previously generated
native shared libraries for the other platforms and compiles the java classes. It then creates one jar
with the java classes and separate jars for each platform containing only the respective native shared
library. The final step is to deploy these jars to the Gluon nexus repository.

The steps for building a new release are:

1. Update the version at the top of `java/build.gradle`, e.g. `0.67.6-gluon-1`
2. Commit and push the changes
3. Create a tag that matches the version and push it, e.g. `v0.67.5-gluon-1`
4. Trigger the workflow: [Upload Java libraries to Sonatype](https://github.com/gluonhq/libsignal/actions/workflows/jni_artifacts.yml) by selecting `Run workflow` from the dropdown and then select the tag for
the `Use workflow from` dropdown.

Syncing with upstream
---------------------

This is the process for syncing with a specific release from the upstream repository:

1. Choose a version that we need to sync to
2. Create a branch from main called `patch-VERSION`, e.g. `patch-v0.67.5`
3. Fetch everything from upstream: `git fetch upstream`
4. Merge the tagged commit with the current branch: `git merge upstream v0.67.5`
5. Resolve any conflicts
6. Update the version in `java/build.gradle` to match the version that we synced with
7. Run a build as described above