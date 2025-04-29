# FT6336G Touchscreen Driver on Orange Pi Zero3

> 本项目记录在 Orange Pi Zero3（H616/H618）上移植 FocalTech FT6336G 电容触摸屏的全过程，包括驱动编译、设备树 overlay 配置、中断调试和 evtest 验证等。
我并没有编写 ft6336的驱动，因为厂商提供了代码，是focaltech_touch。在内核源码里存在野火内核的/drivers/input/touchscreen/focaltech_touch 下 
直接复制出来，修改makefile 然后编译 编译出 focaltech_touch.ko 然后scp到香橙派里。用orangepi-add-overlay ft6336g.dtb 来配置设备树文件，这就可以使用了。
遇见了很多问题，都是设备树语法的问题，因为懒得每次都编译设备树，orangpi有工具可以本地编译，所以使用 这个工具来编译 但是它无法识别 GPIO_ACTIVE_HIGH 类似这种的
宏定义，偷懒就改成了1 。这个内核代码是没问题的。
---

## 📦 项目内容概览

- 适配驱动：`focaltech_core.c`
- 硬件接口：I2C3，GPIO 中断引脚 PC9，RESET 引脚 PC5
- 屏幕分辨率：240x320
- 驱动模式：独立模块编译加载（.ko）

---

## 🔧 1. 设备树 Overlay 配置

创建 `ft6336g-overlay.dts` 内容如下：

```dts
/dts-v1/;
/plugin/;



/ {
    compatible = "xunlong,orangepi-zero3", "allwinner,sun50i-h616";

    fragment@0 {
        target = <&i2c3>;
        __overlay__ {
            status = "okay";

            ft6336g: touchscreen@38 {
                compatible = "focaltech,fts";
                reg = <0x38>;

                interrupt-parent = <&pio>;
                interrupts = <2 9 2>; // PC9 IRQ_TYPE_EDGE_FALLING
                focaltech,irq-gpio = <&pio 2 9 1>;GPIO_ACTIVE_HIGH
                focaltech,reset-gpio = <&pio 2 5 1>; // PC5 GPIO_ACTIVE_HIGH

                focaltech,max-touch-number = <2>;
                touchscreen-size-x = <240>;
                touchscreen-size-y = <320>;
            };
        };
    };
};
```
如果使用 
### 编译与加载

```bash
sudo orangepi-add-overlay ft6336g.dtb
```

---

## 🧰 2. Makefile 编写（独立模块编译）

在驱动目录新建 Makefile：

```makefile
KERNEL_DIR ?= /your/kernel/path

obj-m := focaltech_core.o

all:
	make -C $(KERNEL_DIR) M=$(PWD) modules

clean:
	make -C $(KERNEL_DIR) M=$(PWD) clean
```

> 请将 `KERNEL_DIR` 改为你的内核源码路径，如 `/home/xyz/orangepi-build/kernel/orange-pi-6.1-sun50iw9`

### 编译 & 加载

```bash
make
sudo insmod focaltech_core.ko
```

---

## 🧪 3. evtest 坐标验证

```bash
sudo evtest
```

输出示例：

```text
Event code 53 (ABS_MT_POSITION_X): value 11, range 0–239
Event code 54 (ABS_MT_POSITION_Y): value 33, range 0–319
Event code 57 (ABS_MT_TRACKING_ID): value 40
Event code 330 (BTN_TOUCH): value 1
```

说明驱动已成功注册 `/dev/input/eventX`，并支持多点触控。

---

## 🧠 4. 常见问题排查

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| `invalid GPIO -2` | DTS 中未定义 `focaltech,reset-gpio` | 添加 `reset-gpio` |
| 无事件节点 | `compatible` 未匹配驱动 | 改为 `"focaltech,fts"` |
| probe 失败 | I2C 地址错或驱动未加载 | 检查 `i2cdetect` 和 `dmesg` |

---

## 📌 5. 常用命令速查

```bash
# 编译设备树
dtc -@ -I dts -O dtb -o ft6336g.dtbo ft6336g-overlay.dts

# 加载驱动
modprobe focaltech_touch

# 查看事件节点
ls /dev/input
sudo evtest

# 查看中断
cat /proc/interrupts
```

---

> 项目作者：@你的昵称 ｜ 技术交流欢迎留言讨论