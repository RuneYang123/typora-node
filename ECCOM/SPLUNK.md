### SPLUNK 

#### Apps

在splunk中，App主要是用来数据可视化，数据分析，以及行动的。存在可视化的面板。

#### Add-on

处理数据，优化数据收集的。主要分为三层，Search Tire、Indexer Tire、Forwarder Tier。各个组件保持独立。

![image-20220613191630025](C:\Users\Rune\AppData\Roaming\Typora\typora-user-images\image-20220613191630025.png)

#### 1+1配置

此处的1+1指的是，一台主节点和一台转发器节点。

splunkcore的配置org_all_deploymentclient/local/deploymentclient

```conf
[deployment-client]
# Set the phoneHome at the end of the PS engagement
# 10 minutes
# phoneHomeIntervalInSecs = 600

[target-broker:deploymentServer]
# Change the targetUri
targetUri = 192.168.67.103:8089//此处改为splunkcore的ip地址8089端口
```

org_all_forwarder_outputs/