## Redis 在 Android Emulator 的部署实践

[Redis](https://redis.io/) 是一个键值型的 NoSQL 数据库，我们通常用它来做缓存。随着 Redis 5.0 推出 Redis Stream，Redis 在 IoT 场景的应用将是一种趋势。Redis 已经成功的部署在[树莓派](https://redis.io/topics/ARM)上了。  
本篇将介绍如何将 Redis 跑在 Android 系统上。

### 编译 Redis

要在 Android 上编译 Redis, 我们需要用到 Android NDK。所以这里简单介绍一下关于 Android NDK 的几个概念。

#### Android NDK

[Android NDK](https://developer.android.com/ndk) 是一套允许使用 C 和 C++ 等语言，以原生代码实现部分应用的工具集。它将 C/C++ 代码编译到原生库中，然后 Java 代码可以通过 Java 原生接口 (JNI) 调用原生库中的函数。更多关于 Android NDK 的概念，请查看[这里](https://developer.android.com/ndk/guides/concepts)。  
常用的 NDK 构建代码的方法有：基于 Make 的 [ndk-build](https://developer.android.com/ndk/guides/ndk-build.html) 和 [CMake](https://developer.android.com/ndk/guides/cmake.html)。

#### 基于本地的 NDK 编译

**安装 NDK 环境**  
要使用 NDK 需要先安装相关组件，具体可以参考[官网](https://developer.android.com/ndk/guides#download-ndk), 更建议参考我之前的一篇[使用 Android Studio 构建应用程](../android/20190502-android-studio-build-app.md)。在 Android Studio 从主菜单选择 Tools > Android > SDK Manager。勾选：

- Android 原生开发工具包 (NDK)：这套工具允许您为 Android 使用 C 和 C++ 代码。
- CMake：一款外部构建工具，可与 Gradle 搭配使用来构建原生库。如果您只计划使用 ndk-build，则不需要此组件。

- LLDB：一种调试程序，Android Studio 使用它来调试原生代码。

如果要在 terminal 使用 ndk-build 指令，记得把 ndk-build 的目录加到 PATH。  
**编译 Redis**  
这个 [Repo](https://github.com/wf9a5m75/redis-android) 提供了一个方法。可以下载工程，用 Android Studio 编译。

```
$ git clone https://github.com/wf9a5m75/redis-android
```

也可以直接使用他们提供的编译好的[版本](https://github.com/wf9a5m75/redis-android/releases)。主要是两个文件：`redis-server` 和 `redis-cli`。然后我们把文件通过 ADB 传到 Emulator。下面会介绍。

#### 基于 Docker 的 NDK 编译

我们也可以使用 Docker 来构建。[Android-sdk](https://github.com/bitrise-io/android-ndk) 是一个预安装 NDK 的 Android 镜像。在此基础上，我们在上面仓库的 [jni 文件夹](https://github.com/wf9a5m75/redis-android/tree/master/redis-android/src/main/jni) ndk-build。按照这个思路构建一个 Docker 镜像，然后通过 -v 参数，把生成的文件映射到本机。

```
$ docker build ./ --tag aredis
$ docker run -it -v ~/temp:/build aredis:latest
```

这里就不介绍细节了。

### 使用 ADB 将 redis 部署到 Emulator

#### 准备 Redis 启动文件

上面我们已经得到了 `redis-server` 了。像在 linux 运行一样，我们还需要一个配置文件 `redis.conf`，可以网上下载一个。另外，我们需要将 Redis 开机启动，所以在 `/etc/init` 加个开机启动脚本 `redis.rc`。

```
service redis /system/bin/redis-server /etc/redis.conf
    class main
    user system
    group system root wifi net_raw net_admin
    seclabel u:r:init:s0
```

到现在我们就有了以下几个文件：

- redis.rc
- redis.conf
- redis-cli
- redis-server

#### 创建 AVD

我们可以[使用 Android Studio 来创建 AVD](https://developer.android.com/studio/run/managing-avds?hl=zh-cn), 注意这里我们选择没有 Google Play 的版本，避免需要 root。可以查看现在可以的 emulator。

```
$ emulator -list-avds
```

启动 emulator，这里注意不要使用 Android Studio 启动，可能产生权限问题。

```
$ emulator @<avd name> -writable-system -selinux permissive
```

#### 写入 Redis 文件

要将文件写入 emulator, 我们要用到 ADB。[Android 调试桥 (adb)](https://developer.android.com/studio/command-line/adb) 是一个通用命令行工具，其允许您与模拟器实例或连接的 Android 设备进行通信。关于 ADB 的更多细节，请查看[官网](https://developer.android.com/studio/command-line/adb)。接下来我们就把文件传到 emulator。

- 以 root 权限重启 adbd daemon

```
$ adb root
```

这里如果没有 root 的机器会出错。

```
adbd cannot run as root in production builds
```

AVD 中 Android 7.0 以上默认就没 root，而且带有 Google Play 的就需要自己手动 root 了，参考[这个方法](https://infosectrek.wordpress.com/2017/03/06/rooting-the-android-emulator/)来 root。 我果断选择 Google API 的版本。此时基本就可以成功了，有的还不成功的，可以进入 adb shell，会到`$ 界面`。su 一下到`# 界面`，表示 root 启动成功。

- adb remount  
  adb remount 将 `/system 文件夹` 部分置于可写入的模式，默认情况下是只读模式的。这个命令只适用于已被 root 的设备。在将文件 push 到 system 文件夹之前，必须先输入命令 'adb remount'。它的作用相当于

```
adb shell mount -o rw,remount,rw /system
```

- push 文件到 emulator

```
// 可执行文件放入 system/bin
adb push redis-server /system/bin/
adb push redis-cli /system/bin/
// 配置文件放入 /etc
adb push redis.conf /etc/
// 开机脚本 放入 /etc/init，加执行权限
adb push redis.rc /etc/init/
adb shell chmod 644 /etc/init/redis.rc

// 创建redis工作目录，并授权给system用户
adb shell mkdir /system/redis
adb shell chown -R system:system /system/redis
```

- 重启 emulator 生效

```
$ adb shell reboot
```

### 调试 redis

此时的 emulator 上跑的 Redis，外部还是不能访问，需要做一个端口映射。

- 查看 emulator 的端口

```
$adb devices
List of devices attached
emulator-5554	device
```

发现运行在 5554 端口。

- telnet 连接

```
$4 telnet localhost 5554
Trying ::1...
telnet: connect to address ::1: Connection refused
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
Android Console: Authentication required
Android Console: type 'auth <auth_token>' to authenticate
Android Console: you can find your <auth_token> in
'/Users/dennis/.emulator_console_auth_token'
OK
```

- redir 端口映射
  使用 redir 指令，增加端口映射。

```
$ redir add tcp:5000:6379
// 查看已有映射
$ redir list
```

如果提示没有 redir 指令，需要先 auth。

```
auth <$HOME/.emulator_console_auth_token>
```

此时，在主机连接 emulator 的 redis 成功。

```
$ redis-cli -p 5000
```

好了，可以愉快地操作 Redis 了。

### 参考

[1][ndk 入门指南](https://developer.android.com/ndk/guides)  
[2][rooting the android emulator – on android studio 2.3 ](https://infosectrek.wordpress.com/2017/03/06/rooting-the-android-emulator/)  
[3][adbd cannot run as root in production builds的解决方法](https://www.cnblogs.com/Ph-one/p/5692335.html)
