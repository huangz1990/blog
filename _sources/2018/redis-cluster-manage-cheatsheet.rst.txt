Redis 集群管理常见操作一览
==================================

Redis 内置客户端 ``redis-cli`` 通过 ``--cluster`` 选项集成了多个 Redis 集群管理工具，
这些工具可以执行创建集群、向集群中添加或移除节点、对集群实施重分片以及负载均衡等操作，
它们每一个都非常强大：

::

    $ redis-cli --cluster help
    Cluster Manager Commands:
      create         host1:port1 ... hostN:portN
                     --cluster-replicas <arg>
      check          host:port
      info           host:port
      fix            host:port
      reshard        host:port
                     --cluster-from <arg>
                     --cluster-to <arg>
                     --cluster-slots <arg>
                     --cluster-yes
                     --cluster-timeout <arg>
                     --cluster-pipeline <arg>
      rebalance      host:port
                     --cluster-weight <node1=w1...nodeN=wN>
                     --cluster-use-empty-masters
                     --cluster-timeout <arg>
                     --cluster-simulate
                     --cluster-pipeline <arg>
                     --cluster-threshold <arg>
      add-node       new_host:new_port existing_host:existing_port
                     --cluster-slave
                     --cluster-master-id <arg>
      del-node       host:port node_id
      call           host:port command arg arg .. arg
      set-timeout    host:port milliseconds
      import         host:port
                     --cluster-from <arg>
                     --cluster-copy
                     --cluster-replace
      help

    For check, fix, reshard, del-node, set-timeout you can specify the host and port of any working node in the cluster. 

本文将对上述工具的其中一些进行介绍，
帮助大家初步地了解这些工具的基本用法。


创建集群
-------------

创建集群的工作可以通过 ``create`` 命令来完成，
用户只需要给定各个节点的地址以及想要设置的从服务器数量即可。
比如说，
通过执行以下命令，
我们可以创建出一个由三个主节点和三个从节点组成的集群：

::

    $ redis-cli --cluster create 127.0.0.1:30001 127.0.0.1:30002 127.0.0.1:30003 127.0.0.1:30004 127.0.0.1:30005 127.0.0.1:30006 --cluster-replicas 1
    >>> Performing hash slots allocation on 6 nodes...
    Master[0] -> Slots 0 - 5460
    Master[1] -> Slots 5461 - 10922
    Master[2] -> Slots 10923 - 16383
    Adding replica 127.0.0.1:30004 to 127.0.0.1:30001
    Adding replica 127.0.0.1:30005 to 127.0.0.1:30002
    Adding replica 127.0.0.1:30006 to 127.0.0.1:30003
    >>> Trying to optimize slaves allocation for anti-affinity
    [WARNING] Some slaves are in the same host as their master
    M: 4979f8583676c46039672fb7319e917e4b303707 127.0.0.1:30001
       slots:[0-5460] (5461 slots) master
    M: 4ff303d96f5c7436ce8ce2fa6e306272e82cd454 127.0.0.1:30002
       slots:[5461-10922] (5462 slots) master
    M: 07e230805903e4e1657743a2e4d8811a59e2f32f 127.0.0.1:30003
       slots:[10923-16383] (5461 slots) master
    S: 4788fd4d92387fc5d38a2cd12f0c0d80fc0f6609 127.0.0.1:30004
       replicates 4979f8583676c46039672fb7319e917e4b303707
    S: b45a7f4355ea733a3177b89654c10f9c31092e92 127.0.0.1:30005
       replicates 4ff303d96f5c7436ce8ce2fa6e306272e82cd454
    S: 7c56ffba63e3758bc4c2e9b6a55caf294bb21650 127.0.0.1:30006
       replicates 07e230805903e4e1657743a2e4d8811a59e2f32f
    Can I set the above configuration? (type 'yes' to accept): yes
    >>> Nodes configuration updated
    >>> Assign a different config epoch to each node
    >>> Sending CLUSTER MEET messages to join the cluster
    Waiting for the cluster to join
    ..
    >>> Performing Cluster Check (using node 127.0.0.1:30001)
    M: 4979f8583676c46039672fb7319e917e4b303707 127.0.0.1:30001
       slots:[0-5460] (5461 slots) master
       1 additional replica(s)
    S: 4788fd4d92387fc5d38a2cd12f0c0d80fc0f6609 127.0.0.1:30004
       slots: (0 slots) slave
       replicates 4979f8583676c46039672fb7319e917e4b303707
    S: b45a7f4355ea733a3177b89654c10f9c31092e92 127.0.0.1:30005
       slots: (0 slots) slave
       replicates 4ff303d96f5c7436ce8ce2fa6e306272e82cd454
    S: 7c56ffba63e3758bc4c2e9b6a55caf294bb21650 127.0.0.1:30006
       slots: (0 slots) slave
       replicates 07e230805903e4e1657743a2e4d8811a59e2f32f
    M: 4ff303d96f5c7436ce8ce2fa6e306272e82cd454 127.0.0.1:30002
       slots:[5461-10922] (5462 slots) master
       1 additional replica(s)
    M: 07e230805903e4e1657743a2e4d8811a59e2f32f 127.0.0.1:30003
       slots:[10923-16383] (5461 slots) master
       1 additional replica(s)
    [OK] All nodes agree about slots configuration.
    >>> Check for open slots...
    >>> Check slots coverage...
    [OK] All 16384 slots covered.


