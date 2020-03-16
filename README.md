# Play with native GraalVM

This is a test application to compile a java application on graalvm for windows x64 plattform.

## Setup

There is a significant amount of setup which needs to be done that an application can be compiled natively on windows.

### GraalVM

Community Edition builds are distributed on [Github](https://github.com/graalvm/graalvm-ce-builds/releases). This guide was using version 20.0.0 at the time. You can download it from this [link](https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-20.0.0/graalvm-ce-java11-windows-amd64-20.0.0.zip) for Java 11 support.

Extract the zip to any location to your liking like

    C:\Program Files\Java\GraalVM\20\11\

and export to `PATH`

    > set GRAALVM_HOME=C:\Program Files\Java\GraalVM\20\11
    > set PATH=%GRAALVM_HOME%\bin;%PATH%

This will ensure, that GraalVM is being used and not some already installed JDK/JRE. Extracting the version should result version number 11.0.6.

    > javac -version
    javac 11.0.6

### Native Image

Native Image is not being shipped with GraalVM anymore but requires an installation via GraalVM Updater Tool. Since we chose the community edition we can install Native Image via

    > gu install native-image

alternatively you could download the Native Image jar and install via

    > gu -L install native-image-installable-svm-svmee-java11-windows-amd64-20.0.0.jar

This will grant access to `native-image.cmd` which can compile Java Byte Code to native code.

### Visual Studio 2019 Community Edition

GraalVM requires a compiler for the plattform you want to run the application on. For Java 11 you require at least Visual Studio 2017 15.5.5 but it works with the latest [Visual Studio 2019 16.0.0](https://visualstudio.microsoft.com/downloads/#visual-studio-community-2019). There are probably ways to install less like the [Visual Studio 2019 Build Tool](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2019) which installs the necessary `cl.exe`.

After installation you should be able to choose the directory and prime the environment.

    > cd C:\Program Files (x86)\Microsoft Visual Studio\2019\BuildTools\VC\Auxiliary\Build
    > vcvars64.bat

## Compilation

### Java

Create a new Java class called `A` in a file `A.java` like

    > echo public class A {  public static void main(String[] args) { System.out.println("Hello  Graalvm!"); }} >> A.java

You can compile it with `javac`

    > javac A.java

which results in `A.class`.

### Native

With `native-image.bat` you can compile Java Byte Code to native code using the native compilers for the plattform.

    > native-image.cmd A
    [a:3532]    classlist:   2,551.09 ms,  1.00 GB
    [a:3532]        (cap):   3,666.40 ms,  1.29 GB
    [a:3532]        setup:   7,093.33 ms,  1.29 GB
    [a:3532]   (typeflow):  11,775.28 ms,  1.59 GB
    [a:3532]    (objects):   6,350.12 ms,  1.59 GB
    [a:3532]   (features):     407.08 ms,  1.59 GB
    [a:3532]     analysis:  18,882.42 ms,  1.59 GB
    [a:3532]     (clinit):     303.17 ms,  1.59 GB
    [a:3532]     universe:     982.35 ms,  1.59 GB
    [a:3532]      (parse):   3,217.60 ms,  1.59 GB
    [a:3532]     (inline):   2,456.11 ms,  1.96 GB
    [a:3532]    (compile):  15,663.34 ms,  1.96 GB
    [a:3532]      compile:  22,033.20 ms,  1.96 GB
    [a:3532]        image:   1,733.15 ms,  1.96 GB
    [a:3532]        write:     542.60 ms,  1.96 GB
    [a:3532]      [total]:  54,366.13 ms,  1.96 GB

## Execution

Running `native-image.bat A` generates a native executable on windows called `A.exe` which prints to console.

    > A.exe
    Hello Graalvm!
