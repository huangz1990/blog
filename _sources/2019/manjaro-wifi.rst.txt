为 Manjaro 18.1.4 配置无线网卡
===========================================

最近在用 Manjaro 工作，
想找个合适的无线网卡用但是网上的中文资料并不多，
特作此文希望能帮助到后来者。


系统&设备
----------------

系统版本： Manjaro 18.1.4

Linux 内核版本： 5.3.15

无线网卡： EDIMAX EW-7822UTC （USB 接口，5G 双频）


步骤
------------------

首先使用 ``lsusb`` 命令查看 USB 设备是否已经被正确识别：

::

    $ lsusb -v | more
    ...
    Bus 003 Device 002: ID 7392:c822 Edimax Technology Co., Ltd AC1200 MU-MIMO USB3.0 Adapter
    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               2.10
      bDeviceClass            0 
      bDeviceSubClass         0 
      bDeviceProtocol         0 
      bMaxPacketSize0        64
      idVendor           0x7392 Edimax Technology Co., Ltd
      idProduct          0xc822 
      bcdDevice            2.10
      iManufacturer           1 
      iProduct                2 
      iSerial                 3 
      bNumConfigurations      1

确认了 USB 设备正确识别之后，
接下来要做的就是安装驱动，
但 `Edimax 的官方驱动 <https://www.edimax.com/edimax/download/download/data/edimax/global/download/for_home/wireless_adapters/wireless_adapters_ac1200_dual-band/ew-7822utc>`_\ 最新只支持至 Linux 4.15 。
在网上搜索了下，
发现已经有人做好了驱动，
包名为 `rtl8822bu-dkms-git <https://aur.archlinux.org/packages/rtl8822bu-dkms-git>`_ ，
直接在 AUR 里面安装这个包即可。

另外需要注意的是，
``rtl8822bu-dkms-git`` 包依赖 ``linux-headers`` 包，
我们需要根据自己的 Linux 版本安装相应的 headers 包，
然后再安装网卡驱动：

::

    $ pacman -Ss linux-headers
    ...
        Header files and scripts for building modules for Linux44 kernel
    core/linux49-headers 4.9.206-1
        Header files and scripts for building modules for Linux49 kernel
    core/linux53-headers 5.3.15-1 [已安装]
        Header files and scripts for building modules for Linux53 kernel
    core/linux54-headers 5.4.2-1
        Header files and scripts for building modules for Linux54 kernel
    ...

一切安装完毕之后，
重启系统，
wifi 应该就能够正常工作了。

Ubuntu 上的安装方法也是类似的，
可以参考\ `这个帖子 <https://edimax.freshdesk.com/support/solutions/articles/14000065859-how-to-install-ew-7822ulc-ew-7822utc-in-linux-running-kernel-higher-than-v4-15>`_\ 。

.. tip::

    不知道是驱动的问题还是网卡的问题，
    在连接特定 wifi 网络的时候可能会出现无法连接的情况，
    这时删掉已知的 wifi 网络再重新尝试连接即可，
    还不行就多试几次（笑）。

| 在使用 Linux 路上匍匐前进的黄健宏
| 2019.12.28
