# [build.gradle](https://github.com/VeiZhang/build.gradle)
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
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    testImplementation 'junit:junit:4.12'
    implementation 'com.android.support:appcompat-v7:24.2.1'
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
	            application       : "com.android.application",
	            library           : "com.android.library",
	            maven             : "com.github.dcendents.android-maven",
	            bintray           : "com.jfrog.bintray",
	            novoda            : "com.novoda.bintray-release",
	            greendao          : "org.greenrobot.greendao",
	            "greendao-gradle" : "org.greenrobot:greendao-gradle-plugin:3.2.2"
	    ]
	
	    // 配置
	    android = [
	            /*************************原生配置*************************/
	            compileSdkVersion       : 25,
	            buildToolsVersion       : "25.0.0",
	            minSdkVersion           : 17,
	            targetSdkVersion        : 23,
	            versionCode             : getVersionCode(),
	            versionName             : getVersionName(),
	
	            /*************************自定义配置*************************/
	            androidSupportSdkVersion: "23.0.0"
	    ]
	
	    // 依赖
	    dependencies = [
	            /*************************原生依赖*************************/
	            "appcompat-v7"      : "com.android.support:appcompat-v7:${android["androidSupportSdkVersion"]}",
	            "support-v4"        : "com.android.support:support-v4:${android["androidSupportSdkVersion"]}",
	            "cardview-v7"       : "com.android.support:cardview-v7:${android["androidSupportSdkVersion"]}",
	            "recyclerview-v7"   : "com.android.support:recyclerview-v7:${android["androidSupportSdkVersion"]}",
	            "design"            : "com.android.support:design:${android["androidSupportSdkVersion"]}",
	            "annotations"       : "com.android.support:support-annotations:${android["androidSupportSdkVersion"]}",
	            "gridlayout-v7"     : "com.android.support:gridlayout-v7:${android["androidSupportSdkVersion"]}",
	            "constraint-layout" : "com.android.support.constraint:constraint-layout:1.0.2",
	
	            /*************************第三方依赖*************************/
	            // https://github.com/square/retrofit
	            "retrofit2"           : "com.squareup.retrofit2:retrofit:2.4.0",
	            "converter-scalars"   : "com.squareup.retrofit2:converter-scalars:2.4.0",
	            "converter-gson"      : "com.squareup.retrofit2:converter-gson:2.4.0",
	            "adapter-rxjava"      : "com.squareup.retrofit2:adapter-rxjava:2.4.0",
	            "adapter-rxjava2"     : "com.squareup.retrofit2:adapter-rxjava2:2.4.0",
	            // https://github.com/square/okhttp
	            "okhttp"              : "com.squareup.okhttp3:okhttp:3.11.0",
	            // https://github.com/greenrobot/greenDAO
	            "greendao"            : "org.greenrobot:greendao:3.2.2",
	            // https://github.com/yuweiguocn/GreenDaoUpgradeHelper
	            "greendao-helper"     : "com.github.yuweiguocn:GreenDaoUpgradeHelper:v2.1.0",
	            // https://github.com/bumptech/glide
	            "glide"               : "com.github.bumptech.glide:glide:4.8.0",
	            // https://github.com/square/picasso
	            "picasso"             : "com.squareup.picasso:picasso:2.71828",
	            // https://github.com/facebook/fresco
	            "fresco"              : "com.facebook.fresco:fresco:1.10.0",
	            // https://github.com/greenrobot/EventBus
	            "eventbus"            : "org.greenrobot:eventbus:3.1.1",
	            // https://github.com/BuglyDevTeam/Bugly-Android
	            "bugly"               : "com.tencent.bugly:crashreport:2.6.6.1",
	            "bugly-native"        : "com.tencent.bugly:nativecrashreport:3.3.1",
	            // https://bintray.com/android/android-utils/com.android.volley.volley
	            "volley"              : "com.android.volley:volley:1.1.1",
	            // https://github.com/ReactiveX/RxJava
	            "rxjava"              : "io.reactivex:rxjava:1.3.8",
	            "rxjava2"             : "io.reactivex.rxjava2:rxjava:2.2.2",
	            "rxandroid"           : "io.reactivex:rxandroid:2.1.0",
	            "rxandroid2"          : 'io.reactivex.rxjava2:rxandroid:2.0.2',
	            // https://github.com/JakeWharton/RxBinding
	            "rxbinding"           : 'com.jakewharton.rxbinding2:rxbinding:2.2.0',
	            // https://github.com/google/gson
	            "gson"                : "com.google.code.gson:gson:2.8.5",
	            // https://github.com/alibaba/fastjson
	            "fastjson"            : "com.alibaba:fastjson:1.1.70.android",
	            // https://github.com/apache/commons-lang
	            "commons-lang3"       : "org.apache.commons:commons-lang3:3.8",
	            // https://github.com/square/leakcanary
	            "leakcanary"          : "com.squareup.leakcanary:leakcanary-android:1.6.2",
	            "leakcanary-release"  : "com.squareup.leakcanary:leakcanary-android-no-op:1.6.2",
	            "leakcanary-fragment" : "com.squareup.leakcanary:leakcanary-support-fragment:1.6.2",
	            // https://github.com/YoKeyword/Fragmentation
	            "fragmentation"       : "me.yokeyword:fragmentation:1.3.6",
	
	            /*************************个人依赖*************************/
	            // https://github.com/VeiZhang/BaseToolsLibrary
	            "basetools"           : "com.excellence:basetools:1.2.6",
	            // https://github.com/VeiZhang/Permission
	            "permission"          : "com.excellence:permission:1.0.1",
	            // https://github.com/VeiZhang/RetrofitClient
	            "retrofit-client"     : "com.excellence:retrofit:1.0.5",
	            "retrofit-client2"    : "com.excellence.retrofit:retrofit2:2.0.0",
	            // https://github.com/VeiZhang/QSkinLoader
	            "skinloader"          : "com.excellence:skinloader:1.2.2",
	            // https://github.com/VeiZhang/ToastKit
	            "toast"               : "com.excellence:toast:1.1.0",
	            // https://github.com/VeiZhang/MailSender
	            "mailsender"          : "com.excellence:mailsender:1.0.0",
	            // https://github.com/VeiZhang/Downloader
	            "downloader"          : "com.excellence:downloader:1.2.0",
	            // https://github.com/VeiZhang/AppStatistics
	            "app-statistics"      : "com.excellence:app-statistics:1.0.1",
	            // https://github.com/VeiZhang/AndroidExec
	            "exec"                : "com.excellence:exec:1.1.0",
	            // https://github.com/VeiZhang/AndroidFFmpeg
	            "ffmpeg"              : "com.excellence:ffmpeg:1.1.0",
	            // https://github.com/VeiZhang/ImageLoader
	            "imageloader"         : "com.excellence:imageloader:1.0.0",
	            "imageloader-fresco"  : "com.excellence:imageloader-fresco:1.0.0",
	            "imageloader-picasso" : "com.excellence:imageloader-picasso:1.0.0",
	            "imageloader-glide"   : "com.excellence:imageloader-glide:1.0.0"
	    ]
	}
	
	/***********************APP版本控制的通用方法***********************/
	/**
	 * svn
	 * 直接读取svn版本号作为版本控制
	 */
	/**
	 * git
	 *
	 * git tag作为版本名称
	 * git 版本号有两种方法
	 *  ①一种是把提交次数作为versionCode
	 *  ②另外一种是把tag的数量作为versionCode
	 */
	
	/**
	 * 获取版本
	 * @return
	 */
	def getVersionName() {
	    def date = getDate()
	    def version = getSvnVersionCode()
	    if (version != 0) {
	        return "1.0.${version} [${date}]"
	    }
	    version = getGitTag()
	    if (version != 0) {
	        return "${version} [${date}]"
	    }
	    if (version == 0) {
	        version = 1
	    }
	    /**
	     * 错误的版本信息，请检查
	     */
	    return "0.${version} [${date}]"
	}
	
	/**
	 * 获取版本号
	 * @return
	 */
	def getVersionCode() {
	    def versionCode = getSvnVersionCode()
	    if (versionCode == 0) {
	        versionCode = getGitVersionCode()
	    }
	    if (versionCode == 0) {
	        versionCode = 1
	    }
	    return versionCode
	}
	
	/***********************读取Git信息***********************/
	
	/**
	 * 读取最近的一次git tag
	 * @return tag
	 */
	def getGitTag() {
	    try {
	        def process = ("git describe --abbrev=0 --tags").execute()
	        def tag = process.text.trim()
	        return tag
	    } catch (e) {
	        println e.getMessage()
	    }
	    return 0
	}
	
	/**
	 * 以git tag的数量作为其版本号
	 * @return tag的数量
	 */
	def getGitVersionCode() {
	    try {
	        def process = ("git rev-list HEAD --first-parent --count").execute()
	        def version = process.text.trim().toInteger()
	        return version
	    } catch (e) {
	        println e.getMessage()
	    }
	    return 0
	}
	
	/***********************读取SVN信息***********************/
	
	/**
	 * 根据svn提交版本生成版本号
	 * @return
	 */
	def getSvnVersionCode() {
	    try {
	        def process = ("svnversion -c " + getBuildDir().parent).execute()
	        process.waitFor()
	        def version = process.in.text
	        Pattern pattern = Pattern.compile("(\\d+:)?(\\d+)\\D")
	        Matcher matcher = pattern.matcher(version)
	        if (matcher.find()) {
	            version = matcher.group(matcher.groupCount())
	        }
	        return Integer.parseInt(version)
	    } catch (e) {
	        println e.getMessage()
	    }
	    return 0
	}
	
	/**
	 * 获取日期
	 * @return
	 */
	def getDate() {
	    String date = new SimpleDateFormat("MMddyyyy").format(new Date())
	    return date
	}
	```

	![config.gradle][config.gradle示例]

* 在项目根目录的build.gradle中引用，供其他的Module使用

	**注意：**
	1. **如果想使用ext的值，则只能在项目根目录的build.gradle中引用**
	2. 想让单独的Module使用，则在该Module的build.gradle里引入，但是此时不能使用ext的值，否则会提示无法找到 **"Error:Cannot get property 'xxx' on extra properties extension as it does not exist"**

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


## 示例

* https://github.com/VeiZhang/BaseToolsLibrary 中 [config.gradle][config.gradle]、[application.gradle][application.gradle]


## 第三方依赖版本

| 依赖 | 版本 |
| --- | ---- |
| [retrofit2][retrofit2] | [![Download][retrofit2_download]][retrofit2_latestVersion] |
| [converter-scalars][converter-scalars] | [![Download][converter-scalars_download]][converter-scalars_latestVersion] |
| [converter-gson][converter-gson] | [![Download][converter-gson_download]][converter-gson_latestVersion] |
| [adapter-rxjava][adapter-rxjava] | [![Download][adapter-rxjava_download]][adapter-rxjava_latestVersion] |
| [adapter-rxjava2][adapter-rxjava2] | [![Download][adapter-rxjava2_download]][adapter-rxjava2_latestVersion] |
| [okhttp][okhttp] | [![Download][okhttp_download]][okhttp_latestVersion] |
| [logging-interceptor][logging-interceptor] | [![Download][logging-interceptor_download]][logging-interceptor_latestVersion] |
| [greendao][greendao] | [![Download][greendao_download]][greendao_latestVersion] |
| [greendao-helper][greendao-helper] | [![Download][greendao-helper_download]][greendao-helper_latestVersion] |
| [glide][glide] | [![Download][glide_download]][glide_latestVersion] |
| [picasso][picasso] | [![Download][picasso_download]][picasso_latestVersion] |
| [fresco][fresco] | [![Download][fresco_download]][fresco_latestVersion] |
| [eventbus][eventbus] | [![Download][eventbus_download]][eventbus_latestVersion] |
| [bugly][bugly] | [![Download][bugly_download]][bugly_latestVersion] |
| [bugly-native][bugly-native] | [![Download][bugly-native_download]][bugly-native_latestVersion] |
| [volley][volley] | [![Download][volley_download]][volley_latestVersion] |
| [rxjava][rxjava] | [![Download][rxjava_download]][rxjava_latestVersion] |
| [rxjava2][rxjava2] | [![Download][rxjava2_download]][rxjava2_latestVersion] |
| [rxandroid][rxandroid] | [![Download][rxandroid_download]][rxandroid_latestVersion] |
| [rxbinding][rxbinding] | [![Download][rxbinding_download]][rxbinding_latestVersion] |
| [gson][gson] | [![Download][gson_download]][gson_latestVersion] |
| [fastjson][fastjson] | [![Download][fastjson_download]][fastjson_latestVersion] |
| [commons-lang3][commons-lang3] | [![Download][commons-lang3_download]][commons-lang3_latestVersion] |
| [leakcanary][leakcanary] | [![Download][leakcanary_download]][leakcanary_latestVersion] |
| [leakcanary-release][leakcanary-release] | [![Download][leakcanary-release_download]][leakcanary-release_latestVersion] |
| [leakcanary-fragment][leakcanary-fragment] | [![Download][leakcanary-fragment_download]][leakcanary-fragment_latestVersion] |
| [basetools][basetools] | [![Download][basetools_download]][basetools_latestVersion] |
| [permission][permission] | [![Download][permission_download]][permission_latestVersion] |
| [retrofit-client][retrofit-client] | [![Download][retrofit-client_download]][retrofit-client_latestVersion] |
| [retrofit-client2][retrofit-client2] | [![Download][retrofit-client2_download]][retrofit-client2_latestVersion] |
| [skinloader][skinloader] | [![Download][skinloader_download]][skinloader_latestVersion] |
| [toast][toast] | [![Download][toast_download]][toast_latestVersion] |
| [mailsender][mailsender] | [![Download][mailsender_download]][mailsender_latestVersion] |
| [downloader][downloader] | [![Download][downloader_download]][downloader_latestVersion] |
| [app-statistics][app-statistics] | [![Download][app-statistics_download]][app-statistics_latestVersion] |
| [exec][exec] | [![Download][exec_download]][exec_latestVersion] |
| [ffmpeg][ffmpeg] | [![Download][ffmpeg_download]][ffmpeg_latestVersion] |
| [imageloader][imageloader] | [![Download][imageloader_download]][imageloader_latestVersion] |

[config.gradle示例]:https://github.com/VeiZhang/build.gradle/blob/master/images/config.gradle.png?raw=true "config.gradle"
[config引用]:https://github.com/VeiZhang/build.gradle/blob/master/images/config引用.png?raw=true "config引用"
[config使用]:https://github.com/VeiZhang/build.gradle/blob/master/images/config使用.png?raw=true "config使用"
[远程配置引用]:https://github.com/VeiZhang/build.gradle/blob/master/images/%E8%BF%9C%E7%A8%8B%E9%85%8D%E7%BD%AE%E5%BC%95%E7%94%A8.png?raw=true "远程配置引用"
[继承方式]:https://github.com/VeiZhang/build.gradle/blob/master/images/%E7%BB%A7%E6%89%BF%E6%96%B9%E5%BC%8F.png?raw=true "继承方式"
[config.gradle]:https://github.com/VeiZhang/BaseToolsLibrary/blob/master/config.gradle
[application.gradle]:https://github.com/VeiZhang/BaseToolsLibrary/blob/master/application.gradle

[retrofit2]:https://github.com/square/retrofit
[retrofit2_download]:https://api.bintray.com/packages/bintray/jcenter/com.squareup.retrofit2%3Aretrofit/images/download.svg
[retrofit2_latestVersion]:https://bintray.com/bintray/jcenter/com.squareup.retrofit2%3Aretrofit/_latestVersion
[converter-scalars]:https://github.com/square/retrofit/tree/master/retrofit-converters/scalars
[converter-scalars_download]:https://api.bintray.com/packages/bintray/jcenter/com.squareup.retrofit2%3Aconverter-scalars/images/download.svg
[converter-scalars_latestVersion]:https://bintray.com/bintray/jcenter/com.squareup.retrofit2%3Aconverter-scalars/_latestVersion
[converter-gson]:https://github.com/square/retrofit/tree/master/retrofit-converters/gson
[converter-gson_download]:https://api.bintray.com/packages/bintray/jcenter/com.squareup.retrofit2%3Aconverter-gson/images/download.svg
[converter-gson_latestVersion]:https://bintray.com/bintray/jcenter/com.squareup.retrofit2%3Aconverter-gson/_latestVersion
[adapter-rxjava]:https://github.com/square/retrofit/tree/master/retrofit-adapters/rxjava
[adapter-rxjava_download]:https://api.bintray.com/packages/bintray/jcenter/com.squareup.retrofit2%3Aadapter-rxjava/images/download.svg
[adapter-rxjava_latestVersion]:https://bintray.com/bintray/jcenter/com.squareup.retrofit2%3Aadapter-rxjava/_latestVersion
[adapter-rxjava2]:https://github.com/square/retrofit/tree/master/retrofit-adapters/rxjava2
[adapter-rxjava2_download]:https://api.bintray.com/packages/bintray/jcenter/com.squareup.retrofit2%3Aadapter-rxjava2/images/download.svg
[adapter-rxjava2_latestVersion]:https://bintray.com/bintray/jcenter/com.squareup.retrofit2%3Aadapter-rxjava2/_latestVersion
[okhttp]:https://github.com/square/okhttp
[okhttp_download]:https://api.bintray.com/packages/bintray/jcenter/com.squareup.okhttp3%3Aokhttp/images/download.svg
[okhttp_latestVersion]:https://bintray.com/bintray/jcenter/com.squareup.okhttp3%3Aokhttp/_latestVersion
[logging-interceptor]:https://github.com/square/okhttp/tree/master/okhttp-logging-interceptor
[logging-interceptor_download]:https://api.bintray.com/packages/bintray/jcenter/com.squareup.okhttp3%3Alogging-interceptor/images/download.svg
[logging-interceptor_latestVersion]:https://bintray.com/bintray/jcenter/com.squareup.okhttp3%3Alogging-interceptor/_latestVersion
[greendao]:https://github.com/greenrobot/greenDAO
[greendao_download]:https://api.bintray.com/packages/bintray/jcenter/org.greenrobot%3Agreendao-gradle-plugin/images/download.svg
[greendao_latestVersion]:https://bintray.com/bintray/jcenter/org.greenrobot%3Agreendao-gradle-plugin/_latestVersion
[greendao-helper]:https://github.com/yuweiguocn/GreenDaoUpgradeHelper
[greendao-helper_download]:https://api.bintray.com/packages/yuweiguocn/maven/GreenDaoUpgradeHelper/images/download.svg
[greendao-helper_latestVersion]:https://bintray.com/yuweiguocn/maven/GreenDaoUpgradeHelper/_latestVersion
[glide]:https://github.com/bumptech/glide
[glide_download]:https://api.bintray.com/packages/bintray/jcenter/com.github.bumptech.glide%3Aglide/images/download.svg
[glide_latestVersion]:https://bintray.com/bintray/jcenter/com.github.bumptech.glide%3Aglide/_latestVersion
[picasso]:https://github.com/square/picasso
[picasso_download]:https://api.bintray.com/packages/bintray/jcenter/com.squareup.picasso%3Apicasso/images/download.svg
[picasso_latestVersion]:https://bintray.com/bintray/jcenter/com.squareup.picasso%3Apicasso/_latestVersion
[fresco]:https://github.com/facebook/fresco
[fresco_download]:https://api.bintray.com/packages/facebook/maven/com.facebook.fresco%3Afresco/images/download.svg
[fresco_latestVersion]:https://bintray.com/facebook/maven/com.facebook.fresco%3Afresco/_latestVersion
[eventbus]:https://github.com/greenrobot/EventBus
[eventbus_download]:https://api.bintray.com/packages/bintray/jcenter/org.greenrobot%3Aeventbus-annotation-processor/images/download.svg
[eventbus_latestVersion]:https://bintray.com/bintray/jcenter/org.greenrobot%3Aeventbus-annotation-processor/_latestVersion
[bugly]:https://github.com/BuglyDevTeam/Bugly-Android
[bugly_download]:https://api.bintray.com/packages/bugly/maven/CrashReport/images/download.svg
[bugly_latestVersion]:https://bintray.com/bugly/maven/CrashReport/_latestVersion
[bugly-native]:https://github.com/BuglyDevTeam/Bugly-Android-NDK
[bugly-native_download]:https://api.bintray.com/packages/bugly/maven/NativeCrashReport/images/download.svg
[bugly-native_latestVersion]:https://bintray.com/bugly/maven/NativeCrashReport/_latestVersion
[volley]:https://github.com/google/volley
[volley_download]:https://api.bintray.com/packages/android/android-utils/com.android.volley.volley/images/download.svg
[volley_latestVersion]:https://bintray.com/android/android-utils/com.android.volley.volley/_latestVersion
[rxjava]:https://github.com/ReactiveX/RxJava/tree/1.x
[rxjava_download]:http://img.shields.io/maven-central/v/io.reactivex/rxjava.svg
[rxjava_latestVersion]:https://search.maven.org/artifact/io.reactivex/rxjava
[rxjava2]:https://github.com/ReactiveX/RxJava
[rxjava2_download]:https://api.bintray.com/packages/reactivex/RxJava/RxJava/images/download.svg
[rxjava2_latestVersion]:https://bintray.com/reactivex/RxJava/RxJava/_latestVersion
[rxandroid]:https://github.com/ReactiveX/RxAndroid
[rxandroid_download]:https://api.bintray.com/packages/reactivex/RxJava/RxAndroid/images/download.svg
[rxandroid_latestVersion]:https://bintray.com/reactivex/RxJava/RxAndroid/_latestVersion
[rxbinding]:https://github.com/JakeWharton/RxBinding
[rxbinding_download]:https://api.bintray.com/packages/bintray/jcenter/com.jakewharton.rxbinding2%3Arxbinding/images/download.svg
[rxbinding_latestVersion]:https://bintray.com/bintray/jcenter/com.jakewharton.rxbinding2%3Arxbinding/_latestVersion
[gson]:https://github.com/google/gson
[gson_download]:https://api.bintray.com/packages/bintray/jcenter/com.google.code.gson%3Agson/images/download.svg
[gson_latestVersion]:https://bintray.com/bintray/jcenter/com.google.code.gson%3Agson/_latestVersion
[fastjson]:https://github.com/alibaba/fastjson "Android版本"
[fastjson_download]:https://api.bintray.com/packages/bintray/jcenter/com.alibaba%3Afastjson/images/download.svg?version=1.1.70.android
[fastjson_latestVersion]:https://bintray.com/bintray/jcenter/com.alibaba%3Afastjson/1.1.70.android/link "Android版本"
[commons-lang3]:https://github.com/apache/commons-lang
[commons-lang3_download]:https://api.bintray.com/packages/bintray/jcenter/org.apache.commons%3Acommons-lang3/images/download.svg
[commons-lang3_latestVersion]:https://bintray.com/bintray/jcenter/org.apache.commons%3Acommons-lang3/_latestVersion
[leakcanary]:https://github.com/square/leakcanary
[leakcanary_download]:https://api.bintray.com/packages/pyricau/maven/com.squareup.leakcanary%3Aleakcanary-android/images/download.svg
[leakcanary_latestVersion]:https://bintray.com/pyricau/maven/com.squareup.leakcanary%3Aleakcanary-android/_latestVersion
[leakcanary-release]:https://github.com/square/leakcanary
[leakcanary-release_download]:https://api.bintray.com/packages/pyricau/maven/com.squareup.leakcanary%3Aleakcanary-android-no-op/images/download.svg
[leakcanary-release_latestVersion]:https://bintray.com/pyricau/maven/com.squareup.leakcanary%3Aleakcanary-android-no-op/_latestVersion
[leakcanary-fragment]:https://github.com/square/leakcanary
[leakcanary-fragment_download]:https://api.bintray.com/packages/bintray/jcenter/com.squareup.leakcanary%3Aleakcanary-support-fragment/images/download.svg
[leakcanary-fragment_latestVersion]:https://bintray.com/bintray/jcenter/com.squareup.leakcanary%3Aleakcanary-support-fragment/_latestVersion
[basetools]:https://github.com/VeiZhang/BaseToolsLibrary
[basetools_download]:https://api.bintray.com/packages/veizhang/maven/basetools/images/download.svg
[basetools_latestVersion]:https://bintray.com/veizhang/maven/basetools/_latestVersion
[permission]:https://github.com/VeiZhang/Permission
[permission_download]:https://api.bintray.com/packages/veizhang/maven/permission/images/download.svg
[permission_latestVersion]:https://bintray.com/veizhang/maven/permission/_latestVersion
[retrofit-client]:https://github.com/VeiZhang/RetrofitClient
[retrofit-client_download]:https://api.bintray.com/packages/veizhang/maven/retrofit/images/download.svg
[retrofit-client_latestVersion]:https://bintray.com/veizhang/maven/retrofit/_latestVersion
[retrofit-client2]:https://github.com/VeiZhang/RetrofitClient
[retrofit-client2_download]:https://api.bintray.com/packages/veizhang/maven/retrofit2/images/download.svg
[retrofit-client2_latestVersion]:https://bintray.com/veizhang/maven/retrofit2/_latestVersion
[skinloader]:https://github.com/VeiZhang/QSkinLoader
[skinloader_download]:https://api.bintray.com/packages/veizhang/maven/skinloader/images/download.svg
[skinloader_latestVersion]:https://bintray.com/veizhang/maven/skinloader/_latestVersion
[toast]:https://github.com/VeiZhang/ToastKit
[toast_download]:https://api.bintray.com/packages/veizhang/maven/toast/images/download.svg
[toast_latestVersion]:https://bintray.com/veizhang/maven/toast/_latestVersion
[mailsender]:https://github.com/VeiZhang/MailSender
[mailsender_download]:https://api.bintray.com/packages/veizhang/maven/mailsender/images/download.svg
[mailsender_latestVersion]:https://bintray.com/veizhang/maven/mailsender/_latestVersion
[downloader]:https://github.com/VeiZhang/Downloader
[downloader_download]:https://api.bintray.com/packages/veizhang/maven/downloader/images/download.svg
[downloader_latestVersion]:https://bintray.com/veizhang/maven/downloader/_latestVersion
[app-statistics]:https://github.com/VeiZhang/AppStatistics
[app-statistics_download]:https://api.bintray.com/packages/veizhang/maven/app-statistics/images/download.svg
[app-statistics_latestVersion]:https://bintray.com/veizhang/maven/app-statistics/_latestVersion
[exec]:https://github.com/VeiZhang/AndroidExec
[exec_download]:https://api.bintray.com/packages/veizhang/maven/exec/images/download.svg
[exec_latestVersion]:https://bintray.com/veizhang/maven/exec/_latestVersion
[ffmpeg]:https://github.com/VeiZhang/AndroidFFmpeg
[ffmpeg_download]:https://api.bintray.com/packages/veizhang/maven/ffmpeg/images/download.svg
[ffmpeg_latestVersion]:https://bintray.com/veizhang/maven/ffmpeg/_latestVersion
[imageloader]:https://github.com/VeiZhang/ImageLoader
[imageloader_download]:https://api.bintray.com/packages/veizhang/maven/imageloader/images/download.svg
[imageloader_latestVersion]:https://bintray.com/veizhang/maven/imageloader/_latestVersion
