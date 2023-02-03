# Android 系统设备启动标识及系统更新标识的JNI与原生获取方式

---

## 通过JNI方式获取

### 系统启动标识 boot_mark

文档中提供的获取参考为

```c
static jstring getBoot(JNIEnv *env, jclass) {
    FILE *fp = fopen("/proc/sys/kernel/random/boot_id", "r");
    char boot[TID1_LEN];
    if (fp == nullptr) {
        //获取失败
    } else {
        unsigned char c;
        int i = 0;
        while (i < TID1_LEN) {
            c = fgetc(fp);
            boot[i] = c;
            i = i + 1;
        }
        if (ferror(fp)) {
            //获取失败
        }
    }
    // fgets(t1,sizeof(t1),fp);
    std::string sboot = boot;
    return env->NewStringUTF(sboot.c_str());
}
```

实际测试中发现，虚拟机等部分设备会在`NewStringUTF`方法中出现` NewstringUTF方法会在部分设备上报错 input is not valid Modified UTF-8: illegal start byte 0x**`错误，参照[Android API广告反作弊需求 Native层获取 bootMark奔溃解析
](https://blog.csdn.net/hanshengjian/article/details/120483858)中提到的方法修改后的代码为:
```c
static jstring getBoot(JNIEnv *env, jclass) {
    FILE *fp = fopen("/proc/sys/kernel/random/boot_id", "r");
    char boot[TID1_LEN];
    if (fp == nullptr) {
        //获取失败
    } else {
        unsigned char c;
        int i = 0;
        while (i < TID1_LEN) {
            c = fgetc(fp);
            boot[i] = c;
            i = i + 1;
        }
        if (ferror(fp)) {
            //获取失败
        }
    }
    // fgets(t1,sizeof(t1),fp);
    //std::string sboot = boot;
    //return env->NewStringUTF(sboot.c_str());
    //替换下面的代码
    jclass strClass = (env)->FindClass("java/lang/String");
    jstring encoding = (env)->NewStringUTF("UTF-8");
    jmethodID ctorID = (env)->GetMethodID(strClass, "<init>", "([BLjava/lang/String;)V");
    //建立byte数组
    jbyteArray bytes = (env)->NewByteArray((jsize) TID1_LEN);
    //将char* 转换为byte数组
    (env)->SetByteArrayRegion(bytes, 0, (jsize) TID1_LEN, (jbyte *) boot);
    return (jstring) (env)->NewObject(strClass, ctorID, bytes, encoding);
}
```

### 获取系统更新标识 update_mark

直接使用参考代码:

```c
static jstring getUpdate(JNIEnv *env, jclass) {
    struct stat sb{};
    int updates = 0;
    int updatens = 0;
    if (stat("/data/data", &sb) == -1) {
        //获取失败
    } else {
        updatens = (int) sb.st_atim.tv_nsec;
        updates = (int) sb.st_atim.tv_sec;
    }
    std::string idRes = std::to_string(updates) + "." + std::to_string(updatens);
    return env->NewStringUTF(idRes.c_str());
}
```

## 通过原生获取

### 系统启动标识 boot_mark

获取启动标识要更方便一些
```java
    public static String getBootMark() {
        String ret = "";
        try {
            File bootIdFile = new File("/proc/sys/kernel/random/boot_id");
            if (bootIdFile.exists()) {
                FileInputStream inputStream = null;
                try {
                    inputStream = new FileInputStream(bootIdFile);
                    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
                    ret = reader.readLine().trim();
                    try {
                        ret = ret.substring(0, 36);
                    } catch (Throwable ignore) {

                    }
                } catch (IOException ignore) {

                } finally {
                    if (inputStream != null) try {
                        inputStream.close();
                    } catch (IOException error) {
                        error.printStackTrace();
                    }
                }
            }
        } catch (Throwable tr) {
            tr.printStackTrace();
        }
        return ret;
    }
```

### 获取系统更新标识 update_mark

由于[StructStat#st_atim](https://developer.android.com/reference/android/system/StructStat#st_atim)和[StructTimespec](https://developer.android.com/reference/android/system/StructTimespec)是`API level 27`新增的属性，所以27以下的设备只能获取到秒精度

```java
    private String getUpdateMark() {
        try {
            // 通过os获取StructStat
            StructStat fstat = Os.stat("/data/data");
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O_MR1) {
                // 27及以上新增StructTimespec，支持nsec
                StructTimespec st_atim = fstat.st_atim;
                return st_atim.tv_sec + "." + st_atim.tv_nsec;
            } else {
                // 27以下通过 Seconds part of time of last access 获取
                return fstat.st_atime + "." + "0";
            }
        } catch (Throwable tr) {
            tr.printStackTrace();
        }
        return "";
    }
```