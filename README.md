# Play with native GraalVM

This is a test application to compile a java application on graalvm for windows x64 plattform.

## Setup

There is a significant amount of setup which needs to be done that an application can be compiled natively on windows.

### Automated TL;DR

    > choco install dependencies.config -y
    > set PATH=C:\Program Files (x86)\Microsoft Visual Studio\2017\BuildTools\VC\Tools\MSVC\14.16.27023\bin\Hostx64\x64;%PATH%
    > set GRAALVM_HOME=C:\Program Files\GraalVM\graalvm-ce-java11-20.0.0
    > set JAVA_HOME=%GRAALVM_HOME%
    > set PATH=%GRAALVM_HOME%\bin;%PATH%
    > gu install native-image
    > C:\Program Files (x86)\Microsoft Visual Studio\2017\BuildTools\VC\Auxiliary\Build\vcvars64.bat

### Automated

Chocolatey is a package manager for windows which allows automatic installation of common applications hosted by https://chocolatey.org/.

It supports either packages or package configurations. Package configurations contain the package name and version which could be entered manually. For reference look at [dependencies.config](dependencies.config) used in this project.

    > choco install dependencies.config -y

This installs the build tools and GraalVM onto your machine. Installation does not affect the environment variables, which are required for some tools. To set them up execute

    > set GRAALVM_HOME=C:\Program Files\GraalVM\graalvm-ce-java11-20.0.0
    > set JAVA_HOME=%GRAALVM_HOME%
    > set PATH=%GRAALVM_HOME%\bin;%PATH%

The Visual Studio Build Tools require specific environment variables to be set for the linked files like LIB, INCLUDE and [many more according to SO](https://stackoverflow.com/a/59679967/2787159). To set them you need to run

    > C:\Program Files (x86)\Microsoft Visual Studio\2017\BuildTools\VC\Auxiliary\Build\vcvars64.bat

GraalVM in its naked form does not ship with native image compilation and needs to be installed by the GraalVM Component Updater `gu`

    > gu install native-image

### Manual

There are ways if you do not want to rely on not installed applications and do everything manually.

#### GraalVM

Community Edition builds are distributed on [Github](https://github.com/graalvm/graalvm-ce-builds/releases). This guide was using version 20.0.0 at the time. You can download it from this [link](https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-20.0.0/graalvm-ce-java11-windows-amd64-20.0.0.zip) for Java 11 support.

Extract the zip to any location to your liking like

    C:\Program Files\Java\GraalVM\20\11\

and export to `PATH`

    > set GRAALVM_HOME=C:\Program Files\Java\GraalVM\20\11
    > set JAVA_HOME=%GRAALVM_HOME%
    > set PATH=%GRAALVM_HOME%\bin;%PATH%

This will ensure, that GraalVM is being used and not some already installed JDK/JRE. Extracting the version should result version number 11.0.6.

    > javac -version
    javac 11.0.6

#### Native Image

Native Image is not being shipped with GraalVM anymore but requires a post installation. You can download the Native Image jar from [Github Releases](https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-20.0.0/native-image-installable-svm-java11-windows-amd64-20.0.0.jar) and install via

    > gu -L install native-image-installable-svm-svmee-java11-windows-amd64-20.0.0.jar

This will grant access to `native-image.cmd` which can compile Java Byte Code to native code.

### Visual Studio 2017 Build Tools

GraalVM requires a compiler for the plattform you want to run the application on. For Java 11 you require at least Visual Studio 2017. But since you do not require the whole Visual Studio Editor you can only install the the [Visual Studio 2017 Build Tool](https://my.visualstudio.com/Downloads?q=visual%20studio%202017&wt.mc_id=o~msft~vscom~older-downloads) which installs the necessary `cl.exe`.

After installation you should be able to choose the directory and prime the environment.

    > cd C:\Program Files (x86)\Microsoft Visual Studio\2017\BuildTools\VC\Auxiliary\Build
    > vcvars64.bat

## Compilation

### Java

Create a new Java class called `Main` in a file `Main.java` like

    > echo public class Main {  public static void main(String[] args) { System.out.println("Hello  Graalvm!"); }} >> Main.java

You can compile it with `javac`

    > javac Main.java

which results in `Main.class`.

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

Running `native-image.bat Main` generates a native executable on windows called `Main.exe` which prints to console.

    > Main.exe
    Hello Graalvm!

## Speed Comparison

With Powershell you can invoke

* `Measure-Command { java Main.java }` for Java and
* `Measure-Command { .\Main.exe }` for Native

for timed execution. This is no benchmark but just to showcase the startup reduction potential.

| Java | Native |
| ---- | ------ |
| 720ms | 30ms |
