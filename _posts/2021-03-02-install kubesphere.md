- 确保节点之间 `ssh` 互通

> 略

### 安装依赖

`yum install -y socat conntrack-tools ebtables ipset`

### 利用 `kubekey` 一键安装

`mkdir -pv /opt/kubesphere && cd /opt/kubesphere`
`curl -sfL https://get-kk.kubesphere.io | VERSION=v1.0.1 sh -`

#### 创建配置文件

`./kk create config --with-kubernetes v1.18.6 --with-kubesphere v3.0.0 -f /opt/kubesphere/config.yml`

#### 编辑配置文件

```yaml
apiVersion: kubekey.kubesphere.io/v1alpha1
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - {name: master, address: 192.168.1.2, internalAddress: 192.168.1.2, port: 2233, privateKeyPath: "~/.ssh/id_rsa"}
  - {name: node1, address: 192.168.1.3, internalAddress: 192.168.1.3, port: 2233, privateKeyPath: "~/.ssh/id_rsa"}
  - {name: node2, address: 192.168.1.4, internalAddress: 192.168.1.4, port: 2233, privateKeyPath: "~/.ssh/id_rsa"}
  roleGroups:
    etcd:
    - master
    master:
    - master
    worker:
    - node1
    - node2
  controlPlaneEndpoint:
    domain: lb.kubesphere.local
    address: ""
    port: "6443"
  kubernetes:
    version: v1.18.6
    imageRepo: kubesphere
    clusterName: cluster.local
  network:
    plugin: calico
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
  registry:
    registryMirrors: []
    insecureRegistries: []
  addons: []


---
apiVersion: installer.kubesphere.io/v1alpha1
kind: ClusterConfiguration
metadata:
  name: ks-installer
  namespace: kubesphere-system
  labels:
    version: v3.0.0
spec:
  local_registry: ""
  persistence:
    storageClass: ""
  authentication:
    jwtSecret: ""
  etcd:
    monitoring: true
    endpointIps: localhost
    port: 2379
    tlsEnable: true
  common:
    es:
      elasticsearchDataVolumeSize: 20Gi
      elasticsearchMasterVolumeSize: 4Gi
      elkPrefix: logstash
      logMaxAge: 7
    mysqlVolumeSize: 20Gi
    minioVolumeSize: 20Gi
    etcdVolumeSize: 20Gi
    openldapVolumeSize: 2Gi
    redisVolumSize: 2Gi
  console:
    enableMultiLogin: true  # enable/disable multi login
    port: 30880
  alerting:
    enabled: true
  auditing:
    enabled: true
  devops:
    enabled: true
    jenkinsMemoryLim: 2Gi
    jenkinsMemoryReq: 1500Mi
    jenkinsVolumeSize: 8Gi
    jenkinsJavaOpts_Xms: 512m
    jenkinsJavaOpts_Xmx: 512m
    jenkinsJavaOpts_MaxRAM: 2g
  events:
    enabled: true
    ruler:
      enabled: true
      replicas: 2
  logging:
    enabled: true
    logsidecarReplicas: 2
  metrics_server:
    enabled: true
  monitoring:
    prometheusMemoryRequest: 400Mi
    prometheusVolumeSize: 20Gi
  multicluster:
    clusterRole: none  # host | member | none
  networkpolicy:
    enabled: true
  notification:
    enabled: true
  openpitrix:
    enabled: true
  servicemesh:
    enabled: true
```

#### 使用配置文件创建集群

`./kk create cluster -f config.yaml`

> 这个时候可以去泡一杯咖啡..

**安装成功显示**  
```
#####################################################
###              Welcome to KubeSphere!           ###
#####################################################

Console: http://192.168.1.2:30880
Account: admin
Password: P@88w0rd

NOTES：
  1. After logging into the console, please check the
     monitoring status of service components in
     the "Cluster Management". If any service is not
     ready, please wait patiently until all components
     are ready.
  2. Please modify the default password after login.
...
```

#### 配置LB

> 略

#### 启用 kubectl 自动补全(optional)

`yum install bash-completion -y`

在 `~/.bashrc` 添加 `source /usr/share/bash-completion/bash_completion`

重新加载 `Shell` 并运行 `type _init_completion` 检查 `bash-completion` 是否正确安装

`echo 'source <(kubectl completion bash)' >>~/.bashrc`  
`kubectl completion bash >/etc/bash_completion.d/kubectl`
