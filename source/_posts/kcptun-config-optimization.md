---
title: kcptun config optimization
date: 2019-12-13 16:47:54
tags:
---

## 合理配置 kcptun 进行高效的加速

##### 需调整参数列表
- mtu
- sndwnd、rcvwnd
- datashard、parityshard

##### mtu

这个可以直接用默认的，也可以自己计算。

假设你的服务器 ip 为 1.1.1.1，那么在终端中输入
```bash
ping -l 1472 1.1.1.1
```

如果服务器正常响应，那么 mtu 就是数据包大小 + 28 字节，这里就是 1472+28=1500

如果提示说
```
请求超时。
或者
Packet needs to be fragmented but DF set.
```
说明 1472 大了。调小一些，再进行计算即可。一般默认或 1400 基本都无问题。

##### sndwnd、rcvwnd

服务端的 rcvwnd 要和客户端的 sndwnd 最好一致。256、512 皆可。

服务端的 sndwnd 要和客户端的 rcvwnd 最好一致。
```
近似计算：
    50M  带宽：都设置为 1024
    100M 带宽：都设置为 2048
    200M 带宽：都设置为 4096
```
然后再慢慢自己调。

##### datashard、parityshard

datashard 和 parityshard 是拯救丢包严重的线路用的。大致原理是多发包，丢了包靠多发的包纠错。

计算公式如下：（datashard + parityshard）/ datashard * 之前消耗的带宽 = 纠错后需消耗的带宽

高峰丢包时，先测试丢包率，得到丢包率 nn%。
```
server:
    yum install iperf
    iperf -s -u

client: 多测几次
    iperf -c serverIP -u -b 10M -t 60
```
kcptun 的日志，在执行 `kill -SIGUSR1 pid` 后会有 SNMP 的信息。
需要观察 SNMP 确保 FECRecovered 是否真的接近 FECSegs， 如果是，一定会有质的飞跃。你看到的 ping 丢包率，不一定是实际丢包率，观察 SNMP 是最准确的。


为了发挥 FEC 最佳效果，设置 parityshard / （parity + datashard） > packet loss 比如 5/(5+5) > 30%。

##### dscp
- EF 无阻碍转发（Expedited Forwarding,EF）由 RFC2598 定义，DSCP 值为 46(101110)。EF 服务适用于低丢包率，低延迟，低抖动及保证带宽的业务，如 VOIP。

