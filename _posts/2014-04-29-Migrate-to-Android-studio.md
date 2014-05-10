
# Android studio 从IntelliJ项目的迁移到Gradle 构建环境

------------------------------------------------------

 _Edit by Fooyou Email : <foyou23@qq.com>_ 

    2014/04/29
    
------------------------------------------------------


### Migrating from IntelliJ Projects


    We might provide an automatic migration option in Android Studio in the future.

我们可能会在后续的Android Studio 的发布版本中加入自动转换功能。

    For now, to migrate your IntelliJ project to an Android Gradle project (which can then be imported into IntelliJ and supported directly in IntelliJ), follow the following steps:
    Create a basic "build.gradle" file. Unlike the default Gradle file created by Android Studio when you create a new project, the following gradle file will point the source code directories to the existing folders (e.g. res/, src/) instead of the default new directory structure used for Gradle projects (src/main/java/, src/main/res/, etc). A sample gradle file is given below.


如果你之前一直是使用IntelliJ 开发项目的，而现在想要把你的项目迁移到Android Studio 变成Android Gradle 项目的话，你可以直接导入你的项目到Android Studio中，然后按照如下步骤哦：

1. 创建一个最小化的配置文件 "build.gradle"(如果是Android Studio创建的项目，这个文件工具已经帮你创建了好！)

2. build.gradle包含以下内容，定义源代码的指向目录的位置。比如（res ,src 目录），而Android Studio	    			     创建的项目，默认的目录结构是这样的(src/main/java/, src/main/res/, etc)

3. 这里有个简单的样本文件啦：

```
    build.gradle:
    buildscript {
        repositories {
            mavenCentral()
        }
        dependencies {
            classpath 'com.android.tools.build:gradle:0.5.+'
        }
    }
    apply plugin: 'android'
    
    dependencies {
        compile fileTree(dir: 'libs', include: '*.jar')
    }
    
    android {
        compileSdkVersion 18
        buildToolsVersion "18.0.1"
    
        sourceSets {
            main {
                manifest.srcFile 'AndroidManifest.xml'
                java.srcDirs = ['src']
                resources.srcDirs = ['src']
                aidl.srcDirs = ['src']
                renderscript.srcDirs = ['src']
                res.srcDirs = ['res']
                assets.srcDirs = ['assets']
            }
    
            // Move the tests to tests/java, tests/res, etc...
            instrumentTest.setRoot('tests')
    
            // Move the build types to build-types/<type>
            // For instance, build-types/debug/java, build-types/debug/AndroidManifest.xml, ...
            // This moves them out of them default location under src/<type>/... which would
            // conflict with src/ being used by the main source set.
            // Adding new build types or product flavors should be accompanied
            // by a similar customization.
            debug.setRoot('build-types/debug')
            release.setRoot('build-types/release')
        }
    }
```

简单解释一下：

1.依赖配置外部类库：在这个配置文件中，可以指定你的项目引用到的外部项目源码，通过依赖配置的方式，应该不少开发者，遇到过在引用外部项目的时候，比如 使用ActionBarSherlock的时候，我们通常是需要把源码导入到开发环境中，然后需要在projects文件中配置指定这个路径，有了Gradle以后，你就不需要这样了，这里你只要简单dependencies 指定就好了。Gradle构建系统，会帮你自动边缘，打包到项目中。
 

    Identify which library projects you are using (such as ActionBarSherlock). In Gradle you no longer need to add in these libraries as source code projects; you can simply refer to them as dependencies, and the build system will handle the rest; downloading, merging in resources and manifest entries, etc. For each library, look up the corresponding AAR library dependency name (provided the library in question has been updated as a android library artifact), and add these to the dependency section.

想要了解更多的依赖配置说明，可以上这个网站查看http://gradleplease.appspot.com/

    To find the right declaration statements for your libraries, you might find the following useful: http://gradleplease.appspot.com/


2.如果想测试你的依赖库配置是否正确以否，可以允许这个 assembleDebug 脚步，（当你配置好以后，在Gradle Task 会找到这个脚本的）

    Test your build by running gradle assembleDebug in your project. If you have not built with Gradle before, install it from http://www.gradle.org/downloads. Note that when you create new projects with Studio, we create a gradle wrapper script ("gradlew" and "gradlew.bat") in the project root directory, so any users of the project can just run "gradlew assembleDebug" etc in your project and gradle is automatically installed and downloaded. However, your existing IntelliJ projects presumably does not already have the gradle scripts.)
    Note that IntelliJ projects for Android generally follow the same structure as Eclipse ADT projects, so the instructions in the Eclipse migration guide might be helpful.

    For more information on customizing your build once you have the basics set up, see the new build system user guide. For additional information, see the build system overview page.