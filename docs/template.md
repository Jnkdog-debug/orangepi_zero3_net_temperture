
# 模块名称（如：DHT11 驱动）

## 简介

简要描述这个模块的功能、作用、与系统的集成方式。

例如：
本模块用于驱动 DHT11 温湿度传感器，读取数据后供 UI 显示或通过 Web 上传至远程服务器。

---

## 文件结构

```
路径：drivers/dht11/

├── dht11.c         // 主驱动源码
├── dht11.h         // 头文件
├── Makefile        // 模块编译配置
└── README.md       // 当前文档
```

---

## 编译说明

确保内核路径、CROSS_COMPILE 已设置。

```bash
make -C /path/to/kernel M=$(pwd) modules
```

或者使用项目内统一脚本：

```bash
./scripts/build_driver.sh dht11
```

---

## 加载模块

```bash
sudo insmod dht11.ko
```

查看日志：

```bash
dmesg | tail
```

卸载模块：

```bash
sudo rmmod dht11
```

---

## 设备树配置（如适用）

如果需要通过设备树加载，请说明：

```dts
dht11@0 {
    compatible = "mycompany,dht11";
    gpios = <&pio 1 6 GPIO_ACTIVE_HIGH>;  // 示例 GPIO
    status = "okay";
};
```

路径：`arch/arm64/boot/dts/sunxi/overlay/dht11-overlay.dts`

编译：

```bash
make dtbs
```

---

## 接口说明（API）

```c
int dht11_read_temp_humidity(int *temp, int *humidity);
```

简要说明你对外暴露的函数、IOCTL 命令、sysfs 接口等。

---

## 调试与测试方法

1. 使用 `dmesg` 查看输出
2. 使用脚本或测试程序测试模块功能
3. 示例用户态程序：

```c
int fd = open("/dev/dht11", O_RDONLY);
read(fd, buf, sizeof(buf));
```

---

## 注意事项

- 驱动加载前请确保设备连接正确，GPIO 引脚匹配
- 编译前确认内核头文件路径正确

---

## TODO

- [ ] 添加中断模式支持
- [ ] 将驱动注册为 platform_driver
- [ ] 添加 sysfs 属性节点
