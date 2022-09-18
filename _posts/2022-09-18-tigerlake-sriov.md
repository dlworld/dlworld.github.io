---
layout: post
title: "Enable SR-IOV on Intel Tigerlake iGPU"
categories: [virtualization]
tags: sriov, virtualization
---

My laptop cpu is i7-1165G7 with intergrated Iris Xe Graphics, which support SR-IOV with max 7 virtual functions. I'm trying to enable the iGPU SR-IOV.

<!--break-->


1. Get the linux-intel-lts kernel and checkout 5.15 branch which had patches to support SR-IOV

   ```sh
   # git clone  https://github.com/intel/linux-intel-lts.git
   # git checkout 5.15/ADL-Linux-ER
   ```

2. Get the latest GuC firmware from i915 branch

   ```sh
   # wget https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/tree/i915/tgl_guc_70.1.1.bin
   # mv tgl_guc_70.1.1.bin /lib/firmware/i915/
   ```

3. Patch linux kernel to use the downloaded latest firmware

   ```sh
   diff --git a/drivers/gpu/drm/i915/gt/uc/intel_uc_fw.c b/drivers/gpu/drm/i915/gt/uc/intel_uc_fw.c
   index b16ec1b18fd4..ab508b165397 100644
   --- a/drivers/gpu/drm/i915/gt/uc/intel_uc_fw.c
   +++ b/drivers/gpu/drm/i915/gt/uc/intel_uc_fw.c
   @@ -54,7 +54,7 @@ void intel_uc_fw_change_status(struct intel_uc_fw *uc_fw,
           fw_def(ALDERLAKE_S,  0, guc_def(tgl,  70, 0, 3)) \
           fw_def(DG1,          0, guc_def(dg1,  69, 0, 3)) \
           fw_def(ROCKETLAKE,   0, guc_def(tgl,  70, 0, 3)) \
   -       fw_def(TIGERLAKE,    0, guc_def(tgl,  70, 0, 3)) \
   +       fw_def(TIGERLAKE,    0, guc_def(tgl,  70, 1, 1)) \
           fw_def(JASPERLAKE,   0, guc_def(ehl,  69, 0, 3)) \
           fw_def(ELKHARTLAKE,  0, guc_def(ehl,  69, 0, 3)) \
           fw_def(ICELAKE,      0, guc_def(icl,  69, 0, 3)) \
   ```

4. build and install the kernel

5. set i915 module params in cmdline

   ```sh
   i915.enable_guc=5 i915.max_vfs=7  i915.guc_log_level=5
   ```

   - enable_guc, set bit 2 to enable SRIOV
     - ENABLE_GUC_SUBMISSION  BIT(0)
     - ENABLE_GUC_LOAD_HUC     BIT(1)
     - ENABLE_GUC_SRIOV_PF       BIT(2)
   
6. enable virtual functions

   ```sh
   # echo 7 > /sys/devices/pci0000\:00/0000\:00\:02.0/sriov_numvfs
   ```

   ![image-20220912212525555](/images/image-20220912212525555.png)

7. set up libvirt XML to boot virtual machine with virtual function passthrough

   ```xml
       <hostdev mode='subsystem' type='pci' managed='yes'>
         <source>
           <address domain='0x0000' bus='0x00' slot='0x02' function='0x1'/>
         </source>
         <address type='pci' domain='0x0000' bus='0x07' slot='0x00' function='0x0'/>
       </hostdev>
   ```

