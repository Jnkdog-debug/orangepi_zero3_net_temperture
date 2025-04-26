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
   详细请查询spi通信方式

---

## 📂 文件说明

| 文件名             | 作用                      |
|--------------------|---------------------------|
| `Makefile`         | 编译用的文件               |
| `test_user.c`      | 用户空间测试用例（可选）   |
| `dht11.dts`        | 设备树插件 需要放到overlays文件夹里编译    |   

## 驱动来源
 内核驱动


## 🧪 编译方法

### 驱动编译：

```bash
make
```

## 由于内核存在fb_ili9341的代码
   实际上直接使用就可以
### 使用方法：
   1. 首先查询是否内核是否存在fb_ili9341的驱动
   用命令查看：

   orangepi@orangepizero3:~$ ls /lib/modules/6.1.31-sun50iw9/kernel/drivers/staging/fbtft/
         fb_agm1264k-fl.ko  fb_ili9320.ko  fb_ra8875.ko   fb_ssd1306.ko  fb_tinylcd.ko
         fb_bd663474.ko     fb_ili9325.ko  fb_s6d02a1.ko  fb_ssd1325.ko  fb_tls8204.ko
         fb_hx8340bn.ko     fb_ili9340.ko  fb_s6d1121.ko  fb_ssd1331.ko  fb_uc1611.ko
         fb_hx8347d.ko      fb_ili9341.ko  fb_seps525.ko  fb_ssd1351.ko  fb_uc1701.ko
         fb_hx8353d.ko      fb_ili9481.ko  fb_sh1106.ko   fb_st7735r.ko  fb_upd161704.ko
         fb_hx8357d.ko      fb_ili9486.ko  fb_ssd1289.ko  fb_st7789v.ko
         fb_ili9163.ko      fb_pcd8544.ko  fb_ssd1305.ko  fbtft.ko



2. 加载驱动
   sudo modprobe fb_ili9341

3. 添加设备树文件并重启生效
      1. 新建一个设备树插件文件：
      nano ili9341.dts
      文件内容见[ili9341.dts](./ili9341.dts) 
      
      2. 用命令添加 ili9341的设备树插件文件：
      sudo orangepi-add-overlay ili9341.dts 


      3. 重启使它生效：
      sudo reboot
      观察 /boot/orangepiEnv.txt 文件可以发现，已经添加了设备树驱动：
      verbosity=1
      bootlogo=false
      console=both
      disp_mode=1920x1080p60
      overlay_prefix=sun50i-h616
      rootdev=UUID=2a44ed57-6df2-44b4-9d51-99000c47e55b
      rootfstype=ext4
      **user_overlays=ili9341**
      实际上在设备树overlay文件夹里添加.dtbo文件也是一样的，只不过需要编译和加载
      详细过程和语法见dht11的编译。