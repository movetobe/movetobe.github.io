本文主要总结PCI/PCIe End Point设备的配置，主要从软件的角度，如何配置使用PCI设备。

参考：https://wiki.osdev.org/PCI

### 设备扫描
首先，软件要扫描所有在PCI配置空间中的设备。

PCI配置空间的起始地址为base_addr，例如QEMU模拟ARM，在hw/arm/virt.c中可以找到VIRT_PCIE_ECAM，对应的基地址是0x3f000000。

对于每个设备的配置，按一定的规则存放在PCI配置空间中。存放规则如下：

<image src="pci-device-conf-space-access.png" align=center/>

也就是说， 对于Bus:Device.Function的PCI设备，它的配置存放在地址base_addr + ((Bus << 16) | (Device << 11) | (Function << 8) | (Register Offset & 0xFCU)) 中。
> Register Offset要求是4字节对齐。

### 每个设备配置空间
PCI设备的配置空间有多种格式：
* Header Type 0x00：普通的PCI Endpoints
* Header Type 0x01：PCI-to-PCI bridge
* Header Type 0x02：PCI-to-CardBus bridge

我们这里讨论普通的PCI Endpoints，所以是按照Header Type 0x00的格式来操作每一个PCI设备的配置空间：

<image src="pci-device-conf-space-header.png" align=center/>

1. 读取Vendor ID和Device ID来确定具体的设备。
1. 读取和解析BAR(Base Address Register)空间：读取BAR的0bit，0表示通过内存映射IO（MMIO）的方式操作IO端口，1表示通过IO端口的方式操作IO端口。BAR空间分布如下：

<image src="pci-device-conf-space-bar.png" align=center/>

3.  对于QEMU模拟ARM，在hw/arm/virt.c中可以找到VIRT_PCIE_MMIO，对应的地址从0x10000000开始分配，该地址就作为设备与驱动的共享内存空间。分配的大小在设备初始化的时候注册BAR空间时决定。确定设备的地址空间后，将该地址写回BAR字段。

### PCI设备其他属性
* 设置设备的Command命令字段，如disable interrupt/enable interrupt等属性：

<image src="pci-device-conf-space-command.png" align=center/>

* 可以通过Status状态字段，来查看相关的状态信息：

<image src="pci-device-conf-space-status.png" align=center/>