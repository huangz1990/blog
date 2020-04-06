修改 Ubuntu 软件更新源
====================================

.. image:: image/ubuntu-logo.png

由于设置原因又或者用户疏忽，
某些 Ubuntu 系统在安装完毕之后使用的是国外而不是国内的软件更新源，
导致系统在升级软件的时候无法达到最优的下载速度。
为了解决这个问题，
我们需要将系统的软件更新源修改为本地源。

Ubuntu 在国内有很多源可供选择，
比如清华大学的 TUNA 源就是一个不错的选择。
我们可以根据自己的 Ubuntu 版本在 https://mirror.tuna.tsinghua.edu.cn/help/ubuntu/ 上面找到对应的源地址。

步骤
-----------

Ubuntu 的软件源配置文件为 ``/etc/apt/sources.list`` ，
我们只需要修改这个文件的内容就能够达到改变更新源的目的，
以下是具体的步骤。

1. 备份原来的源文件：

    ::

        # cp /etc/apt/sources.list /etc/apt/sources.list.backup  

2. 了解系统版本：

    ::

        # lsb_release -a
        No LSB modules are available.
        Distributor ID:	Ubuntu
        Description:	Ubuntu 19.10
        Release:	19.10
        Codename:	eoan
        
3. 根据版本（19.10）在上述的 TUNA 帮助页面找到相应的源地址，并复制全部内容：

    ::

        # 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
        deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan main restricted universe multiverse
        # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan main restricted universe multiverse
        deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan-updates main restricted universe multiverse
        # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan-updates main restricted universe multiverse
        deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan-backports main restricted universe multiverse
        # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan-backports main restricted universe multiverse
        deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan-security main restricted universe multiverse
        # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan-security main restricted universe multiverse

        # 预发布软件源，不建议启用
        # deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan-proposed main restricted universe multiverse
        # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ eoan-proposed main restricted universe multiverse

4. 打开文件，将文件内容替换为已复制的内容：

    ::

        # vi /etc/apt/sources.list

5. 从新的软件源中获取包的更新信息：

    ::

        # apt update
        Get:1 https://mirrors.tuna.tsinghua.edu.cn/ubuntu eoan InRelease [255 kB]
        Get:2 https://mirrors.tuna.tsinghua.edu.cn/ubuntu eoan-updates InRelease [97.5 kB]
        Get:3 https://mirrors.tuna.tsinghua.edu.cn/ubuntu eoan-backports InRelease [88.8 kB]
        Get:4 https://mirrors.tuna.tsinghua.edu.cn/ubuntu eoan-security InRelease [97.5 kB]
        Get:5 https://mirrors.tuna.tsinghua.edu.cn/ubuntu eoan/main amd64 Packages [969 kB]
        ...
        Get:80 https://mirrors.tuna.tsinghua.edu.cn/ubuntu eoan-security/multiverse Translation-en [632 B]
        Get:81 https://mirrors.tuna.tsinghua.edu.cn/ubuntu eoan-security/multiverse amd64 c-n-f Metadata [116 B]
        Fetched 43.3 MB in 5s (8,199 kB/s)               
        Reading package lists... Done
        Building dependency tree       
        Reading state information... Done
        All packages are up to date.

6. 更新软件包：

    ::

        # apt upgrade
        Reading package lists... Done
        Building dependency tree
        Reading state information... Done
        Calculating upgrade... Done
        0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.

| 觉得更新系统速度一定要快的黄健宏
| 2020.2.6
