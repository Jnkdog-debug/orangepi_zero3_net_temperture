# FT6336G Touchscreen Driver on Orange Pi Zero3

> 本项目记录在 Orange Pi Zero3（H616/H618）上移植 FocalTech FT6336G 电容触摸屏的全过程，包括驱动编译、设备树 overlay 配置、中断调试和 evtest 验证等。
我并没有编写 ft6336的驱动，因为厂商提供了代码，是focaltech_touch。在内核源码里存在野火内核的/drivers/input/touchscreen/focaltech_touch 下 
直接复制出来，修改makefile 然后编译出 focaltech_touch.ko 然后scp到香橙派里。用sudo orangepi-add-overlay ft6336g.dtb 来配置设备树文件，这就可以使用了。
遇见了很多问题，都是设备树语法的问题，通过查阅和问chatgpt写出了设备树插件代码，因为懒得每次都编译设备树，orangepi有工具可以本地编译，所以使用 这个工具来编译 但是它无法识别 GPIO_ACTIVE_HIGH 类似这种的宏定义，偷懒就改成了1 。这个内核代码是没问题的。
---

## 📦 项目内容概览

- 适配驱动：`focaltech_core.c`
- 硬件接口：I2C3，GPIO 中断引脚 PC9，RESET 引脚 PC5
- 屏幕分辨率：240x320
- 驱动模式：独立模块编译加载（.ko）

---

## 🔧 1. 设备树 Overlay 配置
注意，像这种使用内核源码的 需要自己编写设备树文件的项目，最好直接阅读probe里的代码，
按照代码来编写设备树文件。要保证设备树文件和驱动文件可以匹配的上。因为说白了，设备树就是要被
驱动访问的，是把硬件信息储存在外部的，如果找不到设备树节点，那么驱动就无法得到硬件值，所以就无法正常运行。
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

### 编译与加载
在香橙派里：
```bash
sudo orangepi-add-overlay ft6336g.dtb
```
---

## 🧰 2. Makefile 编写（独立模块编译）
这里建议从内核源码里复制出来
在驱动目录修改 Makefile：

```makefile
# Makefile for single out-of-tree build of focaltech_touch

# 生成 focaltech_touch.ko
obj-m += focaltech_touch.o

# focaltech_touch 模块包含哪些子文件
focaltech_touch-objs := \
    focaltech_core.o \
    focaltech_esdcheck.o \
    focaltech_ex_mode.o \
    focaltech_gesture.o \
    focaltech_point_report_check.o \
    focaltech_i2c.o \
    focaltech_sensor.o

```



### 修改 focaltech 驱动 & 编译  &加载
我使用的是6.1的内核 编译源码时会 报错，所以要修改：

在 Linux 5.18 之后，内核开发者（Greg Kroah-Hartman）改了很多老旧接口：
项目	旧的写法（5.17以前）	新的写法（5.18以后，比如你的6.1）
i2c_driver 的 remove 函数	int (*remove)(struct i2c_client *)	void (*remove)(struct i2c_client *)

也就是说：

    以前 remove() 要返回一个 int

    现在 remove() 什么都不用返回了，直接 void 就行了！

这是为了统一内核风格、简化驱动逻辑。


找到你的 focaltech_core.c 里，
定义 fts_ts_remove 的地方，比如类似：

static int fts_ts_remove(struct i2c_client *client)
{
    ...
    return 0;
}

改成这样：

static void fts_ts_remove(struct i2c_client *client)
{
    ...
    // 注意！不要 return 0；直接 return; 或者什么都不写
}

✅ 改 int → void
✅ 删除 return 0; 结尾！

然后保存、重新 make：
```bash
make -C ~/orangepi/orangepi-build/kernel/orange-pi-6.1-sun50iw9/ M=$(pwd) modules
```
加载驱动
```bash
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
## 接线
使用 i2c3
rst 接
