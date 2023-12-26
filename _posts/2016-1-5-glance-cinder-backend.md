---
layout: post
title: Glance cinder backend
categories: [openstack]
tags: openstack,cinder
---

* 目录
{:toc}


用一个Cinder服务管理多个后端存储器或存储池。在多后端配置中，每个后端都有一个名字（volume_backend_name），通过这个名字与云硬盘类型关联。名字相同时，由调度器决定在哪个后端创建卷。

## 配置
默认配置为/etc/cinder/cinder.conf
1. 设置enabled_backends（在DEFAULT段），不同后端的名字用逗号隔开。

```
enabled_backends=cinder_ceph1,cinder_vmware1
```

1. 用独立小节配置相应后端的存储驱动，需要加入volume_backend_name字段。

```
[cinder_ceph1]
volume_driver=cinder.volume.drivers.rbd.RBDDriver
volume_backend_name=cinder_ceph1
rbd_pool = volumes
rbd_ceph_conf = /etc/ceph/ceph.conf
rbd_flatten_volume_from_snapshot = false
rbd_max_clone_depth = 5
rbd_store_chunk_size = 4
rados_connect_timeout = -1
glance_api_version = 2
rbd_user = cinder
rbd_secret_uuid = 457eb676-33da-42ec-9a8c-9293d545c337
[cinder_vmware1]
volume_driver=cinder.volume.drivers.vmware.vmdk.VMwareVcVmdkDriver
volume_backend_name=cinder_vmware1
vmware_host_ip=172.16.11.200
vmware_host_username=root
vmware_host_password=xxxxxx
vmware_api_retry_count=10
vmware_volume_folder=datastore1
```

1. 调度

```
# Which filter class names to use for filtering hosts when not
# specified in the request. (list value)
#scheduler_default_filters=AvailabilityZoneFilter,CapacityFilter,CapabilitiesFilter
# Which weigher class names to use for weighing hosts. (list
# value)
#scheduler_default_weighers=CapacityWeigher
```

## 关联云硬盘
创建云硬盘类型并与相应的存储后端关联

```
[root@controller ~]# source admin-openrc.sh 
[root@controller ~]# cinder --os-username admin --os-tenant-name admin type-create ceph1
[root@controller ~]# cinder --os-username admin --os-tenant-name admin type-key ceph1 set volume_backend_name=cinder_ceph1
[root@controller ~]# cinder --os-username admin --os-tenant-name admin type-create vmware1
[root@controller ~]# cinder --os-username admin --os-tenant-name admin type-key vmware1 set volume_backend_name=cinder_vmware1
```

查看类型的额外属性

```
[root@controller ~]# cinder --os-username admin --os-tenant-name admin extra-specs-list
+--------------------------------------+---------+---------------------------------------------+
|                  ID                  |   Name  |                 extra_specs                 |
+--------------------------------------+---------+---------------------------------------------+
| 0bc8497c-7461-4e0d-b467-4fdf33561c44 | vmware1 | {u'volume_backend_name': u'cinder_vmware1'} |
| 53eeca13-af22-4b80-9631-1f1c8882788e |  ceph1  |  {u'volume_backend_name': u'cinder_ceph1'}  |
| bac77519-7c12-48cb-bac7-563d280f2a78 |  xsxsxx |                      {}                     |
+--------------------------------------+---------+---------------------------------------------+
```
﻿
创建云硬盘时指定类型

```
[root@controller ~]# cinder create --display-name test --volume-type ceph1 1
+---------------------+--------------------------------------+
|       Property      |                Value                 |
+---------------------+--------------------------------------+
|     attachments     |                  []                  |
|  availability_zone  |                cinder                |
|       bootable      |                false                 |
|      created_at     |      2015-11-20T08:36:20.908686      |
| display_description |                 None                 |
|     display_name    |                 test                 |
|      encrypted      |                False                 |
|          id         | 97f64d3e-a774-4a24-9f60-7e4fc99ad03b |
|       metadata      |                  {}                  |
|         size        |                  1                   |
|     snapshot_id     |                 None                 |
|     source_volid    |                 None                 |
|        status       |               creating               |
|     volume_type     |                ceph1                 |
+---------------------+--------------------------------------+
```
