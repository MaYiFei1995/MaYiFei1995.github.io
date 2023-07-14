# WINDOWS 环境下编译 OLLVM 替换到 NDK 环境

---

## 编译 OLLVM

### 环境准备

这里使用的是`AGP 7.2.2`、`NDK 25.2.9519653`、`llvm 14.0.7`、`cmake 3.22.1`、`python39`

#### git

用来下载源码

#### python

搞到这一步环境变量里应该已经有[python](https://www.python.org/downloads/windows/)了吧

#### NDK

AGP 的 7.2.2 版本默认使用的 NDK 版本为`21.4.7075529`，对应的 LLVM 为`9.0.9`。

![ndk](https://MaYiFei1995.github.io/img/2023-07-14-0.png)

需要根据实际情况选择 NDK 对应的 LLVM 版本，编译 OLLVM，LLVM 版本号可以通过`Sdk_DIR\ndk\$version\toolchains\llvm\prebuilt\windows-x86_64\AndroidVersion.txt`文件看到，如SDK Manager 中的最新版本`25.2.9519653`为：
```
14.0.7
based on r450784d1
for additional information on LLVM revision and cherry-picks, see clang_source_info.md
```

在 module 的`build.gradle`中指定 NDK 版本
```groovy
android {
    ndkVersion "25.2.9519653"
}
```

#### OLLVM

可以在 [github/heroims/obfuscator](https://github.com/heroims/obfuscator) 仓库的分支中找到对应的移植源码，

![heroims](https://MaYiFei1995.github.io/img/2023-07-14-1.png)

`13x`和`14x`版本也可以在 [github/yangyiyu08/ollvm-project](https://github.com/yangyiyu08/ollvm-project) 仓库的对应分支中获取。`14x`版本还可以直接在 Releases 中下载作者编译好的文件，跳过编译的步骤。但这个文件在我的环境下会有NDK编译错误的情况，我自己编译出来的文件运行正常。

这里使用`NDK 25.2.9519653`，需要下载 14x 的源码。

#### CMAKE

AGP 的 7.2.2 版本默认使用的 cmake 版本为`3.18.1`，在指定 NDK 版本到 25.2.9519653 后，会在编译时提示需要升级到 3.19 版本以上的信息。所以这里在编译前就通过 SDK Manager 下载 3.22.1 版本，并把所在目录**添加到环境变量**。

同时在`build.gradle`和`CMakeLists`中修改工程的 cmake 版本。

### 开始编译

在源码的目录执行以下命令构建 cmake 配置
`cmake -S llvm -B build -G Ninja -DLLVM_ENABLE_PROJECTS="clang" -DCMAKE_BUILD_TYPE=Release -DLLVM_INCLUDE_TESTS=OFF -DLLVM_ENABLE_NEW_PASS_MANAGER=OFF`

对应的参数:
> -G Ninja: 使用 ninja 进行编译源码
> -DLLVM_ENABLE_PROJECTS="clang": 启用clang，有多个选择 但我们只需要clang，官方文档有说明
> -DCMAKE_BUILD_TYPE=Release: 构建 release 版本，比 debug 版本编译快很多
> -DLLVM_INCLUDE_TESTS=OFF: 关闭 llvm 的头文件测试，也是为了加快编译速度
> -DLLVM_ENABLE_NEW_PASS_MANAGER=OFF: 这个非常重要，llvm-12.x 开始默认使用 newPM进行编译源码，导致 ollvm 不起作用！因此，需要加上这个参数禁用掉 newPM。(在每个编译时增加flag -flegacy-pass-manager 让 llvm 不走 newPM 编译也可以，但没必要）

执行以上命令后若提示 Configuration done. 则配置成功。接下来执行以下命令开始编译：
`cmake --build build -j16`

其中`-j16`为指定的线程数，需要根据 CPU 调整。然后等待编译完成

![waiting...](https://MaYiFei1995.github.io/img/2023-07-14-2.png)

### 编译完成

编译完成后打开`build/bin`目录，找到`clang.exe`、`clang++.exe`、`clang-cl.exe`，可以看到三个文件的MD5是相同的。

![certutil](https://MaYiFei1995.github.io/img/2023-07-14-3.png)

编译后的文件大小为 137MB，对比 NDK 目录下的 clang.exe 仅有 88.6MB。编译后的文件可以通过`strip clang.exe`命令剥离可执行文件减小到 113MB。

## 替换到 NDK 环境

### 备份与替换

首先打开当前 NDK 的 llvm 目录，将`clang.exe`、`clang++.exe`、`clang-cl.exe`备份，然后把上一步编译后的文件复制到当前目录。

![bak](https://MaYiFei1995.github.io/img/2023-07-14-4.png)

### 复制 lib 库

此时编译会出现找不到`libunwind`等库的错误，错误信息显示目录文件不存在
```log
  CMake Error at SDK_DIR/cmake/3.22.1/share/cmake-3.22/Modules/CMakeTestCCompiler.cmake:69 (message):
    The C compiler
  
      "SDK_DIR/ndk/25.2.9519653/toolchains/llvm/prebuilt/windows-x86_64/bin/clang.exe"
  
    is not able to compile a simple test program.
  
    It fails with the following output:
  
      Change Dir: APPLICATION_DIR/app/.cxx/RelWithDebInfo/703a16l3/arm64-v8a/CMakeFiles/CMakeTmp
      
      Run Build Command(s):SDK_DIR\\cmake\3.22.1\bin\ninja.exe cmTC_c87d1 && [1/2] Building C object CMakeFiles/cmTC_c87d1.dir/testCCompiler.c.o
      [2/2] Linking C executable cmTC_c87d1
      FAILED: cmTC_c87d1 
      cmd.exe /C "cd . && SDK_DIR\\ndk\25.2.9519653\toolchains\llvm\prebuilt\windows-x86_64\bin\clang.exe --target=aarch64-none-linux-android26 --sysroot=SDK_DIR/ndk/25.2.9519653/toolchains/llvm/prebuilt/windows-x86_64/sysroot -g -DANDROID -fdata-sections -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -D_FORTIFY_SOURCE=2 -Wformat -Werror=format-security -static-libstdc++ -Wl,--build-id=sha1 -Wl,--no-rosegment -Wl,--fatal-warnings -Wl,--no-undefined -Qunused-arguments CMakeFiles/cmTC_c87d1.dir/testCCompiler.c.o -o cmTC_c87d1  -latomic -lm && cd ."
      ld: error: unable to find library -latomic
      ld: error: cannot open SDK_DIR/ndk/25.2.9519653/toolchains/llvm/prebuilt/windows-x86_64/lib/clang/14.0.0/lib/linux/libclang_rt.builtins-aarch64-android.a: No such file or directory
      ld: error: unable to find library -l:libunwind.a
      ld: error: cannot open SDK_DIR/ndk/25.2.9519653/toolchains/llvm/prebuilt/windows-x86_64/lib/clang/14.0.0/lib/linux/libclang_rt.builtins-aarch64-android.a: No such file or directory
      ld: error: unable to find library -l:libunwind.a
      clang: error: linker command failed with exit code 1 (use -v to see invocation)
      ninja: build stopped: subcommand failed.
      
      
  
    
  
    CMake will not be able to correctly generate this project.
  Call Stack (most recent call first):
    CMakeLists.txt:10 (project)
```

需要把`SDK_DIR/ndk/25.2.9519653/toolchains/llvm/prebuilt/windows-x86_64/lib64/`目录下的`calng`目录，复制到`/lib`目录中，并把`clang/14.0.7`修改为`14.0.0`。

这里的`14.0.0`是根据错误日志中出现的路径提取出来的，在控制台中输入`clang -v`查看当前 clang 的版本，按照日志中出现或当前使用的 clang 版本调整。

![clang](https://MaYiFei1995.github.io/img/2023-07-14-5.png)

至此，NDK 中 集成 OLLVM 已经完成了。接下来是配置和使用 OLLVM。

## 配置 OLLVM

这部分网上参考的文档很多，这里也只是简单介绍一下参数

### 参数

|参数|说明|
|:-:|:-:|
|-mllbm -sub|激活指令替换|
|-mllvm -sub_loop=3|如果激活了传递，则在函数上应用3次。默认值：1|
|-mllvm -bcf|激活虚假控制流程|
|-mllvm -bcf_loop=3|如果激活了传递，则在函数上应用3次。默认值：1|
|-mllvm -bcf_prob=40|如果激活了传递，基本块将以40％的概率进行模糊处理。默认值：30|
|-mllvm -fla|激活控制流扁平化|
|-mllvm -split|激活基本块分割。在一起使用时改善展平|
|-mllvm -split_num=3|如果激活了传递，则在每个基本块上应用3次。默认值：1|


### 拓展参数

[heroims](https://heroims.github.io/)在移植 OLLVM 时，集成了[Armariris](https://github.com/GoSSIP-SJTU/Armariris)的字符串混淆功能。

|参数|说明|
|:-:|:-:|
|-mllvm -sobf|编译时候添加选项开启字符串加密|
|-mllvm -seed=|指定随机数生成器种子|

### 使用

可以在`build.gradle`中进行配置，如:
```groovy
android {
    defaultConfig {
        extrnalNativeBuild {
            cmake {
                cppFlags '-mllvm -fla'
            }
        }
    }
}
```

也可以在`CMakeLists`中进行配置，如：
```
SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mllvm -fla -mllvm -sub -mllvm -sobf")
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mllvm -fla -mllvm -sub -mllvm -sobf")
```

## 输出比对

以配置了`-fvisibility=hidden -ffunction-sections -fdata-sections` 的获取 updatemark 为例，源码为：
```c++
static jstring getUpdate(JNIEnv *env, jobject clazz) {
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

### before

![before](https://MaYiFei1995.github.io/img/2023-07-14-6.png)

### after

![after](https://MaYiFei1995.github.io/img/2023-07-14-7.png)

## 遇到的问题

### 虚假控制流程的问题

实际编译的过程中，增加了`-mllvm -bcf`参数后，编译超过半个小时还是没有完成，移除后正常编译。
搜索了相关的问题后，发现可能是 ndk 的编译器优化 flag 在 release 时是 -O2 导致的。
> This is specifically an issue with llvm / android-ndk when compiling with thumb mode. You'll either need to disable thumb compilation (annoying) or patch the llvm to not generate this type of instructions; it's not actually an obfuscator-llvm issue.
> 
> Potentially try upgrading your ndk as well, though I'm doubtful that will fix this issue. If I have extra time later I can try to find the patch I needed to create for llvm to specifically work around this issue.

但在我这边，debug 依然是没有响应，只能是去掉`-bcf`。

### CMAKELIST 的 Release 不生效问题

`CMAKE_C_FLAGS`和`CMAKE_C_FLAGS_DEBUG`配置的参数都可以正常生效，但`CMAKE_C_FLAGS_RELEASE`配置无法在 release 时生效。只能配置在`CMAKE_C_FLAGS`中，然后在 CMakeLists 中判断当前环境是否为 DEBUG。

## 参考资料

- [yangyiyu08/【清羽】Windows10下编译OLLVM-14.x](https://blog.csdn.net/qq_41923691/article/details/123258565)
- [heroims/OLLVM代码混淆移植与使用](https://heroims.github.io/2019/01/06/OLLVM%E4%BB%A3%E7%A0%81%E6%B7%B7%E6%B7%86%E7%A7%BB%E6%A4%8D%E4%B8%8E%E4%BD%BF%E7%94%A8/)
- [obfuscator/issues/-bcf crashes when compiling ndk project](https://github.com/obfuscator-llvm/obfuscator/issues/78#issuecomment-320805930)