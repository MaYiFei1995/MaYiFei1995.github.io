# AndroidStudio 中 gradle.properties 的中文值获取乱码问题
---
## 0x01 现象
在`gradle.properties`中定义了全局变量，然后从 build.gradle 中设置 app_name：
```groovy
resValue "string", "app_name", "\"" +  project.app_name + "\""
```
gradle 生成的 xml 文件出现了中文乱码
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <!-- Automatically generated file. DO NOT MODIFY -->

    <!-- Value from default config. -->
    <string name="app_name" translatable="false">"æµè¯åºç¨"</string>

</resources>
```
---
## 0x02 查错
检查了 gradle.properties 和 build.gradle ，确定编码是 UTF-8，也确认了 AndroidStudio 的编码是中文。
最后通过 google 解决了问题，[stackoverflow](https://stackoverflow.com/questions/30956495/gradle-uses-wrong-encoding-latin-1-for-property-file) 中提到的 [documentation of java.util.Properties](https://docs.oracle.com/javase/7/docs/api/java/util/Properties.html) 中有写到
> The load(Reader) / store(Writer, String) methods load and store properties from and to a character based stream in a simple line-oriented format specified below. The load(InputStream) / store(OutputStream, String) methods work the same way as the load(Reader)/store(Writer, String) pair, **except the input/output stream is encoded in ISO 8859-1 character encoding.** Characters that cannot be directly represented in this encoding can be written using Unicode escapes as defined in section 3.3 of The Java™ Language Specification; only a single 'u' character is allowed in an escape sequence. The native2ascii tool can be used to convert property files to and from other character encodings.

---
## 0x03 解决
确定了问题，解决就简单了，直接修改 gradle :
```groovy
resValue "string", "app_name", "\"" + new String(project.APP_NAME.getBytes("iso8859-1"), "UTF-8") + "\""
```
gradle 生成的 xml 文件：
```
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <!-- Automatically generated file. DO NOT MODIFY -->

    <!-- Value from default config. -->
    <string name="app_name" translatable="false">"测试应用"</string>

</resources>
```
***