查看集群信息
----------------

通过执行 ``info`` 命令，
用户可以看到集群包含的主从节点数量，
以及集群中的键、槽分布信息：

::

    $ redis-cli --cluster info 127.0.0.1:30001
    127.0.0.1:30001 (4979f858...) -> 1 keys | 5461 slots | 1 slaves.
    127.0.0.1:30002 (4ff303d9...) -> 4 keys | 5462 slots | 1 slaves.
    127.0.0.1:30003 (07e23080...) -> 3 keys | 5461 slots | 1 slaves.
    [OK] 8 keys in 3 masters.
    0.00 keys per slot on average.


添加节点/移除节点
------------------------

在拥有了集群之后，
用户就可以通过执行 ``add-node`` 命令或者 ``del-node`` 命令，
向集群里面添加新节点又或者移除已有的节点。

比如说，
通过执行以下命令，
我们可以把节点 30007 添加至集群当中：

::

    redis-cli --cluster add-node 127.0.0.1:30007 127.0.0.1:30001
    >>> Adding node 127.0.0.1:30007 to cluster 127.0.0.1:30001
    >>> Performing Cluster Check (using node 127.0.0.1:30001)
    ...
    [OK] All nodes agree about slots configuration.
    >>> Check for open slots...
    >>> Check slots coverage...
    [OK] All 16384 slots covered.
    >>> Send CLUSTER MEET to node 127.0.0.1:30007 to make it join the cluster.
    [OK] New node added correctly.

又或者通过执行以下命令，
将 ID 为 ``e1971e...`` 的节点 30007 移出集群：

::
    
    redis-cli --cluster del-node 127.0.0.1:30001 e1971eef02709cf4698a6fcb09935a910982ab3b
    >>> Removing node e1971eef02709cf4698a6fcb09935a910982ab3b from cluster 127.0.0.1:30001
    >>> Sending CLUSTER FORGET messages to the cluster...
    >>> SHUTDOWN the node.


重分片
----------

通过执行 ``reshard`` 命令，
用户可以将指定数量的槽从一个节点迁移至另一个节点。
比如说，
通过执行以下命令，
我们可以将 ID 为 ``4979f...`` 的节点 30001 目前负责的其中 10 个槽转交给 ID 为 ``1cd87d...`` 的节点 30007 负责：

::

    $ redis-cli --cluster reshard 127.0.0.1:30001 --cluster-from 4979f8583676c46039672fb7319e917e4b303707 --cluster-to 1cd87d132101893b7aa2b81cf333b2f7be9e1b75 --cluster-slots 10
    >>> Performing Cluster Check (using node 127.0.0.1:30001)
    M: 4979f8583676c46039672fb7319e917e4b303707 127.0.0.1:30001
       slots:[0-5460] (5461 slots) master
       1 additional replica(s)
    S: 4788fd4d92387fc5d38a2cd12f0c0d80fc0f6609 127.0.0.1:30004
       slots: (0 slots) slave
       replicates 4979f8583676c46039672fb7319e917e4b303707
    S: b45a7f4355ea733a3177b89654c10f9c31092e92 127.0.0.1:30005
       slots: (0 slots) slave
       replicates 4ff303d96f5c7436ce8ce2fa6e306272e82cd454
    M: 1cd87d132101893b7aa2b81cf333b2f7be9e1b75 127.0.0.1:30007
       slots: (0 slots) master
    S: 7c56ffba63e3758bc4c2e9b6a55caf294bb21650 127.0.0.1:30006
       slots: (0 slots) slave
       replicates 07e230805903e4e1657743a2e4d8811a59e2f32f
    M: 4ff303d96f5c7436ce8ce2fa6e306272e82cd454 127.0.0.1:30002
       slots:[5461-10922] (5462 slots) master
       1 additional replica(s)
    M: 07e230805903e4e1657743a2e4d8811a59e2f32f 127.0.0.1:30003
       slots:[10923-16383] (5461 slots) master
       1 additional replica(s)
    [OK] All nodes agree about slots configuration.
    >>> Check for open slots...
    >>> Check slots coverage...
    [OK] All 16384 slots covered.

    Ready to move 10 slots.
      Source nodes:
        M: 4979f8583676c46039672fb7319e917e4b303707 127.0.0.1:30001
           slots:[0-5460] (5461 slots) master
           1 additional replica(s)
      Destination node:
        M: 1cd87d132101893b7aa2b81cf333b2f7be9e1b75 127.0.0.1:30007
           slots: (0 slots) master
      Resharding plan:
        Moving slot 0 from 4979f8583676c46039672fb7319e917e4b303707
        Moving slot 1 from 4979f8583676c46039672fb7319e917e4b303707
        Moving slot 2 from 4979f8583676c46039672fb7319e917e4b303707
        Moving slot 3 from 4979f8583676c46039672fb7319e917e4b303707
        Moving slot 4 from 4979f8583676c46039672fb7319e917e4b303707
        Moving slot 5 from 4979f8583676c46039672fb7319e917e4b303707
        Moving slot 6 from 4979f8583676c46039672fb7319e917e4b303707
        Moving slot 7 from 4979f8583676c46039672fb7319e917e4b303707
        Moving slot 8 from 4979f8583676c46039672fb7319e917e4b303707
        Moving slot 9 from 4979f8583676c46039672fb7319e917e4b303707
    Do you want to proceed with the proposed reshard plan (yes/no)? yes
    Moving slot 0 from 127.0.0.1:30001 to 127.0.0.1:30007:
    Moving slot 1 from 127.0.0.1:30001 to 127.0.0.1:30007:
    Moving slot 2 from 127.0.0.1:30001 to 127.0.0.1:30007:
    Moving slot 3 from 127.0.0.1:30001 to 127.0.0.1:30007:
    Moving slot 4 from 127.0.0.1:30001 to 127.0.0.1:30007:
    Moving slot 5 from 127.0.0.1:30001 to 127.0.0.1:30007:
    Moving slot 6 from 127.0.0.1:30001 to 127.0.0.1:30007:
    Moving slot 7 from 127.0.0.1:30001 to 127.0.0.1:30007:
    Moving slot 8 from 127.0.0.1:30001 to 127.0.0.1:30007:
    Moving slot 9 from 127.0.0.1:30001 to 127.0.0.1:30007:


