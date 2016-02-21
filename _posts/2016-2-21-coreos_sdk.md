---
layout: post
title: coreos sdk开发环境配置
categories: [coreos]
tags: coreos
---


## 环境准备

### 安装repo

CoreOS包括很多子项目，**repo**工具可以帮助管理这些git仓库。

```
mkdir ~/bin 
export PATH="$PATH:$HOME/bin" 
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo 
chmod a+x ~/bin/repo
```

### 准备项目目录

```
mkdir coreos-sdk; cd coreos-sdk
```

### 初始化项目

```
dlw@dlw:coreos-sdk$ repo init -u https://github.com/coreos/manifest.git
dlw@dlw:coreos-sdk$ repo sync
Fetching project coreos/updateservicectl
Fetching project coreos/sysroot-wrappers
Fetching project coreos/scripts
Fetching project coreos/etcd
Fetching projects:   2% (1/39)  Fetching project coreos/coreos-buildbot
Fetching projects:   5% (2/39)  Fetching project coreos/systemd
Fetching projects:   7% (3/39)  Fetching project coreos/installer
Fetching projects:  10% (4/39)  Fetching project appc/spec
Fetching projects:  12% (5/39)  Fetching project chromiumos/third_party/pyelftools
Fetching projects:  15% (6/39)  Fetching project coreos/bootengine
Fetching projects:  17% (7/39)  Fetching project chromiumos/platform/crostestutils
Fetching projects:  20% (8/39)  Fetching project coreos/coretest
Fetching projects:  23% (9/39)  Fetching project coreos/grub
Fetching projects:  25% (10/39)  Fetching project coreos/locksmith
Fetching projects:  28% (11/39)  Fetching project chromiumos/platform/factory-utils
Fetching projects:  30% (12/39)  Fetching project coreos/portage-stable
Fetching projects:  33% (13/39)  Fetching project coreos/coreos-overlay
Fetching projects:  35% (14/39)  Fetching project coreos/chromite
Fetching projects:  38% (15/39)  Fetching project coreos/shim
Fetching projects:  41% (16/39)  Fetching project coreos/mayday
Fetching projects:  43% (17/39)  Fetching project coreos/efunctions
Fetching projects:  46% (18/39)  Fetching project chromiumos/repohooks
Fetching projects:  48% (19/39)  Fetching project coreos/coreos-metadata
Fetching projects:  51% (20/39)  Fetching project coreos/docker
Fetching projects:  53% (21/39)  Fetching project coreos/coreos-cloudinit
Fetching projects:  56% (22/39)  Fetching project coreos/seismograph
Fetching projects:  58% (23/39)  Fetching project coreos/mantle
Fetching projects:  61% (24/39)  Fetching project coreos/baselayout
Fetching projects:  64% (25/39)  Fetching project coreos/dev-util
Fetching projects:  66% (26/39)  Fetching project coreos/etcdctl
Fetching projects:  69% (27/39)  Fetching project coreos/sdnotify-proxy
Fetching projects:  71% (28/39)  Fetching project coreos/ignition
Fetching projects:  74% (29/39)  Fetching project coreos/nss-altfiles
Fetching projects:  76% (30/39)  Fetching project coreos/rkt
Fetching projects:  79% (31/39)  Fetching project coreos/fleet
Fetching projects:  82% (32/39)  Fetching project coreos/init
Fetching projects:  84% (33/39)  Fetching project appc/acbuild
Fetching projects:  87% (34/39)  Fetching project coreos/toolbox
Fetching projects:  89% (35/39)  Fetching project coreos/update_engine
Fetching projects: 100% (39/39), done.  
Syncing work tree: 100% (39/39), done.  
Your sources have been sync'd successfully.  

dlw@dlw:coreos-sdk$ ls
AUTHORS  chromite  git-repo  LICENSE  src
```

## QEMU 交叉编译环境

环境用于编译arm64系统，需要配置binfmt服务

