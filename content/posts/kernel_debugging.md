---
title: "Kernel Debugging Setup"
date: 2024-03-02T15:00:00Z
image: /images/post/kernel_debug_title.jpeg
categories: ["Linux", "Kernel", Debugging]
featured: true
draft: false
---

All right folks, I've been procrastinating a lot because of the workload and have been laying low, enjoying my comfort zone. I've been learning Linux kernel internals for a while, and if you're like me, you might have the urge to debug the kernel to see how it actually works and what magic **Linus Torvalds** and others at the Linux Foundation have crafted to get our system running. I'll try to keep this post as simple and beginner-friendly as possible, so if you're just starting to look into the kernel, you don't have to spend a month properly understanding and setting up the environment. Let's jump in and configure our cool Linux kernel debugging environment!

## Resonating with Linux Spirit

First things first, in the old-fashioned Linux way, we need to update our system and install tons of dependencies. You can do this using the following commands:

<Code language="sh">
{` 
   #This will take time depending on your internet connection
   sudo apt update && sudo apt-get upgrade -y
   sudo apt install gcc make bc build-essential libelf-dev libssl-dev 
   sudo apt install bison flex initramfs-tools qemu-system-x86 cpio
   sudo apt install git-core libncurses5-dev dwarves zstd

`}
</Code>

## Downloading the Source

Sure, here is the corrected grammar, spelling, and punctuation:

Once you're done installing the dependencies, the first thing you'll need is the kernel you want to debug. You can download the source code for the Linux kernel from the [kernel.org](https://kernel.org) website. If you're interested in older kernel versions, you can visit the [mirror site](https://mirrors.edge.kernel.org/pub/linux/kernel/). And if you're a true Linux kernel enthusiast, you can use the `wget` command to download the source code.


<Notice type="tip">
   If you decided to learn linux internals, I suggest to pickup an kernel version and stick with it for consistent learning.
</Notice>

<Code language="sh">
{` 
   set $KERNEL_VERSION=6.5
   set $BASE=6.x
   wget -cv https://mirrors.edge.kernel.org/pub/linux/kernel/v$BASE/linux-$KERNEL_VERSION.tar.gz

`}
</Code>

<Notice type="info">
  You can set the `KERNEL_VERSION` and `BASE` environment variables accordingly here the BASE is set as **6.x** because the kernel version i want to use is **6.5** if you are interested in let's say **5.15.80** then you have to set the `BASE` as **5.x** and the `KERNEL_VERSION` as **5.15.80**
</Notice>

Once, the **wget** completes the downloading you have tarball of the linux kernel source code uncompress it using the following command

<Code language="sh">
{` 
   tar -xvf linux-$KERNEL_VERSION.tar.gz

`}
</Code>

## Configure the Kernel

Contrary to popular belief, compiling the kernel is not difficult. In fact, it's quite similar to compiling other utilities from source code. In true Linux tradition, we'll configure the Linux kernel by editing the `config` file.

<Code language="sh">
{`
   make defconfig  
   vim .config

`}
</Code>

### Important configuration

Here are the few configurations you must enable to make your debugging process easier and more enjoyable. After all, who wants to set breakpoints at machine code offsets? We want to be able to set them like this: `break fs/pipe.c:496`.

<Code language="sh">
{`
   # Make sure to have these configs on 
   "CONFIG_NET_9P=y" 
   "CONFIG_NET_9P_DEBUG=n" 
   "CONFIG_9P_FS=y" 
   "CONFIG_9P_FS_POSIX_ACL=y" 
   "CONFIG_9P_FS_SECURITY=y" 
   "CONFIG_NET_9P_VIRTIO=y" 
   "CONFIG_VIRTIO_PCI=y" 
   "CONFIG_VIRTIO_BLK=y" 
   "CONFIG_VIRTIO_BLK_SCSI=y" 
   "CONFIG_VIRTIO_NET=y" 
   "CONFIG_VIRTIO_CONSOLE=y" 
   "CONFIG_HW_RANDOM_VIRTIO=y" 
   "CONFIG_DRM_VIRTIO_GPU=y" 
   "CONFIG_VIRTIO_PCI_LEGACY=y" 
   "CONFIG_VIRTIO_BALLOON=y" 
   "CONFIG_VIRTIO_INPUT=y" 
   "CONFIG_CRYPTO_DEV_VIRTIO=y" 
   "CONFIG_BALLOON_COMPACTION=y" 
   "CONFIG_PCI=y" 
   "CONFIG_PCI_HOST_GENERIC=y" 
   "CONFIG_GDB_SCRIPTS=y" 
   "CONFIG_DEBUG_INFO=y" 
   "CONFIG_DEBUG_INFO_REDUCED=n" 
   "CONFIG_DEBUG_INFO_SPLIT=n" 
   "CONFIG_DEBUG_FS=y" 
   "CONFIG_DEBUG_INFO_DWARF4=y" 
   "CONFIG_DEBUG_INFO_BTF=y" 
   "CONFIG_FRAME_POINTER=y"

`}
</Code>

<Notice type="note">
 The above configuration list is not exhaustive. The kernel has thousands of configuration options, and they are set based on your specific needs. If you plan to debug a specific module or file system, make sure to enable the relevant configuration options for it. 
</Notice>

## Happy Make time

With the configuration complete, compilation is straightforward. Use the `make` command to build the Linux kernel. This process might take some time depending on your system resources.

<Code language="sh">
{`
   make -j $(nproc)

`}
</Code>

Tada! You're done with the compilation process. I decided to keep things simple and ignore many details. If you want to explore this process in detail, you can check out the blog written by my friend Ali [Linux Kernel Compilation](https://medium.com/@alirazamumtaz/linux-kernel-compilation-and-addling-a-custom-system-call-d6060baff429) and the handout by **Dr. Arif Butt** [Linux Kernel Compilation Handouts](http://www.arifbutt.me/wp-content/uploads/2022/04/Handout-Linux-Kernel-Compilation-1.pdf).


## Setting up the GDB


![image](/images/post/kernel_debugging_gdb.jpeg)

If you're used to debugging with GDB, you might be tempted to skip this section. But I highly recommend reading it! While I won't waste time explaining how to install GDB (you can use your package manager), I do want to emphasize the importance of using specific tools for the job.

While userland debuggers like GDB wrappers (GEF, pwndbg, Peda) are fantastic, they're not ideal for debugging the kernel. That's why [bata24 ](https://github.com/bata24) created an enhanced version of [GEF](https://github.com/bata24/gef/), specifically designed for kernel debugging. 

If you find this wrapper helpful, be sure to express your gratitude to both [bata24](https://github.com/bata24) and [Dawar Zaman](https://www.linkedin.com/pub/dir/Qadeer/Zaman)). Dawar introduced me to this enhanced GEF version.

The installation process for the enhanced GEF is quite straightforward. You can follow the instructions provided on the GitHub repository.

## Setting up the Qemu Environment 

![image](/images/post/kernel_debugging_qemu.jpeg)

Qemu is open source emulator which will run over linux the best thing about the Qemu is that it provides you option to debug the kernel over the socket connection.

### Clone the Qemu Environment Repository 

1. The first step for the qemu environment setup is to clone the repository from [github](https://github.com/DeathNet123/Kernel_Debugging_Env/)

Once you have the repository clone let's look at the 
