# 为 APK 文件增加右键菜单组实现快捷安装与快捷反编译

***

## 0.结果
![效果图](https://MaYiFei1995.github.io/img/2021-03-03-1.png)
![效果图](https://MaYiFei1995.github.io/img/2021-03-03-2.png)
---

## 1.需求
迫于每次都要打开 Powershell 手动敲 ```adb install xxx.apk``` 太麻烦，就想通过注册表搞一个右键菜单，实现快捷安装 apk 的功能。
最后决定先实现三个功能：
- adb install -r
- adb install -t
- 使用 jarsigner 重签名

可是对 windows 一窍不通，只能去网上抄代码。

**更新**
把 apk 复制到 as 中再通过 `ApkAnalyzer` 查看 `AndroidManifest`太麻烦，加上不想每次敲`java -jar apktool.jar d a.apk --only-main-classes`这么一大串，就增加了三个新功能
- 导出 apk 的 `AndroidManifest.xml`
- apk 右键`apktool d`
- 目录右键`apktool b`

接下来要做的就是反编译各类型文件格式转换了，`dx.jar`、`d2j`、`baksmali.jar`、`smali.jar`这些工具的快捷命令

---

## 2.抄代码
照着几个现成的代码抄，又测了半天，最后发现无论是注册`HKEY_CLASSES_ROOT\.apk\`还是`HKEY_CLASSES_ROOT\apk_auto_file\`、无论配置`SubCommands`还是直接在`\.apk\shell`下面增加条目，都没办法在apk文件的右键中展示。

反正也是自己用，图方便就直接对`HKEY_CLASSES_ROOT\*\`进行注册了。

调用的代码是`Powershell -noexit 'command'`，第一次写 shell 代码，这个`-noexit`还查了老半天。

查看 AndroidManifest 的部分，尝试使用`aapt2 dump xmltree xxx.apk AndroidManifest.xml`来实现，但导出的 xml 是解析后的内容，不方便直接查看，没办法替代复制到AS中用`ApkAnalyzer`的功能。

参照[apkanalyzer command](https://developer.android.com/studio/command-line/apkanalyzer.html)文档，尝试使用命令行调用` apkanalyzer manifest print xxx.apk`，但``ApkAnalyzer`是 unix 的 shell，没办法在 windows 环境下直接调用。

在[StackOverflow](https://stackoverflow.com/questions/47081004/apkanalyzer-is-not-recognized-as-an-internal-or-external-command/51905063#51905063)搜了下，找到了解决办法，按照文中配置了 cmd 后，将目录添加到了 Path 中，就可以配置注册表了。

---

## 3.代码
```shell
@echo off

::##############################################################################
::##
::##  apkanalyzer.cmd
::##
::##  apkanalyzer start up script for Windows
::##
::##  converted by ewwink
::##
::##############################################################################

::Attempt to set APP_HOME

SET SAVED=%cd%
SET APP_HOME=C:\android\sdk\tools
SET APP_NAME="apkanalyzer"

::Add default JVM options here. You can also use JAVA_OPTS and APKANALYZER_OPTS to pass JVM options to this script.
SET DEFAULT_JVM_OPTS=-Dcom.android.sdklib.toolsdir=%APP_HOME%

SET CLASSPATH=%APP_HOME%\lib\dvlib-26.0.0-dev.jar;%APP_HOME%\lib\util-2.2.1.jar;%APP_HOME%\lib\jimfs-1.1.jar;%APP_HOME%\lib\annotations-13.0.jar;%APP_HOME%\lib\ddmlib-26.0.0-dev.jar;%APP_HOME%\lib\repository-26.0.0-dev.jar;%APP_HOME%\lib\sdk-common-26.0.0-dev.jar;%APP_HOME%\lib\kotlin-stdlib-1.1.3-2.jar;%APP_HOME%\lib\protobuf-java-3.0.0.jar;%APP_HOME%\lib\apkanalyzer-cli.jar;%APP_HOME%\lib\gson-2.3.jar;%APP_HOME%\lib\httpcore-4.2.5.jar;%APP_HOME%\lib\dexlib2-2.2.1.jar;%APP_HOME%\lib\commons-compress-1.12.jar;%APP_HOME%\lib\generator.jar;%APP_HOME%\lib\error_prone_annotations-2.0.18.jar;%APP_HOME%\lib\commons-codec-1.6.jar;%APP_HOME%\lib\kxml2-2.3.0.jar;%APP_HOME%\lib\httpmime-4.1.jar;%APP_HOME%\lib\annotations-12.0.jar;%APP_HOME%\lib\bcpkix-jdk15on-1.56.jar;%APP_HOME%\lib\jsr305-3.0.0.jar;%APP_HOME%\lib\explainer.jar;%APP_HOME%\lib\builder-model-3.0.0-dev.jar;%APP_HOME%\lib\baksmali-2.2.1.jar;%APP_HOME%\lib\j2objc-annotations-1.1.jar;%APP_HOME%\lib\layoutlib-api-26.0.0-dev.jar;%APP_HOME%\lib\jcommander-1.64.jar;%APP_HOME%\lib\commons-logging-1.1.1.jar;%APP_HOME%\lib\annotations-26.0.0-dev.jar;%APP_HOME%\lib\builder-test-api-3.0.0-dev.jar;%APP_HOME%\lib\animal-sniffer-annotations-1.14.jar;%APP_HOME%\lib\bcprov-jdk15on-1.56.jar;%APP_HOME%\lib\httpclient-4.2.6.jar;%APP_HOME%\lib\common-26.0.0-dev.jar;%APP_HOME%\lib\jopt-simple-4.9.jar;%APP_HOME%\lib\sdklib-26.0.0-dev.jar;%APP_HOME%\lib\apkanalyzer.jar;%APP_HOME%\lib\shared.jar;%APP_HOME%\lib\binary-resources.jar;%APP_HOME%\lib\guava-22.0.jar

SET APP_ARGS=%*
::Collect all arguments for the java command, following the shell quoting and substitution rules
SET APKANALYZER_OPTS=%DEFAULT_JVM_OPTS% -classpath %CLASSPATH% com.android.tools.apk.analyzer.ApkAnalyzerCli %APP_ARGS%

::Determine the Java command to use to start the JVM.
SET JAVACMD="java"
where %JAVACMD% >nul 2>nul
if %errorlevel%==1 (
  echo ERROR: 'java' command could be found in your PATH.
  echo Please set the 'java' variable in your environment to match the
  echo location of your Java installation.
  echo.
  exit /b 0
)

:: execute apkanalyzer

%JAVACMD% %APKANALYZER_OPTS%
```

```shell
Windows Registry Editor Version 5.00

; ApkHelper.reg

[HKEY_CLASSES_ROOT\*\Shell\ApkHelper]

"MUIVerb"="APK Helper"

"SubCommands"=""

"Position"="Center"

[HKEY_CLASSES_ROOT\*\shell\ApkHelper\Shell]

[HKEY_CLASSES_ROOT\*\shell\ApkHelper\Shell\InstallR]

@="adb install -r"

[HKEY_CLASSES_ROOT\*\shell\ApkHelper\Shell\InstallR\command]

@="PowerShell -noexit adb install -r \"%1\" "

[HKEY_CLASSES_ROOT\*\shell\ApkHelper\Shell\InstallT]

@="adb install -t"

[HKEY_CLASSES_ROOT\*\shell\ApkHelper\Shell\InstallT\command]

@="PowerShell -noexit adb install -r -t \"%1\" "

[HKEY_CLASSES_ROOT\*\shell\ApkHelper\Shell\SignNew]

@="Jarsigner"

[HKEY_CLASSES_ROOT\*\shell\ApkHelper\Shell\SignNew\command]

@="PowerShell -noexit jarsigner -verbose -keystore F:\Decompile\windows签名工具\Test.keystore -storepass 123456 -signedjar \"%1.signed.apk\"  \"%1\"  test -digestalg SHA1 -sigalg MD5withRSA  "

[HKEY_CLASSES_ROOT\*\shell\ApkHelper\Shell\ShowManifest]

@="Manifest"

[HKEY_CLASSES_ROOT\*\shell\ApkHelper\Shell\ShowManifest\command]

@="PowerShell -noexit apkanalyzer manifest print \"%1\" >  \"%1.AndroidManifest.xml\"  "

```
```
Windows Registry Editor Version 5.00

; apktool.reg

[HKEY_CLASSES_ROOT\*\shell\APKTool]

"MUIVerb"="APKTool"

"SubCommands"=""

"Position"="Center"

[HKEY_CLASSES_ROOT\*\shell\APKTool\shell]

[HKEY_CLASSES_ROOT\*\shell\APKTool\shell\apktoold]

@="apktool d"

[HKEY_CLASSES_ROOT\*\shell\APKTool\shell\apktoold\command]

@="PowerShell -noexit java -jar .\\apktool_2.5.0.jar d \"%1\" --only-main-classes"

[HKEY_CLASSES_ROOT\Folder\shell\APKTool]

"MUIVerb"="APKTool"

"SubCommands"=""

"Position"="Center"

[HKEY_CLASSES_ROOT\Folder\shell\APKTool\shell]

[HKEY_CLASSES_ROOT\Folder\shell\APKTool\shell\apktoolb]

@="apktool b"

[HKEY_CLASSES_ROOT\Folder\shell\APKTool\shell\apktoolb\command]

@="PowerShell -noexit java -jar .\\apktool_2.5.0.jar b \"%1\" -o new.apk"
```
---

# 4.Todo
- 1.只为 apk 文件注册右键菜单组
- 2.配置命令执行 python 脚本对重签名的文件名进行优化
- 3.为 dex 文件增加常用的 d2j 命令
