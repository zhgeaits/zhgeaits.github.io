---
layout: post
title:  "项目构建神器之Maven学习笔记，常用使用记录&Nexus仓库管理"
date:   2011-11-20 07:30:03
categories: other
type: other
---

maven这个高大上的东西，在没有上项目管理的课程时候是不知道什么是项目构建的，虽然就是打包，但是包含很多知识和技术，我们工作室很早就开始使用maven，从大二开始使用，因为飞哥去淘宝实习，从那里学习得到回来教我们的。

**环境搭建**  
这个简单到和搭建JDK环境一样。  
第一步：去官网下载一个版本，然后解压到本地随便一个目录。  
第二步：在环境变量设置一个M2_HOME，指向解压的目录，再去PATH路径加入%M2_HOME%\bin。这时候可以再控制台输入mvn -version了。  
第三步：随便在某一个地方设置一个仓库目录，例如在M2_HOME\resposity, 然后在conf\settings.xml里面设置<localRepository>D:/maven3.0.5/repository</localRepository>。
当然，根据自己所在团队要详细设置settings了。例如，xhome要设置自己的镜像，YY要设置自己的仓库等等。

命令大全：  
1.帮助信息查询：mvn help:describe -Dplugin=help -Dfull  
2.查看全局的pom：mvn help:effective-pom  
3.打包项目：mvn package  
4.打包并安装项目：mvn install 要打开调试标记运行就：mvn install -X  
5.清理项目：mvn clean  
6.创建项目：mvn archetype:create -DgroupId=org.sonatype.mavenbook.ch03 -DartifactId=simple -DpackageName=org.sonatype.mavenbook
或者：mvn archetype:create -DgroupId=org.sonatype.mavenbook.ch05 -DartifactId=simple-webapp -DpackageName=org.sonatype.mavenbook -DarchetypeArtifactId=maven-archetype-webapp  
7.生存站点和报告命令：mvn site  
8.运行java项目实例：mvn exec:java -Dexec.mainClass=org.sonatype.mavenbook.weather.Main  
9.打印项目依赖列表：mvn dependency:resolve 或者以树状形式显示：mvn dependency:tree  
10.只是运行测试：mvn test 使用-e 或者-X显示调试过程。测试时可以设置忽略错误：mvn test -Dmaven.test.failure.ignore=true  
11.安装时跳过测试：mvn install -Dmaven.test.skip=true  
12.创建一个打包好的命令行应用程序:配置maven-assembly-plugin，然后运行：mvn assembly:assembly  
13.分析依赖：mvn dependency:analyze  
14.安装是跳过测试：mvn install -Dmaven.test.skip=true  
15.打源码包：mvn clean javadoc:jar source:jar install -Dmaven.test.skip=true  
发布包：  
mvn install:install-file -Dfile=外部包的路径 \  
	-DgroupId=外部包的groupID \  
	-DartifactId=外部包的artifactId \  
	-Dversion=外部包的版本号 \  
	-Dpackage=jar  
	
	
以前在工作室是用maven构建web项目，去阿里那里也是，而且那里的maven是阿里自己修改过的。。。现在来到YY，也用maven构建android项目。

对于android，maven构建jar时是不支持aidl的，so库这些的,而apklib支持，apklib只是对源码和资源文件的打包，所以apklib是maven自己搞的东西，但是已经不支持和流行了，官方文档都没了，现在已经被gradle的aar锁代替了。

## Troubleshooting

1.构建android的时候，突然出现这样一个错误，说是debug的证书过期了，google了以后发现说证书过期就去删掉debug证书就可以了，debug证书在c盘用户目录的.android目录下。

>[ERROR] Failed to execute goal com.simpligility.maven.plugins:android-maven-plugin:4.3.0:apk (default-apk) on project xxx: Debug Certifica
te expired on xxx 下午4:23 -> [Help 1]

然而我删掉了以后发现还是不行，那就说明不是那个证书，也试过重开，重启，清缓存，都解决不了。于是加了-X选项重新运行mvn命令，发现，错误的地方在：

>at com.simpligility.maven.plugins.android.phase09package.ApkMojo.doAPKWithAPKBuilder(ApkMojo.java:710)

再去github上找到simpligility/android-maven-plugin，找到了ApkMojo.doAPKWithAPKBuilder方法，发现这样获取debug证书：

>final String debugKeyStore = signWithDebugKeyStore ? ApkBuilder.getDebugKeystore() : null;

ApkBuilder.getDebugKeystore()是android的工具，于是在google，发现源码如下：

地址：

https://android.googlesource.com/platform/sdk/+/e162064a7b5db1eecec34271bc7e2a4296181ea6/sdkmanager/libs/sdklib/src/com/android/sdklib/build/ApkBuilder.java

{% highlight java %}

public static String getDebugKeystore() throws ApkCreationException {
    try {
        return DebugKeyProvider.getDefaultKeyStoreOsPath();
    } catch (Exception e) {
        throw new ApkCreationException(e, e.getMessage());
    }
}
{% endhighlight %}

继续找到DebugKeyProvider.getDefaultKeyStoreOsPath：

{% highlight java %}

public static String getDefaultKeyStoreOsPath()
            throws KeytoolException, AndroidLocationException {
    String folder = AndroidLocation.getFolder();
    if (folder == null) {
        throw new KeytoolException("Failed to get HOME directory!\n");
    }
    String osKeyStorePath = folder + "debug.keystore";
    return osKeyStorePath;
}

{% endhighlight %}

原来都在AndroidLocation.getFolder()这里：

{% highlight java %}

String home = findValidPath(new EnvVar[] { EnvVar.ANDROID_SDK_HOME,
                                                       EnvVar.USER_HOME,
                                                       EnvVar.HOME });

{% endhighlight %}

其实分别从三个地方获取，sdk目录，用户主目录和环境变量的目录，先去看sdk目录，果然发现了，然后问题就解决了。

慢慢再记录。