---
layout: page
title: Cinder对接华为存储
categories: [storage]
tags: cinder
#permalink: /archive/
#banner_image: sample-banner-image-3.jpg
---



<!--break-->


﻿
OpenStack Cinder项目发展比较成熟，支持大部分主流存储厂商。但国内只有华为和联想，但联想的是OEM DotHill，驱动直接继承自dothill。

华为存储驱动进入Cinder项目较早，但主干版本中的驱动更新并不及时，官方配置文档内容并没有完全更新。

以Ｌ版本的iSCSI驱动对接华为OceanStor 6800V3为例。

## iSCSI对接

1. 通过存储Web管理平台DeviceManager创建存储池
2. 华为驱动配置文件

```
<?xml version='1.0' encoding='UTF-8'?>
<config>
<Storage>
	<Product>V3</Product>
	<Protocol>iSCSI</Protocol>
	<RestURL>https://192.168.2.6:8088/deviceManager/rest/</RestURL>
	<UserName>admin</UserName>
	<UserPassword>xxxxxxxx</UserPassword>
</Storage>
<LUN>
	<LUNType>Thick</LUNType>
	<WriteType>1</WriteType>
	<MirrorSwitch>0</MirrorSwitch>
	<LUNcopyWaitInterval>5</LUNcopyWaitInterval>
	<Timeout>432000</Timeout>
	<StoragePool>StoragePool001</StoragePool>
</LUN>
<iSCSI>
	<DefaultTargetIP>192.168.200.6</DefaultTargetIP>
	<Initiator Name="iqn.1994-05.com.redhat:f824e8232f3" TargetIP="192.168.200.6"/>
	<Initiator Name="iqn.1993-08.org.debian:01:cb55fa9ff2e8" TargetIP="192.168.200.6"/>
</iSCSI>
<Host OSType="Linux" HostIP="192.168.200.11,192.168.200.12"/>
</config>
```

3. Cinder配置文件

```
volume_driver = cinder.volume.drivers.huawei.huawei_driver.HuaweiISCSIDriver
cinder_huawei_conf_file = /etc/cinder/cinder_huawei_conf.xml
```

文档中给的驱动为HuaweiV3ISCSIDriver，而代码中并没有该驱动．而且Cinder自带的代码有Bug，需要从华为存储的github(https://github.com/huaweistorage/OpenStack_Driver)上下载．


## 多路径配置

1. 通过存储Web管理平台DeviceManager创建端口组，并将存储的相应端口加入该端口组。
2. 将端口组加入驱动配置文件，让发起端通过端口组与目标端连接，并重启服务。

```
<iSCSI>
	<DefaultTargetIP>192.168.200.6</DefaultTargetIP>
	<Initiator Name="iqn.1994-05.com.redhat:f824e8232f3" TargetPortGroup="PortGroup001"/>
</iSCSI>
```

3. 计算服务（nova）配置文件中启用多路径，并重启服务

```
# nova.conf, [libvirt]
iscsi_use_multipath = True
```

## 参数说明

** 必填参数 **

参数 | 默认值 | 描述 | 适用存储类型
--- | --- | --- | ---
Product | 	-	 | 存储产品，包括：TV2, 18000 和 V3．	 | All
Protocol | 	-	 | 连接协议， 'iSCSI' 或 'FC'. | 	All
RestURL | 	-	 | 管理接口地址，如：https://x.x.x.x:8088/deviceManager/rest/，如果多个RestURL用分号隔开(;). | 	All
UserName | 	-	 | 存储管理员用户名	 | All
UserPassword	 | -	 | 存储管理员密码 | All
StoragePool	 | -	 | 使用的存储池名称，如果多个存储池用分号隔开(;). | 	All


** 可选参数 **


参数 | 默认值 | 描述 | 适用存储类型
--- | --- | --- | ---
LUNType | 	Thin	 | 创建的LUN类型，Thick 或 Thin，使用Thin需要存储有相应的授权 | All
WriteType	 | 1	 | 写缓存类型：1 (write back), 2 (write through), and 3 (mandatory write back).	 | All
MirrorSwitch | 	1 | 	Cache mirroring or not, possible values are: 0 (without mirroring) or 1 (with mirroring).	 | All
LUNcopyWaitInterval | 	5	 | 如果启用LUN复制，查询拷贝进度的频率 | T series V2 V3 18000
Timeout	 | 432000	 | 等待存储LUN拷贝超时时间，单位是秒	 | T series V2 V3 18000
Initiator Name | 	-	 | 计算节点发起端名称，/etc/iscsi/initiatorname.iscsi	 | All
Initiator TargetIP | 	- | 为计算节点提供服务的iSCSI端口IP地址 | All
Initiator TargetPortGroup | 	-	 | 为计算节点提供服务的端口组，如多路径场景	 | T series V2 V3 18000
DefaultTargetIP | 	-	 | Target默认IP | All
OSType	 | Linux	 | Operating system of the Nova compute node’s host.	 | All
HostIP | 	-	 | 要连接存储的计算节点IP | 	All


## 参考文档

- [华为存储OpenStack驱动](https://github.com/huaweistorage/OpenStack_Driver/tree/master/Cinder)
- [Cinder华为存储驱动配置](http://docs.openstack.org/newton/config-reference/block-storage/drivers/huawei-storage-driver.html)


