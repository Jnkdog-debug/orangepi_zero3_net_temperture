
# 编译内核与设备树指南（Orange Pi Zero3）

## 1. 更改权限

为防止编译过程中出现权限错误，先将内核源码目录的所有权更改为当前用户：

```bash
sudo chown -R xyz:xyz /home/xyz/orangepi/orangepi-build/
```

> 请将 `xyz` 替换为你实际的用户名。

---

## 2. 设置交叉编译环境变量

由于香橙派提供的 Linux SDK 工具链位于 `toolchains/` 目录中，因此需要将其加入环境变量。
如果国内github不好访问，那么有越过github下载的办法，详情看这篇博客https://blog.csdn.net/m0_63674807/article/details/146135169
编辑 `~/.bashrc`：

```bash
nano ~/.bashrc
```

在末尾添加以下内容：

```bash
export ARCH=arm64
export CROSS_COMPILE=aarch64-none-linux-gnu-
export PATH=/home/xyz/orangepi/orangepi-build/toolchains/gcc-arm-11.2-2022.02-x86_64-aarch64-none-linux-gnu/bin:$PATH
export GCC_COLORS=auto
```

保存后使其立即生效：

```bash
source ~/.bashrc
```

---

## 3. 编译内核

进入内核源码目录：

```bash
cd ~/orangepi/orangepi-build/kernel/orange-pi-6.1-sun50iw9
```

运行 `menuconfig` 配置内核：

```bash
make menuconfig
```

步骤：

1. 选择 `<Load>`，载入 `linux-6.1-sun50iw9-next.config` 文件；
2. 选择 `<Save>`，保存为相同文件名；
3. 选择 `<Exit>` 退出配置界面；

然后开始编译内核：

```bash
make -j$(nproc)
```

---

## 4. 编译设备树

将自定义的设备树 overlay 插件文件（`.dts`）复制到：

```bash
orangepi-build/kernel/orange-pi-6.1-sun50iw9/arch/arm64/boot/dts/allwinner/overlay/
```

执行设备树编译：
- 先 在内核源码下
make menuconfig
按照选项配置.config 主要是要load 
- 然后
```bash
make dtbs
```

（这里太玄学了，不知道怎么就可以了）
- [ ]使用设备树 需要编辑 /boot/uEnv.txt 文件，添加这个设备树插件 **todo**

- 香橙派设备树文档 
- [sun50i-h618-orangepi-zero3.dts](/~/orangepi/orangepi-build/kernel/orange-pi-6.1-sun50iw9/arch/arm64/boot/dts/allwinner/sun50i-h618-orangepi-zero3.dts)
---


## 5. 编译内核模块（单个驱动）

进入驱动所在目录，确保该目录中有 `Makefile`，执行：

```bash
make
```

即可使用当前设置的内核源码进行模块编译。

---

## 常见问题

- **编译失败提示找不到命令？**  
  请确保 `.bashrc` 环境变量已生效，或重新打开终端。

- **设备树修改无效？**  
  请确认你已经将 `.dtbo` 正确复制到开发板 `/boot/dtb/overlay/` 目录，并在启动配置中启用。

- **想更新内核后重新构建？**  
  请执行 `make clean` 或 `make mrproper` 清理旧配置。
