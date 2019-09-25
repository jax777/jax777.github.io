---
layout: post
title: gopacket库 路由选择操作的bug
categories: linux,go
tag: 路由,go
---
> https://github.com/google/gopacket/blob/master/routing/routing.go

## 说明
最近用到go的gopacket库自行构造poc数据包并发送，偶然发现发送至docker 网段ip的流量也被发送到了公网网卡。

## 排查linux 路由表
```shell
docker0   Link encap:Ethernet  HWaddr 02:42:e9:6c:2f:4e  
          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1535 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1824 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:88837 (88.8 KB)  TX bytes:2555107 (2.5 MB)

eth0      Link encap:Ethernet  HWaddr 00:16:3e:00:c9:6f  
          inet addr:172.31.91.12  Bcast:172.31.95.255  Mask:255.255.240.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:641466 errors:0 dropped:0 overruns:0 frame:0
          TX packets:211231 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:74829986 (74.8 MB)  TX bytes:154754150 (154.7 MB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

vethdba2c30 Link encap:Ethernet  HWaddr 8e:5f:64:fc:0d:5d  
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1535 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1824 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:110327 (110.3 KB)  TX bytes:2555107 (2.5 MB)



# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.31.95.253   0.0.0.0         UG    0      0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
172.31.80.0     0.0.0.0         255.255.240.0   U     0      0        0 eth0

#ip route 
default via 172.31.95.253 dev eth0 
172.17.0.0/16 dev docker0  proto kernel  scope link  src 172.17.0.1 
172.31.80.0/20 dev eth0  proto kernel  scope link  src 172.31.91.12
```
测试发现，正常调用系统socket操作，数据包路由正常，显然是gopacket库的路由表发生了问题。
公网网卡是默认路由，正常情况下，目的ip未在其他路由中找到时才使用默认路由。

## 排查

### out of order iface error
由于云主机里的有个网卡 vethdba2c Index 不是顺序递增了，导致出现错误。
这个错误可以直接忽略的，同时修改一下其他存储代码即可。
`routing.go`
```go
line 68:    type router struct {
line 69:    	ifaces [100]net.Interface // max interface index count 100
line 70:    	addrs  []ipAddrs
line 71:    	v4, v6 routeSlice
line 72:    }

line 214：   for i, iface := range ifaces {
line 215：   	if i != iface.Index-1 {
line 216：   		return nil, fmt.Errorf("out of order iface %d = %v", i, iface)
line 217：   	}
line 218：   	rtr.ifaces = append(rtr.ifaces, iface)
```
查看结构体
```go
type Interface

Interface represents a mapping between network interface name and index. It also represents network interface facility information.

type Interface struct {
    Index        int          // positive integer that starts at one, zero is never used
    MTU          int          // maximum transmission unit
    Name         string       // e.g., "en0", "lo0", "eth0.100"
    HardwareAddr HardwareAddr // IEEE MAC-48, EUI-48 and EUI-64 form
    Flags        Flags        // e.g., FlagUp, FlagLoopback, FlagMulticast
}
```

`routing.go`修改为
```go
line 68:    type router struct {
line 69:    	ifaces [100]net.Interface // max interface index count 100
line 70:    	addrs  []ipAddrs
line 71:    	v4, v6 routeSlice
line 72:    }

line 214:   for _, iface := range ifaces {
line 215:   	//if i != iface.Index-1 {
line 216:   	//	return nil, fmt.Errorf("out of order iface %d = %v", i, iface)
line 217:   	//}
line 218:   	rtr.ifaces[iface.Index] = iface
```

### 路由表

```go
line 132:    for _, rt := range routes {
line 133:    	 if rt.InputIface != 0 && rt.InputIface != inputIndex {
line 134:    	 	continue
line 135:    	 }
line 136:    	 if rt.Src != nil && !rt.Src.Contains(src) {
line 137:    	 	continue
line 138:    	 }
line 139:    	 if rt.Dst != nil && !rt.Dst.Contains(dst) {
line 140:    	 	continue
line 141:    	 }
line 142:    	 return int(rt.OutputIface), rt.Gateway, rt.PrefSrc, nil
```
对路由依次进行匹配,但是默认路由在第一个，导致直接匹配了默认路由，没有走到正常路由。

> https://stackoverflow.com/questions/15668653/how-to-find-the-default-networking-interface-in-linux

修改将Dst为nil的重新组织为默认路由表。先匹配正常路由，最后匹配默认路由。
![](/styles/images/2019-8/defaultroute.png)