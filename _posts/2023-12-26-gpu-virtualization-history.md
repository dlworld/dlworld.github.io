---
layout: post
title: "History of GPU Virtualization"
categories: [virtualization]
tags: gpu, virtualization
---

The development process of GPU virtualization is actually closely related to the popularity of the public cloud market and cloud computing application scenarios. If we talked about cloud computing 15 years ago, most people's reaction would be "unintelligible". However, with the popularity of cloud computing scenarios and the concept becoming more and more popular, everyone gradually has a clearer concept and practical understanding of cloud computing. Naturally, as application scenarios expand from the application of computing units that rely solely on CPUs to multiple architectures, and with the advent of heterogeneous computing scenarios, virtualization and cloud migration are also proposed for professional computing chips such as GPU, FPGA, and TPU. strongly demand. Especially in recent years, the rapid development of machine learning, deep learning and other fields has given rise to an upsurge in the migration of heterogeneous computing scenarios to the cloud.
Since GPU is the main force of heterogeneous computing, let us review the development history of GPU virtualization and make a horizontal comparison of various GPU manufacturers. 

<!--break-->


- *2008: Preface*
VMware's GPU full virtualization VSGA technology is the first attempt at GPU shared virtualization. It was first launched in VMware's commercialized Workstation 6.5 and Fusion 2.0 versions at the end of 2008, and was later used in the data center-oriented product vSphere. integrated. However, this is a VMware proprietary closed-source solution. It has not been widely used in the open source community and products outside of VMware, so it is not the focus of this article.
- *2012: Beginning*
With the introduction of the kernel VFIO module and the gradual popularity of pass-through devices, the road to GPU virtualization has been opened. The beginning of large-scale application is generally accompanied by the successful implementation of the VFIO module. In fact, around 2012, GPU pass-through technology has been an important application scenario of the VFIO module.
- *2013: First product versus the rest*
Nvidia released the GRID K1 product in 2013, which marked the maturity of GPU virtualization and gradually started the rapid development process of heterogeneous computing virtualization.
In fact, in the same year of 2013, Intel OTC's GVT-d and GVT-g GPU virtualization solutions for HSW have been developed for more than a year. The original hardware was based on SNB/HSW, and the prototype code was based on Xen Hypervisor.
Intel maintains a keen technical insight into the development of the GPU industry. It had already initiated a proposal for GPU virtualization as early as 2011. However, due to insufficient attention, it was not until 2014, three years later, that there was a GVT-g-based solution. XenClient product is launched.
In the same year: The community maintainer of the VFIO module also officially released the VGA assignment on the KVM Forum.
At the beginning of the same year: AMD has also started a GPU virtualization solution (Tonga architecture) based on SRIOV, and began to develop the GIM driver and vGPU scheduling system of SRIOV PF. It is speculated that the hardware implementation of SRIOV should have been completed about half a year in advance. It was not until two years later that AMD finally ushered in its first GPU SRIOV product: FirePro S7150 (released in early 2016).
As the leader in the GPU industry, Nvidia is basically 1-2 years ahead of its competitors in the research and development and productization of GPU virtualization. As a competitor, AMD has been catching up since then. Intel was basically a runner during that period.
2014: The birth of vGPU shard virtualization
One year later, in 2014, with the publication of a Usenix ATC paper: "A Full GPU Virtualization Solution with Mediated Pass-Through", an unknown new technology of GPU virtualization officially entered everyone's attention: GPU Sharding virtualization (let’s call it this in Chinese for now, because the name mediated passh-through doesn’t make people understand what this is at all).
This paper was published by two Principal Engineers from Intel OTC, and it also represents Intel's technology accumulation in the field of GPU virtualization.
It should be said that Nvidia, as an industry leader, plays a crucial role in promoting shard virtualization in the community. In fact, VFIO's mdev framework was introduced by Nvidia for the GRID vGPU product line. The concept of mdev was first proposed by Nvidia and merged into Linux kernel 4.10. People who play in closed source ecosystems are also beginning to embrace open source.
There is no news about AMD in 2014. It should continue to develop the world's first SRIOV-based GPU solution.
- *2015: Divergence*
In cooperation with Citrix, Intel has released XenClient and XenServer products based on GVT-d and GVT-g for shard virtualization. These products represented the benchmark for the GPU virtualization industry in the Xen community at that time. 
Intel has also begun to promote GVT-g technology at major internal and external conferences. Of course, it hopes that its technology can be commercialized and have a good market prospect. For example, at the Intel Developer Conference (IDF) that year, it took the lead in releasing a multimedia video processing cloud solution based on GVT-g. There were more than 100 people listening, and many were interested. As a system that uses free GPU for audio and video processing, it is much more cost-effective than using E5 Server alone. But unfortunately, no product was launched in the end. The reason is still the internal positioning problem of Intel GPU. The fatal flaws and pain points of the Intel GVT-g solution will be discussed later.
AMD continues to develop the world's first SRIOV GPU.
While others are playing with technology, Nvidia has already begun its industrial layout. In the same year, various GRID-based solutions on AWS were released in cooperation with VMware, such as the very cool Game Streaming.
In fact, GRID is a big concept. Represents Nvidia's broad portfolio of GPU virtualization products. Among them, GRID vGPU is a shard virtualization solution based on mdev.
- *2016, 2017: Returns*
In 2016, AMD brought the world's first GPU SRIOV graphics card FirePro S7150x2. This product for graphics rendering applications has become a must-have business for major public cloud vendors. This is the only graphics rendering virtualization that is cost-effective.
Intel continues to vigorously promote Intel GVT-g technology in major forums. For the first time in technology, Nvidia, the industry leader, took the lead in implementing vGPU live migration technology. It can be said that Intel OTC's virtualization department has achieved the ultimate in GVT-g within its own capabilities. However, it has failed on the road to productization. The further we go, the harder it gets.
At this time, Nvidia was riding on the popularity of AI, and was increasingly improving GRID technology and sharding virtualization, leaving its opponents far behind. At this time Nvidia also began to show its face in the open source community. And on the second day of the 2016 KVM Forum, Nvidia architect Neo grandly introduced GRID vGPU technology. 
It has to be said that in the early years, Intel, as a representative of intergrated graphics, and Nvidia, as a representative of discrete graphics cards, had in-depth cooperation in GPU research and development. And then cooperated with AMD to develop CPU+GPU chips. And the recent cooperation between Intel and AMD to fight Nvidia's squeeze in the GPU field.
The above three are both rivals and friends.
- *2020: New frontiers*
Nvidia released the Ampere GPU with MIG(Multi-Instance GPU), which is a revolutionary technology that carves up the mighty power of Ampere Tensor Core GPU into up to seven independent instances, each with its own dedicated resources like high-bandwidth memory, cache, and compute cores. 
AMD also has follow-up product announcements. For example, the release of MI25, a Deep Learning benchmark against old rival Nvidia.
With the popularity of GPU virtualization applications, the application scenarios of GPU virtualization are no longer limited to the cloud computing market. Various emerging industries have also begun to apply GPU virtualization technology. The most direct one is the in-vehicle entertainment system, referred to as IVI (In-vehicle Information system). So the three old friends are also old rivals, and they all began to compete in the fields of IVI and autonomous driving. This has also brought about a turning point for the implementation of Intel GVT-g technology. So Intel took the lead in releasing the virtualization solution (ACRN) based on the Internet of Things, and started again with GVT-g's shard virtualization technology.

References
- (阿里云郑晓：浅谈GPU虚拟化技术)[https://developer.aliyun.com/article/590909]