```
dlw@dlw:coreos-sdk$ cat /etc/binfmt.d/qemu-aarch64.conf
:qemu-aarch64:M::\x7fELF\x02\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\xb7:\xff\xff\xff\xff\xff\xff\xff\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff:/usr/bin/qemu-aarch64-static:
```

运行或重启服务

```
systemctl restart systemd-binfmt.service
```

## 编译镜像

下载编译器和其它工具：

```
dlw@dlw:coreos-sdk$ ./chromite/bin/cros_sdk
Attempting download: https://commondatastorage.googleapis.com/builds.developer.core-os.net/sdk/amd64/949.0.0/coreos-sdk-amd64-949.0.0.tar.bz2
<>
These are the packages that would be merged, in order:
Calculating dependencies... done!
Total: 0 packages, Size of downloads: 0 KiB
INFO    update_chroot: Removing obsolete packages
Scanning Configuration files...
Exiting: Nothing left to do; exiting. :)
INFO    update_chroot: Elapsed time (update_chroot): 87m30s
INFO    cros_sdk:make_chroot: Elapsed time (make_chroot.sh): 90m7s
cros_sdk:make_chroot: All set up.  To enter the chroot, run:
$ cros_sdk --enter 
CAUTION: Do *NOT* rm -rf the chroot directory; if there are stale bind
mounts you may end up deleting your source tree too.  To unmount and
delete the chroot cleanly, use:
$ cros_sdk --delete 
dlw@dlw ~/trunk/src/scripts $ 
```

进入chroot环境

```
dlw@dlw:coreos-sdk$ ./chromite/bin/cros_sdk --enter
```

删除SDK chroot

```
dlw@dlw:coreos-sdk$ ./chromite/bin/cros_sdk --delete
```

#### 设置core用户密码：

```
dlw@dlw ~/trunk/src/scripts $ ./set_shared_user_password.sh 
Enter password for shared user account: Password set in /etc/shared_user_passwd.txt
```

#### 选择编译的平台架构

如果在一个SDK下编译不同的targets，在后面build_packages和build_image阶段也可以加参数*--board=*。

```
dlw@dlw ~/trunk/src/scripts $ ./setup_board --default --board=arm64-usr
INFO    update_chroot: Setting up portage...
find: `/mnt/host/source/config/portage/repos': No such file or directory
INFO    update_chroot: Setting up crossdev...
INFO    update_chroot: Updating chroot:
INFO    update_chroot:  chroot version: 955.0.0+2016-02-14-2253
INFO    update_chroot:  CoreOS version: 955.0.0+2016-02-14-2253
INFO    update_chroot: Updating basic system packages
!!! Binhost package index  has no TIMESTAMP field.
>>> Jobs: 0 of 0 complete                           Load avg: 0.52, 0.45, 0.48
INFO    update_chroot: Updating cross aarch64-cros-linux-gnu toolchain
!!! Binhost package index  has no TIMESTAMP field.
-------------------------------------------------------------------------------------------------------------
 * crossdev version:      20140917
 * Host Portage ARCH:     amd64
 * Target Portage ARCH:   arm64
 * Target System:         aarch64-cros-linux-gnu
 * Stage:                 4 (C/C++ compiler)
 * ABIs:                  arm64

 * binutils:              binutils-[stable]
 * gcc:                   gcc-[stable]
 * headers:               linux-headers-[stable]
 * libc:                  glibc-[stable]
 * Extra: gdb:            DO IT  

 * CROSSDEV_OVERLAY:      /usr/local/portage/crossdev
 * PORT_LOGDIR:           /var/log/portage
 * PORTAGE_CONFIGROOT:    
 * Portage flags:          --quiet -uNv --with-bdeps=y --select --usepkg --getbinpkg                         --jobs=4
  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  - 
 * leaving sys-devel/binutils in /usr/local/portage/crossdev
 * leaving sys-devel/gcc in /usr/local/portage/crossdev
 * leaving sys-kernel/linux-headers in /usr/local/portage/crossdev
 * leaving sys-libs/glibc in /usr/local/portage/crossdev
 * leaving sys-devel/gdb in /usr/local/portage/crossdev
 * leaving metadata/layout.conf alone in /usr/local/portage/crossdev
  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  - 
 * Log: /var/log/portage/cross-aarch64-cros-linux-gnu-binutils.log
 * Emerging cross-binutils ...
 * Messages for package sys-devel/gcc-4.9.3 merged to /build/arm64-usr/:
 * If you have issues with packages unable to locate libstdc++.la,
 * then try running 'fix_libtool_files.sh' on the old gcc versions.
 * You might want to review the GCC upgrade guide when moving between
 * major versions (like 4.2 to 4.3):
 * http://www.gentoo.org/doc/en/gcc-upgrading.xml
