# BLUETOOTH 使用

## Bluetooth 启动信息

系统启动时会打印如下 Bluetooth 信息：

```
root@imx8mmevk:/# dmesg | grep -i Bluetooth
[    1.297069] Bluetooth: Core ver 2.22
[    1.304662] Bluetooth: HCI device and connection manager initialized
[    1.311046] Bluetooth: HCI socket layer initialized
[    1.315947] Bluetooth: L2CAP socket layer initialized
[    1.321031] Bluetooth: SCO socket layer initialized
[    3.370819] Bluetooth: HCI UART driver ver 2.3
[    3.375287] Bluetooth: HCI UART protocol H4 registered
[    3.380435] Bluetooth: HCI UART protocol BCSP registered
[    3.385784] Bluetooth: HCI UART protocol LL registered
[    3.390932] Bluetooth: HCI UART protocol ATH3K registered
[    3.396344] Bluetooth: HCI UART protocol Three-wire (H5) registered
[    3.402776] Bluetooth: HCI UART protocol Broadcom registered
[    5.046528] Bluetooth: RFCOMM TTY layer initialized
[    5.051424] Bluetooth: RFCOMM socket layer initialized
[    5.056589] Bluetooth: RFCOMM ver 1.11
[    5.060352] Bluetooth: BNEP (Ethernet Emulation) ver 1.3
[    5.065677] Bluetooth: BNEP filters: protocol multicast
[    5.070915] Bluetooth: BNEP socket layer initialized
[    5.075891] Bluetooth: HIDP (Human Interface Emulation) ver 1.2
[    5.081822] Bluetooth: HIDP socket layer initialized
```

## 设备识别

在系统启动完成后通过指令``hciconfig``查看 Bluetooth 连接状态。

```
root@imx8mmevk:/# hciconfig hci0 up
root@imx8mmevk:/# hciconfig
hci0:   Type: Primary  Bus: UART
        BD Address: EC:2E:98:28:95:1C  ACL MTU: 1021:8  SCO MTU: 64:1
        UP RUNNING 
        RX bytes:1451 acl:0 sco:0 events:81 errors:0
        TX bytes:1225 acl:0 sco:0 commands:81 errors:0
```

通过指令``hcitool``查看当前可用的 Bluetooth MAC 地址。

```
root@imx8mmevk:/# hcitool dev
Devices:
        hci0    EC:2E:98:28:95:1C
```

## Bluetooth 连接

通过指令``bluetoothctl``可以管理 Bluetooth 的连接操作。

```
# bluetoothctl
[bluetooth]# power on
[bluetooth]# agent off
[bluetooth]# agent on
[bluetooth]# default-agent
[bluetooth]# pairable on

Push the connect button in the device

[bluetooth]# scan on

Copy mac address

[bluetooth]# scan off
[bluetooth]# pair <mac address>

Approve pairing on Device if required

[bluetooth]# trust <mac address>
[bluetooth]# connect <mac address>
[bluetooth]# quit
```