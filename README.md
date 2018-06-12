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
	    // 插件
	    plugins = [
	            application      : "com.android.application",
	            library          : "com.android.library",
	            maven            : "com.github.dcendents.android-maven",
	            bintray          : "com.jfrog.bintray",
	            novoda           : "com.novoda.bintray-release",
	            greendao         : "org.greenrobot.greendao",
	            'greendao-gradle': "org.greenrobot:greendao-gradle-plugin:3.2.2"
	    ]
	
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
	            "appcompat-v7"     : "com.android.support:appcompat-v7:${android["androidSupportSdkVersion"]}",
	            "support-v4"       : "com.android.support:support-v4:${android["androidSupportSdkVersion"]}",
	            "cardview-v7"      : "com.android.support:cardview-v7:${android["androidSupportSdkVersion"]}",
	            "recyclerview-v7"  : "com.android.support:recyclerview-v7:${android["androidSupportSdkVersion"]}",
	            "design"           : "com.android.support:design:${android["androidSupportSdkVersion"]}",
	            "annotations"      : "com.android.support:support-annotations:${android["androidSupportSdkVersion"]}",
	            "gridlayout-v7"    : "com.android.support:gridlayout-v7:${android["androidSupportSdkVersion"]}",
	
	            /*************************第三方依赖*************************/
	            // https://github.com/square/retrofit
	            "retrofit2"        : "com.squareup.retrofit2:retrofit:2.2.0",
	            "converter-scalars": "com.squareup.retrofit2:converter-scalars:2.2.0",
	            "converter-gson"   : "com.squareup.retrofit2:converter-gson:2.2.0",
	            "adapter-rxjava"   : "com.squareup.retrofit2:adapter-rxjava:2.2.0",
	            // https://github.com/square/okhttp
	            "okhttp"           : "com.squareup.okhttp3:okhttp:3.10.0",
	            // https://github.com/greenrobot/greenDAO
	            "greendao"         : "org.greenrobot:greendao:3.2.2",
	            // https://github.com/bumptech/glide
	            "glide"            : "com.github.bumptech.glide:glide:4.7.1",
	            // https://github.com/square/picasso
	            "picasso"          : "com.squareup.picasso:picasso:2.71828",
	            // https://github.com/facebook/fresco
	            "fresco"           : "com.facebook.fresco:fresco:1.9.0",
	            // https://github.com/greenrobot/EventBus
	            "eventbus"         : "org.greenrobot:eventbus:3.1.1",
	            // https://bugly.qq.com/docs/user-guide/instruction-manual-android/?v=20170912151050
	            "bugly"            : "com.tencent.bugly:crashreport:latest.release",
	            // https://bintray.com/android/android-utils/com.android.volley.volley
	            "volley"           : "com.android.volley:volley:1.1.0",
	            // https://github.com/ReactiveX/RxJava
	            "rxjava"           : "io.reactivex:rxjava:1.3.0",
	            "rxjava2"          : "io.reactivex.rxjava2:rxjava:x.y.z",
	            "rxandroid"        : "io.reactivex:rxandroid:1.2.1",
	
	            "gson"             : "com.google.code.gson:gson:2.2.4",
	            "commons-lang3"    : "org.apache.commons:commons-lang3:3.3.2",
	
	            /*************************个人依赖*************************/
	            // https://github.com/VeiZhang/BaseToolsLibrary
	            "basetools"        : "com.excellence:basetools:1.2.5",
	            // https://github.com/VeiZhang/Permission
	            "permission"       : "com.excellence:permission:1.0.1",
	            // https://github.com/VeiZhang/RetrofitClient
	            "retrofit-client"  : "com.excellence:retrofit:1.0.1",
	            // https://github.com/VeiZhang/QSkinLoader
	            "skinloader"       : "com.excellence:skinloader:1.2.2",
	            // https://github.com/VeiZhang/ToastKit
	            "toast"            : "com.excellence:toast:1.0.0",
	            // https://github.com/VeiZhang/MailSender
	            "mailsender"       : "com.excellence:mailsender:1.0.0",
	            // https://github.com/VeiZhang/Downloader
	            "downloader"       : "com.excellence:downloader:1.1.0"
	    ]
	}
	
	def getVersionName() {
	    String prefix = "1.0." + getSvnVersionCode()
	    return prefix + ' [' + getDate() + ']'
	}
	
	// 根据svn提交版本生成版本号
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
	    return 1
	}
	
	def static getDate() {
	    String date = new SimpleDateFormat("MMddyyyy").format(new Date())
	    return date
	}
	```

	![config.gradle][config.gradle]

* 在项目根目录的build.gradle中引用，供其他的Module使用

	**注意：** 
	1. **如果想使用ext的值，则只能在项目根目录的build.gradle中引用**
	2. 想让单独的Module使用，则在该Module的build.gradle里引入，但是此时不能使用ext的值，否则会提示无法找到"Error:Cannot get property 'xxx' on extra properties extension as it does not exist"

	```
	apply from: "config.gradle"
	```

	![config引用][config引用]


* 在Module目录的build.gradle中使用变量
	
	![config使用][config使用]


## 远程配置

远程配置配置其他步骤与[本地配置](#本地配置)是一样，不同的是引用的方式，导入的不是路径里的文件，而是一个文件链接

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
[远程配置引用]:https://github.com/VeiZhang/build.gradle/blob/master/images/%E8%BF%9C%E7%A8%8B%E9%85%8D%E7%BD%AE%E5%BC%95%E7%94%A8.png?raw=true "远程配置引用"
[继承方式]:https://github.com/VeiZhang/build.gradle/blob/master/images/%E7%BB%A7%E6%89%BF%E6%96%B9%E5%BC%8F.png?raw=true "继承方式"