INFO    setup_board: Elapsed time (setup_board): 40m44s
INFO    setup_board: The SYSROOT is: /build/arm64-usr
```

#### 编译链接系统二进制

编译所有目标二进制包

```
./build_packages
```

#### 编译GRUB

Aarch64架构，编译*arm64-usr*镜像前需要先编译GRUB。向
/etc/portage/package.use/grub文件添加：

```
sys-boot/grub grub_platforms_arm64
```

```
sudo emerge -v grub
```

#### 生成CoreOS镜像

用前面编译的包生成镜像

```
# ./build_image dev
...
Formatting partition 1 (EFI-SYSTEM) as vfat
Formatting partition 3 (USR-A) as ext2
Formatting partition 6 (OEM) as ext4
Formatting partition 9 (ROOT) as ext4
...
2016/02/21 14:43:51 - generate_au_zip.py - INFO    : Generated /mnt/host/source/src/build/images/arm64-usr/developer-955.0.0+2016-02-21-1024-a1/coreos_production_update.zip
COREOS_BUILD=955
COREOS_BRANCH=0
COREOS_PATCH=0
COREOS_VERSION=955.0.0+2016-02-21-1024
COREOS_VERSION_ID=955.0.0
COREOS_BUILD_ID="2016-02-21-1024"
COREOS_SDK_VERSION=949.0.0
Done. Image(s) created in /mnt/host/source/src/build/images/arm64-usr/developer-955.0.0+2016-02-21-1024-a1
Developer image created as coreos_developer_image.bin
To convert it to a virtual machine image, use:
  ./image_to_vm.sh --from=../build/images/arm64-usr/developer-955.0.0+2016-02-21-1024-a1 --board=arm64-usr 
The default type is qemu, see ./image_to_vm.sh --help for other options.
```


二进制镜像转换成虚拟机

```
dlw@dlw ~/trunk/src/scripts $ ./image_to_vm.sh --help
...
Resizing partition 9 (ROOT) to 6627000320 bytes
ROOT: 8812/196608 files (0.0% non-contiguous), 22940/196608 blocks
resize2fs 1.42.13 (17-May-2015)
Resizing the filesystem on /dev/loop0 to 1617920 (4k) blocks.
The filesystem on /dev/loop0 is now 1617920 (4k) blocks long.  

