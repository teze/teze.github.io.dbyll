
#Gradle for Android (Part 2)

-------------------------------------------

 _Edit by Fooyou Email : <foyou23@qq.com>_ 
    
    10:56 2014-5-5
    
---------------------------------------------

 在前一部分讲述了，Gradle for Android 的一些基本配置，可以满足基本的构建需求，但是有了一定规模的APP的需求却会越来越复杂的，所以，会用到更复杂的配置。接下来，就来看看稍微复杂一点点的咯。
 
引用JNI ，So库作为lib的情景
------------------------------
  
  如果你的项目有用到So库，要如何在`gradle`中，把so库也编译到apk中呢？现在，还是使用之前的`HelloWorld` 项目为例子。在`gradle.build`文件中，`Android` 大括号中添加如下脚本,编写2个task：

```
        //这个任务的意思是，把libs下所有的`So`文件，打包成`zip`，并且复制到`buildDir` native-libs目录下面。
      	task nativeLibsToJar(type: Zip, description: 'create a jar archive of     the native libs') {
        destinationDir file("$buildDir/native-libs")
        baseName 'native-libs'
        extension 'jar'
        from fileTree(dir: 'libs', include: '**/*.so')
        into 'lib/'
    }
    
    //这个任务则是把，上面任务打包好的so文件，加入参与编译。
    tasks.withType(Compile) {
        compileTask -> compileTask.dependsOn(nativeLibsToJar)
    }
```
配置是比较简单了，试试！看看能否运行。到目前好像Android stuio 有更好的办法，来做这件事情了，到时候在看咯。
**记住，这里说的配置，都是Android的方法块中添加。**
  
引用外部项目，作为当前项目的lib的情景
-------------------------------------
在项目开发中，有时候要用第三方库，（比如：`Android-PullToRefresh`，`ActionBarSherlock`）而这些库本身是一个Android项目，又不能直接打包成普通的Jar包，如果是Eclipse ADT开发，有直接的配置，比较简单，Android studio 配置也很简单，但是如果在Eclipse 使用Gradle 呢？

首先新建一个libProject，**(现在,就有2个项目了libProject 和 HelloWorld)**
    
> **tips**：`project.properties` 配置文件，添加一行 `android.library=true`，这个项目就会被编译成`lib`项目了。本例子中`libProject` 就是如此。

我们来看看新的文件目录结构：

```
android
├─Hellowold
└─LibProject
```
现在我们需要新增配置文件，在根目录android 下新增3个配置文件，分别是：`build.gradle`
  `local.properties`  `settings.gradle`

```
android
├ build.gradle
├ local.properties
├ settings.gradle
├─Hellowold
└─LibProject
    build.gradle
```
`build.gradle` 文件内容如下：
```
//这个文件是整个项目（本文中的android）全局配置，每一个子模块中也有相应的`build.gradle`文件。Hellowold ，LibProject 下都有的。

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:0.9.+'
    }
}
```

`local.properties` 文件内容如下：
```
sdk.dir=F:\\Android\\sdk // 跟Helloworld项目的一样，指定sdk路径
```

`settings.gradle` 文件内容如下：
```
    //这个文件内容是整个项目，模块配置，表明包含哪些模块。
    include ':LibProject'      // 当前项目包含LibProject
    include ':Hellowold'     // 当前项目包含Hellowold
```

`LibProject` 下的  `build.gradle` 文件内容如下：
```
    //你会发现，这个文件基本和Helloworld下的配置差不多，配置的参数意思是相同的，但有一点注意，也是最大的区别。 apply plugin: 'android-library' 这句。
    
    
    apply plugin: 'android-library'  //没错就是这里，要写成'android-library'
                                     //表明这是一个library    
        dependencies {
            compile fileTree(dir: 'libs', include: '*.jar')
    }
    
    android {
        compileSdkVersion 19
        buildToolsVersion "19.0.3"
    
       
        lintOptions {
            abortOnError false
        }
    }
```
Ok ,有了上面的配置，应该可以正常运行，试试咯。

