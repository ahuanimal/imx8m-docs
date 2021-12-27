# LED 使用

## 以设备的方式控制 LED

标准的 Linux 专门为 LED 设备定义了 LED 子系统，用户可以通过 ``/sys/class/leds/`` 目录控制这两个 LED。更详细的说明请参考 【leds-class.txt](https://www.kernel.org/doc/Documentation/leds/leds-class.txt)。

用户可以通过 echo 向其 trigger 属性输入命令控制每一个 LED：

```
root@imx8mmevk:/# echo default-on > /sys/class/leds/green-bt/trigger 
root@imx8mmevk:/# echo heartbeat > /sys/class/leds/green-bt/trigger
```

用户还可以使用 cat 命令获取 trigger 的可用值：

```
root@imx8mmevk:/# cat /sys/class/leds/green-bt/trigger                 
none bluetooth-power kbd-scrolllock kbd-numlock kbd-capslock kbd-kanalock kbd-shiftlock kbd-altgrlock kbd-ctrllock kbd-altlock kbd-shiftllock kbd-shiftrlock kbd-ctrlllock kbd-ctrlrlock mmc0 mmc1 mmc2 [heartbeat] cpu cpu0 cpu1 cpu2 cpu3 default-on ethlink 30be0000.ethernet-1:00:1Gbps 30be0000.ethernet-1:00:100Mbps 30be0000.ethernet-1:00:10Mbps hci0-power 
```

## 在内核中操作 LED

在内核中操作 LED 的步骤如下：

1、在 dts 文件中定义 LED 节点“leds” 在 kernel/arch/arm/boot/dts/freescale/fsl-imx8mm-evk.dts 文件中定义LED节点，具体定义如下：

```
        leds {
                compatible = "gpio-leds";
                pinctrl-names = "default";
                pinctrl-0 = <&pinctrl_gpio_led>;
                status = "okay";
                status {
                        label = "status";
                        gpios = <&gpio1 6 0>;
                        default-state = "off";
                        linux,default-trigger = "heartbeat";
                };

                bt_led {
                        label = "green-bt";
                        gpios = <&gpio3 19 0>; /* bt */
                        linux,default-trigger = "hci0-power";
                        default-state = "off";
                };
        };

```

注意：compatible 的值要跟 drivers/leds/leds-gpio.c 中的 .compatible 的值要保持一致。

2、在驱动文件包含头文件

```
#include <linux/leds.h>
```

3、在驱动文件中控制 LED。

（1）定义 LED 触发器

```
DEFINE_LED_TRIGGER(ledtrig_ir_click);
```

（2）注册该触发器

```
led_trigger_register_simple("ir-power-click", &ledtrig_ir_click);
```

（3）控制 LED 的亮灭。

```
led_trigger_event(ledtrig_ir_click, LED_FULL); //亮 
led_trigger_event(ledtrig_ir_click, LED_OFF); //灭
```