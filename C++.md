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

关于Conan的问题，犹豫不决就直接去删包

1. 某个包被lock

如果remove lock不管用直接到` ~/.conan/data`删掉对应的包重新下载

2. 遇到类似这种问题也删掉重新安装

> ERROR: The 'libzip/1.7.3' package has 'exports_sources' but sources not found in local cache. Probably it was installed from a remote that is no longer available.

3. 在macOS下使用Conan编译flatbuffers2.0的flatc可执行文件时出错，提示` ld: library not found for -lcrt0.o`

+ 产生这个错误的根本原因是macOS不支持非kernel的包的`- static`选项，导致macOS缺少很多动态链接的库

macOS下`man ld`的结果信息

> Produces a mach-o file that does not use the dyld.  Only used building the kernel.

+ 产生这个错误的很重要的原因是conan-center-index有一套自己的conanfile.py，而flatbuffers官方项目下的conanfile中一直未设置过`FLATBUFFERS_STATIC_FLATC`

最后修改了conan-center-index的flatbuffers的conanfile.py。最终去掉了

```python
self._cmake.definitions["FLATBUFFERS_STATIC_FLATC"] = not self.options.shared
```

在合并之前公司项目里有自己的conan源，直接更改了自己源中的对应选项

关于沟通过程，详见这个pr：https://github.com/conan-io/conan-center-index/pull/6796

4. 报错：incorrect 'clang' is not one detected by CMake: 'GNU'

指定编译器为clang的情况下会遇到这个情况

在~/.conan/profiles/default中设置env的值

```
[env] 
CC=/usr/bin/clang 
CXX=/usr/bin/clang++
```

5. manjaro下安装某个conan包的时候提示找不到libturbojpeg.a

到/usr/lib下不存在该文件，只有.so

原因是manjaro官方源的对应包中不包含静态链接库，需要手动编译安装libjpeg-turbo到对应路径下[当前日期:2021.8.28]

## 编译

如果使用make -j有可能出现内存不足的问题，需要手动指定线程数或者再make -j一次（但是如果要编的太大，会再次遇到这种问题）

遇到奇怪的编译器bug，不影响使用的情况下升级编译器也能解决部分问题

## 链接

先检查头文件的符号与实现的符号是否一样，尤其是注意是否const对应好

\` objdump -t libfile`查看库中的符号，如果链接失败的话查看生成的内容是否存在对应链接失败的符号

检查cmake中是否添加了对应的library到target中

### MSVC

某个符号如果未被使用则不会被导出，需要找一个地方使用一次才行

c4273：

可能出现的一种情况是当前库与被链接库的dll导出导入不同。一个库是\_\_declspec(dllexport)，另一个库是\__declspec(dllimport)

## 第三方库与工具

截止2021.6.7，xtensor的adapt并没有适配含有padding的数据，即stride不等于使用shape算出来的默认stride

clang-format不同版本的结果格式不同，哪怕是小版本也会有差异

pybind11

` py::dtype::of<>`如果想要获取非标准类型，需要通过` py::dtype(str)`来获取，比如说float16

objdump -t查看符号

### 查看依赖

macOS: otool -L/-l

linux: ldd

windows: dependencies

## 安装conda后出现GLIBCXX_XXX not found

查看这个符号是否存在与libstdc++中

strings /usr/lib/libstdc++.so.6 | grep GLIBCXX_3.4.29

存在的话进入anaconda的安装目录，用类似的命令查看anaconda中的库是否包含这个符号，不包含则执行以下命令

sudo mv libstdc++.so.6 backpacklibstdc++.so.6

sudo ln -s /usr/lib/libstdc++.so.6 ./libstdc++.so.6

https://askubuntu.com/questions/575505/glibcxx-3-4-20-not-found-how-to-fix-this-error/764572#764572

## CLion

### 环境变量

CLion不会读取shell中的变量，但是貌似会读取/etc/environment目录下的全局变量（macOS无法验证这个问题）

CLion的cmake寻找库和build以及运行的时候需要链接动态库都会遇到这个问题

需要在项目里面手动添加环境变量。如果项目里需要调python，那么还需要在这里修改PYTHONPATH

### 连接远程开发

截止2020年的所有版本连接远程的时候都是会在后台上传文件到远程或者下载文件到远程，因此会出现文件更新不及时导致的bug

清空缓存再试

### 其他

遇到过按shift+1的时候打不出！而是跳出某个窗口，最后查看keymap不知道怎么多了一个macOS copy键位，切回原来的键位即可恢复正常

## vscode

见到同事出现过vscode调试的时候调试信息显示数据值和实际值不同
