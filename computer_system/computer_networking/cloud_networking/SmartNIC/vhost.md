### dpdk librte_vhost

#### vhost-user/Qemu之间的消息解析：
***VHOST_USER_GET_FEATURES***：vhost-user收到此消息，需要将自己所支持的特性，用”|”组成u64的值，放在VhostUserMsg里面的u64回复给Qemu，这里面的features包含GSO/GRO等信息

***VHOST_USER_SET_FEATURES***：vhost-user收到此消息，消息体中带有u64的前端支持的features，前后端都支持的features生效

***VHOST_USER_SET_MEM_TABLE***: 主要做共享内存的映射
- 判断nregions是否超过最大值，目前是8片
- 如果dev->mem已经映射过，判断是否跟之前一致，若一致则关闭每个region的fd
- 如果映射过，且与之前不一致，则free掉旧的，重新映射
- 清空iotlb
- 分配dev->guest_pages在普通堆上
- 开始填充memory regions：对于每一片内存，一一对应

<img src="vhost-memory-region.png" align=center/>

因此:

```
mmap_addr = mmap(…, (size + mmap_offset), …, fd, …);
host_user_addr = mmap_addr + mmap_offset;
```

<img src="vhost-mmap.png" align=center width=500/>

如果virtqueue里面的desc/avail/used vring是空，则后面VHOST_USER_SET_VRING_ADDR会做vring地址从qva_to_vva()的转换。

如果virtqueue里面的desc/avalil/used vring非空，就说明之前初始化过，这次memory table更新，需要重新更新virtqueue。

<img src="vhost-virtqueue.png" align=center width=350/>

```
/* Converts QEMU virtual address to Vhost virtual address. */
static uint64_t qva_to_vva(struct virtio_net *dev, uint64_t qva, uint64_t *len)
{
    struct rte_vhost_mem_region *r;
    uint32_t i;
    if (unlikely(!dev || !dev->mem))
        goto out_error;
    /* Find the region where the address lives. */
    for (i = 0; i < dev->mem->nregions; i++) {
        r = &dev->mem->regions[i];
       if (qva >= r->guest_user_addr && qva < r->guest_user_addr + r->size) {
            if (unlikely(*len > r->guest_user_addr + r->size - qva))
                *len = r->guest_user_addr + r->size - qva;
            return qva - r->guest_user_addr + r->host_user_addr;
        }
    }
out_error:
    *len = 0;
    return 0;
}
```

根据上面从qemu得到的内存信息，更新virtqueue, 需要将qemu发送下来的desc vring/avail vring/used vring的地址通过qva_to_vva()转换成vhost-user看到的地址对于qva_to_vva()也比较好理解，根据qemu传下来的vring地址，和memory regions的地址，找到在哪一片region；然后在region的偏移量offset = qva - guest_user_addr;在vhost-user中看到的地址就是这一片region的起始地址host_user_addr + offset; 至此，VHOST_USER_SET_MEM_TABLE就结束了。

***VHOST_USER_SET_VRING_NUM***：队列深度，desc数量

***VHOST_USER_SET_VRING_ADDR***：将qemu空间的desc/avail/used vring的地址转换成vhost-user空间中的virtual memory

***VHOST_USER_SET_VRING_BASE***：qemu告诉vhost-user，avail vring让vhost-user开始使用的位置。index是rx queue or tx queue，num是idx

***VHOST_USER_SET_VRING_ENABLE***：enable队列，同样是使用vhost_vring_state的数据结构,index代表rx queue还是tx queue，num代表enable/disable

#### virtio-device收发报文
* virtio-device发送报文(virtio-device -> NIC):

    available ring就是由virtio-device从avail ring中读取报文描述符，把报文拷贝到mbuf中。used ring就是virtio-device在从avail ring中读取报文后，会把用完的描述符放入used ring中。
	
* virtio-device接收报文(NIC -> virtio-device):

    avail ring是virtio-device从avail ring中读取可用的描述符。used ring是virtio-device会把要发送的报文从mbuf拷贝到描述符指向的地址中，并将描述符id放入used ring[]中，virtio-net通过used ring来读取收到的报文。