指定文件目录配置
----------------
本文中的`HelloWorld`项目是在`Eclipse`中创建的，其目录结构是 ADT 模式的目录结构，如果要用gradle 编译的，可能还需要指定目录配置。同样在Android方法块中 新增一个任务，如下：

```
//这是指定项目，模块，对应的文件路径，文件名字
sourceSets {

       main {
           manifest.srcFile 'src/main/AndroidManifest.xml'
           java.srcDirs = ['src']
           resources.srcDirs = ['src']
           aidl.srcDirs = ['src']
           renderscript.srcDirs = ['src']
           res.srcDirs = ['res']
           assets.srcDirs = ['assets']
       }

       instrumentTest.setRoot('tests')  /* 测试目录*/
    }
```
如果是用 Android studio 创建的项目，就不需要这么配置了，因为它有默认约定的目录结构。

指定独立的res，资源文件的情景
----------------------------------
如果，想要给不同渠道的apk包，指定不同的res资源文件呢？比如，一个App 使用不同的drawble，不同icon，不同的string 等等...

现假设，需要给GooglePlay这个版本的渠道，指定独立的res资源，可以如下操作：

- 在项目跟目录新增一个新的目录GooglePlay，然后把对应的res文件放入其中，新的目录结构如下：

 ```
    │  build.gradle
    │  local.properties
    │  settings.gradle
    │
    ├─Hellowold
    │  └─GooglePlay
    │      └─res
    └─LibProject
            build.gradle
 ```

- 修改`sourceSets`任务，内容如下：

 ```
sourceSets {

       main {
           manifest.srcFile 'src/main/AndroidManifest.xml'
           java.srcDirs = ['src']
           resources.srcDirs = ['src']
           aidl.srcDirs = ['src']
           renderscript.srcDirs = ['src']
           res.srcDirs = ['res']
           assets.srcDirs = ['assets']
       }

       instrumentTest.setRoot('tests')  /* 测试目录*/
       
       //注意到这里，给GooglePlay指定特定的res资源。
        GooglePlay { 
            res.srcDirs = ['GooglePlay/res']
        }
    }
    
 ```
- 以此同时，你可能还需要在productFlavors 任务中新增一个GooglePlay渠道。
 
 ```
 productFlavors { 
	   // 同样可以在这里，配置GooglePlay的其他参数，
        GooglePlay { 
	   
	   }
 }
 ```
 
其他资源文件类似，src，assets 等待。


指定独立的AndroidManifest的情景
-------------------------------
这个部分，或许很有用处，尤其是在打多渠道包的时候。也稍微有点复杂。理论上可以跟res一样，指定特地的AndroidManifest配置文件，但是，gradle 做文件合并的时候，不够智能，总不能符合预定的效果。所以会采用更为复杂一点的办法。原理很简单的：就是新建一个task 把对应的 AndroidManifest 文件复制到编译目录下。例子如下：

```
android.applicationVariants.all { variant ->
        def flavor = variant.productFlavors[0].name
               
	         variant.processManifest.doLast{ 
		        copy{
		            from("${buildDir}/manifests"){
		                include "${variant.dirName}/AndroidManifest.xml" //把编译目录下的AndroidManifest复制一份
		            }
		            into("${buildDir}/manifests/$flavor")//复制 AndroidManifest到对应的渠道目录下
		 
					// 你想要做的操作，这里是更改AndroidManifest的某个字符。例子中是替换UMENG_CHANNEL_VALUE 的值为 对应的渠道名字
		            filter{
		                String line -> line.replaceAll("UMENG_CHANNEL_VALUE", "$flavor")//动态替换AndroidManifest的字符
		            } 
		            variant.processResources.manifestFile = file("${buildDir}/manifests/$flavor/${variant.dirName}/AndroidManifest.xml")
		          } 
		     }
    }
```
嗯，基本就是这样了。

总结
-----
先写的这里把，后续可能还会写这方面相关的内容。
Gradle的优势就是可以使用`groovy`脚本语言，来自定义各种`task`，这样就显得功能强大了，随心所欲了。








 
