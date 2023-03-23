买不起ARM设备，直接在ubuntu上起一个qemu进行模拟arm开发环境

# 配置环境

```shell
sudo apt-get install gcc gcc-arm-linux-gnueabi binutils-arm-linux-gnueabi qemu-user gdb-multiarch
```

然后再继续安装qemu的依赖项

```shell
sudo apt-get install build-essential gcc pkg-config glib-2.0 libglib2.0-dev libsdl1.2-dev libaio-dev libcap-dev libattr1-dev libpixman-1-dev
```

# 下载qemu

```shell
wget https://download.qemu.org/qemu-3.0.0.tar.xz
```

最简单的就是下载qemu的源码本地编译

建议提前建一个文件夹，然后解压这个文件

```shell
tar xvJf qemu-3.0.0.tar.xz
```

先编译，`cd qemu-3.0.0`

```shell
./configure
```

可能的报错：

- make的时候总是报错，就找到那个报错的文件，把报错的那一行的static关键字删除
- 关于python的报错：`ERROR: Python not found. Use --python=/path/to/python`：
  - 直接`apt install python`, 就行

然后最后一步，可能需要一点时间

```shell
make&&make install
```

