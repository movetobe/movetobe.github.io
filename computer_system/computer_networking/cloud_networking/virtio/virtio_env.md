OVS+DPDK的VIRTIO后端进行网络端口管理，Libvirt+Qemu+Kvm进行虚拟机管理。
分几步进行搭建：

* [宿主机(ubuntu20.04)，直接安装ovs+dpdk，安装libvirt+qemu](#install_virt_env)

* [配置vm-vhostuser-ovsdpdkbr-vhostuser-vm环境](#config_vhost)

* [编译、安装ovs+dpdk](#build_virt_env)

* [搭建环境中碰到的问题](#problems)

> ### <span id = "install_virt_env"> 宿主机(ubuntu20.04)，直接安装ovs+dpdk，安装libvirt+qemu</span>

#### 1. 安装ovs+dpdk
```
#apt install dpdk
#apt install openvswitch-switch
#apt install openvswitch-switch-dpdk
```
#### 2. 安装libvirt+qemu环境
```
#apt install virt-manager
#apt install libvirt-clients
#apt install qemu
#apt install qemu-kvm
#apt install qemu-system
```


> ### <span id = "config_vhost"> 配置vm-vhostuser-ovsdpdkbr-vhostuser-vm环境</span>

#### 1. 配置大页(默认是2048KB,即2M)

配置大页数量有两种方式：
* 永久性配置，将大页配置(大页数量)写入配置文件中，机器重启自动配置
```
#echo 'vm.nr_hugepages=4096' > /etc/sysctl.d/hugepages.conf
```
* 临时性配置，机器重启后配置丢失，需要重新配置
```
#sysctl -w vm.nr_hugepages=4096
```

挂载大页
```
#mount -t hugetlbfs none /dev/hugepages
```

检查/查看大页相关信息
```
#cat /proc/meminfo |grep Huge
```

如果想配置1G大页，按照如下几个步骤配置：
* 在内核启动项，/boot/grub/grub.cfg中添加，需要重启宿主机
```
default_hugepagesz=1GB hugepagesz=1GB hugepages=4
```
* 挂载大页
```
# mount -t hugetlbfs -o pagesize=1G none /dev/hugepages
```

#### 2. 启动ovs+dpdk

检查ovs版本：
```
#ovs-vsctl show 或者
#ovs-vsctl -V
```

配置ovs使用用户态数据路径(dpdk)
```
#ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-init=true
#ovs-vsctl get Open_vSwitch . dpdk_initialized
```

重启ovs服务
```
#service openvswitch-switch restart
```

运行时很多应用的日志都在/var/log/中，所有ovs和libvirt的日志也在/var/log中，如ovs日志：
```
#/var/log/openvswitch/ovs-vswitch.log
```

ovs中添加用户态网桥和vhostuser端口，这里采用的dpdkvhostuserclient类型的端口：
```
#ovs-vsctl add-br ovsdpdkbr0 -- set bridge datapath_type=netdev
#ovs-vsctl add-port br0 vhost-user-1 -- set Interface vhost-user-1 type=dpdkvhostuserclient options:vhost-server-path=/tmp/vhost-user-1
#ovs-vsctl add-port br0 vhost-user-2 -- set Interface vhost-user-2 type=dpdkvhostuserclient options:vhost-server-path=/tmp/vhost-user-2
#ovs-vsctl show
```

#### 3. 配置vm的xml文件(centos-vm1.xml)
```
<domain type='kvm'>
    <name>centos8_vm1</name>
    <uuid>4a9b3f53-fa2a-47f3-a757-dd87720d9d1d</uuid>
    <memory unit='KiB'>2097152</memory>
    <currentMemory unit='KiB'>2097152</currentMemory>
    <memoryBacking>
        <hugepages>
            <page size='2' unit='M' nodeset='0'/>
        </hugepages>
    </memoryBacking>
    <vcpu placement='static'>1</vcpu>
    <cputune>
        <shares>2048</shares>
        <vcpupin vcpu='0' cpuset='0'/>
        <emulatorpin cpuset='0'/>
    </cputune>
    <os>
        <type arch='x86_64' machine='pc'>hvm</type>
        <boot dev='hd'/>
    </os>
    <features>
        <acpi/>
        <apic/>
    </features>
    <cpu mode='host-model'>
        <model fallback='allow'/>
        <numa>
            <cell id='0' cpus='0' memory='2097152' unit='KiB' memAccess='shared'/>
        </numa>
    </cpu>
    <on_poweroff>destroy</on_poweroff>
    <on_reboot>restart</on_reboot>
    <on_crash>destroy</on_crash>
    <devices>
        <emulator>/usr/bin/qemu-system-x86_64</emulator>
        <disk type='file' device='disk'>
            <driver name='qemu' type='qcow2' cache='none'/>
            <source file='/home/wenjian/vm/centos-vm1.qcow2'/>
            <target dev='vda' bus='virtio'/>
        </disk>
        <interface type='bridge'>    #虚拟机网络连接方式 
            <mac address='52:54:00:78:f9:6a'/>
            <source bridge='br0'/>
            <target dev='vnet27'/>
            <model type='virtio'/>
            <alias name='net0'/>
        </interface>

        <interface type='vhostuser'>
            <mac address='00:00:00:00:00:01'/>
            <source type='unix' path='/tmp/vhost-user-client-1' mode='server'/>
            <model type='virtio'/>
            <driver queues='2'>
                <host mrg_rxbuf='on'/>
            </driver>
        </interface>
        <serial type='pty'>
            <target port='0'/>
        </serial>
        <console type='pty'>
            <target type='serial' port='0'/>
        </console>
        <graphics type='vnc' port='5915' autoport='yes' listen='0.0.0.0'>
            <listen type='address' address='0.0.0.0'/>
        </graphics>
    </devices>
</domain>
```

然后使用virsh管理vm
```
virsh create centos_vm1.xml
```

可以通过virsh串口进入vm
```
virsh console centos_vm1
```

也可以通过vnc登录，安装使用[VNC Viewer](https://www.realvnc.com/en/connect/download/viewer/)



> ### <span id = "build_virt_env"> 编译、安装ovs+dpdk</span>

> ### <span id = "problems"> 搭建环境中碰到的问题 </span>

#### 1. 镜像下载(centos)
* [中国科大镜像](http://mirrors.ustc.edu.cn/centos-cloud/centos/)
* 用户名和密码
    - 安装libguestfs-tools
    ```
    #apt install libguestfs-tools
    ```
    - 使用工具virt-customize对镜像进行自定义
    ```
    #virt-customize -a centos-vm1.qcow2 --root-password password:your-password
    ```

#### 2. 
