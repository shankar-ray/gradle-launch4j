[ ![Get automatic notifications about new "gradle-launch4j" versions](https://www.bintray.com/docs/images/bintray_badge_color.png)](https://bintray.com/theboegl/gradle-plugins/gradle-launch4j?source=watch) [ ![Download](https://api.bintray.com/packages/theboegl/gradle-plugins/gradle-launch4j/images/download.svg) ](https://bintray.com/theboegl/gradle-plugins/gradle-launch4j/_latestVersion) [ ![Build status](https://ci.appveyor.com/api/projects/status/xscd7594tneg721r/branch/master?svg=true)](https://ci.appveyor.com/project/TheBoegl/gradle-launch4j/branch/master)

Go To section
* [Introduction](#introduction)
* [Tasks](#tasks)
* [Configuration](#configuration)
* [Launch4jLibraryTask](#launch4jlibrarytask)
* [Launch4jExternalTask](#launch4jexternaltask)
* [Contributors](#contributors)
* [Version](#version)

# Introduction

The gradle-launch4j plugin uses [launch4j](http://launch4j.sourceforge.net/) to create windows .exe files for java applications.
This plugin is compatible with the Gradle versions 2 and later.

# Tasks

There are 3 tasks:

* **createExe** - Backward compatible task to generate an .exe file. *Execute this task to generate an executable.* With default settings this creates the executable under `${project.buildDir}/launch4j` and puts all runtime libraries into the lib subfolder. 
* createAllExecutables - Helper task to run all tasks of the `Launch4jExternalTask` and `Launch4jLibraryTask` type.
* ~~launch4j~~ - Deprecated placeholder task that depends on the above. This task was deprecated in favor of the createExe task and to avoid the name conflict of launch4j on the project.

Launch4j no longer needs to be installed separately, but if you want, you can still use it from the *PATH*. Since version 2.0 use the [Launch4jExternalTask](#launch4jexternaltask) to create your executable.

# Configuration

The configuration follows the structure of the launch4j xml file. The gradle-launch4j plugin tries to pick sensible defaults based on the project. The only required
value is the `mainClassName`.

## How to include
An example configuration within your `build.gradle` for use in all Gradle versions might look like:

    buildscript {
      repositories {
        maven {
          url 'https://plugins.gradle.org/m2/'
        }
      }
      dependencies {
        classpath 'gradle.plugin.edu.sc.seis.gradle:launch4j:2.1.0'
      }
    }

    repositories {
      mavenCentral()
    }

    apply plugin: 'java'
    apply plugin: 'edu.sc.seis.launch4j'

    launch4j {
      mainClassName = 'com.example.myapp.Start'
      icon = 'icons/myApp.ico'
    }

The same script snippet for new, incubating, plugin mechanism introduced in Gradle 2.1:

    apply plugin: 'java'

    plugins {
      id 'edu.sc.seis.launch4j' version '2.1.0'
    }

    launch4j {
      mainClassName = 'com.example.myapp.Start'
      icon = 'icons/myApp.ico'
    }


If no repository is configured before applying this plugin the *Maven central* repository will be added to the project.

See the [Gradle User guide](http://gradle.org/docs/current/userguide/custom_plugins.html#customPluginStandalone) for more information on how to use a custom plugin and the [plugin page](https://plugins.gradle.org/plugin/edu.sc.seis.launch4j) for the above settings.

## How to configure

The values configurable within the launch4j extension along with their defaults are:

 *    String outputDir = "launch4j"
 *    String libraryDir = "lib"
 *    Object copyConfigurable

&nbsp;  

 *    String xmlFileName = "launch4j.xml"
 *    String mainClassName
 *    boolean dontWrapJar = false
 *    String headerType = "gui"
 *    String jar = "lib/"+project.tasks[JavaPlugin.JAR_TASK_NAME].archiveName or "", if the JavaPlugin is not loaded
 *    String outfile = project.name+'.exe'
 *    String errTitle = ""
 *    String cmdLine = ""
 *    String chdir = '.'
 *    String priority = 'normal'
 *    String downloadUrl = "http://java.com/download"
 *    String supportUrl = ""
 *    boolean stayAlive = false
 *    boolean restartOnCrash = false
 *    String manifest = ""
 *    String icon = ""
 *    String version = project.version
 *    String textVersion = project.version
 *    String copyright = "unknown"
 *    String companyName = ""
 *    ~~String description = project.name~~ deprecated use fileDescription instead
 *    String fileDescription = project.name
 *    String productName = project.name
 *    String internalName = project.name
 *    String trademarks
 *    String language = "ENGLISH_US"
 *    String opt = ""
 *    String bundledJrePath
 *    boolean bundledJre64Bit = false
 *    boolean bundledJreAsFallback = false
 *    String jreMinVersion = project.targetCompatibility or the current java version, if the property is not set
 *    String jreMaxVersion
 *    String jdkPreference = "preferJre"
 *    String jreRuntimeBits = "64/32"
 *    String mutexName
 *    String windowTitle
 *    String messagesStartupError
 *    String messagesBundledJreError
 *    String messagesJreVersionError
 *    String messagesLauncherError
 *    Integer initialHeapSize
 *    Integer initialHeapPercent
 *    Integer maxHeapSize
 *    Integer maxHeapPercent
 *    String splashFileName
 *    boolean splashWaitForWindows = true
 *    Integer splashTimeout = 60
 *    boolean splashTimeoutError = true

Removed properties
*    ~~String launch4jCmd = "launch4j"~~ (use the [Launch4jExternalTask](#launch4jexternaltask) instead)
*    ~~boolean externalLaunch4j = false~~ (use the [Launch4jExternalTask](#launch4jexternaltask) instead)

### Configurable input configuration

In order to configure the input of the *copyL4jLib* task set the `copyConfigurable` property.
The following example shows how to use this plugin hand in hand with the shadow plugin:

    launch4j {
        outfile = 'TestMain.exe'
        mainClassName = project.mainClassName
        copyConfigurable = project.tasks.shadowJar.outputs.files
        jar = "lib/${project.tasks.shadowJar.archiveName}"
    }

If you use the outdated fatJar plugin the following configuration correctly wires the execution graph:

    fatJar {
        classifier 'fat'
        manifest {
            attributes 'Main-Class': project.mainClassName
        }
    }
    
    copyL4jLib.dependsOn fatJar
    fatJarPrepareFiles.dependsOn jar
    
    launch4j {
        outfile = 'TestMain.exe'
        mainClassName = project.mainClassName
        copyConfigurable = project.tasks.fatJar.outputs.files
        jar = "lib/${project.tasks.fatJar.archiveName}"
    }

# Launch4jLibraryTask
This task type can be used to build multiple executables with Launch4j. The default launch4j configuration from [how to configure](#how-to-configure) is used for the default values but can be adjusted.
To avoid replacing the resulting xml file or executable on each invocation, `xmlFileName` and `outfile` are set to the task name (`name.xml` and `name.exe` respectively).

Creating three executables is as easy as:

    launch4j {
        outfile = 'MyApp.exe'
        mainClassName = 'com.example.myapp.Start'
        icon = 'icons/myApp.ico'
        productName = 'My App'
    }
    
    task createFastStart(type: edu.sc.seis.launch4j.tasks.Launch4jLibraryTask) {
        outfile = 'FastMyApp.exe'
        mainClassName = 'om.example.myapp.FastStart'
        icon = "icons/myAppFast.ico"
        fileDescription = 'The lightning fast implementation'
    }
    
    task "MyApp-memory"(type: edu.sc.seis.launch4j.tasks.Launch4jLibraryTask) {
        fileDescription = 'The default implementation with increased heap size'
        maxHeapPercent = 50
    }

Running the `createAllExecutables` task will create the following executables in the launch4j folder located in the buildDir:
* MyApp.exe
* FastMyApp.exe
* MyApp-memory.exe

# Launch4jExternalTask
The [section from above](#launch4jlibrarytask) applies to this task, too.
This task type has the following additional property:

* String launch4jCmd = "launch4j"

In order to use a launch4j instance named 'launch4j-test' located in the PATH create a task like the following:

    launch4j {
        mainClassName = 'com.example.myapp.Start'
    }
    
    task createMyApp(type: edu.sc.seis.launch4j.tasks.Launch4jExternalTask) {
        launch4jCmd = 'launch4j-test'
        outfile = 'MyApp.exe'
    }

# Contributors

* [Sebastian Bögl](https://github.com/TheBoegl) (Maintainer)
* [Philip Crotwell](https://github.com/crotwell) (Creator)
* [Sebastian Schuberth](https://github.com/sschuberth)
* [FourtyTwo](https://github.com/FFourtyTwo)

# Version

See [VERSION.md](VERSION.md) for more information.
