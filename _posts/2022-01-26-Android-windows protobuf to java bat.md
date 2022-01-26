# windows 环境下 ProtoBuf 编译 Java 文件的简单脚本
---
- 在 ProtoBuf 的 [release](https://github.com/protocolbuffers/protobuf/releases/) 页面下载 win64 版本，解压。

- 编写一个简单的编译脚本，遍历目录下的`.proto`文件，并编译输出到 out 目录

    ```sh
    @echo off

    echo "compile proto to java"

    set proto_path=".\files"
    set java_out_path="..\java"

    for %%f in ( %proto_path%\*.proto ) do ( cmd /c " .\bin\protoc.exe "%%f" --proto_path=%proto_path% --java_out=%java_out_pa
    ```