# I2C 使用

## 前言

本文主要描述如何在该开发板上配置 I2C，配置 I2C 可分为两大步骤：

* 定义和注册 I2C 设备

* 定义和注册 I2C 驱动

下面以配置 AM1805 为例。

## 定义和注册 I2C 设备

在注册I2C设备时，需要结构体 i2c_client 来描述 I2C 设备。然而在标准Linux中，用户只需要提供相应的 I2C 设备信息，Linux就会根据所提供的信息构造 i2c_client 结构体。

用户所提供的 I2C 设备信息以节点的形式写到 dts 文件中，如下所示：

```
&i2c2 {
        clock-frequency = <400000>;
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_i2c2>;
        status = "okay";

        rtc_am1805_bridge:am1805_bridge@69{
                status = "okay";
                reg = <0x69>;
                compatible = "qcom,rtc_am1805";
                dev_name = "rtc_am1805";
                init_date = "2015/01/01";
        };
}
```

## 定义和注册 I2C 驱动

### 定义 I2C 驱动

在定义 I2C 驱动之前，用户首先要定义变量 of_device_id 和 i2c_device_id 。of_device_id 用于在驱动中调用dts文件中定义的设备信息，其定义如下所示：

```
static const struct of_device_id meson_rtc_dt_match[] = {
        { .compatible = "qcom,rtc_am1805",.data = NULL},
        {},
};

MODULE_DEVICE_TABLE(of, meson_rtc_dt_match);
```

定义变量 i2c_device_id：

```
static const struct i2c_device_id am1805_i2c_id[] = {
        { "qcom,rtc_am1805", 0 },
        { }
};

MODULE_DEVICE_TABLE(i2c, am1805_i2c_id);
```

i2c_driver 如下所示：

```
struct i2c_driver rtc_am1805_driver = {
        .driver = {
                .name = "rtc_am1805",
                .of_match_table = meson_rtc_dt_match,
                .pm     = &rtc_am1805_pm_ops,
        },
        .probe = rtc_am1805_probe,
        .remove = rtc_am1805_remove,
        .shutdown = rtc_am1805_shutdown,
        .id_table = am1805_i2c_id,
};
```

注：变量id_table指示该驱动所支持的设备。

### 注册 I2C 驱动

使用i2c_add_driver函数注册 I2C 驱动。

```
static int  __init rtc_am1805_init(void)
{
    RTC_DBG(RTC_DBG_VAL, "rtc_am1805 --rtc_am1805_init\n");
    return i2c_add_driver(&rtc_am1805_driver);
}
```

在调用 i2c_add_driver 注册 I2C 驱动时，会遍历 I2C 设备，如果该驱动支持所遍历到的设备，则会调用该驱动的 probe 函数。

### 通过 I2C 收发数据

在注册好 I2 C 驱动后，即可进行 I2C 通讯。

* 向从机发送信息

```
static int rtc_am1805_i2c_write(unsigned char *buff, __u32 addr, unsigned len)
{
        int res = 0;
        unsigned char *kBuf;
        int i;
        struct i2c_msg msg;

        if (rtc_info->pI2cClient == NULL){
                pr_err("%s i2c client is null.\n",__func__);
                return -1;
        }

        kBuf = kmalloc(len+1, GFP_KERNEL);
        memcpy(kBuf+1, buff, len);
        kBuf[0] = addr&0xFF;
        msg.addr = rtc_info->pI2cClient->addr;
        msg.flags = 0;
        msg.len = len+1;
        msg.buf = kBuf;

        pr_debug("%s:i2c write register\n",__func__);
        for (i=0;i<=len;i++){
            pr_debug("%s i2c write %02x \n",__func__,kBuf[i]);
        }

        res = i2c_transfer(rtc_info->pI2cClient->adapter, &msg, 1);
        if (res < 0)
                pr_err("%s: i2c transfer failed ,ret:%d .\n", __func__,res);
        else
                res = 0;

        kfree(kBuf);
        return res;
}
```

* 向从机读取信息

```
static int rtc_am1805_i2c_read(unsigned char *buff, u8 reg, unsigned len)
{       
        int res = 0;
        struct i2c_msg msgs[2]; 
        unsigned char ctrBuf[3] = {(reg&0xff), 0, 0};
        if (rtc_info->pI2cClient == NULL){
                pr_err("%s :i2c client is null\n",__func__);
                return -1;
        }
        
        msgs[0].addr    = rtc_info->pI2cClient->addr;
        msgs[0].flags   = 0;
        msgs[0].len     = sizeof(reg);
        msgs[0].buf     = &reg;
        msgs[1].addr    = rtc_info->pI2cClient->addr;
        msgs[1].flags   = I2C_M_RD;
        msgs[1].len     = len;
        msgs[1].buf     = buff;
        
        res = i2c_transfer(rtc_info->pI2cClient->adapter, msgs, 2);
        if (res < 0){
                pr_err("%s: i2c transfer failed(%d), addr:%x\n",
                                __func__, res, rtc_info->pI2cClient->addr);
        }else{  
                res = 0;
        }
        return res;
}

```

注:msgs[0] 是要向从机发送的信息，告诉从机主机要读取信息。msgs[1] 是主机向从机读取到的信息。

至此，主机可以使用函数 rtc_am1805_i2c_read 和 rtc_am1805_i2c_write 和从机进行通讯。