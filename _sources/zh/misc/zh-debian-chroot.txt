====================================================
ArmHF Debian Chroot
====================================================

在 Linux 系统上进行嵌入式开发时，通常使用交叉编译或者qemu虚拟环境进行软件的构建，生成可以在目标机器上运行的可执行程序。对于 Debian ,为了加快软件开发速度，同时避免交叉编译时自行构建各个依赖库产生各种不必要的干扰，可以选择基于 QEMU 虚拟环境的 Debian Chroot 环境进行软件的开发，构建，以及调试。本篇主要对 Debian Chroot 环境的构建，开发，调试环境进行介绍，方便开发人员能够快速的搭建相关环境，介入软件开发过程。

配置 chroot 环境
====================================================

安装 qemu 和 debootstrap 等工具
----------------------------------------------------

.. code-block:: shell

   sudo apt-get install binfmt-support qemu qemu-user-static debootstrap 

获取 armhf 系统包
----------------------------------------------------

Debian Arm 硬浮点系统包有两种方式可以获取，包括 *debootstrap* 和 *qemu-debootstrap* ，两者获取方式如下说明。

基于 debootstrap 获取
""""""""""""""""""""""""""""""""""""""""""""""""""""

1. 安装 debootstrap, qemu, qemu-arm 等工具。


2. 获取 chroot 系统及软件包。

   .. code-block:: shell
   
      sudo debootstrap --no-check-gpg --arch=armhf sid ./sid-armhf ftp://ftp.debian.org/debian/
  
   .. note::
      
      对于上述指令中的 **sid** 为 Debian 的每日构建版本，属于 Factory 版本，软件时时更新，可能并不稳定。如果希望使用稳定版本，可以选择 **jessie** (Debian 8)，**wheezy** (Debian 7)， **squeeze** (Debian 6)等稳定版本，不过建议使用最新的稳定版本 **jessie** 作为生产系统，如果是用于测试或者开发者环境，可以选择 **sid** 。
   

基于 qemu-debootstrap 获取
""""""""""""""""""""""""""""""""""""""""""""""""""""

.. warning::
   
   对于 **qemu-debootstrap** 仅适用于 Debian 或者 Ubuntu 系统，以及这两个系统的衍生系统版本。

   
对于 **x86** 机器，则可以使用 qemu-user-static 包中的 qemu-debootstrap 来获取系统包。系统选择参照备注里的说明。
   
.. code-block:: shell

   qemu-debootstrap --no-check-gpg --arch=armhf sid /chroots/sid-armhf ftp://ftp.debian.org/debian/
      
切入 Chroot 环境
----------------------------------------------------

在完成 Arm 硬浮点架构的系统包获取之后，可以切入 Chroot 环境，进行第一次环境设置。

1. 进入 chroot 环境，挂载 */proc* 文件系统

   .. code-block:: shell
   
      sudo cp /usr/bin/qemu-arm-static sid-armhf/usr/bin
      sudo chroot /chroots/sid-armhf
      
      mount -t proc proc /proc
   
   .. note::
     
      如果使用的是 *qemu-bootstrap* 和 *schroot* ，那么 */proc* 的挂载将会有配置文件自动执行。
      
      对于 **openSUSE** ， 则需要拷贝 */usr/bin/qemu-arm* 和 */usr/bin/qemu-arm-binfmt* 到 *sid-armhf/usr/bin* 目录。

2. 首次进入系统设置。

   在第一次进入系统之后，需要对系统进行设置。在首次进入系统后，需要对系统进行设置，包括阻止系统服务启动，配置软件源等。
      
   .. tip::
      
      建议创建 **/usr/sbin/policy-rc.d** 阻止 chroot 环境中的后台服务启动。该文件权限需要修改为 **0755** ，其中的内容如下（内容取自pbuilder）。
      
      .. code-block:: shell
      
         echo "************************************" >&2
         echo "All rc.d operations denied by policy" >&2
         echo "************************************" >&2
         exit 101
         
   系统更新源配置位于 **/etc/apt/sources.list** ，由于 debootstrap 生成的软件源不可用，我们需要在其中添加可用的软件源，如下是 armhf 的二进制和源代码源：
   
   ::
   
     deb http://ftp.debian.org/debian sid main
     deb-src http://ftp.debian.org/debian sid main
     
   .. note::
   
     软件源需要根据自己的系统版本进行设置，这里使用的是 **sid** ，对于生产用系统需要根据选择的系统版本进行设置，如 **jessie** ， **wheezy** ，或者 **sequeeze** 等。
     
   在完成软件源的添加之后，可以根据需要更新系统，或者添加自己需要的软件源。
   
   .. code-block:: shell
     
      sudo apt-get update

准备开发环境
====================================================    

安装 gcc, build-essential, scons 等开发工具



