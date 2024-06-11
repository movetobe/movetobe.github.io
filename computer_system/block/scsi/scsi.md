SCSI (Small Computer System Interface)

SCSI总线的寻址方式，按照 控制器 -> 通道 -> SCSI ID -> LUN ID 来寻址。
> * LUN, Logic Unit Number
> * 通道，即SCSI总线
> * SCSI ID，即设备ID

SCSI简要数据的读写过程：

写入数据，控制器（initiator）向磁盘设备（target）发送数据。
1. 发起方在获得总线仲裁（在16条总线中竞争取胜）之后，会发送一个SCSI Command写命令帧，其中包含对应的LUN号以及LBA地址段。
1. 接收端接收后，向发起方回送一个XFER_RDY帧，表示已经准备好接收数据了。
1. 发起方收到XFER_RDY帧，开始发送数据，每发送一帧数据，接收方都会应答一个XFER_RDY帧。如此反复直到数据发送完成。
1. 当接收方执行完命令后，再发送一个RESPONSE帧，表示写入结束。

读取数据，控制器（initiator）从磁盘设备（target）获取数据。
1. 发起方在获得总线仲裁（在16条总线中竞争取胜）之后，会发送一个SCSI Command读命令帧。
1. 接收端接收后，按LUN号以及LBA地址段对应的所有扇区数据读出，传回给发起方。 
1. 所有数据传输完成后，目标端发送一个RESPONSE帧，表示这条SCSI命令执行完毕。

