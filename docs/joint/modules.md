# 第二阶段：子模块设计

硬件转发引擎的主要模块包括转发逻辑、转发表（路由表）以及邻居缓存。

## 转发表设计与实现

IP 分组转发的依据是转发表。路由器需要使用 IP 分组头部的目标 IP 地址查询转发表，从而确定下一跳接口（出接口）以及下一跳的 IP 地址。其中，查询转发表时需要采用最长前缀匹配算法，其性能要求高，会影响转发速率，应当用硬件实现。根据计算和模拟测试，在本实验框架下，若转发引擎采用流水线设计，查询转发表为流水线的某一级，且为状态机多周期实现，那么在查询转发表的时钟周期数不超过 10 个周期时，小包转发速率才可达到线速。对于转发表的更新操作（插入、更新或删除），其性能要求相对不高，因此可以根据实验者的设计，由软件或者硬件实现。一般地，软件实现更新操作可能较为简单。此外，更新转发表时则应当按照精确匹配算法。

转发表可以采用流水线设计，也可以采用状态机多周期设计。可供参考的数据结构有如下几种：

* 模拟实现 T-CAM（Ternary Content-Addressable Memory）：FPGA 中实现大容量具有挑战，实际中较多使用 T-CAM 专用芯片，但本实验中不推荐。
* 蛮力查找：实现简单，性能不佳，本实验中不推荐。
* 基于树的数据结构
    * Trie：将 IP 地址前缀视作字符串，实现最长前缀匹配。根据教学实践，对于 IPv4 转发表，32 级流水线的 Trie 树（每级恰好访问一层）似乎是一种受欢迎的设计。但请注意，本实验需要实现 IPv6 转发表。此外，实验者可以根据实际路由表前缀长度的分布，合理配置每一级的容量，以达到在该分布下，整体转发表容量的提升。
    * 多叉 Trie、层压缩或路径压缩等压缩 Trie：减少树的深度，优化查询周期数。
    * 其他高级数据结构与算法：Luleå、Poptrie、……
* 基于散列表的数据结构：按照前缀长度将路由条目存放在不同散列表中，查询时按照前缀长度从大到小逐一尝试，精确匹配。

在实现转发表时，实验者可能还需要考虑以下问题：

!!! question "思考"

    1. 数据结构：蛮力查找？T-CAM？Trie？Radix Tree？散列表？其他？
    2. 存储位置：LUTRAM？BRAM？SRAM？
    3. 接口设计：暴露存储？提供插入、删除、更新及查询接口？
    4. 最大容量：最大需要存储多少路由表项？
    5. 如何设计数据结构以及接口才能方便软件修改转发表？

在完成转发引擎的整体搭建前，实验者可以暂不过度追求转发表的大容量和高性能。此处建议实验者先设计一个基础且可以正常工作的简单模块，然后配合一个比较完善的 testbench，用以测试基本功能。实验者还可以用脚本生成一些测试数据，然后在仿真时测试转发表的正确性。

测试时，实验者可以通过一些方式将一些路由表项预置在转发表内。对于 FPGA 片内的 RAM，实验者可以用代码或 IP 核的参数配置内容；一般地，实验者可以写一段逻辑，功能为，在复位后插入几条固定的路由表项。值得说明的是，在实际工作时，实验路由器的转发表初始仅包含数条直连路由，其余路由信息由 RIP 路由协议进行交换，并通过运行在 CPU 上的软件写入转发表。

!!! question "思考"

    1. 为什么要重视 testbench？
    2. 实验板上 BRAM 、 LUTRAM 和 SRAM 资源大致容量如何？
    3. BRAM 和 SRAM 的延迟谁更低？
    4. FPGA 片内都提供了哪几种 RAM？Latency 的配置会有哪些影响？
    5. LUTRAM 与 BRAM 的区别如何？
    6. 如何表示一条直连路由表项？

## 邻居缓存设计与实现

邻居缓存负责缓存 IP 地址与 MAC 地址的对应关系。与转发表不同，其匹配算法采用精确匹配，因此较为简单。根据本实验的实际情况，邻居缓存所需支持的容量无需太大，十余条即可。实现邻居缓存时，实验者可以采用 CAM（Content-Addressable Memory）、蛮力查找或散列表等数据结构，同时可以加入过期自动删除的功能。其中，实验者也可以不实现过期自动删除的功能。与转发表类似，实验者需要为邻居缓存设计好接口，写好 testbench，并进行充分测试。

!!! question "思考"

    1. 如何实现过期自动删除的功能？
    2. IP 地址会有局部性吗？
    3. 如何处理邻居缓存满的情况？

