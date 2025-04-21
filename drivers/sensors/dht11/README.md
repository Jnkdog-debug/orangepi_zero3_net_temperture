# DHT11 温湿度驱动

## 📌 简介

该驱动是香橙派zero3的驱动


## 📎 接线说明
- Data 引脚接 PC6
- 电源 3.3V / GND


## DHT11 驱动说明
- 驱动类型：Platform Driver  **字符设备驱动**
- 驱动协议：串行通信（单线串行口）
- 支持的功能：温湿度读取、中断支持、定时读取、上报 input_event 
- 注册方式（内核模块 / 静态编译）


## DHT11 协议说明

### 📦 数据格式

DHT11 通过单总线发送 40 位数据，格式如下：

```
[8bit 湿度整数] + [8bit 湿度小数] + [8bit 温度整数] + [8bit 温度小数] + [8bit 校验]
```

- 湿度/温度的小数部分通常为 0  
- 校验位 = 前四段数据相加的低 8 位

---

### 🔌 通讯流程

1. **初始化等待**
   - DHT11 上电后需等待 1 秒稳定，期间不可发送指令。

2. **启动信号**
   - MCU 设置 DATA 为输出并拉低 ≥18ms，然后释放并切换为输入，等待响应。

3. **应答信号**
   - DHT11 检测到低电平后，发出：
     - 83μs 低电平（响应）
     - 87μs 高电平（准备发送数据）

4. **发送数据**
   - 共 40 位数据
   - 每个位由 54μs 低电平开始：
     - 高电平 ~26μs 表示“0”
     - 高电平 ~70μs 表示“1”

5. **结束信号**
   - 数据传输后，DHT11 发送 54μs 低电平并进入待机状态，准备下一次通讯。

---

## 📂 文件说明

| 文件名             | 作用                      |
|--------------------|---------------------------|
| `dht11_driver.c`   | 主驱动源码                |
| `dht11_driver.h`   | 头文件，结构体、函数声明等 |
| `Makefile`         | 编译用的文件               |
| `test_user.c`      | 用户空间测试用例（可选）   |
| `dht11.dts`        | 设备树插件 需要放到overlays文件夹里编译    |   

## 驱动来源
   野火 内核源码 dht11驱动
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
   
  模仿led 添加 
  		dht11: dht11 {
		compatible = "dht11_pin";
			label = "dht11_pin";  //这里的详细定义我还并不清楚，但是在
			gpios = <&pio 2 6 GPIO_ACTIVE_HIGH>; /* PC12 */		
		};
1. 回kernel目录  make dtbs
2. 产生的dtb文件 scp到orangepi 的 /boot/dtb/allwinner 下
3. reboot 


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

