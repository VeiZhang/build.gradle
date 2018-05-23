# build.gradle
统一管理android studio中的gradle编译脚本

## 目的

**当工程集成很多Modules时，每个Module都有一个build.gradle，并且带有如下重复的代码；对每个build.gradle修改很麻烦，因此统一管理build.gradle文件是必要的**

```
// 重复代码
android {
    compileSdkVersion 25
    buildToolsVersion "25.0.0"

    defaultConfig {
        minSdkVersion 15
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:24.2.1'
}
```

* **优化代码**
* **使用相同的编译配置**
* **统一管理远程依赖**
* 减少`sync project`次数


## 本地配置<a name="本地配置">

* 在项目根目录下创建config.gradle文件，作为管理配置文件
	
	```
	import java.text.SimpleDateFormat
	import java.util.regex.Matcher
	import java.util.regex.Pattern
	
	ext {
	    // 配置
	    android = [
	            /*************************原生配置*************************/
	            compileSdkVersion       : 25,
	            buildToolsVersion       : "25.0.0",
	            minSdkVersion           : 17,
	            targetSdkVersion        : 23,
	            versionCode             : getSvnVersionCode(),
	            versionName             : getVersionName(),
	
	            /*************************自定义配置*************************/
	            androidSupportSdkVersion: "23.0.0"
	    ]
	
	    // 依赖
	    dependencies = [
	            /*************************原生依赖*************************/
	            "appcompat-v7"   : "com.android.support:appcompat-v7:${android["androidSupportSdkVersion"]}",
	            "support-v4"     : "com.android.support:support-v4:${android["androidSupportSdkVersion"]}",
	            "cardview-v7"    : "com.android.support:cardview-v7:${android["androidSupportSdkVersion"]}",
	            "recyclerview-v7": "com.android.support:recyclerview-v7:${android["androidSupportSdkVersion"]}",
	            "design"         : "com.android.support:design:${android["androidSupportSdkVersion"]}",
	            "annotations"    : "com.android.support:support-annotations:${android["androidSupportSdkVersion"]}",
	            "gridlayout-v7"  : "com.android.support:gridlayout-v7:${android["androidSupportSdkVersion"]}",
	
	            /*************************第三方依赖*************************/
	            "basetools"      : "com.excellence:basetools:1.2.5",
	            "commons-lang3"  : "org.apache.commons:commons-lang3:3.3.2",
	            "rxjava"         : 'io.reactivex:rxjava:1.3.0',
	            "rxandroid"      : 'io.reactivex:rxandroid:1.2.1',
	            "gson"           : "com.google.code.gson:gson:2.2.4"
	    ]
	}
	
	def getVersionName() {
	    String prefix = "1.0." + getSvnVersionCode()
	    return prefix + ' [' + getDate() + ']'
	}
	
	def getSvnVersionCode() {
	    def process = ("svnversion -c " + getBuildDir().parent).execute()
	    process.waitFor()
	    def version = process.in.text
	    Pattern pattern = Pattern.compile("(\\d+:)?(\\d+)\\D")
	    Matcher matcher = pattern.matcher(version)
	    if (matcher.find()) {
	        version = matcher.group(matcher.groupCount())
	    }
	    try {
	        return Integer.parseInt(version)
	    } catch (e) {
	        println e.getMessage()
	    }
	    return 0
	}
	
	def static getDate() {
	    String date = new SimpleDateFormat("MMddyyyy").format(new Date())
	    return date
	}
	```

	![config.gradle][config.gradle]

* 在项目根目录的build.gradle中引用，供其他的Module使用；只想让单独的Module使用，则在该Module的build.gradle里引入

	```
	apply from: "config.gradle"
	```

	![config引用][config引用]


* 在Module目录的build.gradle中使用变量
	
	![config使用][config使用]


## 远程配置

* 远程配置配置其他步骤与**[本地配置](#本地配置)**是一样，不同的是引用的方式，导入的不是路径里的文件，而是一个文件链接

	```
	apply from: "https://github.com/VeiZhang/build.gradle/blob/master/config.gradle?raw=true"
	```
	
	![远程配置引用][远程配置引用]


## 继承方式

本地、远程的配置都只是保存了变量，但是如果想引用gradle文件里面的函数，其实很简单，与上面是相同的做法，必须注意的是，**apply from:** 放置的顺序很重要！

**注意**：引入位置在文件的开头、中间、结尾等处使用。因为gradle脚本编译是顺序执行的，如果父脚本与子脚本有相同的方法，此时父脚本引入的顺序就非常重要，不同的位置，执行先后不一样。

![继承方式][继承方式]



[config.gradle]:https://github.com/VeiZhang/build.gradle/blob/master/images/config.gradle.png?raw=true "config.gradle"
[config引用]:https://github.com/VeiZhang/build.gradle/blob/master/images/config引用.png?raw=true "config引用"
[config使用]:https://github.com/VeiZhang/build.gradle/blob/master/images/config使用.png?raw=true "config使用"
[远程配置引用]:https://github.com/VeiZhang/build.gradle/blob/master/远程配置引用.png?raw=true "远程配置引用"
[继承方式]:https://github.com/VeiZhang/build.gradle/blob/master/继承方式.png?raw=true "继承方式"