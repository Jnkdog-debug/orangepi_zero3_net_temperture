# ili9341屏幕驱动

## 📌 简介

该驱动是香橙派zero3的驱动

## 需求与需求分析

- 该驱动应该可以驱动这块ili9341的屏幕
- 可以显示图片
- 可以使用framebuffer显示命令行
- 使用设备树或者设备树插件
- 后续可以显示qt组件

## 开发方法
1. 伪代码
2. 开发循环 ： 有可执行的demo ，然后逐步迭代
3. 借助开源项目

## 📎 接线说明


- Data 引脚接 PC6
- 电源 3.3V / GND


## ili9341 驱动说明
- 驱动类型：spi Driver  **字符设备驱动**
- 驱动协议：串行通信（单线串行口）
- 支持的功能：温湿度读取、中断支持、定时读取、上报 input_event 
- 注册方式（内核模块 / 静态编译）


## spi 协议说明
   详细查询spi通信方式




---





## 📂 文件说明

| 文件名             | 作用                      |
|--------------------|---------------------------|
| `ili9341_driver.c`   | 主驱动源码                |
| `ili9341_driver.h`   | 头文件，结构体、函数声明等 |
| `Makefile`         | 编译用的文件               |
| `test_user.c`      | 用户空间测试用例（可选）   |
| `dht11.dts`        | 设备树插件 需要放到overlays文件夹里编译    |   

## 驱动来源
   野火 内核源码 驱动
       设备树源码


## 🧪 编译方法

### 驱动编译：

```bash
make
```

### 设备树编译 
1. 修改
    /home/xyz/orangepi/orangepi-build/kernel/orange-pi-6.1-sun50iw9/arch/arm64/boot/dts/allwinner/sun50i-h618-orangepi-zero3.dts
   文件
**未知如何修改 todo**

2. 回kernel目录  make dtbs
3. 产生的dtb文件 scp到orangepi 的 /boot/dtb/allwinner 下
4. reboot 





/********************问题非常大用设备树插件，不知道为什么无法使用************************/
### 设备树插件编译：

- 复制设备树插件
   把 dht11.dts 放到 
  /home/xyz/orangepi/orangepi-build/kernel/orange-pi-6.1-sun50iw9/arch/arm64/boot/dts/allwinner/overlay/ 
  文件夹下
- 修改makefile文件
   修改allwinner 的makefile 文件：
         sun50i-h616-pi-uart3.dtbo \
         sun50i-h616-pi-uart4.dtbo  \
   在这一行后添加：
         dht11.dtbo
- 编译
   有关设备树插件的编译，请参考 [build_guide.md](/docs/build_guide.md)。



- 使用
   1. 将编译出的dht11.dtbo文件放到开发板的 /boot/dtb/allwinner/overlay/下
   2. 修改dht11 名字 成sun50i-h616-dht11.dtbo 否则无法正常使用
   3. 编辑 /boot/orangepiEnv.txt 添加：

      overlays=dht11 //前期博客有误导，不添加前缀，直接添加设备树节点也存在 注意，等号之间不能有空格，否则加载不上。

   4. 重启开发板，ls /proc/device-tree/ | grep dht11 检查是否存在 dht11设备。
   5. 验证设备是否正常 
      ls /proc/device-tree/dht11@0/
      cat /proc/device-tree/dht11@0/compatible 

