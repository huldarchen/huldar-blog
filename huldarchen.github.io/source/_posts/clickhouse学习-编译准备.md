---
title: clickhouse学习-编译准备
date: 2021-09-24 11:43:35
tags: clickhouse
---

# 环境准备

系统：macOS

1. Homebrewan安装

```sh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2. 安装基础组件

```sh
brew install cmake ninja libtool gettext llvm gcc binutils
```

3. 安装CLang

```sh
# 检查
brew info llvm
# 安装
brew install llvm
```

安装完成后记录信息

```sh
To use the bundled libc++ please add the following LDFLAGS:
  LDFLAGS="-L/usr/local/opt/llvm/lib -Wl,-rpath,/usr/local/opt/llvm/lib"

llvm is keg-only, which means it was not symlinked into /usr/local,
because macOS already provides this software and installing another version in
parallel can cause all kinds of trouble.

If you need to have llvm first in your PATH, run:
  echo 'export PATH="/usr/local/opt/llvm/bin:$PATH"' >> ~/.zshrc

For compilers to find llvm you may need to set:
  export LDFLAGS="-L/usr/local/opt/llvm/lib"
  export CPPFLAGS="-I/usr/local/opt/llvm/include"

==> mercurial
zsh completions have been installed to:
  /usr/local/share/zsh/site-functions
```

# CLion导入源码

将clickhouse根目录导入到CLion。

![image-20210809192415716](https://gitee.com/huldar/image/raw/master/uPic/2021-08-09/image-20210809192415716.png)

配置正确的CLang路径

默认情况下，clion可以探测出正确的编译器，但是在mac上，至少我的环境中探测出的编译器是App，并不work

![img](https://gitee.com/huldar/image/raw/master/uPic/2021-08-09/webp.jpg)

配置正确的CLang地址

![image-20210809193705568](https://gitee.com/huldar/image/raw/master/uPic/2021-08-09/image-20210809193705568.png)

# CLion进行编译

![image-20210809193805830](https://gitee.com/huldar/image/raw/master/uPic/2021-08-09/image-20210809193805830.png)

编译报错

```sh
cmake .. -DCMAKE_CXX_COMPILER=`which clang++` -DCMAKE_C_COMPILER=`which clang` -DCMAKE_BUILD_TYPE=Debug -DCMAKE_BUILD_WITH_INSTALL_RPATH=ON
-- IPO/LTO not enabled.
-- CMAKE_BUILD_TYPE: Debug
CMake Error at CMakeLists.txt:149 (message):
  Cannot find objcopy.
```

`Cannot find objcopy.` 解决办法

1. 安装objcopy

   ```sh
   brew install binutils
   ```



2. 查找objcopy

   ```sh
   mdfind -name objcopy
   ```



3. 在根目录下的CMakelist.txt修改

```sh
# find_program (OBJCOPY_PATH NAMES "llvm-objcopy" "llvm-objcopy-12" "llvm-objcopy-11" "llvm-objcopy-10" "llvm-objcopy-9" "llvm-objcopy-8" "objcopy")

find_program (OBJCOPY_PATH NAMES "llvm-objcopy" "llvm-objcopy-12" "llvm-objcopy-11" "llvm-objcopy-10" "llvm-objcopy-9" "llvm-objcopy-8" "objcopy"  PATHS "/usr/local/Cellar/binutils/2.36.1/x86_64-apple-darwin20.2.0/bin")

```

调整后就可以进行编译了，编译project后，运行

```sh
./cmake-build-debug//programs/clickhouse-server --config-file ./programs/server/config.xml
```

报错

```sh
<jemalloc>: ../contrib/jemalloc/include/jemalloc/internal/tsd_generic.h:145: Failed assertion: "tsd_booted"
[1]    81768 abort      ./clickhouse --vresion
```

解决办法 编译时候增加参数`-DENABLE_JEMALLOC=0` 后重新编译，编译完成后即可运行成功

![image-20210809194107415](https://gitee.com/huldar/image/raw/master/uPic/2021-08-09/image-20210809194107415.png)
