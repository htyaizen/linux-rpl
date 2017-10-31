作者使用linux和contiki以完成beaglebones、树莓派与contiki系统的传感器之间的通信。开始使用SimpleRPL，但是没有处理NUD，所以启用了Linux RPL的支持。清理代码到linux内核的最佳实践，联系内核网络维护者，以了解谁在内核中获得RPL。

作者在linux中使用了6Llowpan堆栈的RPL，但是在beaglebone和rpi官方内核中有一些缺失的修复程序，需要从这个库中合并大量补丁，以使所有工作都符合项目要求。

Linux实现6lowpan堆栈和contiki实现效果相当。请注意，系统集成了很多错误和不当行为，但是由于linux中并没有原生实现任何RPL，也没将6lowpan与linux集成在一起，所以任何工作都是一个很好的起点。

作者的rpl实现支持使用CORE仿真器测试它的命名空间。RPL内核实现监视NUD并且如果需要的话。预计允许多个DODAG。它还提供了一个OF API，以便将来实现。现在只执行了0OF。rpl内核实现实现了Linux引导触发器支持，以允许绑定beaglebones和rpi以“加入任何DAG”反馈。

要在linux中与RPL进行交互，有一个用户界面工具“rpl-ctl”，它是基于“iz”的优秀工具。使用该工具，用户可以设置RPL功能，列出RPL的内部状态并触发维修。它还没有实现，但最好能根据RPL中发生的情况来实现用户界面事件触发。

如果有人想尝试一下，内核补丁可以在这里找到：https：//github.com/joaopedrotaveira/linux-rpl

由于作者一直在使用几种版本的内核到RPi和BBW / B的几个功能，所以一直难以把RPL和项目所需的其他功能保存到单个存储库。
#linux-rpl

##**RPL: IPv6 Routing Protocol for Low-Power and Lossy Networks for Linux**

### rpl-userspace-tools
用户空间工具用来设置管理RPL配置

https://github.com/joaopedrotaveira/rpl-userspace-tools

### rpl-kernel
Kernel patches

* Mainline
	* v3.10
	* v3.11
	* v3.12
* raspberrypi 3.10.y

https://github.com/joaopedrotaveira/linux-rpl

## Using RPL

### Setting up root
```
# root模式下，内核会使用iface地址当做DODAG的ID
$ sudo ip addr add 2001:aaaa:beef:c0fe::1/64 dev lowpan0

# 设置iface为DODAG的root
$ sudo sysctl -w net.ipv6.conf.lowpan0.rpl_dodag_root=1

# 在iface上使能RPL
$ sudo sysctl -w net.ipv6.conf.lowpan0.rpl_enabled=1
```

### Setting up RPL router
```
# enable without rpl-userspace-tools
$ sudo sysctl -w net.ipv6.conf.lowpan0.rpl_enabled=1
```
```
# enable with rpl-userspace-tools
$ sudo rpl-ctl enable lowpan0
```

### Disable IPv6 Privacy Extensions
RPL module announces all global IPv6 addresses of RPL enabled iface. It might not be desired to announce 2 or 3 addresses of each node to DAG.RPL模式通告所有iface使能RPL的IPv6地址，

To disable IPv6 Privacy Extensions:
```
$ sudo sysctl -w net.ipv6.conf.all.use_tempaddr = 0
$ sudo sysctl -w net.ipv6.conf.default.use_tempaddr = 0
```
