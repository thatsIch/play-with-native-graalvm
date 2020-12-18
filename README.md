# Play with native GraalVM

This is a test application to compile a java application on graalvm for windows x64 plattform.

## Setup

There is a significant amount of setup which needs to be done that an application can be compiled natively on windows.

### Automated TL;DR

    > choco install dependencies.config -y
    > set PATH=C:\Program Files (x86)\Microsoft Visual Studio\2017\BuildTools\VC\Tools\MSVC\14.16.27023\bin\Hostx64\x64;%PATH%
    > set GRAALVM_HOME=C:\Program Files\GraalVM\graalvm-ce-java11-20.3.0
    > set JAVA_HOME=%GRAALVM_HOME%
    > set PATH=%GRAALVM_HOME%\bin;%PATH%
    > gu install native-image
    > C:\Program Files (x86)\Microsoft Visual Studio\2017\BuildTools\VC\Auxiliary\Build\vcvars64.bat

### Automated

[Chocolatey](https://chocolatey.org/) is a package manager for Windows which allows automatic installation of common applications hosted by themself.

It supports either packages or package configurations. Package configurations contain the package name and version which could be entered manually. For reference look at the [dependencies.config](dependencies.config) used in this project. To install a package configuration run

    > choco install dependencies.config -y

This installs the build tools and GraalVM onto your machine. Installation does not affect the environment variables, which are required for some tools. To set GraalVM up execute

    > set GRAALVM_HOME=C:\Program Files\GraalVM\graalvm-ce-java11-20.3.0
    > set JAVA_HOME=%GRAALVM_HOME%
    > set PATH=%GRAALVM_HOME%\bin;%PATH%

The Visual Studio Build Tools require specific environment variables to be set for the linked files like LIB, INCLUDE and [many more according to SO](https://stackoverflow.com/a/59679967/2787159). To set them you need to run

    > C:\Program Files (x86)\Microsoft Visual Studio\2017\BuildTools\VC\Auxiliary\Build\vcvars64.bat

GraalVM in its naked form does not ship with native image compilation and needs to be installed by the GraalVM Component Updater `gu`

    > gu install native-image

### Manual

There are ways if you do not want to rely on installing more applications than necessary and do everything manually.

#### GraalVM

Community Edition builds are distributed on [Github](https://github.com/graalvm/graalvm-ce-builds/releases). You can download version 20.3.0 from this [link](https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-20.3.0/graalvm-ce-java11-windows-amd64-20.3.0.zip) for Java 11 support. 

Extract the zip to any location to your liking like

    C:\Program Files\Java\GraalVM\20\11\

and export to `PATH`

    > set GRAALVM_HOME=C:\Program Files\Java\GraalVM\20\11
    > set JAVA_HOME=%GRAALVM_HOME%
    > set PATH=%GRAALVM_HOME%\bin;%PATH%

This will ensure, that GraalVM is being used and not some already installed JDK/JRE. Extracting the version should result version number `11.0.6`.

    > javac -version
    javac 11.0.6

#### Native Image

Native Image is not being shipped with GraalVM anymore but requires a post installation. You can download the Native Image jar from [Github Releases](https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-20.3.0/native-image-installable-svm-java11-windows-amd64-20.3.0.jar) and install via

    > gu -L install native-image-installable-svm-svmee-java11-windows-amd64-20.3.0.jar

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

With `native-image.cmd` you can compile Java Byte Code to native code using the native compilers for the plattform.

    > native-image.cmd Main
    [main:10724]    classlist:   1,099.78 ms,  0.96 GB
    [main:10724]        (cap):   3,229.82 ms,  0.96 GB
    [main:10724]        setup:   4,524.37 ms,  0.96 GB
    [main:10724]     (clinit):     197.28 ms,  1.20 GB
    [main:10724]   (typeflow):   4,476.40 ms,  1.20 GB
    [main:10724]    (objects):   3,368.92 ms,  1.20 GB
    [main:10724]   (features):     234.61 ms,  1.20 GB
    [main:10724]     analysis:   8,442.70 ms,  1.20 GB
    [main:10724]     universe:     396.98 ms,  1.22 GB
    [main:10724]      (parse):   1,048.61 ms,  1.22 GB
    [main:10724]     (inline):   1,035.53 ms,  1.66 GB
    [main:10724]    (compile):   5,776.59 ms,  2.25 GB
    [main:10724]      compile:   8,268.98 ms,  2.25 GB
    [main:10724]        image:   1,029.04 ms,  2.25 GB
    [main:10724]        write:     486.25 ms,  2.25 GB
    [main:10724]      [total]:  24,409.37 ms,  2.25 GB

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
| 1048ms | 21ms |
