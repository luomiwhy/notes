# Hadoop原理机制

## 名词说明

* namespace
* namenode
* datanode

## 写数据流程

1. 客户端将文件分块，hadoop2是128MB为一个块
2. 客户端请求上传文件
3. namenode检测文件系统目录树
4. 通知客户端可以上传文件
5. 客户端请求上传blk-1  备份数3
6. namenode检测datanode信息池，找到可用的3个datanode ip（dn1，dn2，dn3），策略一般是本地一个，同机架另一个dn，不同机架的一个dn
7. 返回可用dn（dn1，dn2，dn3），返回地址按网络拓扑上的距离来排序，最近的在最前面
8. 客户端请求数据传输，与dn1建立pipline，dn1与dn2建立pipline，dn2与dn3建立pipeline
9. dn3，dn2，dn1，客户端，逐级通知pipline建立完毕
10. 建立数据传输的stream，以packet\(64KB\)为单位发送数据
11. dn1接收后，保存传递过来的源源不断的packet，dn2，dn3 同样
12. 逐级返回数据校验，数据保存成功，直到返回到客户端
13. 开始传输blk-2等等

## 读数据流程

1.

