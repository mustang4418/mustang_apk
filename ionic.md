
# Android打包教程(Ionic)
Android 要求所有的程序必须有签名，否则就无法安装该程序。
通过对我们发布的apk文件进行唯一签名，保证我们应用的一致性，才能正确的对应用进行版本更新。同时也防止其他开发商通过相同的Package Name来混淆替换我们已经安装的应用。

### 准备工作
#### 签名的步骤：

1. 使用 keytool 工具创建key，它是一个有效的安全钥匙和证书的管理工具。
2. 使用 jarsigner 工具根据上一步生成的key对apk进行签名，它是JDK中包含的用于JAR文件签名和验证的工具。
#### 将工具添加到环境变量

为了方便我们后面进行签名操作，我们把这两个工具所在的目录添加到系统的环境变量中，避免了后续操作来回切换目录。
~~~
keytool 和 jarsigner 工具位于JDK目录，我电脑上的目录为 D:\Java\jdk1.8.0_91\bin 。
~~~
注意：替换为你自己的JDK目录

## 打包apk

ionic 功能开发完成后，就需要将应用打包发布到应用市场。
之前我们调试应用的时候，直接使用ionic的 **build** 命令进行生成

~~~
$ ionic build android
~~~
或者使用 **run** 命令直接运行APP进行调试：

~~~
$ ionic run android
~~~
这两种方法都会生成 **debug** 版本的apk应用包 **android-debug.apk** ，我们可以在项目根目录下的 platforms\android\build\outputs\apk 目录下找到它。

我们现在发布app，就需要生成对应的release应用包，使用以下命令：

~~~
$ ionic build --release android
~~~
注意：加上–release 参数
同样的，在 **platforms\android\build\outputs\apk** 目录下回生成未签名的release应用包 **android-release-unsigned.apk** 。

接下来我们就需要对这个 **android-release-unsigned.apk** 包进行签名了。

### 查看apk签名

首先通过命令行工具，导航到生成的apk目录下：

~~~
$ cd platforms\android\build\outputs\apk
~~~
使用以下命令查看apk的签名：

~~~
$ keytool -list -printcert -jarfile "android-release-unsigned.apk"
~~~
提示：不是已签名的 jar 文件
因为我们还没有对它进行签名。
如果我们查看调试时生成的 android-debug.apk 的签名的话：

~~~
$ keytool -list -printcert -jarfile "android-debug.apk"
~~~
提示如下：

~~~
签名者 #1:
签名:
所有者: C=US, O=Android, CN=Android Debug
发布者: C=US, O=Android, CN=Android Debug
序列号: 1
有效期开始日期: Wed Jun 01 15:00:26 CST 2016, 截止日期: Fri May 25 15:00:26 CST 2046
证书指纹:
         MD5: 4C:03:B5:D8:7D:37:C6:8C:94:5C:A5:FD:F7:BC:F3:F9
         SHA1: 35:9A:49:01:02:36:C0:E0:61:68:30:3D:AD:5D:A5:32:3F:2A:8E:71
         SHA256: 73:06:CE:61:BA:92:A3:B4:98:C4:EA:BA:9E:E5:FE:E4:86:C0:C4:B4:6F:03:D1:CE:F3:3D:7F:1A:D2:87:E7:C6
         签名算法名称: SHA1withRSA
         版本: 1
~~~         
说明该apk已经进行过签名，这是开发工具对apk自动进行的签名，这样才能直接将apk安装在设备上供我们调试使用。当我们要发布自己的apk时，就需要手动对其进行正式的签名了。

### 生成签名key

通过JDK的keytool工具生成签名key，因为我们已经将工具添加到系统的环境变量了，所以在要签名的apk目录下，执行下面命令：

~~~
$ keytool -genkey -v -keystore android.keystore -alias android.keystore -keyalg RSA -keysize 2048 -validity 10000
~~~
**android.keystore** 即是生成的安全密钥文件的名称
执行后回让你填写一些信息：

~~~
Enter keystore password: <输入密码>
Re-enter new password: <输入密码>
What is your first and last name? 
What is the name of your organizational unit?
What is the name of your organization?
What is the name of your City or Locality?
~~~
**请记住你输入的密码，进行签名时需要验证**

生成完成后，就可以看到目录下有一个 **android.keystore** 文件了，该文件必须妥善保存，之后再次进行签名时还要使用。

### 进行签名

使用 **jarsigner** 工具为apk签名，命令如下：

~~~
$ jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore android.keystore android-release-unsigned.apk android.keystore
~~~
执行过程中会要求你输入密码，请输入生成密钥文件时录入的密码
执行完成后，该 **android-release-unsigned.apk** 应用包就已经签名好了，就可以将这个apk对外发布了。
提示：签名完成后，我们可以通过前面的方法再次查看这个apk是否包含签名信息

## 优化（非必需）

使用 Zipalign 工具，它是一个android平台上整理APK文件的工具。它能够对打包的Android应用程序进行优化，以使Android操作系统与应用程序之间的交互作用更有效率，这能够让应用程序和整个系统运行得更快。

zipalign 工具位于你的android sdk 的Home目录的build-tools的某个版本的SDK下面。
比如我电脑上的目录为： D:\Android\sdk\build-tools\23.0.2
执行以下命令：

~~~
zipalign -v 4 android-release-unsigned.apk MyApp.apk
~~~
