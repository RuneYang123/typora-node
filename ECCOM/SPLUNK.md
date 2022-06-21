### SPLUNK 

#### Apps

在splunk中，App主要是用来数据可视化，数据分析，以及行动的。存在可视化的面板。

#### Add-on

处理数据，优化数据收集的。主要分为三层，Search Tire、Indexer Tire、Forwarder Tier。各个组件保持独立。

![image-20220613191630025](C:\Users\Rune\AppData\Roaming\Typora\typora-user-images\image-20220613191630025.png)

#### 节点

主要有Search节点、index节点、fwd节点以及管理节点。管理节点包含Deploment Service（负责fwd的配置下发）、Cluster Master集群的管理节点、Deployer发配置、MC监控节点 。

##### Deploment Service

Deploment Service可以按照共同特征对Splunk Enterprise组件进行分组（serverclass），然后分发根据这些组分发内容（一般是Add-on）。通过IP地址，主机名或其他字段进行分组。

其中核心conf文件deplomentclient.conf，告诉它Deployment Server的位置。每个client定期轮询Deployment Server识别自己，确定自己的serverclass然后部署应用程序集。

#### 实验搭建分布式集群

##### 确认机器角色

本次分布式实验，用到了5台机器。由一台搜索头（SH）、两台索引器（IDX）、一台分发节点及监控节点（DS、MC）。

```
172.30.242.137(SH)
172.30.242.138(IDX1)
172.30.242.139(IDX2)
172.30.242.140(DS,MC)
172.30.242.142(UF)
```

##### 配置机器环境

###### 检查环境

```sh
sudo cat /proc/cpuinfo #检查CPU
sudo free -h #MEM
sudo cat /etc/redhat-release #OS版本
sudo cat /proc/version  #内核版本
sudo df -h #磁盘空间
```

###### 新建用户组、用户（Splunk）

```sh
sudo groupadd splunk #splunk组
sudo useradd -g splunk splunk #添加用户 所有机器
```

###### 新建文件夹

存放Splunk安装包以及安装位置，并把权限给Splunk用户

```sh
sudo mkdir /software
sudo mkdir /software/packages
sudo chown -R splunk:splunk /software
```

###### 上传文件

```sh
sudo wget -O splunk-8.2.5-77015bc7a462-Linux-x86_64.tgz "https://download.splunk.com/products/splunk/releases/8.2.5/linux/splunk-8.2.5-77015bc7a462-Linux-x86_64.tgz" #除去UF都需要SPlunkCore安装包
sudo wget -O splunkforwarder-8.2.5-77015bc7a462-Linux-x86_64.tgz "https://download.splunk.com/products/universalforwarder/releases/8.2.5/linux/splunkforwarder-8.2.5-77015bc7a462-Linux-x86_64.tgz" #UF安装包
```

###### 解压文件

```sh
sudo tar -zxvf splunk安装包 -C /software
sudo chown -R splunk:splunk /software
```

###### 上传配置文件

```sh
sudo mv C配置文件包 /software
sudo chown -R splunk:splunk /software
```

###### deplomentclient.conf文件配置

将targetUri修改为自己的DS8089端口。然后将配置下发到除了DS的所有机器上。每台机器都要操作一次，包括UF转发器。

```c
[deployment-client]
# Set the phoneHome at the end of the PS engagement
# 10 minutes
# phoneHomeIntervalInSecs = 600

[target-broker:deploymentServer]
# Change the targetUri DS的8089端口
targetUri = deploymentserver.splunk.mycompany.com:8089
```

```sh
cp -ar /software/C配置文件包/org_all_deplomentclient.conf /software/splunk/ect/apps #将配置文件修改后下发
cp -ar /software/C配置文件包/org_all_deplomentclient.conf /software/splunk/ect/de-apps#DS上配置
```

