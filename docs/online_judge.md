# 如何进行在线测试

!!! fail "禁止作弊"

    正如创建仓库时的提示，你提交到 GitLab 上的 **所有代码** 将被作为查重的依据，进行横向和纵向的交叉查重，代码库包括了本实验框架启用以来的所有代码。恶意篡改 Git Commit 时间也视为作弊（比如在 DDL 之后把时间调到 DDL 之前）。一旦确认作弊，你将失去 **实验部分所有的分数** 。请三思而后行！

## 编程作业

同学在 TanLabs 上创建好自己的作业仓库后，就可以往自己的仓库提交自己的代码更新。对于每次 `git push`，GitLab CI 都会进行评测，然后把评测结果推送到 TanLabs 上，然后你就可以在 TanLabs 上看到最新的评测信息。GitLab CI 会进行和本地一样的测试（有兴趣可以查看项目根目录中的 `.gitlab-ci.yml`），数据也完全一致。你提交的代码会用于判断你掌握的程度和代码查重。一般来说，你只需要修改每个编程作业题的源文件，其他文件不需要编辑。如果你有编辑的需求，为了防止影响评测的结果，请找助教确认。需要注意的是，`master` 分支是受保护的，你不能进行 Force Push，并且最终成绩的评测也必须是 `master` 分支上的。

!!! attention "DDL"

    截止时间是针对 GitLab 上 CI 评测时间来说的，其中评测的时间很短，评测的时间约等于你 Push 提交代码的时间。所以请勿在 DDL 前修改代码而不 Push，只有在 DDL 之前 Push 到 GitLab 的代码的才有效。如果出现了因为 GitLab 自身问题导致超过 DDL 的，请联系助教。

对于同学提交的作业，利用 GitLab CI，在仓库中设置了 `.gitlab-ci.yml` 文件，评测的流程：

1. 同学修改代码，通过 GitLab CI 编译和评测
2. 评测结果保留在 GitLab 的 Artifacts 中
3. 评测完成时，通过 WebHook 通知 TanLabs，TanLabs 从 GitLab 取回评测结果
4. 同学在 TanLabs 上浏览评测信息，选择作为最终成绩的评测结果

## 真机测试

在 TanLabs 上，你可以提交真机测试，只需要选择你的 Git Commit，就可以进行个人的真机测试。对于组队测试，需要选择三位同学的 Git Commit，进行一次真机测试。

教学团队在实验室部署了树莓派集群，当同学提交评测任务的时候：

1. 分配若干个树莓派，下发涉及到的代码和配置，对应拓扑中的各个网络设备
2. 分配一个 VLAN 编号，在树莓派上创建对应的 VLAN Interface，放到 netns 中
3. 在各个树莓派上依次完成评测所需要完成的任务
4. 回传日志、评测结果和抓包结果
