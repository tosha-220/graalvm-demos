# GraalVM demos: Java Kotlin interop ahead-of-time compilation demo

This repository contains the code for a demo application for [GraalVM](graalvm.org).

## Prerequisites
* [Maven](https://maven.apache.org/)
* [GraalVM](http://graalvm.org)

## Preparation

This is a simple Java / Kotlin application, where a Java method accesses a String from Kotlin and calls a Kotlin function, which later accesses a String from a Java class. This example demonstrates how easy it is to interop between Java and Kotlin.

1. [Download GraalVM](https://www.graalvm.org/downloads/), unzip the archive, export the GraalVM home directory as the `$GRAALVM_HOME` and add `$GRAALVM_HOME/bin` to the `PATH` environment variable.

On Linux:
```
export GRAALVM_HOME=/home/${current_user}/path/to/graalvm
export PATH=$GRAALVM_HOME/bin:$PATH
```
On macOS:
```
export GRAALVM_HOME=/Users/${current_user}/path/to/graalvm/Contents/Home
export PATH=$GRAALVM_HOME/bin:$PATH
```

2. Install [Native Image](https://www.graalvm.org/docs/reference-manual/native-image/#install-native-image). For GraalVM Community users, run `gu install native-image`.
For GraalVM Enterprise users, download the Native Image JAR file from [Oracle Technology Network](https://www.oracle.com/downloads/graalvm-downloads.html) and install it by running `gu -L install component.jar`, where -L option, equivalent to --local-file or --file, tells to install a component from a downloaded archive.

3. Download or clone the repository and navigate into the `java-kotlin-aot` directory:

```
git clone https://github.com/shelajev/graalvm-demos
cd graalvm-demos/java-kotlin-aot
```

Before running this example, you need to build the application:
```
./build.sh
```

Let's look at the important line of the `build.sh` which creates a native image
from our Java application. The `native-image` utility is a part of GraalVM. It
is used to compile applications ahead-of-time for faster startup and lower
general overhead at runtime.

```
$GRAALVM_HOME/bin/native-image -cp ./target/mixed-code-hello-world-1.0-SNAPSHOT.jar -H:Name=helloworld -H:Class=hello.JavaHello -H:+ReportUnsupportedElementsAtRuntime --allow-incomplete-classpath
```

It takes a couple of parameters, the class path, the main class of the
application with the `-H:Class=...` and the name of the resulting executable
with `-H:Name=...`.

After executing the `native-image` command, check the directory, it should have
produced an executable file `helloworld`.

## Running the application

To run the application, you need to execute the fat jar file in the `target`
directory. You can run it as a normal Java application using `java`. Or, since
we have a native image prepared, you can run that directly.

The `run.sh` file executes it both ways and times them with the `time` utility.
```
time java -cp ./target/mixed-code-hello-world-1.0-SNAPSHOT-jar-with-dependencies.jar hello.JavaHello
time ./helloworld
```

On my machine `run.sh` produces approximately the following output:
```
→ ./run.sh
+ java -cp ./target/mixed-code-hello-world-1.0-SNAPSHOT-jar-with-dependencies.jar hello.JavaHello
Hello from Kotlin!
Hello from Java!

real	0m0.176s
user	0m0.119s
sys	0m0.039s
+ ./helloworld
Hello from Kotlin!
Hello from Java!

real	0m0.008s
user	0m0.003s
sys	0m0.003s
```

The performance gain of the native version is largely due to the faster startup.

## License

The sample application in this directory is taken from the JetBrains' [Kotlin-examples repository](https://github.com/JetBrains/kotlin-examples/tree/master/maven/mixed-code-hello-world).

The code from that repository is distributed under the Apache License 2.0.
The code in this directory is also distributed under the Apache License 2.0, see LICENSE.md for more details.
