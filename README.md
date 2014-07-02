# Overview
A collection of basic projects to demonstrate how Gradle works with
multi-language builds. If you want to build the native projects, then your best
bet is to be in \*nix. I didn't take the time to make sure everyone who checks
this repository out could build the native projects without some strings
attached (of course, some strings are always attached when it comes to native
programming).

# Basic Things That Might Make You Twitch

## Gradle Wrapper
The Gradle wrapper is pretty slick. This is a self-contained distribution of
Gradle. Committing the files generated by `gradle wrapper` into version control
allows users to quickly checkout the repository and run Gradle commands without
installing Gradle! If you have Gradle installed, then running `gradle wrapper`
will produce the following items:

+ gradlew
+ gradlew.bat
+ gradle/

The `gradlew` and `gradle.bat` files are the Gradle wrappers that should always
be used when working in a Gradle project (this is because one project may use a
different version of Gradle then another). By default, running any command with
the wrapper will download the specified version of Gradle. No need to manually
install your own distribution of Gradle. You can change the wrapper settings to
download from a different location if you are in a network-restricted area.
This is a really nice feature.

To run any of the Gradle tasks, use `./gradlew` on \*nix or `gradle.bat` on
Windows. I'll be mainly using `./gradlew` in this document to refer to the
wrapper.

## Gradle Daemon
When you execute a Gradle command, Gradle is doing a lot under the covers, so
Gradle comes off as slow. However, turning the Gradle daemon on by default for
a developer environment ([see
here](https://github.com/k-mack/gradle-examples/blob/master/gradle.properties#L1)
helps significantly since Gradle uses the same JVM for each build. The daemon
expires after three hours. The Gradle daemon is probably not what you want when
doing continuous integration builds (builds that take a really long time).

## Root Project
Gradle can only have one root project (shocking, I know). All subprojects are
traced back through their directories to the root.

## The Colon (`:`)
The colon is used to denote the path to a project or a task (which is
associated with a project).  For example, when setting up the `simple-java`
project in the root's `build.gradle`, I used `project('jvm:simple-java')` ([see
here](https://github.com/k-mack/gradle-examples/blob/master/build.gradle#L49)).
I could have used `project(':jvm:simple-java')` with a colon in the beginning
to tell Gradle the project's fully qualified name, but since the project
configuration is occurring in the build file located in the root project, I
don't need to specify the first colon.

# Setting Things Up
If you haven't ran `./gradew` (or `gradlew.bat` on Windows) yet, then do so.
This will setup the Gradle wrapper for the repository.

## Defining Projects
Projects can be define either in their own Gradle build file or in the root
project's build file. The former method is used for the `native` projects; the
latter is used for the `jvm` projects ([see
here](https://github.com/k-mack/gradle-examples/blob/master/build.gradle#L5)).

## Including Projects
You need to tell Gradle about which projects it should manage. These projects
are included by specifying them in the `settings.gradle` file.

To see the projects that Gradle will oversee, run `./gradlew projects`.

## Gradle Tries to Find Itself
Odd subsection title. What am I saying? If you have Gradle installed, navigate
to a project that does not have a Gradle build file, such `jvm/simple-java`,
and run `gradle build`, Gradle will trace its way up the project directory's
hierarchy to try and find the project's root directory. Of course, this will
bypass the Gradle wrapper if one exists in the root directory... rules, who
needs 'em.

## Running Tasks
To get a list of available tasks for a project, run `./gradlew tasks`. For
example, `./gradlew jvm:jni:tasks`

You can call a task by its fully qualified Gradle path (usually a good idea),
such as `./gradlew level1:level2:levelN:taskName` or by just the task's name,
such as `./gradlew taskName`. In the latter, all tasks of `taskName` in the
Gradle project will be executed.

For example, the following two commands will build the JNI shared library:
`./gradlew jniPlatformSharedLibrary` or `./gradlew
jvm:jni:jniPlatformSharedLibrary`

### The Native Install\* Tasks
If you ran `./gradlew tasks`, then you probably saw the native install tasks,
such as `installMainExecutable`. These tasks are really nice for native
projects because they produce a binary that is "ready to go." For example,
providing you have GCC install that supports c11, you can run `./gradlew
native:simple-c-lib-dependent:installMainExecutable` (or just `./gradlew
installMainExecutable`), which will produce the binary
`native/simple-c-lib-dependent/build/install/mainExecutable/main`. The
`native:simple-c-lib-dependent` project depends on `native:simple-c-lib`, so if
you just ran `./gradlew native:simple-c-lib-dependent:mainExecutable` you'll
get a binary, but when you run it you'll get:

    $ native/simple-c-lib-dependent/build/binaries/mainExecutable/main
    native/simple-c-lib-dependent/build/binaries/mainExecutable/main: error while loading shared libraries: libstringutils.so: cannot open shared object file: No such file or directory

You can update `LD_LIBRARY_PATH` and do the same above or just run the
`installMainExecutable` task.

# Examples

## JNI
Run the JNI project by doing the following steps:

    $ ./gradlew jvm:jni:build
    $ java -Djava.library.path=jvm/jni/build/binaries/jniPlatformSharedLibrary -cp jvm/jni/build/libs/jni.jar example.jni.app.Main [optional_arg]

