# 在 SUSE Linux Enterprise Server 12 上安装 Oracle RAC (Oracle Real Application Clusters, Oracle实时应用集群)

文档原作者：
- Chen Wu, ISV 技术经理, SUSE
- Arun Singh, ISV 技术经理, SUSE

## Oracle RAC 12.1.0.2.0 和 SUSE Linux Enterprise Server 12 (x86_64)

在数据库计算领域，Oracle RAC 为Oracle数据库环境下的集群和高可用性提供软件支持。Oracle提供了RAC软件标准版本，其中节点通过使用Oracle Clusterware软件组成集群。

SUSE Linux Enterprise Server 为Oracle客户提供了一个可拓展平台，以适应诸如Oracle数据库这样的~~高要求工作栈和应用~~。Oracle和SUSE一道为Linux提供了Oracle数据库的支持，并为今天IT数据中心提供~~real-grid benefits~~ 提供了一流的解决方案。这篇文档主要为如何在SUSE Linux Enterprise Server 12 操作系统上安装Oracle RAC 12.1.0.2.0 提供细节上的帮助。

## 1.0 介绍

这篇文档为在 SUSE Linux Enterprise Server 12 上安装 Oracle RAC 12.1.0.2.0 提供帮助。在下面的例子中，我们将使用x86_64版本的 Oracle Database 12c Enterprise 和 SUSE Linux Enterprise Server。相似的步骤也适用于其他如x86，IA64等平台。如有任何问题，~~请向...发送您的请求。~~

Oracle产品的官方文档可在此网站获取：http://docs.oracle.com/en/

## 2.0 硬件和软件配置要求

硬件要求：

![表1 硬件配置要求](_v_images/表1硬件配置要求_1539143551_7759.png)

软件要求：

