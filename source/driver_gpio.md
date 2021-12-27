# GPIO 使用

## GPIO 状态

通过command可以后去系统gpio的利用状态，如下：

```
root@imx8mmevk:/# cat /sys/kernel/debug/gpio 
gpiochip0: GPIOs 0-31, parent: platform/30200000.gpio, 30200000.gpio:
 gpio-5   (                    |lt9611-irq-gpio     ) in  hi IRQ
 gpio-6   (                    |lt9611-reset-gpio   ) out hi    
 gpio-8   (                    |gpio_lte_pwr_off    ) out hi    
 gpio-9   (                    |radio on            ) out hi    
 gpio-14  (                    |PCIe CLKREQ         ) out lo    
 gpio-15  (                    |cd                  ) in  hi IRQ

gpiochip1: GPIOs 32-63, parent: platform/30210000.gpio, 30210000.gpio:
 gpio-38  (                    |?                   ) out hi    
 gpio-42  (                    |reset               ) out hi    
 gpio-51  (                    |VSD_3V3             ) out lo    

gpiochip2: GPIOs 64-95, parent: platform/30220000.gpio, 30220000.gpio:
 gpio-83  (                    |green-bt            ) out lo    

gpiochip3: GPIOs 96-127, parent: platform/30230000.gpio, 30230000.gpio:
 gpio-117 (                    |PCIe reset          ) out hi    

gpiochip4: GPIOs 128-159, parent: platform/30240000.gpio, 30240000.gpio:
 gpio-131 (                    |headphone detect    ) in  hi IRQ
 gpio-137 (                    |spi_imx             ) out hi  
```

每个GPIO被定义为输入(in)或输出(out)并显示状态为高电平(lo)或低电平(hi)。
如 pin 15 作为 SD card 的 card-detect，当 SD card 插入时，状态为：

```
gpio-15  (                    |cd                  ) in  lo IRQ

```

当 SD card 拔出时，状态为：

```
gpio-15  (                    |cd                  ) in  hi IRQ

```

## 控制单个 GPIO

i.MX 平台一个GPIO group 有 32 个 GPIO。
例如GPIO1_3属于第一个 group 的第3pin，因此它的GPIO number是3。
GPIO4_21 则是 (4-1) * 32+21=117。
假设一个 GPIO 定义在你一个 device tree中，下面演示如何通过command操作它。

将 GPIO 导入 userspace 使用：

```
$ echo 117 > /sys/class/gpio/export
```

配置成输出模式：

```
$ echo out > /sys/class/gpio/gpio117/direction
```

将 GPIO 拉高：

```
$ echo 1 > /sys/class/gpio/gpio117/value
```

将 GPIO 拉低：

```
$ echo 0 > /sys/class/gpio/gpio117/value
```

配置成输入模式：

```
$ echo in > /sys/class/gpio/gpio117/direction
```

释放 GPIO：

```
$ echo 117 > /sys/class/gpio/unexport
```

## 内核设备树 GPIO 配置

### pin 定义文件

在kernel/include下可以找到 pin 脚定义文件 pins-imx8mm.h
如果搜索 GPIO4_IO2可以看到一组与之相同前缀(MX8MM_IOMUXC_SAI1_RXD0)的宏定义。

```
#define MX8MM_IOMUXC_SAI1_RXD0_SAI1_RX_DATA0                                0x164 0x3CC 0x000 0x0 0x0
#define MX8MM_IOMUXC_SAI1_RXD0_SAI5_RX_DATA0                                0x164 0x3CC 0x4D4 0x1 0x1
#define MX8MM_IOMUXC_SAI1_RXD0_PDM_DATA0                                    0x164 0x3CC 0x534 0x3 0x1
#define MX8MM_IOMUXC_SAI1_RXD0_CORESIGHT_TRACE0                             0x164 0x3CC 0x000 0x4 0x0
#define MX8MM_IOMUXC_SAI1_RXD0_GPIO4_IO2                                    0x164 0x3CC 0x000 0x5 0x0
#define MX8MM_IOMUXC_SAI1_RXD0_CCMSRCGPCMIX_BOOT_CFG0                       0x164 0x3CC 0x000 0x6 0x0
#define MX8MM_IOMUXC_SAI1_RXD0_SIM_M_HADDR17                                0x164 0x3CC 0x000 0x7 0x0
```

根据不同的需求可以将pin脚配置成不同的定义，具体定义可以参考芯片手册。

### 在设备树中配置GPIO

编译 arch/arm64/boot/dts/freescale/fsl-imx8mm-evk.dts，在 iomuxc node 中增加需要的 GPIO pin 脚配置，如下：

```
&iomuxc {
    pinctrl-names = "default";
    pinctrl-0 = <&pinctrl_tas5825_pdn>;

    imx8mm-evk  {
        pinctrl_tas5825_pdn: tas5825_pdn {
            fsl,pins = <
                /* Add your GPIO definitions here */ 
            >;
        };
    };
…
};
```