INFO    image_to_vm.sh: Mounting image to ../build/images/arm64-usr/developer-955.0.0+2016-02-21-1024-a1/coreos_developer_qemu_image.img.vmtmpdir/rootfs
Cleaning up mounts
INFO    image_to_vm.sh: Writing qcow2 image coreos_developer_qemu_image.img
INFO    image_to_vm.sh: Writing qemu configuration
INFO    image_to_vm.sh: Cleaning up temporary files  

    .  o ..
    o . o o.o
         ...oo_
           _[__\___
        __|_o_o_o_o\__
    OK  \' ' ' ' ' ' /
    ^^^^^^^^^^^^^^^^^^^^  

INFO    image_to_vm.sh: Elapsed time (image_to_vm.sh): 1m24s
INFO    image_to_vm.sh: Files written to ../build/images/arm64-usr/developer-955.0.0+2016-02-21-1024-a1
INFO    image_to_vm.sh:  - coreos_developer_qemu_image.img
INFO    image_to_vm.sh:  - coreos_developer_qemu.sh
INFO    image_to_vm.sh:  - coreos_developer_qemu.README
If you have qemu installed (or in the SDK), you can start the image with:
  cd path/to/image
  ./coreos_developer_qemu.sh -curses -bios QEMU_EFI.fd  

If you need to use a different ssh key or different ssh port:
  ./coreos_developer_qemu.sh -a ~/.ssh/authorized_keys -p 2223 -- -curses -bios QEMU_EFI.fd  

If you rather you can use the -nographic option instad of -curses. In this
mode you can switch from the vm to the qemu monitor console with: Ctrl-a c
See the qemu man page for more details on the monitor console.  

SSH into that host with:
  ssh 127.0.0.1 -p 2222  

﻿A prebuilt QEMU EFI firmware can be downloaded at the following link:
http://snapshots.linaro.org/components/kernel/leg-virt-tianocore-edk2-upstream/latest/QEMU-AARCH64/RELEASE_GCC48/QEMU_EFI.fd
```

### 启动

用qemu启动镜像

```
dlw@dlw ~/trunk/src/scripts $ cd ../build/images/arm64-usr/developer-955.0.0+2016-02-21-1024-a1/
dlw@dlw developer-955.0.0+2016-02-21-1024-a1 $ wget http://snapshots.linaro.org/components/kernel/leg-virt-tianocore-edk2-upstream/latest/QEMU-AARCH64/RELEASE_GCC48/QEMU_EFI.fd
dlw@dlw developer-955.0.0+2016-02-21-1024-a1 $ ./coreos_developer_qemu.sh -curses -bios QEMU_EFI.fd
```

登录虚拟机

```
$ chmod 600 ~/.ssh/config
$ ssh core@127.0.0.1 -p 2222
core@127.0.0.1's password: 
CoreOS developer (955.0.0+2016-02-21-1024)
Update Strategy: No Reboots
core@coreos_developer_qemu-955-0-0-20 ~ $ uname -a
Linux coreos_developer_qemu-955-0-0-20 4.4.1-coreos #2 SMP PREEMPT Sun Feb 21 13:57:23 CST 2016 aarch64 GNU/Linux
```


## 修改

### git和repo

CoreOS用*repo*管理，最初是Android项目中用于管理大量git仓库。repo用XML格式的清单（manifest）文件描述上游仓库在哪，怎么合并到一个工作区（working checkout）。

### 更新repo清单

CoreOS的清单文件在*.repo/manifests*下，修改defaut.xml就行。
repo用‘default’分支追溯上游分支（）。

## 编译镜像

编译生产镜像和开发镜像的流程不同。

### 开发镜像

#### 镜像更新包

编译一个VM镜像的过程很花时间，镜像开发环境上可以用gmerge在自己的工作主机上编译包，然后放到目标虚拟机上运行。

在工作机SDK chroot上启动开发服务器

```
start_devserver --port 8080
```

在*/etc/coreos/update.conf*的*DEVSERVER*配置指向工作机的IP/hostname和端口，然后执行gmerge：

```
gmerge coreos-base/update_engine
```


# 错误记录

```
dlw@dlw ~/trunk/src/scripts $ ./update_chroot --toolchain_boards=arm64-usr --usepkg --nousepkgonly --getbinpkg --job=4
INFO    update_chroot: Setting up portage...
find: `/mnt/host/source/config/portage/repos': No such file or directory
INFO    update_chroot: Setting up crossdev...
INFO    update_chroot: Updating chroot:
INFO    update_chroot:  chroot version: 955.0.0+2016-02-14-2253
INFO    update_chroot:  CoreOS version: 955.0.0+2016-02-14-2253
INFO    update_chroot: Updating basic system packages  

!!! Binhost package index  has no TIMESTAMP field.
-------------------------------------------------------------------------------------------
 * crossdev version:      20140917
 * Host Portage ARCH:     amd64
 * Target Portage ARCH:   arm64
 * Target System:         aarch64-cros-linux-gnu
 * Stage:                 4 (C/C++ compiler)
 * ABIs:                  arm64
 * binutils:              binutils-[stable]
 * gcc:                   gcc-[stable]
 * headers:               linux-headers-[stable]
 * libc:                  glibc-[stable]
 * Extra: gdb:            DO IT
 * CROSSDEV_OVERLAY:      /usr/local/portage/crossdev
 * PORT_LOGDIR:           /var/log/portage
 * PORTAGE_CONFIGROOT:    
 * Portage flags:          --quiet -uNv --with-bdeps=y --select --usepkg --getbinpkg                         --jobs=4
  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  - 
 * leaving sys-devel/binutils in /usr/local/portage/crossdev
 * leaving sys-devel/gcc in /usr/local/portage/crossdev
 * leaving sys-kernel/linux-headers in /usr/local/portage/crossdev
 * leaving sys-libs/glibc in /usr/local/portage/crossdev
 * leaving sys-devel/gdb in /usr/local/portage/crossdev
 * leaving metadata/layout.conf alone in /usr/local/portage/crossdev
  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  -  ~  -  _  - 
 * Log: /var/log/portage/cross-aarch64-cros-linux-gnu-binutils.log
 * Emerging cross-binutils ...
 * binutils failed :(
 * 
 * If you file a bug, please attach the following logfiles:
 * /var/log/portage/cross-aarch64-cros-linux-gnu-info.log
 * /var/log/portage/cross-aarch64-cros-linux-gnu-binutils.log.xz
 * /var/tmp/portage/cross-aarch64-cros-linux-gnu/binutils*/temp/binutils-config.logs.tar.xz
dlw@dlw ~/trunk/src/scripts $ 
```


## 网络代理

使用代理仍会出错：

```
dlworld@dlw:coreos-sdk$ curl --socks5 127.0.0.1:1080 -I https://commondatastorage.googleapis.com/builds.developer.core-os.net/sdk/amd64/949.0.0/coreos-sdk-amd64-949.0.0.tar.bz2
curl: (35) Unknown SSL protocol error in connection to commondatastorage.googleapis.com:443 
```

通过浏览器等其它途径访问目标地址，获取其IP然后写入hosts文件。

```
216.58.199.4 www.google.com
216.58.199.16 commondatastorage.googleapis.com
```

### emerge设置git代理

```
>>> Unpacking source...
 * Falling back to fetching from remote git repository...
 * Fetching https://chromium.googlesource.com/chromiumos/third_party/rootdev.git ...
git fetch https://chromium.googlesource.com/chromiumos/third_party/rootdev.git --prune +refs/heads/*:refs/heads/* +refs/tags/*:refs/tags/* +refs/notes/*:refs/notes/* +HEAD:refs/git-r3/HEAD
fatal: unable to access 'https://chromium.googlesource.com/chromiumos/third_party/rootdev.git/': Failed to connect to chromium.googlesource.com port 443: Connection timed out
 * ERROR: sys-apps/rootdev-0.0.1-r11::coreos failed (unpack phase):
 *   Unable to fetch from any of EGIT_REPO_URI
 * 
 * Call stack:
 *     ebuild.sh, line  133:  Called src_unpack
 *   environment, line 2658:  Called cros-workon_src_unpack
 *   environment, line  612:  Called git-r3_src_unpack
 *   environment, line 1822:  Called git-r3_src_fetch
 *   environment, line 1816:  Called git-r3_fetch
 *   environment, line 1742:  Called die
 * The specific snippet of code:
 *       [[ -n ${success} ]] || die "Unable to fetch from any of EGIT_REPO_URI";
```

emerge获取代码使用代理

```
diff --git a/eclass/git-r3.eclass b/eclass/git-r3.eclass
index eca443b..cab4adb 100644
--- a/eclass/git-r3.eclass
+++ b/eclass/git-r3.eclass
@@ -265,6 +265,9 @@ _git-r3_env_setup() {
                eerror "unpack function. If necessary, please declare proper src_unpack()."
                die "EGIT_NOUNPACK has been removed."
        fi
+
+git config --global http.proxy 'socks5://127.0.0.1:1080'       
+git config --global https.proxy 'socks5://127.0.0.1:1080'      
 }  
 
 # @FUNCTION: _git-r3_set_gitdir
```