SUSE：
- [SUSE Linux Enterprise Server 12 SP1 (x86_64)](http://download.suse.de/install)

Oracle：
- [Database 12c Enterprise/Standard Editions (x86_64)](http://www.oracle.com/technetwork/indexes/downloads/index.html#database)

## 3.0 关于四节点测试集群

- HP DL360 Gen9 服务器 (Intel Xeon 2x12 core ~ 48 CPU)，配备64GB内存
- 每台服务器配备4个网卡 (~~two bonded as active/passive~~), 及一个静态IP地址
- 本地硬盘 (500 GB)
- 三个共享的SAN分区 ( ASM为30 GB，NFS为400 GB, 其他为600 B)
- SUSE Linux Enterprise Server 12 SP1 (x86_64)
- 内核版本: 3.12.49-11-default

## 4.0 前提

### 4.1 安装SUSE Linux Enterprise Server 12

在集群的每个节点上都安装 SUSE Linux Enterprise Server 12。具体的步骤可以在以下网站中参考官方文档：https://www.suse.com/documentation/sles-12/

### 4.2 网络配置

在安装 Oracle RAC 过程中，网络配置应按以下参数进行：

```shell
            #Private:
            10.1.1.1        c2n1-priv
            10.1.1.2        c2n2-priv
            10.1.1.3        c2n3-priv
            10.1.1.4        c2n4-priv
            
            #Public:
            137.65.135.72   c2n1.provo.novell.com c2n1
            137.65.135.73   c2n2.provo.novell.com c2n2
            137.65.135.74   c2n3.provo.novell.com c2n3
            137.65.135.75   c2n4.provo.novell.com c2n4
                
            #Virtual
            137.65.135.76		c2n1-vip	c2n1-vip.provo.novell.com
            137.65.135.77		c2n2-vip	c2n2-vip.provo.novell.com
            137.65.135.78		c2n3-vip	c2n3-vip.provo.novell.com
            137.65.135.79		c2n4-vip	c2n4-vip.provo.novell.com
                
            #SCAN:
            c2-scan.provo.novell.com (137.65.135.87)
            c2-scan.provo.novell.com (137.65.135.148)
            c2-scan.provo.novell.com (137.65.135.149)
```

## 5.0 安装 Oracle RAC

下面的章节将向您介绍如何安装Oracle RAC。

### 5.1 安装Oracle Grid Infrastructure

#### 准备工作

作为非管理员用户登录到 SUSE Linux Enterprise Server 12 64位操作系统中，并下载相应版本（Linux x86-64）的 Oracle Database 12c Release 1 Grid Infrastructure (12.1.0.2.0)。

解压 grid.zip 文件并从 Grid ShipHome 运行安装文件'./runinstaller'。

#### 安装

实际的安装过程请参考以下步骤：

1. 选择 '安装选项'

![图1 安装选项](_v_images/图1安装选项_1539232732_15431.png)

选择 '为集群安装并配置Oracle Grid Infrastructure’，然后点击‘下一步’。

2. 进入下一项‘集群类型’

![图2 集群类型](_v_images/图2集群类型_1539233829_18084.png)

选择‘配置一个标准集群’，然后点击‘下一步’。

3. 进入下一项‘安装类型’

![图3 安装类型](_v_images/图3安装类型_1539233941_10431.png)

选择‘高级安装’，然后点击‘下一步’

4. 进入下一项‘产品语言’

![图4 产品语言](_v_images/图4产品语言_1539233995_26824.png)

选择所有语言，然后点击‘下一步’

5. 填写必要的‘Grid plug and play information’

![图5 Grid Plug and Play Information](_v_images/图5gridplug_1539234083_30670.png)

按照截图中的信息进行填写，然后点击‘下一步’

关于GNS设置的更多信息请在以下网站中参考官方Oracle文档：https://docs.oracle.com/database/121/CWLIN/toc.htm

6. 进入下一项‘集群节点信息’

![图6 集群节点信息](_v_images/图6集群节点信息_1539234292_5838.png)

为所有节点填入相应的公共主机名和虚拟主机名，然后点击‘下一步’。

7. 配置‘网络接口使用’

![图7 网络接口使用](_v_images/图7网络接口使用_1539234463_26953.png)

填写Oracle Grid使用接口的相关具体信息，然后点击‘下一步’。

8. 填写‘存储选项信息’

![图8 存储选项信息](_v_images/图8存储选项信息_1539237395_19400.png)

选择‘使用标准ASM用于存储’，然后点击‘下一步’。

9. 进入下一项‘创建ASM磁盘组’

![图9 创建ASM磁盘组](_v_images/图9创建asm磁盘组_1539237470_23050.png)

根据您的需求，创建一个ASM磁盘组，然后点击‘下一步’。

10. 设置‘ASM密码’
![图10 ASM密码](_v_images/图10asm密码_1539237546_11997.png)

按照截图填写ASM密码，然后点击‘下一步’

11. 定义‘故障隔离支持’

![图11 故障隔离支持](_v_images/图11故障隔离支持_1539237645_17226.png)

选择‘不要使用IPMI’，然后点击‘下一步’。

12. 配置‘管理选项’

![图12 管理选项](_v_images/图12管理选项_1539237733_22594.png)

根据需求选择或不选‘注册EM’这一项，然后进入下一步。

13. 进入下一项‘优先操作系统组’

![图13 优先操作系统组](_v_images/图13优先操作系统组_1539237930_16847.png)

操作系统组是默认选择的。点击‘下一步’继续。

14. 设置‘安装路径’

![图14 安装路径](_v_images/图14安装路径_1539238005_28216.png)

填写Oracle base和软件路径，然后点击‘下一步’。

15. 创建‘库存’

![图15 创建库存](_v_images/图15创建库存_1539238188_6923.png)

如果是主机上的第一次安装，请为安装元数据文件选择一个存储路径。进入下一步。

16. 进入下一项‘根脚本执行配置’

![图16 根脚本执行配置](_v_images/图16根脚本执行配置_1539238296_25460.png)

在配置软件过程中，某些操作的执行需要管理员权限。按照上图所示选择选项，安装器将会自动执行这些操作。为管理员配置密码，然后进入下一步。

17. 进入下一项‘条件检查’

![图17 条件检查1](_v_images/图17条件检查1_1539238512_23947.png)

如上图所示进行条件检查，点击‘修复/再次检查’以对系统进行再次检查。

![图18 条件检查2](_v_images/图18条件检查2_1539238592_17669.png)

按照提示符，在每个节点上以管理员权限手动运行修复脚本。点击确认。

```shell
c2n1:/home # /tmp/CVU_12.1.0.2.0_oracle/runfixup.sh
All Fix-up operations were completed successfully.
```

![图19 条件检查3](_v_images/图19条件检查3_1539247037_20875.png)

在修复结果页面中，确认选择‘忽略所有’。然后点击‘下一步’。

18. 查看‘摘要’

![图20 摘要](_v_images/图20摘要_1539247056_15210.png)

查看安装摘要，如上图所示。如果一切正确，点击安装以继续。

19. 安装产品

![图21 安装产品 - 进展](_v_images/图21安装产品进展_1539247085_15844.png)

包将被安装。

![图22 安装产品 - 配置过程](_v_images/图22安装产品配置过_1539247112_23392.png)

将完成‘准备进行相关配置’步骤。此时你需要先安装Oracle Patch 18456643。这之后点击Yes以运行配置脚本。

![图23 安装产品 - 验证工具](_v_images/图23安装产品验证工_1539247147_9378.png)

错误信息‘Oracle集群验证工具失败’将会弹出。忽略这个错误并点击OK继续。

![图24 安装产品 - 设置完成](_v_images/图24安装产品设置完_1539247180_29295.png)

当设置完成时，点击Next以进行下一步。

![图25 安装产品 - 设置完成信息](_v_images/图25安装产品设置完_1539247208_8252.png)

一条错误信息将会弹出，告诉您一些配置帮助失败。点击Yes确认。

![图26 安装产品 - 结束](_v_images/图26安装产品结束_1539247226_21867.png)

现在一个集群的Oracle Grid Infrastructure安装已经完成。

#### 安装后的检查
1. 检查Oracle Clusterware运行状况

```shell
oracle@c2n1:~> /home/grid/bin/crsctl check cluster -all
**************************************************************
c2n1:
CRS-4537: Cluster Ready Services is online
CRS-4529: Cluster Synchronization Services is online
CRS-4533: Event Manager is online
**************************************************************
c2n2:
CRS-4537: Cluster Ready Services is online
CRS-4529: Cluster Synchronization Services is online
CRS-4533: Event Manager is online
**************************************************************
c2n3:
CRS-4537: Cluster Ready Services is online
CRS-4529: Cluster Synchronization Services is online
CRS-4533: Event Manager is online
**************************************************************
c2n4:
CRS-4537: Cluster Ready Services is online
CRS-4529: Cluster Synchronization Services is online
CRS-4533: Event Manager is online
```

2. 检查Oracle Clusterware资源

```shell
oracle@c2n1:~> /home/grid/bin/srvctl status nodeapps
VIP c2n1-vip.provo.novell.com is enabled
VIP c2n1-vip.provo.novell.com is running on node: c2n1
VIP c2n2-vip.provo.novell.com is enabled
VIP c2n2-vip.provo.novell.com is running on node: c2n2
VIP c2n3-vip.provo.novell.com is enabled
VIP c2n3-vip.provo.novell.com is running on node: c2n3
VIP c2n4-vip.provo.novell.com is enabled
VIP c2n4-vip.provo.novell.com is running on node: c2n4
Network is enabled
Network is running on node: c2n1
Network is running on node: c2n4
Network is running on node: c2n3
Network is running on node: c2n2
ONS is enabled
ONS daemon is running on node: c2n1
ONS daemon is running on node: c2n4
ONS daemon is running on node: c2n3
ONS daemon is running on node: c2n2
```

3. 检查指定资源状态

```shell
oracle@c2n1:~> /home/grid/bin/crsctl stat res -t
--------------------------------------------------------------------------------
Name        Target State      Server                   State details
--------------------------------------------------------------------------------
Local Resources
--------------------------------------------------------------------------------
ora.LISTENER.lsnr
         ONLINE  ONLINE       c2n1                     STABLE
         ONLINE  ONLINE       c2n2                     STABLE
         ONLINE  ONLINE       c2n3                     STABLE
         ONLINE  ONLINE       c2n4                     STABLE
ora.SUSETEST.dg
         ONLINE  ONLINE       c2n1                     STABLE
         ONLINE  ONLINE       c2n2                     STABLE
         ONLINE  ONLINE       c2n3                     STABLE
         ONLINE  ONLINE       c2n4                     STABLE
ora.SUSETEST01.dg
         ONLINE  ONLINE       c2n1                     STABLE
         ONLINE  ONLINE       c2n2                     STABLE
         ONLINE  ONLINE       c2n3                     STABLE
         ONLINE  ONLINE       c2n4                     STABLE
ora.asm
         ONLINE  ONLINE       c2n1                     Started,STABLE
         ONLINE  ONLINE       c2n2                     Started,STABLE
         ONLINE  ONLINE       c2n3                     Started,STABLE
         ONLINE  ONLINE       c2n4                     Started,STABLE
ora.net1.network
         ONLINE  ONLINE       c2n1                     STABLE
         ONLINE  ONLINE       c2n2                     STABLE
         ONLINE  ONLINE       c2n3                     STABLE
         ONLINE  ONLINE       c2n4                     STABLE
ora.ons
         ONLINE  ONLINE       c2n1                     STABLE
         ONLINE  ONLINE       c2n2                     STABLE
         ONLINE  ONLINE       c2n3                     STABLE
         ONLINE  ONLINE       c2n4                     STABLE
--------------------------------------------------------------------------------
Cluster Resources
--------------------------------------------------------------------------------
ora.LISTENER_SCAN1.lsnr
    1    ONLINE  ONLINE       c2n4                     STABLE
ora.LISTENER_SCAN2.lsnr
    1    ONLINE  ONLINE       c2n3                     STABLE
ora.LISTENER_SCAN3.lsnr
    1    ONLINE  ONLINE       c2n1                     STABLE
ora.MGMTLSNR
    1    ONLINE  ONLINE       c2n1                     169.254.38.57 10.1.1.1,STABLE
ora.c2n1.vip
    1    ONLINE  ONLINE       c2n1                     STABLE
ora.c2n2.vip
    1    ONLINE  ONLINE       c2n2                     STABLE
ora.c2n3.vip
    1    ONLINE  ONLINE       c2n3                     STABLE
ora.c2n4.vip
    1    ONLINE  ONLINE       c2n4                     STABLE
ora.cvu
    1    ONLINE  ONLINE       c2n1                     STABLE
ora.mgmtdb
    1    ONLINE  ONLINE       c2n1                     Open,STABLE
ora.oc4j
    1    ONLINE  ONLINE       c2n1                     STABLE
ora.scan1.vip
    1    ONLINE  ONLINE       c2n4                     STABLE
ora.scan2.vip
    1    ONLINE  ONLINE       c2n3                     STABLE
ora.scan3.vip
    1    ONLINE  ONLINE       c2n1                     STABLE
--------------------------------------------------------------------------------
```

4. 检查OCR和Voting磁盘文件

```shell
oracle@c2n1:~> /home/grid/bin/ocrcheck
Status of Oracle Cluster Registry is as follows :
        Version                  :          4
        Total space (kbytes)     :     409568
        Used space (kbytes)      :       1696
        Available space (kbytes) :     407872
        ID                       : 1286433802
        Device/File Name         :  +SUSETEST
                     Device/File integrity check succeeded
    
                     Device/File not configured
    
                     Device/File not configured
    
                     Device/File not configured
    
                     Device/File not configured
    
        Cluster registry integrity check succeeded
    
        Logical corruption check bypassed due to non-privileged user   
        
oracle@c2n1:~> /home/grid/bin/crsctl query css votedisk
##  STATE    File Universal Id                File Name Disk group
--  -----    -----------------                --------- ---------
1. ONLINE   3195ba343d234f0cbf92bf2df0f7d3b9 (/dev/oradata/disk1) [SUSETEST]
2. ONLINE   3410554dce114fa1bfb86b8cd1f3288b (/dev/oradata/disk2) [SUSETEST]
3. ONLINE   69bb2fac7bac4f71bf4b3ec63154f76c (/dev/oradata/disk3) [SUSETEST]
Located 3 voting disk(s).        
```

### 5.2 安装Oracle数据库

#### 准备

作为非管理员用户登录到 SUSE Linux Enterprise Server 12 64位操作系统中，并下载相应版本（Linux x86-64）的 Oracle Database 12c Release 1 (12.1.0.2.0)。

解压 grid.zip 文件并从 Grid ShipHome 运行安装文件'./runinstaller'。

#### 安装

实际的安装过程请参考以下步骤：

1. 选择‘配置安全更新’

![图27 配置安全更新](_v_images/图27配置安全更新_1539247263_11660.png)

填写接收安全问题提醒的邮箱地址，然后下一步。

2. 进入下一项‘安装选项’

![图28 安装选项](_v_images/图28安装选项_1539247277_25551.png)

选择‘只安装数据库软件’，然后下一步。

3. 进入下一项‘Grid 安装选项’

![图29 Grid安装选项](_v_images/图29grid安装选_1539247297_21504.png)

选择‘Oracle 实时应用集群数据库安装’，然后下一步。

4. 在‘节点选择’中选择一系列节点

![图30 节点选择](_v_images/图30节点选择_1539247313_7597.png)

选择集群中所有节点，然后下一步。

5. 进行下一项‘产品语言’

![图31 产品语言](_v_images/图31产品语言_1539247327_14424.png)

选择所有语言，然后下一步。

6. 选择‘数据库版本’

![图32 数据库版本](_v_images/图32数据库版本_1539247341_28142.png)

选择‘企业版’，然后下一步。

7. 配置‘安装路径’

![图33 安装路径](_v_images/图33安装路径_1539247356_13915.png)

配置Oracle base和软件路径，如上图所示，然后下一步。

8. 设置优先操作系统组

![图34 操作系统组](_v_images/图34操作系统组_1539247371_29624.png)

选择默认设置，然后下一步。

9. 执行条件检查

![图35 条件检查](_v_images/图35条件检查_1539247387_16677.png)

如上图执行条件检查，点击‘修复&再次检查’以继续。

![图36 修复脚本](_v_images/图36修复脚本_1539247414_11263.png)

参照提示符，在每个节点上以管理员权限手动运行修复脚本，然后点击OK。

你应该会看到如下输出：

```shell
c2n1 ~ #: /tmp/CVU_12.1.0.2.0_oracle/runfixup.sh
All Fix-up operations were completed successfully.
```

查看检查结果：

![图37 检查结果](_v_images/图37检查结果_1539247465_1792.png)

查看验证结果：

![图38 验证结果](_v_images/图38验证结果_1539247480_9344.png)

选择‘忽略所有’，然后下一步。

10. 查看摘要

![图39 摘要](_v_images/图39摘要_1539247492_31973.png)

如果摘要如上图所示，点击安装。

11. 进入下一项‘安装产品’

![图40 安装产品](_v_images/图40安装产品_1539247505_27580.png)

当产品加载时，会有窗口弹出以执行你的配置脚本。

![图41 执行配置脚本](_v_images/图41执行配置脚本_1539247525_22290.png)

在每个集群节点上以管理员权限执行root.sh，然后点击OK继续。

你会看到如下输出：

```shell
c2n1:~ # /home/oracle/app/product/12.1.0/dbhome_1/root.sh
Performing root user operation.

The following environment variables are set as:
    ORACLE_OWNER=oracle
    ORACLE_Home=/home/oracle/app/product/12.1.0/dbhome_1
    
Enter the full pathname of the local bin directory: [/usr/local/bin]
The contents of "dbhome" have not changed. No need to overwrite.
The contents of "oraenv" have not changed. No need to overwrite.
The contents of "coraenv" have not changed. No need to overwrite.

Entries will be added to the /etc/oratab file as needed by
Database Configuration Assistant when a database is created
Finished running generic part of root script.
Now product-specific root actions will be performed.
```

下面的截图意味着安装成功。

![图42 结束](_v_images/图42结束_1539247541_19889.png)

Oracle数据库的安装到此结束，点击Close关闭。

#### 创建ASM磁盘组

使用ASM配置助手创建ASM磁盘组，用于数据文件存储。

![图43 ASM配置助手](_v_images/图43asm配置助手_1539247560_24417.png)

参照执行屏幕显示的操作。

#### 创建Oracle RAC 12.1.0.2数据库

通过使用数据库配置助手，执行以下操作以创建Oracle RAC 12.1.0.2数据库：

1. 选择‘数据库操作’

![图44 数据库操作](_v_images/图44数据库操作_1539247574_11098.png)

选中‘创建数据库’，然后下一步。

2. 进入下一项‘创建模式’

![图45 创建模式](_v_images/图45创建模式_1539247591_24386.png)

选择‘高级模式’，然后下一步。

3. 配置‘数据库模版’

![图46 数据库模版](_v_images/图46数据库模版_1539247603_27461.png)

选择你想配置的数据库类型，然后下一步。

4. 进入下一项‘数据库标识’

![图47 数据库标识](_v_images/图47数据库标识_1539247616_17022.png)

如上图填写‘全局数据库名称’，然后下一步。

```note
备注：容器数据库
Oracle 数据库12C 支持创建容器数据库，可根据需求创建。
```

5. 进入下一项‘数据库存放’

![图48 数据库存放](_v_images/图48数据库存放_1539247644_13244.png)

如上图所示，检查~~服务器池~~（server pools）信息，然后下一步。

6. 设置‘管理选项’

![图49 管理选项](_v_images/图49管理选项_1539247657_311.png)

为数据库设置管理选项，然后下一步。

7. 进入下一项‘数据库凭据’

![图50 数据库凭据](_v_images/图50数据库凭据_1539247674_25429.png)

为数据库设置管理员密码，然后下一步。

8. 进入下一项‘存储路径’

![图51 存储路径](_v_images/图51存储路径_1539247687_4798.png)

如上图配置数据库文件存储路径，然后下一步。

9. 进入下一项‘数据库选项’

![图52 数据库选项](_v_images/图52数据库选项_1539247700_8935.png)

根据需求选择是否添加，然后下一步。

10. 进入下一项‘初始化参数’

![图53 初始化参数](_v_images/图53初始化参数_1539247714_30770.png)

选择‘典型设置’，并根据需求调整参数，然后下一步。

11. 进入下一项‘创建选项’

![图54 创建选项](_v_images/图54创建选项_1539247728_1071.png)

如上图所示配置数据库创建选项，然后下一步。

12. 执行‘条件检查’

![图55 条件检查](_v_images/图55条件检查_1539247740_9481.png)

选择‘忽略所有’，然后下一步。

13. 检查‘摘要’

![图56 摘要](_v_images/图56摘要_1539247756_31414.png)

检查确认数据库配置摘要中的信息，如上图所示。点击‘结束’

14. 检查‘进度页’

![图57 进度页](_v_images/图57进度页_1539247772_31986.png)

数据库的创建过程如上图所示，等待创建过程结束即可。

15. ‘结束’

![图58 结束](_v_images/图58结束_1539247785_10410.png)

当数据库创建完成时，如上图所示会有一些信息提示。点击‘关闭’。

#### 安装后的检查

1. 确认数据库状态和配置

```shell
oracle@c2n1:~> export ORACLE_HOME=/home/oracle/app/product/12.1.0/dbhome_1
oracle@c2n1:~> /home/oracle/app/product/12.1.0/dbhome_1/bin/srvctl status database -d susedb
Instance SUSEDB_1 is running on node c2n2
Instance SUSEDB_2 is running on node c2n3
Instance SUSEDB_3 is running on node c2n4
Instance SUSEDB_4 is running on node c2n1
```

```shell
oracle@c2n1:~> /home/oracle/app/product/12.1.0/dbhome_1/bin/srvctl config database -d susedb -a
Database unique name: SUSEDB
Database name: SUSEDB
Oracle home: /home/oracle/app/product/12.1.0/dbhome_1
Oracle user: oracle
Spfile: +SUSETEST01/SUSEDB/PARAMETERFILE/spfile.274.908484433
Password file: +SUSETEST01/SUSEDB/PASSWORD/pwdsusedb.256.908484093
Domain:
Start options: open
Stop options: immediate
Database role: PRIMARY
Management policy: AUTOMATIC
Server pools: susepool
Disk Groups: SUSETEST01
Mount point paths:
Services:
Type: RAC
Start concurrency:
Stop concurrency:
Database is enabled
Database is individually enabled on nodes:
Database is individually disabled on nodes:
OSDBA group: dba
OSOPER group:
Database instances:
Configured nodes:
Database is policy managed
```

```shell

SUSE DOCUMENTATION
SUSE DOCUMENTATION: SUSE BEST PRACTICES DOCUMENTATION
Installing Oracle RAC on SUSE Linux Enterprise Server 12
1.0 Introduction
2.0 Hardware and Software Requirements
3.0 Testing 4-Node Cluster Information
4.0 Prerequisites
5.0 Oracle RAC Installation
6.0 Additional Comments
7.0 GNU Free Documentation License
Installing Oracle RAC on SUSE Linux Enterprise Server 12
Oracle RAC 12.1.0.2.0 and SUSE Linux Enterprise Server 12 (x86_64)
SUSE Linux Enterprise Server
Chen Wu, ISV Technical Manager, SUSE

Arun Singh, ISV Technical Manager, SUSE

In database computing, Oracle Real Application Clusters (RAC) provides software for clustering and high availability in Oracle database environments. Oracle includes RAC with the Standard Edition, provided the nodes are clustered using Oracle Clusterware.

SUSE Linux Enterprise Server provides Oracle customers a scalable platform for demanding workloads and applications such as Oracle Database. Oracle and SUSE have teamed up to enable Linux in enterprises with Oracle Database and have created a world-class solution that offers real-grid benefits for IT data centers today. This document provides the details for installing Oracle RAC 12.1.0.2.0 on SUSE Linux Enterprise Server 12 OS.

1.0 Introduction#
This document provides the details for installing Oracle RAC 12.1.0.2.0 on SUSE Linux Enterprise Server 12. For the example at hand, the x86_64 version of both the Oracle Database 12c Enterprise and SUSE Linux Enterprise Server is used. Similar steps apply to other platforms (x86, IA64, etc.). If you encounter any problem or have general questions, submit your query to .

The official Oracle product documentation is available at: http://docs.oracle.com/en/

2.0 Hardware and Software Requirements#
Hardware Requirements
Table 1 Hardware Requirements

Requirement

Minimum

RAM

32 GB

Swap space

Approximately twice the size of RAM

Disk space in /tmp

8 GB

Disk space for software files

8 GB

Disk space for database files

8 GB

Software Requirements
SUSE:

SUSE Linux Enterprise Server 12 SP1 (x86_64)

(http://download.suse.de/install)

Oracle:

Database 12c Enterprise/Standard Editions (x86_64)

(http://www.oracle.com/technetwork/indexes/downloads/index.html#database)

3.0 Testing 4-Node Cluster Information#
HP DL360 Gen9 Server (Intel Xeon 2x12 core ~ 48 CPU) with 64GB RAM

4 NIC per server (two bonded as active/passive), plus a Static IP Address

Local HDD (500 GB)

Three shared SAN Partition ( ASM: 30 GB and NFS: 400 GB, Other: 600 B)

SUSE Linux Enterprise Server 12 SP1 (x86_64)

Kernel version: 3.12.49-11-default

4.0 Prerequisites#
4.1 Installing SUSE Linux Enterprise Server 12#
Install SUSE Linux Enterprise Server 12 on each node of the cluster. To do so, follow the instructions in the official SUSE Linux Enterprise Server documentation at https://www.suse.com/documentation/sles-12/

4.2 Network Configuration#
The network configuration for your Oracle RAC installation should look as follows:

            #Private:
            10.1.1.1        c2n1-priv 
            10.1.1.2        c2n2-priv 
            10.1.1.3        c2n3-priv 
            10.1.1.4        c2n4-priv 
                
            #Public:
            137.65.135.72   c2n1.provo.novell.com c2n1
            137.65.135.73   c2n2.provo.novell.com c2n2
            137.65.135.74   c2n3.provo.novell.com c2n3
            137.65.135.75   c2n4.provo.novell.com c2n4
                
            #Virtual
            137.65.135.76		c2n1-vip	c2n1-vip.provo.novell.com
            137.65.135.77		c2n2-vip	c2n2-vip.provo.novell.com
            137.65.135.78		c2n3-vip	c2n3-vip.provo.novell.com
            137.65.135.79		c2n4-vip	c2n4-vip.provo.novell.com
                
            #SCAN:
            c2-scan.provo.novell.com (137.65.135.87)
            c2-scan.provo.novell.com (137.65.135.148)
            c2-scan.provo.novell.com (137.65.135.149)             
5.0 Oracle RAC Installation#
The following sections explain how to perform your Oracle RAC installation.

5.1 Installing Oracle Grid Infrastructure#
Preparation#
Log in to the SUSE Linux Enterprise Server 12 64-bit OS as a non-administration user. Download the Oracle Database 12c Release 1 Grid Infrastructure (12.1.0.2.0) for Linux x86-64.

Extract grid.zip and run the installer './runInstaller' from Grid ShipHome.

Installation#
For the actual installation, follow the steps below:

Select Installation Option.

Figure 1 Installation Option


Choose the option Install and Configure Oracle Grid Infrastructure for a Cluster", then click Next to continue.

Proceed to Cluster Type.

Figure 2 Cluster Type


Choose the option Configure a Standard Cluster", then click Next to continue.

Proceed to Installation Type.

Figure 3 Installation Type


Choose the option Advanced Installation", then click Next to continue.

Proceed to Product Languages.

Figure 4 Product Languages


Select all languages, then click Next to continue.

Fill in the required Grid Plug and Play Information.

Figure 5 Grid Plug and Play Information


Fill in the information as displayed in the screenshot, then click Next to continue.

For more details about the GNS configuration consult the official Oracle documentation at https://docs.oracle.com/database/121/CWLIN/toc.htm.

Proceed to Cluster Node Information.

Figure 6 Cluster Node Information


Provide the list of nodes with their public host name and virtual host name, then click Next to continue.

Specify the Network Interface Usage .

Figure 7 Network Interface Usage


Provide the details of the interfaces that are used by Oracle Grid for public and private traffic, then click Next to continue.

Fill in the Storage Option Information.

Figure 8 Storage Option Information


Choose the option Use Standard ASM for storage, then click Next to continue.

Proceed to Create ASM Disk Group.

Figure 9 Create AMS Disk Group


Depending on your needs, create an ASM Disk Group , then click Next to continue.

Specify the ASM Password.

Figure 10 ASM Password


Fill in the ASM Password as shown in the screenshot above, then click Next to continue.

Define the Failure Isolation Support.

Figure 11 Failure Isolation Support


Choose the option Do not use IPMI, then click Next to continue.

Specify the Management Options.

Figure 12 Management Options


If you select or deselect the option Register with EM depends on your needs. After you made your choice click Next to continue.

Proceed to Privileged Operating System Groups.

Figure 13 Privileged Operating System Groups


The operating system gorups are selected by default. Click Next to continue.

Specify the Installation Location.

Figure 14 Installation Location


Fill in the Oracle base and the Software location, then click Next to continue.

Create the Inventory.

Figure 15 Create Inventory


If this is your first installation on this host, specify a directory for the installation metadata files. Click Next to continue.

Proceed to Root Script Execution Configuration.

Figure 16 Root Script Execution Configuration


While configuring the software, you need to perform certain operations as root user. Choose the option as shown above. The Installer will then perform these operations automatically. Fill in the password for root user. Click Next to continue.

Proceed to Prerequisite Checks.

Figure 17 Prerequisite Checks 1


Perform the prerequisite check as shown on the screen above. Click Fix&Check Again to recheck the system.

Figure 18 Prerequisite Checks 2


Follow the prompts, and run the fixup script as root manually on each node. Click OK.

c2n1:/home # /tmp/CVU_12.1.0.2.0_oracle/runfixup.sh
All Fix-up operations were completed successfully.
c2n1:/home #
Figure 19 Prerequiste Checks 3


On the Fixup Result tab, checkmark the option Ignore All. Click Next to continue.

Review the Summary.

Figure 20 Summary


Review the installation Summary as shown above. If everything is correct, click Install to continue.

Install the product.

Figure 21 Install Product - Progress


The packages are being installed.

Figure 22 Install Product - Configuration Steps


The action prepare for configuration steps is to be completed. At this point you first need to install the Oracle Patch 18456643. After that, click Yes to run the configuration scripts.

Figure 23 Install Product - Verification Utility


An error message will pop up Oracle Cluster Verification Utility failed. Ignore the error and click OK to continue.

Figure 24 Install Product - Setup Completed


When the setup is completed, click Next to continue.

Figure 25 Install Product - Setup Completed Message


An error message will pop up to inform you about some configuration assistants that failed. Click Yes to confirm.

Figure 26 Install Product - Finish


You have now finished the installation of the Oracle Grid Infrastructure for a Cluster.

Post-Installation Checks#
Check the Oracle Clusterware health:

oracle@c2n1:~> /home/grid/bin/crsctl check cluster -all
**************************************************************
c2n1:
CRS-4537: Cluster Ready Services is online
CRS-4529: Cluster Synchronization Services is online
CRS-4533: Event Manager is online
**************************************************************
c2n2:
CRS-4537: Cluster Ready Services is online
CRS-4529: Cluster Synchronization Services is online
CRS-4533: Event Manager is online
**************************************************************
c2n3:
CRS-4537: Cluster Ready Services is online
CRS-4529: Cluster Synchronization Services is online
CRS-4533: Event Manager is online
**************************************************************
c2n4:
CRS-4537: Cluster Ready Services is online
CRS-4529: Cluster Synchronization Services is online
CRS-4533: Event Manager is online
**************************************************************
Check the Oracle Clusterware resources:

oracle@c2n1:~> /home/grid/bin/srvctl status nodeapps
VIP c2n1-vip.provo.novell.com is enabled
VIP c2n1-vip.provo.novell.com is running on node: c2n1
VIP c2n2-vip.provo.novell.com is enabled
VIP c2n2-vip.provo.novell.com is running on node: c2n2
VIP c2n3-vip.provo.novell.com is enabled
VIP c2n3-vip.provo.novell.com is running on node: c2n3
VIP c2n4-vip.provo.novell.com is enabled
VIP c2n4-vip.provo.novell.com is running on node: c2n4
Network is enabled
Network is running on node: c2n1
Network is running on node: c2n4
Network is running on node: c2n3
Network is running on node: c2n2
ONS is enabled
ONS daemon is running on node: c2n1
ONS daemon is running on node: c2n4
ONS daemon is running on node: c2n3
ONS daemon is running on node: c2n2
Check the status of the designated resources:

oracle@c2n1:~> /home/grid/bin/crsctl stat res -t
--------------------------------------------------------------------------------
Name        Target State      Server                   State details
--------------------------------------------------------------------------------
Local Resources
--------------------------------------------------------------------------------
ora.LISTENER.lsnr
         ONLINE  ONLINE       c2n1                     STABLE
         ONLINE  ONLINE       c2n2                     STABLE
         ONLINE  ONLINE       c2n3                     STABLE
         ONLINE  ONLINE       c2n4                     STABLE
ora.SUSETEST.dg
         ONLINE  ONLINE       c2n1                     STABLE
         ONLINE  ONLINE       c2n2                     STABLE
         ONLINE  ONLINE       c2n3                     STABLE
         ONLINE  ONLINE       c2n4                     STABLE
ora.SUSETEST01.dg
         ONLINE  ONLINE       c2n1                     STABLE
         ONLINE  ONLINE       c2n2                     STABLE
         ONLINE  ONLINE       c2n3                     STABLE
         ONLINE  ONLINE       c2n4                     STABLE
ora.asm
         ONLINE  ONLINE       c2n1                     Started,STABLE
         ONLINE  ONLINE       c2n2                     Started,STABLE
         ONLINE  ONLINE       c2n3                     Started,STABLE
         ONLINE  ONLINE       c2n4                     Started,STABLE
ora.net1.network
         ONLINE  ONLINE       c2n1                     STABLE
         ONLINE  ONLINE       c2n2                     STABLE
         ONLINE  ONLINE       c2n3                     STABLE
         ONLINE  ONLINE       c2n4                     STABLE
ora.ons
         ONLINE  ONLINE       c2n1                     STABLE
         ONLINE  ONLINE       c2n2                     STABLE
         ONLINE  ONLINE       c2n3                     STABLE
         ONLINE  ONLINE       c2n4                     STABLE
--------------------------------------------------------------------------------
Cluster Resources
--------------------------------------------------------------------------------
ora.LISTENER_SCAN1.lsnr
    1    ONLINE  ONLINE       c2n4                     STABLE
ora.LISTENER_SCAN2.lsnr
    1    ONLINE  ONLINE       c2n3                     STABLE
ora.LISTENER_SCAN3.lsnr
    1    ONLINE  ONLINE       c2n1                     STABLE
ora.MGMTLSNR
    1    ONLINE  ONLINE       c2n1                     169.254.38.57 10.1.1.1,STABLE
ora.c2n1.vip
    1    ONLINE  ONLINE       c2n1                     STABLE
ora.c2n2.vip
    1    ONLINE  ONLINE       c2n2                     STABLE
ora.c2n3.vip
    1    ONLINE  ONLINE       c2n3                     STABLE
ora.c2n4.vip
    1    ONLINE  ONLINE       c2n4                     STABLE
ora.cvu
    1    ONLINE  ONLINE       c2n1                     STABLE
ora.mgmtdb
    1    ONLINE  ONLINE       c2n1                     Open,STABLE
ora.oc4j
    1    ONLINE  ONLINE       c2n1                     STABLE
ora.scan1.vip
    1    ONLINE  ONLINE       c2n4                     STABLE
ora.scan2.vip
    1    ONLINE  ONLINE       c2n3                     STABLE
ora.scan3.vip
    1    ONLINE  ONLINE       c2n1                     STABLE
--------------------------------------------------------------------------------
 
Check OCR and the Voting disk files:

oracle@c2n1:~> /home/grid/bin/ocrcheck
Status of Oracle Cluster Registry is as follows :
        Version                  :          4
        Total space (kbytes)     :     409568
        Used space (kbytes)      :       1696
        Available space (kbytes) :     407872
        ID                       : 1286433802
        Device/File Name         :  +SUSETEST
                     Device/File integrity check succeeded
    
                     Device/File not configured
    
                     Device/File not configured
    
                     Device/File not configured
    
                     Device/File not configured
    
        Cluster registry integrity check succeeded
    
        Logical corruption check bypassed due to non-privileged user   
        
oracle@c2n1:~> /home/grid/bin/crsctl query css votedisk
##  STATE    File Universal Id                File Name Disk group
--  -----    -----------------                --------- ---------
1. ONLINE   3195ba343d234f0cbf92bf2df0f7d3b9 (/dev/oradata/disk1) [SUSETEST]
2. ONLINE   3410554dce114fa1bfb86b8cd1f3288b (/dev/oradata/disk2) [SUSETEST]
3. ONLINE   69bb2fac7bac4f71bf4b3ec63154f76c (/dev/oradata/disk3) [SUSETEST]
Located 3 voting disk(s).        
   
5.2 Installing Oracle Database#
Preparation#
Log in to the SUSE Linux Enterprise Server 12 64-bit OS as a non-admin user. Download the Oracle Database 12c Release 1 (12.1.0.2.0) for Linux x86-64.

Extract grid.zip and run the installer './runInstaller' from Database ShipHome.

Installation#
For the actual installation, follow the steps below:

Select Configure Security Updates

Figure 27 Configure Security Updates


Provide your e-mail address to be informed of security issues, then click Next to continue.

Proceed to Installation Option.

Figure 28 Installation Option


Choose the option Install database software only, then click Next to continue.

Proceed to Grid Installation Options.

Figure 29 Grid Installation Options


Choose the option Oracle Real Application Clusters database installation, then click Next to continue.

Go to Nodes Selection to select from the list of nodes.

Figure 30 Nodes Selection


Select all nodes in the cluster, then click Next to continue.

Proceed to Product Languages.

Figure 31 Product Languages


Select all languages, then click Next to continue.

Select the Database Edition.

Figure 32 Database Edition


Choose the option Enterprise Edition, then click Next to continue.

Specify the Installation Location.

Figure 33 Installation Location


Fill in the Oracle base and the Software location as shown above, then click Next to continue.

Define the privileged Operating System Groups.

Figure 34 Operating System Groups


The priviledged Operating System groups are selected by default. Click Next to continue.

Perform the Prerequisite Checks .

Figure 35 Prerequisite Checks


Perform the pre-check as shown above; to recheck the system. Click Fix&Check Again to continue.

Figure 36 Fixup Script


Follow the prompts. Manually run the fixup script as root user on each node, then click OK.

You should see the following output:

c2n1 ~ #: /tmp/CVU_12.1.0.2.0_oracle/runfixup.sh
All Fix-up operations were completed successfully.
Check the Fixup Results here:

Figure 37 Fixup Results


Check the Verification Results here:

Figure 38 Verification Results


Select the option Ignore All, then click Next to continue.

Review the Summary.

Figure 39 Summary


If you see the installation summary as shown above, click Install to continue.

Proceed to Install Product.

Figure 40 Install Product


While the product loads, a window pops up where you can execute the configuration scripts.

Figure 41 Execute Configuration Scripts


Execute root.sh as the root user in each cluster node, then click OK to continue.

You will see the following output:

c2n1:~ # /home/oracle/app/product/12.1.0/dbhome_1/root.sh
Performing root user operation.

The following environment variables are set as:
    ORACLE_OWNER=oracle
    ORACLE_Home=/home/oracle/app/product/12.1.0/dbhome_1
    
Enter the full pathname of the local bin directory: [/usr/local/bin]
The contents of "dbhome" have not changed. No need to overwrite.
The contents of "oraenv" have not changed. No need to overwrite.
The contents of "coraenv" have not changed. No need to overwrite.

Entries will be added to the /etc/oratab file as needed by
Database Configuration Assistant when a database is created
Finished running generic part of root script.
Now product-specific root actions will be performed.
The next screen indicates that the installation has been finished successfully.

Figure 42 Finish


The installation of your Oracle Database is finished. Click Close to close the screen.

Create ASM Disk Group#
Use the ASM Configuration Assistant to create the ASM disk group for datafile storage.

Figure 43 ASM Configuration Assistant


Follow the instructions displayed on the screen.

Create Oracle RAC 12.1.0.2 Database#
Perform the following steps to create your Oracle RAC 12.1.0.2 database by using the Database Configuration Assistant:

Select Database Operation.

Figure 44 Database Operation


Choose the option Create Database, then click Next to continue.

Proceed to Creation Mode.

Figure 45 Creation Mode


Choose the option Advanced Mode, then click Next to continue.

Define the Database Template.

Figure 46 Database Template


Select the type of database you want to configure, then click Next to continue.

Proceed to Database Identification.

Figure 47 Database Identification


Fill in the Global Database Name as shown above, then click Next to continue.

NOTE: Container Database

Oracle Database 12C supports the creation of a Container Database. This means you can also create a Container Database depending on your needs.

Proceed to Database Placement.

Figure 48 Database Placement


Review the Server Pools information as shown above. Click Next to continue.

Specifiy the Management Options.

Figure 49 Management Options


Specify the management options for the database, then click Next to continue.

Proceed to Database Credentials.

Figure 50 Database Credentials


Specify the administrative password for the database users, then click Next to continue.

Proceed to Storage Locations.

Figure 51 Storage Locations


Specify the database files storage information as shown above, then click Next to continue.

Proceed to Database Options.

Figure 52 Database Options


According to your needs, specify whether to add the schemas to your database. Click Next to continue.

Proceed to Initialization Parameters.

Figure 53 Initialization Parameters


Choose the option Typical Settings and adjust the parameters to meet your requirements. Click Next to continue.

Proceed to Creation Options.

Figure 54 Creation Options


Select the database creation options as shown above, then click Next to continue.

Perform the Prerequisite Checks.

Figure 55 Prerequisite Checks


Select the option Ignore All, then click Next to continue.

Review the Summary.

Figure 56 Summary


Review and check the information displayed in the Database Configuration Summary as shown above. Click Finish to continue.

Review the Progress Page.

Figure 57 Progress Page


The creation of your database is now progressing as shown above. Wait until the creation process is completed.

Proceed to Finish.

Figure 58 Finish


When the creation of the database is completed, some information is displayed on the screen as shown above. Click Close to close the screen.

Post-Installation Checks#
Verify the database status and the configuration.

oracle@c2n1:~> export ORACLE_HOME=/home/oracle/app/product/12.1.0/dbhome_1
oracle@c2n1:~> /home/oracle/app/product/12.1.0/dbhome_1/bin/srvctl status database -d susedb
Instance SUSEDB_1 is running on node c2n2
Instance SUSEDB_2 is running on node c2n3
Instance SUSEDB_3 is running on node c2n4
Instance SUSEDB_4 is running on node c2n1
oracle@c2n1:~> /home/oracle/app/product/12.1.0/dbhome_1/bin/srvctl config database -d susedb -a
Database unique name: SUSEDB
Database name: SUSEDB
Oracle home: /home/oracle/app/product/12.1.0/dbhome_1
Oracle user: oracle
Spfile: +SUSETEST01/SUSEDB/PARAMETERFILE/spfile.274.908484433
Password file: +SUSETEST01/SUSEDB/PASSWORD/pwdsusedb.256.908484093
Domain:
Start options: open
Stop options: immediate
Database role: PRIMARY
Management policy: AUTOMATIC
Server pools: susepool
Disk Groups: SUSETEST01
Mount point paths:
Services:
Type: RAC
Start concurrency:
Stop concurrency:
Database is enabled
Database is individually enabled on nodes:
Database is individually disabled on nodes:
OSDBA group: dba
OSOPER group:
Database instances:
Configured nodes:
Database is policy managed
oracle@c2n1:~> /home/grid/bin/crsctl stat res -t
--------------------------------------------------------------------------------
Name        Target State      Server                   State details
--------------------------------------------------------------------------------
Local Resources
--------------------------------------------------------------------------------
ora.LISTENER.lsnr
        ONLINE  ONLINE       c2n1                     STABLE
        ONLINE  ONLINE       c2n2                     STABLE
        ONLINE  ONLINE       c2n3                     STABLE
        ONLINE  ONLINE       c2n4                     STABLE
ora.SUSETEST.dg
        ONLINE  ONLINE       c2n1                     STABLE
        ONLINE  ONLINE       c2n2                     STABLE
        ONLINE  ONLINE       c2n3                     STABLE
        ONLINE  ONLINE       c2n4                     STABLE
ora.SUSETEST01.dg
        ONLINE  ONLINE       c2n1                     STABLE
        ONLINE  ONLINE       c2n2                     STABLE
        ONLINE  ONLINE       c2n3                     STABLE
        ONLINE  ONLINE       c2n4                     STABLE
ora.asm
        ONLINE  ONLINE       c2n1                     Started,STABLE
        ONLINE  ONLINE       c2n2                     Started,STABLE
        ONLINE  ONLINE       c2n3                     Started,STABLE
        ONLINE  ONLINE       c2n4                     Started,STABLE
ora.net1.network
        ONLINE  ONLINE       c2n1                     STABLE
        ONLINE  ONLINE       c2n2                     STABLE
        ONLINE  ONLINE       c2n3                     STABLE
        ONLINE  ONLINE       c2n4                     STABLE
ora.ons
        ONLINE  ONLINE       c2n1                     STABLE
        ONLINE  ONLINE       c2n2                     STABLE
        ONLINE  ONLINE       c2n3                     STABLE
        ONLINE  ONLINE       c2n4                     STABLE
--------------------------------------------------------------------------------
Cluster Resources
--------------------------------------------------------------------------------
ora.LISTENER_SCAN1.lsnr
    1    ONLINE  ONLINE       c2n4                     STABLE
ora.LISTENER_SCAN2.lsnr
    1    ONLINE  ONLINE       c2n3                     STABLE
ora.LISTENER_SCAN3.lsnr
    1    ONLINE  ONLINE       c2n1                     STABLE
ora.MGMTLSNR
    1    ONLINE  ONLINE       c2n1                     169.254.38.57 10.1.1.1,STABLE
ora.c2n1.vip
    1    ONLINE  ONLINE       c2n1                     STABLE
ora.c2n2.vip
    1    ONLINE  ONLINE       c2n2                     STABLE
ora.c2n3.vip
    1    ONLINE  ONLINE       c2n3                     STABLE
ora.c2n4.vip
    1    ONLINE  ONLINE       c2n4                     STABLE
ora.cvu
    1    ONLINE  ONLINE       c2n1                     STABLE
ora.mgmtdb
    1    ONLINE  ONLINE       c2n1                     Open,STABLE
ora.oc4j
    1    ONLINE  ONLINE       c2n1                     STABLE
ora.scan1.vip
    1    ONLINE  ONLINE       c2n4                     STABLE
ora.scan2.vip
    1    ONLINE  ONLINE       c2n3                     STABLE
ora.scan3.vip
    1    ONLINE  ONLINE       c2n1                     STABLE
ora.susedb.db
    1    ONLINE  ONLINE       c2n2                     Open,STABLE
    2    ONLINE  ONLINE       c2n3                     Open,STABLE
    3    ONLINE  ONLINE       c2n4                     Open,STABLE
    4    ONLINE  ONLINE       c2n1                     Open,STABLE    
--------------------------------------------------------------------------------
```

2. 确认Oracle 企业管理工具可用

![图59 Oracle企业管理工具](_v_images/图59oracle企_1539247822_6574.png)

## 6.0 备注

1. 在 database/stage/cvu/cv/admin/cvu_config 和 grid/stage/cvu/cv/admin/cvu_config 中编辑参数CV_ASUME_DISTID=SUSE11

2. 应用Patch 20737462解决参照数据丢失引起的CVU问题

3. 安装libcap1包（libcap2包已被默认安装）；如 libcap1-1.10-59.61.x86_64 或 libcap1-32bit-1.10-59.61.x86_64

4. mksh（mksh-50-2.13.x86_64）取代了ksh

5. libaio已被重新命名为libaio1（libaio1-0.3.109-17.15.x86_64）；确保libaio1已安装

6. 可以配合 -ignoreSysPreqs 使用OUI 以暂时解决CVU检查失败的问题

## 7.0 GNU自由文档授权

著作权所有 (C) 2000, 2001, 2002 Free Software Foundation, Inc. 51 Franklin St, Fifth Floor, Boston, MA 02110-1301 USA。允许每个人复制和发布本授权文件的完整副本，但不允许对它进行任何修改。

### 0. 导言

本授权的目的在于作为一种手册、教科书或其它的具有功能性的有用文件获得自由：确保每一个人都具有复制和重新发布它的有效自由，而不论是否作出修改，也不论其是否具有商业行为。其次，本授权保存了作者以及出版者由于他们的工作而 得到 名誉的方式，同时也不被认为应该对其他人所作出的修改而担负责任。

本授权是一种’copyleft’，这表示文件的衍生作品本身必须具有相同的自由涵义。它补足了GNU 公共通用授权-- 一种为了自由软件而设计的’copyleft’授权。

我们设计了本授权是为了将它使用到自由软件的使用手册上，因为自由软件需要自由的文档：一种自由的程序应该提供与此软件具有相同的自由的使用手册。但是本授权并不被限制在软件使用手册的应用上；它可以被用于任何以文字作基础的作品，而不论其主题内容，或者它是否是一个被出版的印刷书籍。我们建议本授权主要应用在以使用说明或提供参考作为目的的作品上。

### 1. 效力与定义

本授权的效力在于任何媒体中的任何的使用手册或其它作品，只要其中包含由版权所有人所指定的声明，说明它可以在本授权的条款下被发布。这样的一份声明提供了全球范围内的,免版税的和没有期限的许可，在此所陈述条件下使用那个作品。以下所称的文件，指的是任何像这样的使用手册或作品。公众中的任何成员都是被许可人，并且称作为你。如果你以一种需要在版权法下取得允许的方式进行复制、修改或发布作品，你就接受了这项许可。

“修改版本”指的是任何包含文件或是它的其中一部份，不论是逐字的复制或是经过修正，或翻译成其它语言的任何作品。

“次要章节”是一个具名的附录，或是文件的本文之前内容的章节，专门用来处理文件的出版者或作者，与文件整体主题(或其它相关内容)的关系，并且不包含任何可以直接落入那个整体主题的内容。(因此，如果文件的部分内容是作为数学教科书，那么其次要章节就可以不用来解释任何数学。)它的关系可以是与主题相关的历史连接，或是与其相关的法律、商业、哲学、伦理道德或政治立场。

“不变章节”是标题已被指定的某些次要章节，在一个声明了是以本授权加以发行的文件中，依此作为不变章节。如果一个章节并不符合上述有关于次要的定义时，则它并不允许被指定为不变。文件可以不包含不变章节。如果文件并没有指出任何不变章节，那么就是没有。

“封面文字”是某些被加以列出的简短文字段落，在一个声明了是以本授权加以发行的文件中，依此作为前封面文字或后封面文字。前封面文字最多可以包含 5 个单词，后封面文字最多可以包含 25 个单词。

文件的”透明”拷贝指的是一份机器可读的拷贝，它以一种一般公众可以取得其规格说明的格式来表现，适合于直接用一般文字编辑器、一般点阵图像程序用于由图元像素构成的影像或一些可以广泛取得的绘图程序用于由向量绘制的图形直接地进行修订；并且适合于输入到文字格式化程式，或是可以自动地转换到适合于输入到文字格式化程序的各种格式。一份以透明以外的档案格式所构成的拷贝，其标记或缺少标记，若是被安排成用来挫折或是打消读者进行其后续的修改，则此拷贝并非透明。一种影像格式，如果仅仅是用来充斥文本的资料量时，就不是透明的。一个不是透明的拷贝被称为混浊。

透明拷贝适合格式的例子包括有：没有标记的纯 ASCII、Texinfo 输入格式、LaTeX 输入格式、使用可以公开取得其 DTD 的 SGML 或 XML、合乎标准的简单 HTML、PostScript 或 PDF。透明影像格式的例子有 PNG、XCF 和 JPG 。混浊格式包括只能够以私人文书处理器阅读以及编辑的私人格式、DTD 以及或处理工具不能够一般地加以取得的 SGML 或 XML、以及由某些文书处理器只是为了输出的目的而做出的，由机器制作的 HTML、PostScript 或 PDF 。

标题页对一本印刷书籍来说，指的是标题页本身，以及所需要用来容纳本授权必须出现在标题页的易读内容的，如此的接续数页。对于并没有任何如此页面的作品的某些格式，标题页指的是本文主体开始之前作品标题最显着位置的文字。

出版者指的是把那些文本副本发布给公众的任何人或实体。

一个标题为 XYZ的章节指的是文件的一个具名的次要单元，其标题精确地为 XYZ 或是将 XYZ 包含在跟着翻译为其它语言的 XYZ 文字后面的括号内 -- 这里 XYZ 代表的是名称于下提及的特定章节，像是感谢、贡献、背书或历史。当你修改文件时，给像这样子的章节保存其标题指的是，它保持为一个根据这个定义的标题为 XYZ的章节。

文件可以在用来陈述本授权效力及于文件的声明后，包括担保放弃。这些担保放弃被考虑为以提及的方式，包括在本授权中，但是只被看作为放弃担保之用：任何这些担保放弃可能会有的其它暗示都是无效的，并且也对本授权的含义没有影响。

以下是有关复制、发布及修改的明确条款及条件。

### 2. 逐字的复制

你可以复制或发布文件于任何媒体，而不论其是否具有商业买卖行为，其条件为具有本授权、版权声明和许可声明，说明本授权效力于文件的所有重制拷贝，并且你没有增加任何其它条件到本授权的条件中。你不可以使用技术手段，来妨碍或控制你所制作或发布的拷贝阅读或进一步的发布。然而，你可以接受补偿以作为拷贝的交换。如果你发布了数量足够大的拷贝，你也必须遵循第三条的条件。

你也可以在上述的相同条件下借出拷贝，并且你可以公开地陈列拷贝。

### 3. 大量地复制

如果你出版文件的印刷拷贝或者通常具有印刷封面的媒体的拷贝，数量上超过一百个单位，而且文件许可声明要求有封面文字，那么你必须将这些拷贝附上清楚且易读的文字：前封面文字于前封面上、后封面文字于后封面上。这两种封面必须清楚易读地辨认出，你是这些拷贝的出版者。前封面文字必须展示完整的标题，而标题的文字应当同等地显著可见。你可以增加额外的内容于封面上。仅在封面作出改变的复制，只要它们保存了文件的标题，并且满足了这些条件，可以在其它方面被看作为逐字的复制。

如果对于任意一个封面所需要的文字，数量过于庞大以至于不能符合易读的原则，你应该在实际封面的最前面列出所能符合易读原则的内容，然后将剩下的接续在相邻的页面。

如果你出版或发布数量超过一百个单位文件的混浊拷贝，你必须与此混浊拷贝一起包含一份机器可读的透明拷贝，或是与一份混浊拷贝一起或其陈述一个电脑网络位址，使一般的网络使用公众具有存取权，可以使用公开标准的网络协定，下载一份文件的完全透明拷贝，此拷贝中并且没有增加额外的内容。如果你使用后面的选项，当你开始大量地发布混浊拷贝时，你必须采取合理的审慎步骤，以保证这个透明拷贝将会在发布的一开始就保持可供存取，直到你最后一次发布那个发行版的一份混浊拷贝给公众后，至少一年为止。以保证这个透明拷贝，将会在所陈述的位址保持如此的可存取性，直到你最后一次发布那个发行版直接或经由你的代理商或零售商的一份混浊拷贝给公众后，至少一年为止。

你被要求，但不是必须，在重新发布任何大数量的拷贝之前与文件的作者联络，给予他们提供你一份文件的更新版本的机会。

### 4. 修改

你可以在上述第二条和第三条的条件下，复制和发布文件的修改版本，其条件为你要精确地在本授权下发布修改版本，且修改版本补足了文件的角色，从而允许修改版本的发布和修改权利给任何拥有它拷贝的人。另外，你必须在修改版本中做这些事：


第一款、 在标题页或在封面上使用，如果有与先前版本不同的文件，应该被列在文件的历史章节不同的标题。如果版本的原始出版者允许，你可以使用与某一个先前版本相同的标题。

第二款、 在修改版本的标题页上列出担负作者权的一个或多个人或实体作为作者，并且列出至少五位文件的主要作者。如果少于五位，则列出全部的主要作者，除非他们免除了你这个要求。

第三款、 在标题页陈述修改版本的出版者的名称作为出版者。

第四款、 保存文件的所有版权声明。

第五款、 为你的修改增加一个与其它版权声明相邻的适当的版权声明。

第六款、 在版权声明后面，以授权附录所显示的形式，包括一个给予公众在本授权条款下使用修改版本的许可声明。

第七款、 在那个许可声明中保存恒常章节和文件许可声明中必要封面文字的全部列表。

第八款、 包括一个未被改变的本授权的副本。

第九款、 保存标题为历史的章节和其标题，并且增加一项至少陈述如同在标题页中所给的修改版本的标题、年份、新作者和出版者。如果在文件中没有标题为历史的章节，则制作出一个陈述如同在它的标题页中所给的文件的标题、年份、新作者和出版者，然后增加一项描述修改版本如前面句子所陈述的情形。

第十款、 如果有的话，保存在文件中为了给公众存取文件的透明拷贝，而给予的网络位址，以及同样地在文件中为了它所根据的先前版本，而给予的网络位址。这些可以被放置在历史章节。你可以省略一个在文件本身之前，已经至少出版了四年的作品的网络位址，或是如果它所参照的那个版本的原始出版者给予允许的情形下也可以省略它。

第十一款、 在任何标题为感谢或贡献的章节，保存章节的标题，并且在那章节保存到那时候为止，每一个贡献者的感谢以及或贡献的所有声色。

第十二款、 保存文件的所有恒常章节，于其文字以及标题皆不得变更。章节号码或其同等物并不被认为是章节标题的一部份。

第十三款、 删除任何标题为背书的章节。这样子的章节不可以被包括在修改版本中。

第十四款、 不要重新命名任何现存的章节，而使其标题为背书，或造成与任何恒常章节相冲突的标题。

第十五款、 保存任何的担保放弃。

如果修改版本包括新的本文之前内容的章节，或合乎作为次要章节的附录，并且没有包含复制自文件的内容，则你具有选择可以指定一些或全部这些章节为恒常的。要这样做，将它们的标题增加到在修改版本许可声明中的恒常章节列表中。这些标题必须可以和任何其它章节标题加以区别。

你可以增加一个标题为背书的章节，其条件为它仅只包含由许多团体所提供的你的修改版本的背书 -- 举例来说，同侪评审的说明，或本文已经被一个机构认可为一个标准的权威定义。

你可以增加一个作为前封面文字的最多五个字的段落，以及一个作为后封面文字的最多二十五个字的段落，到修改版本的封面文字列表的后面。前封面文字和后封面文字都只能有一个段落，可以经由任何一个实体，或经由任何一个实体所作出的安排而被加入。如果文件已经在同样的封面包括了封面文字 -- 先前由你或由你所代表的相同实体所作出的安排而加入，则你不可以增加另外一个；但是你可以在先前出版者的明确允许下替换掉旧的。
文件的作者和出版者并不由此授权，而给予允许使用他们的名字以为了或经由声称或暗示任何修改版本背书为自己所应得的方式而获得名声的权利。

### 5. 组合文件

你可以在上述第四条的条款中对于修改版本的定义之下，将文件与其它在本授权下发行的文件组合起来，其条件是你要在组合品中，包括所有原始文件的所有恒常章节，不做修改，同时在组合作品的许可声明中将它们全部列为恒常章节，并且你要保存它们所有的担保放弃。

组合作品只需包含本授权的一份副本，并且重复的恒常章节可以仅以单一个拷贝来取代。如果名称重复但内容不同的恒常章节，则将任此章节的标题，以在它的后面增加的方式加以独特化，如果已知的话，于括号中指出那个章节的原始作者或出版者的名称，或是指定一个独特的号码。在此组合作品许可声明中恒常章节的列表中，对其章节标题也作出相同的调整。

在组合品中，你必须组合在不同原始文件中，标题为历史的任何章节，形成一个标题为历史的章节；同样组合任意标题为感谢或贡献的章节。你必须删除标题为背书的所有章节。

### 6. 文件的收集

你可以制作含有文件以及其它以本授权发行文件的收集品，并且将本授权对不同文件中的个别副本，以单一个包括在收集品的副本取代，其条件是你要遵循在其它方面，给予一个文件逐字复制的允许本授权的规则。

你可以从这样的一个收集品中抽取出一份单一的文件，并且在本授权下将它单独地发布，其条件是你要在抽取出的文件中插入本授权的一份副本，并且在关于那份文件的逐字的复制的所有其它方面，遵循本授权。

### 7. 独立作品的聚集

一个文件的编辑物，其中或附加于储存物或发布媒体的一册的，具有其它分别且独立的文件或作品的衍生品，如果经由编辑而产生的版权，并没有用来限制此编辑物使用者的法律权力，而超过了个别的作品所允许的，则被称为一个聚集品。当文件中包括一个聚集品，本授权的效力并不仅在于此聚集品中的，于其本身并非文件的衍生作品的其它作品。
如果第三条的封面文字要求效力于这些文件的拷贝，并且文件的篇幅少于整个聚集品的一半，则文件的封面文字可以被放在只围绕着文件，并于聚集品内部的封面或是电子的封面同等物上，如果文件是以电子的形式出现的话。否则它们必须出现在绕着整个聚集品的印刷封面上。

### 8. 翻译

翻译被认为一种修改，因此你可以在第四条的条款下发布文件的翻译。用翻译更换恒常章节需要取得版权所有者的特别允许，但是你可以包括部份或所有恒常章节的翻译，使其附加到这些恒常章节的原始版本之中。你可以包括本授权、文件中的所有许可声明和任何的担保放弃的翻译，其条件为你也必须包括本授权的原始英文版本，以及这些声明与放弃的原始版本。如果发生翻译与本授权、声明或放弃的原始版本有任何的不同意时，将以原始版本为准。

如果在文件中的章节被标题为感谢、贡献或历史，则保存标题第一条的必要条件第四条，典型上将会需要去更动实际的标题。

### 9. 终止

你不可以复制、修改、在本授权下再设定额外条件的次授权或发布文件，除非明白地表示是在本授权所规范的条件下进行。任何其它的复制、修改、在本授权下再设定额外条件的次授权、或发布文件的意图都是无效的，并且将会自动地终止你在本授权下所被保障的权利。

然而，如果你终止所有违反本授权的行为，特定版权所有人会暂时恢复你的授权直到此版权所有者明确并最终地终止你的授权。或者特定版权所有人永久地恢复你的授权如果此版权所有人在停止违反授权后的60天内没有通过合理的方式通知你违反授权。

此外，此版权所有者用一些合理的方式通知你违反了授权规定，也是你第一次从此版权所有者收到违反授权的通知，并且你在收到通知后的30天内终止了这种行为，那么此版权所有者会永久地恢复你的授权。

你的权利在此章节的终止并不代表终止得到你的拷贝和权利的当事人的授权。如果你的权利被终止而没有被永久性地恢复，那么你将没有任何权利去使用此章节的全部或部分资料和资料的拷贝。

### 10. 本授权的未来改版

自由软件基金会可能偶尔会出版自由文件授权的新修订过的版本。这种新版本在精神上将会类似于现在的版本，但在细节上可能会有不同，以对应新的问题或相关的事。请见 http://www.gnu.org/copyleft/。

本授权的任何版本都被指定一个可供区别的版本号码。如果文件指定一个效力于它的特定号码版本的本授权或任何以后的版本，你就具有选择遵循指定的版本，或任何已经由自由软件基金会出版的后来版本并且不是草稿的条款和条件。如果文件并没有指定一个本授权的版本号码，你就可以选择任何一个曾由自由软件基金会所出版不是草稿的版本。如果文件指定一个代理能够决定本授权未来的那一种版本可以用，而且还指定代理公开声明如果你接受一种版本你将会永久地被授权为文本选择此版本的权利。
11.重新授权

“MMC 网站”是指任何发布有著作版权作品的网站服务器，也为任何人提供卓越的设施去编辑一些作品。任何人都可编辑的一种公众维基就是这种服务器的一个例子。包含在这个网站的”MMC”是指任何一套在MMC网站上发布的具有著作版权的作品。

“CC-BY-SA”是指 Creative Commons Attribution-Share Alike 3.0 授权，它是被“知识共享组织” 颁布的，“知识共享组织” 是一家非赢利性的，在圣弗朗西斯科，加利福尼亚具有重要的商业地位的组织。 而且未来的“copyleft”版本的授权也是被同一个组织发布的。

“合并”是指以整体或作为另一个文本的部分发布或重新发布一个文本。

MMC有重新授权的资格，如果它是在MMC下授权；或者所有的作品首次发布并非在MMC下授权，后来以整体或部分合并到MMC下，它们没有覆盖性文本或不变的章节并且在2008年11月1日之前合并的。

那么MMC网站的操作者会在2009年8月1日之前的任何时间在同一网站重新发布包含在这一网站的MMC是经过CC-BY-SA 授权的，只要那个MMC有资格重新授权。

授权附录：如何使用本授权用于你的文件

为使用本授权成为你撰写成的一份文件，必须在文件中包括本授权的一份复本，以及标题页的后面包括许可声明：

Copyright (c) YEAR YOUR NAME. Permission is granted to copy, distribute and/or modify this document under the terms of the GNU Free Documentation License, Version 1.2 or any later version published by the Free Software Foundation; with no Invariant Sections, no Front-Cover Texts, and no Back-Cover Texts. A copy of the license is included in the section entitled "GNU Free Documentation License".

如果你有恒常章节、前封面文字和后封面文字，请将with... Texts这一行以这些文本取代：
with the Invariant Sections being LIST THEIR TITLES, with the Front-Cover Texts being LIST, and with the Back-Cover Texts being LIST.

如果你有不具封面文字的恒常章节，或一些其它这三者的组合，将可选择的二项合并以符合实际情形。

如果你的文件中包含有并非微不足道的程序码范例，我们建议这些范例平行地在你的自由软件授权选择下，比如以 GNU General Public License 的自由软件授权来发布，从而允许它们作为自由软件而使用。