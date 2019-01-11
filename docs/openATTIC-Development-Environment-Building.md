# 搭建openATTIC开发环境

## 目录:

-   [简要版](#简要版)
-   [详细内容](#详细内容)
    1.   [（基于虚拟机的）Ceph集群部署](#1.(基于虚拟机)的ceph集群部署)
    2.   [openATTIC服务安装与配置](#2.openattic服务安装与配置)
    3.   [openATTIC开发环境搭建](#3.openattic开发环境搭建)
-   [理解openATTIC架构](#理解openattic架构)
-   [参考](#参考)

## 简要版

1. 按文档搭建Ceph集群
2. 安装openATTIC并使其连通集群
- 本地安装openATTIC: zypper in openattic
- 将ceph集群admin节点/etc/ceph目录下的key和conf文件拷贝到本地
-  修改/etc/sysconfig/openattic中相关的salt API，rgw，grafana参数
3. 将本地环境转变为开发环境
- 安装配置git，克隆bitbucket上的openATTIC源码到本地；安装nodejs和npm
- 修改/etc/apache2/conf.d/openattic.conf文件，使其指向本地克隆源码路径
- 修改/etc/sysconfig/openattic中的OADIR变量，使其指向本地克隆代码路径
- 启动相关web ui和oaconfig服务，即完成所有操作。

## 详细内容

### 1.(基于虚拟机的)Ceph集群部署

#### 1.1 利用virtual-machine-manager创建虚拟机

- 利用网页控制界面（？fiberhome）的远程控制工具console redirection给服务器添加virtual storage，即系统安装介质openSUSE_Leap_42.3.ios，我们就是利用这个介质来创建虚拟机。

- 打开virtual machine manager界面，添加connection。

![添加VM Host][添加机器连接]

- 在显示的连接主机列表中找到我们要安装虚拟机的主机，右键选择新建虚拟机。选择本地安装介质，即之前我们为服务器加载的虚拟介质。（当然也可以选择给出的其他方法，如网络安装，虚拟机上已有的系统镜像等。）

![创建新虚拟机][创建虚拟机]

- 选择使用CDROM/DVD。如果没有自动检测出系统信息可自己从列表中选择。

![选择使用CD/DVD介质][定位安装介质]

- 选择分配给虚拟机的内存和CPU。

- 为虚拟机创建存储空间，可以默认创建，也可以自己选择。

- 安装前最后确认，选择‘安装前定义配置’。
    - '启动选项'中选择允许启动列表，并把CDROM放在第一位（安装完成后调回Disk第一位，并不勾选CEROM，这样就直接从装好的硬盘中启动，不再需要添加的虚拟介质了）。
    - ‘IDE CDROM’选择connect，并确认连接上我们之前添加的ios。
    - ‘Display spice’中选择类型为VNC server。最后应用更改，并开始安装。

剩下的安装过程和在本地安装系统并无不同。在安装完成后关闭虚拟机，然后在virtual-machine-manager主界面找到我们刚创建的虚拟机，点击右键克隆选项，就能克隆出整个集群所需的足够数量的虚拟机，之后只需为每台虚拟机做相应的更改，例如ip地址，hostname等。

#### 1.2 SES/Ceph集群部署

可参考SES5部署手册。

### 2.openATTIC服务安装与配置

- 确保本机能与Ceph集群建立通信，配置与搭建集群时对各节点的操作类似，包括修改hostname,ntp服务，关闭防火墙等
- 操作系统： openSUSE Leap 42.3
- 关键目录：
    - /etc/ceph
    - /etc/sysconfig/openattic

#### 2.1 安装openATTIC

按以下命令添加源并直接安装openATTIC：

```shell
zypper addrepo http://download.opensuse.org/repositories/filesystems:/ceph:/luminous/openSUSE_Leap_42.3/filesystems:ceph:luminous.repo
zypper addrepo http://download.opensuse.org/repositories/filesystems:openATTIC:3.x/openSUSE_Leap_42.3/filesystems:openATTIC:3.x.repo
zypper refresh
zypper install openattic
```

openATTIC依赖librados的python包来实现对于ceph集群的管理和监控等功能。

#### 2.2 拷贝添加keyring和conf文件

配置openATTIC并实现和ceph集群的连接就要首先将keys拷贝到装有openATTIC的系统（openATTIC节点）中。这里说的keys包括ceph的admin keyring（ceph.client.admin.keyring）和ceph的配置文件（ceph.conf），可以在ceph集群的admin节点中/etc/ceph文件目录下找到。除了手动拷贝之外，也可以在admin节点上运行以下命令实现：

```shell
ceph-deploy admin openattic.yourdomain.com
```

这里的‘openattic.yourdomain.com’指代openATTIC节点的ip，因此需要admin节点能够和openATTIC节点通信，例如从admin节点通过ssh登录openATTIC节点。

此外，需要注意的是，要确保这两个文件能够对名为openattic的user group可读，具体可执行如下命令：

```shell
chgrp openattic /etc/ceph/ceph.conf /etc/ceph/ceph.client.admin.keyring
chmod g+r /etc/ceph/ceph.conf /etc/ceph/ceph.client.admin.keyring
```

在/etc/sysconfig/openattic中修改CEPH_CLUSTERS和CEPH_KEYRING_...变量，可参考下一步(2.3)中的格式。

#### 2.3 管理多个Ceph cluster

openATTIC支持管理多个Ceph集群，只需要他们有不同的名字和fsid（默认的集群名是ceph）。

拷贝key的时候要把相应的文件名做更改，例如一个集群名为dev，就把‘ceph.conf’改为‘dev.conf’，‘ceph.client.admin.keyring’改为‘dev.client.admin.keyring’。

在/etc/sysconfig/openattic中也要进行相应的调整。

- CEPH_CLUSTERS中用';'将多个集群的配置文件分开
    - 例如： CEPH_CLUSTERS="/etc/ceph/ceph.conf;/home/user/ceph/build/ceph.conf"
- 对于每个集群都通过CPEH_KEYRING+'大写的fsid'的形式定义keyirng的变量名(fsid可通过在admin节点运行’ceph -s‘命令获得)
    - 例如： CEPH_KEYRING_123ABCDE_4567_ABCD_1234_567890ABCDEF="/home/user/ceph/build/keyring"


此时，执行

```shell
oaconfig install
```

即可完成openATTIC的重新配置，连接上Ceph集群。

**注意**：但若要在后续配置gateway，deepsea等服务则先不执行此命令 ；要在后续配置开发环境也先不执行此命令。


#### 2.4 在openATTIC中集成Deepsea

DeepSea是SUSE基于Salt开发的Ceph安装和管理框架，可以用于对整个Ceph集群和各组成部分的部署，配置和管理。

一些openATTIC的特性，例如iSCSI和Ceph obeject gateway（RGW）的管理就依赖于通过Salt REST API和DeepSea的通信。如果要启用DeepSea的REST API可以在Salt主节点上执行以下命令：

```shell
salt-call state.apply ceph.salt-api
```

openATTIC默认Salt主节点的hostname是'salt'， API端口是'8000'， API用户名是'admin'。 如果要更改这些默认配置需要在/etc/sysconfig/openattic中进行相应修改。包括以下设定：

```text
SALT_API_HOST="admin"
SALT_API_PORT="8000"
SALT_API_USERNAME="admin"
SALT_API_PASSWORD="admin"
SALT_API_EAUTH="sharedsecret"
SALT_API_SHARED_SECRET="11dddddd-4567-3dd4-vg09-c93hdnsi3nfd" ## example
```

admin节点/etc/salt/master.d目录下有相关的sharedsecret,eauth信息


#### 2.5 配置Ceph object gateway(RGW)和Grafana管理服务

在/etc/sysconfig/openattic中以下设定进行相应修改即可：

```text
RGW_API_HOST="node4"
RGW_API_PORT=80
RGW_API_SCHEME="http"
RGW_API_ACCESS_KEY="VFEG733GBY0DJCIV6NK0" ## example
RGW_API_SECRET_KEY="lJzPbZYZTv8FzmJS5eiiZPHxlT2LMGOMW8ZAeOAq" ## example
RGW_API_USER_ID="admin"

GRAFANA_API_HOST="admin"
GRAFANA_API_PORT="3000"
GRAFANA_API_USERNAME="admin"
GRAFANA_API_PASSWORD="admin"
```

相应的所需要的集群RGW信息可通过在admin节点上执行以下命令获得：

```shell
radosgw-admin user info --uid=admin
```

或

```shell
salt-run ui_rgw.credentials
```

在配置好上述服务后执行命令“oaconfig install”完成openATTIC的配置，启用一系列服务，初始化数据库等工作。再执行“oaconfig start”命令就可以在浏览器中访问localhost地址进入openATTIC用户界面。

```
注意：
实际配置过程中，在RGW参数设置上填入命令输出的参数，认证无法成功（用户界面-System-setting中可看到）。此时选择让deepsea接管才能顺利连接。
```

#### 2.6 登录web ui及修改默认用户密码

如果不想修改默认用户'openattic'的密码，那么现在即可用浏览器访问http://127.0.0.1/openattic进入openATTIC的web用户界面，并用默认的用户名和密码（均为'openattic'）进行登录。

如果想修改密码（出于安全考虑），则可通过执行以下命令并按提示完成相应修改：

```shell
oaconfig changepassword openattic
```

### 3.openATTIC开发环境搭建

#### 3.1 配置git，创建本地代码仓库

首先下载安装最新版本的git: zypper in git

设置用户名和邮箱（用于对commit'签名'）（开发者社区要求commit提交能够可查，所以最好是使用真名）: 

```shell
git config --global user.name "你的用户名"
git config --global user.email "你的邮箱地址"
```

然后注册一个bitbucket帐号，配置bitbucket的ssh连接，用于代码的git clone。

- 生成SSH密钥，并按提示保存和加密密钥（可直接按Enter键选择默认配置）。

```shell
  ssh-keygen -t rsa -b 4096 -C "之前填写的邮箱地址" 
```

- 为ssh-agent添加SSH密钥： 

```shell
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

- 复制公钥，并将复制到的内容添加到bitbucket（左下角用户头像 - Bitbucket settings - SSH keys - Add key）：

```shell
cat ~/.ssh/id_rsa.pub # 输出公钥内容
```

- 验证连接

```shell
ssh -T git@bitbucket.org
```

可以看到'logged in as'字样即表示验证成功。

给自己fork一份bitbucket上的openATTIC官方仓库，并clone自己帐号中fork好的代码到本地的/srv目录下。

```shell
cd /srv
git clone git@bitbucket.org:<your_username>/openattic.git
git remote add upstream ssh://git@bitbucket.org/openattic/openattic.git
git fetch upstream
git checkout master
```

#### 3.2 安装开发工具

为了防止系统更新造成配置更改，运行如下命令为已安装包添加lock:

```shell
zypper al 'openattic*'
```

安装Node.js和npm(官方文档标注是支持nodejs6版本，所以最好也是安装v6版本):

```shell
zypper in nodejs npm
```

#### 3.3 修改相关的服务配置文件

编辑/etc/apache2/conf.d/openattic.conf文件：

- 将所有有关'/usr/share/openattic'的路径改为'/srv/openattic/backend'

- 添加如下内容：

```txt
<Directory /srv/openattic>
    Require all granted
</Directory>
```

编辑/etc/sysconfig/openattic文件,修改OADIR变量：

```txt
OADIR="/srv/openattic/backend"
```

#### 3.4 启动openATTIC服务

搭建web UI界面，更新配置并启动openATTIC服务：

```shell
cd /srv/openattic/webui
npm run build  ## build web app
oaconfig install ## configure openATTIC
oaconfig start  ## start openATTIC service
```

至此我们就搭建完成本地开发环境，openATTIC的web界面也能够在本地浏览器访问 http://localhost/openattic/ 看到（默认用户名和密码均为openattic）。

![openATTIC用户界面展示][openATTIC主界面]

修改/srv/openattic中的代码，openATTIC进程、GUI和oaconfig工具等会自动读取并做出相应改变。

## 理解openATTIC架构

以上内容让我们对openATTIC有了一定的认识，涉及到的几个关键目录和文件也告诉了我们一些信息，例如：
- /etc/ceph目录下的key和conf文件是集群对本机节点认证的所需文件
- /etc/sysconfig/openattic文件中的设置是用于相关API的通信连接
- /srv/openattic/webui目录包含了用于构建oA的web界面所需的代码
- /srv/openattic/backend目录包含了oA用于和集群建立通信，管理等的一些关键服务

但从整体上了解openATTIC的架构，将对我们对配置过程的理解以及解决配置过程中可能遇到的问题会有更多的帮助。所以这里将简要介绍一下openATTIC的架构。

~~待完成。。。~~

## 参考

- [官方文档](http://docs.openattic.org/en/latest/): 大部分的配置信息及方法在官方文档中都已经具体给出，如有问题可先到官方文档中寻找解决办法。
- [Google Group](https://groups.google.com/forum/#!forum/openattic-users): 也可以在group中咨询相关问题，但最好先搜索是否已经有类似的问题出现，并建议阅读[提问的方法](http://catb.org/~esr/faqs/smart-questions.html)。
- git配置
    - [设置SSH key](https://confluence.atlassian.com/bitbucket/set-up-an-ssh-key-728138079.html#SetupanSSHkey-ssh2) 


[openATTIC主界面]: ../images/oAdev/openATTIC_Dashboard.png "openATTIC UI - Dashboard"
[添加机器连接]: ../images/oAdev/add_connection.png
[创建虚拟机]: ../images/oAdev/create_new_vm.png
[定位安装介质]: ../images/oAdev/add_ios.png