负载均衡
------------

当集群中各个节点的槽配置趋于不均衡的时候，
各个节点负担的载荷就可能会出现区别，
并导致某些节点超负荷运行，
而某些节点则长时间空置。
为了解决这个问题，
用户可以通过执行 ``rebalance`` 命令，
让各个节点负责的槽数量重新回到平均状态。

比如说，
对于以下这个由三个节点组成，
并且三个节点分别负责 2000 、 11384 和 3000 个槽的集群来说：

::

    $ redis-cli --cluster info 127.0.0.1:30001
    127.0.0.1:30001 (61d1f17b...) -> 0 keys | 2000 slots | 1 slaves.
    127.0.0.1:30002 (101fbae9...) -> 0 keys | 11384 slots | 1 slaves.
    127.0.0.1:30003 (31822e0a...) -> 0 keys | 3000 slots | 1 slaves.
    [OK] 0 keys in 3 masters.
    0.00 keys per slot on average.

我们可以通过执行以下命令来重新分配各个节点负责的槽：

::

    $ redis-cli --cluster rebalance 127.0.0.1:30001
    >>> Performing Cluster Check (using node 127.0.0.1:30001)
    [OK] All nodes agree about slots configuration.
    >>> Check for open slots...
    >>> Check slots coverage...
    [OK] All 16384 slots covered.
    >>> Rebalancing across 3 nodes. Total weight = 3.00
    Moving 3462 slots from 127.0.0.1:30002 to 127.0.0.1:30001
    #####...#####
    Moving 2461 slots from 127.0.0.1:30002 to 127.0.0.1:30003
    #####...#####

在复杂均衡操作执行完毕之后，
集群各个节点负责的槽数量将趋于平均，
这样它们各自负担的载荷也会重新回到平均状态：

::

    $ redis-cli --cluster info 127.0.0.1:30001
    127.0.0.1:30001 (61d1f17b...) -> 0 keys | 5462 slots | 1 slaves.
    127.0.0.1:30002 (101fbae9...) -> 0 keys | 5461 slots | 1 slaves.
    127.0.0.1:30003 (31822e0a...) -> 0 keys | 5461 slots | 1 slaves.
    [OK] 0 keys in 3 masters.
    0.00 keys per slot on average.


在集群的所有节点中执行命令
-----------------------------

如果用户想要对集群中的所有节点执行同一个命令，
那么可以通过 ``call`` 命令来完成。
比如说，
通过执行以下命令，
用户就可以让集群中的每个节点都执行 ``DBSIZE`` 命令，
从而得到各个节点各自的数据库大小：

::

    $ redis-cli --cluster call 127.0.0.1:30001 DBSIZE
    >>> Calling DBSIZE
    127.0.0.1:30001: (integer) 0

    127.0.0.1:30004: (integer) 0

    127.0.0.1:30005: (integer) 3

    127.0.0.1:30006: (integer) 0

    127.0.0.1:30002: (integer) 3

    127.0.0.1:30003: (integer) 0


小结
---------

好的，
关于 ``redis-cli --cluster`` 选项的简单介绍就到此结束了，
希望这些介绍可以帮助大家更好地了解到这些命令。
大家有兴趣的话，
也可以以此为起点，
更加深入地学习这些命令。

本文节选自尚未发布的《Redis使用教程》（\ `RedisGuide.com <http://redisguide.com>`_\ ），
大家有兴趣的话也可以关注一下本书，
里面有关于 ``redis-cli --cluster`` 工具更深入和更详细的介绍。

| 黄健宏
| 2018.11.21
