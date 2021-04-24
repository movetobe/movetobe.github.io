在QEMU上创建一个虚拟设备的时候，通常需要考虑设备产生中断的方式。当前PCI设备产生中断的方式有传统的PIN脚电平触发中断，还有消息触发的MSI(Message Signaled Interrupts)中断。

本文主要讨论QEMU模拟ARM Cortex-A53环境下，PCI设备电平触发中断，后面有机会再补充MSI/MSIX中断。

在QEMU上创建的PCI设备，通常都挂载在GPEX（Generic PCI Express Bridge）上，传统的中断方式，中断都是通过GPEX上传到GIC（Generic Interrupt Controller），然后到达CPU。这种中断方式是一种共享的中断方式，有中断数目受限、中断处理复杂等缺点。

我们这里想要讨论的是，PCI设备中断触发流程，以及中断到达软件后，中断号是多少的问题。

### PCI设备中断触发后到达CPU的中断号
首先，是QEMU的GPIO（General Purpose Input/Output）模型如下：

Device.[GPIO_OUT] -> [GPIO_IN].GIC.[GPIO_OUT] -> [GPIO_IN].CPU

GIC和CPU会分别创建GPIO_IN，Device和GIC的GPIO_OUT不需要创建，只需要与对应的GPIO_IN做关联

创建设备的时候，要将设备的GPIO_OUT与GIC关联：

<image src="pci-device-gpio-gic.png" align=center/>

4个引脚（#A/#B/#C/#D）对应的中断是irqmap[VIRT_PCIE]，对应到hw/arm/virt.c中的a15irqmap[VIRT_PCIE]的中断范围是3->6。

GIC初始化的时候需要创建GPIO_IN和将GPIO_OUT与CPU做关联:

<image src="gic-gpio-cpu.png" align=center/>

此时，需要将刚才得出的3->6的中断号转换到全局中断号，全局1->32是CPU私有中断(SPIs和PPIs，所以PCIE的全局中断范围为35->38。

### PCI设备中断触发
PCI设备的触发流程，在QEMU中比较成熟，只需要调用qemu_irq_raise()既可以触发中断。

<image src="pci-device-interrupt.png" align=center/>

GIC在内核初始化的时候会相应的进行设置，同样在hw/arm/virt.c中，QEMU模拟ARM的GIC Distributor的基地址是0x08000000，GIC CPU的基地址是0x08010000。所以在PCI设备触发中断时，中断一路从gpex送到gic，然后更新GIC->irq_state[irq]的状态，然后一路再调到cpu的generic_handle_interrupt()，设置中断掩码。

CPU在运行中，会检查是否有中断需要处理，根据中断处理的类型，触发CPU的Exception，跳转到Exception Vector Irq Addr去处理，从而进入软件处理流程。