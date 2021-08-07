# C++相关

## 语法

private成员变量如果是unused的，clang会警告，而如果加上[[maybe_unused]]，gcc又会attribute ignored‘。如果开启了-Werror之类的会直接报错卡住

解决方案：

```c++
#if define(__clang__)
#pragma clang diagnostic ignored "-Wunused-private-field"
#endif
```

### EXC_BAD_INSTRUCTION

macOS下没有返回值会出现这个问题（这是出现这个错误的情况之一）

## CMake

添加 -DCMAKE_EXPORT_COMPILE_COMMANDS=ON查看编译命令

macOS下cmake寻找python会默认去找系统自带的版本，即` /usr/local/Frameworks/...`哪怕使用conda依然如此，需要手动指定路径才行

看到错误信息提示span非std成员，应该想到cmake中设置的cpp标准的问题

## Conan

提示某个包被lock了，如果remove lock不管用直接到` ~/.conan/data`删掉对应的包重新下载

关于Conan的问题，犹豫不决就直接去删包

> ERROR: The 'libzip/1.7.3' package has 'exports_sources' but sources not found in local cache. Probably it was installed from a remote that is no longer available.

遇到类似这种问题也删掉重新安装

## 编译

如果使用make -j有可能出现内存不足的问题，需要手动指定线程数或者再make -j一次（但是如果要编的太大，会再次遇到这种问题）

遇到奇怪的编译器bug，不影响使用的情况下升级编译器也能解决部分问题

## 链接

` objdump -t libfile`查看库中的符号，如果链接失败的话查看生成的内容是否存在对应链接失败的符号

检查cmake中是否添加了对应的library到target中

## 第三方库与工具

截止2021.6.7，xtensor的adapt并没有适配含有padding的数据，即stride不等于使用shape算出来的默认stride

clang-format不同版本的结果格式不同，哪怕是小版本也会有差异

pybind11

` py::dtype::of<>`如果想要获取非标准类型，需要通过` py::dtype(str)`来获取，比如说float16

objdump -t查看符号

otool -L/-l 查看链接的依赖（windows下使用dependencies）

## CLion

### 环境变量

CLion不会读取shell中的变量，但是貌似会读取/etc/environment目录下的全局变量（macOS无法验证这个问题）

CLion的cmake寻找库和build以及运行的时候需要链接动态库都会遇到这个问题

需要在项目里面手动添加环境变量。如果项目里需要调python，那么还需要在这里修改PYTHONPATH

### 连接远程开发

截止2020年的所有版本连接远程的时候都是会在后台上传文件到远程或者下载文件到远程，因此会出现文件更新不及时导致的bug

清空缓存再试

## vscode

见到同事出现过vscode调试的时候调试信息显示数据值和实际值不同