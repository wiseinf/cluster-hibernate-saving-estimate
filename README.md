# cluster-hibernate-saving-estimate

## Introduction

`cluster-hibernate-saving-estimate` is a tool for estimating saving when using cluster hibernate for kubernetes clusters.

## Prerequisites

1. A kubernetes cluster using Alibaba Cloud ACK.
1. A kubeconfig file for the cluster with following permissions:

    | 资源 | 权限 | 说明 |
    | --- | --- | --- |
    | Node | List | 获取节点信息 |
    | Pod | List | 获取Pod信息 |
    | Namespace | List | 获取Namespace信息 |
    | Deployment| List | 获取Deployment信息 |
    | CronJob | List  | 获取CronJob信息 |

    权限文件如下：
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: optimizer
    rules:
      - apiGroups:
          - ""
        resources:
          - nodes
          - pods
          - namespaces
        verbs:
          - list
      - apiGroups:
          - "apps/v1"
        resources:
          - deployments
        verbs:
          - list
      - apiGroups:
          - "batch/v1"
        resources:
          - cronjobs
        verbs:
          - list
    ```

1. A access key id and secret for Alibaba Cloud.

    | 服务 | 权限 | 说明 |
    | --- | --- | --- |
    | 云服务器（ECS） | ecs:DescribeInstances | 获取实例信息 |
    | 容器服务Kubernetes版（CS） |  cs:DescribeClusterNodePools<br/> cs:DescribeClusterNodePoolDetail<br/> cs:DescribeClusterNodes | 获取集群、节点池、节点等信息 |

    其权限策略文件描述如下，您可以直接在阿里云控制台中导入该策略，具体请参考[通过脚本编辑模式创建自定义权限策略](https://help.aliyun.com/zh/ram/user-guide/create-a-custom-policy#section-kwn-gu8-48m)
    ```json
    {
      "Version": "1",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "ecs:DescribeInstances"
          ],
          "Resource": [
            "*"
          ],
          "Condition": {}
        },
        {
          "Effect": "Allow",
          "Action": [
            "cs:DescribeClusterNodePools",
            "cs:DescribeClusterNodePoolDetail",
            "cs:DescribeClusterNodes"
          ],
          "Resource": [
            "*"
          ],
          "Condition": {}
        }
      ]
    }
    ```

## Quick Start

1. Clone the repo using `git clone https://github.com/wiseinf/cluster-hibernate-saving-estimate.git` or download the zip file from from `https://github.com/wiseinf/cluster-hibernate-saving-estimate/tags` and unzip it.
1. Go to the `cluster-hibernate-saving-estimate` directory and run command:

    ```sh
    ./bin/${arch}/cluster-hibernate-saving-estimate --kubeconfig=kubeconfigPath --access-key-id=your_access_key_id --access-key-secret=your_access_key_secret --hibernate-cron="0 10 * * *" --awaken-cron="0 8 * * *" --format=text --output=cluster-hibernate-saving-estimate.txt
    ```

    In which, `${arch}` depends on your platform, currently, `${arch}` can be `linux-amd64`(which runs on linux amd64 archtecture) or `darwin-amd64`(which runs on mac os amd64 archtecture). `--kubeconfig` is a path to a kubeconfig file. `--access-key-id` and `--access-key-secret` are your access key id and secret. `--hibernate-cron` and `--awaken-cron` are cron expressions for hibernation and awakening schedule, respectively. `--format` is the output format, default is text, available values are text and json. `--output` is the output file location, default is `cluster-hibernate-saving-estimate.txt`.

1. The output file will be generated in the current directory. For example:

    ```text
    NodeGroup: as-cpu-ng(np151adb107448039712d3a24f0d50a)  Total Nodes: 12  Autoscaling: true
    OnDemandNodes: 6  cpu: 192 cores, memory: 384 gib
        Node: cn-beijing.171.18.106.158(i-bp6f7txtyecxuauzgk6m) NoSpot, InstanceType: ecs.hfc7.8xlarge, ChargeType: PrePaid
        Node: cn-beijing.171.18.106.76(i-bp6f7txtyecxuauzgk6o) NoSpot, InstanceType: ecs.hfc7.8xlarge, ChargeType: PrePaid
        Node: cn-beijing.171.18.106.69(i-bp6f7txtyecxuauzgk6j) NoSpot, InstanceType: ecs.hfc7.8xlarge, ChargeType: PrePaid
        Node: cn-beijing.171.18.106.79(i-bp6f7txtyecxuauzgk6k) NoSpot, InstanceType: ecs.hfc7.8xlarge, ChargeType: PrePaid
        Node: cn-beijing.171.18.106.159(i-bp6f7txtyecxuauzgk6h) NoSpot, InstanceType: ecs.hfc7.8xlarge, ChargeType: PrePaid
        Node: cn-beijing.171.18.106.125(i-bp6f7txtyecxuauzgk6g) NoSpot, InstanceType: ecs.hfc7.8xlarge, ChargeType: PrePaid
    SpotNodes: 0  
    ReservedNodes: 6    cpu: 192 cores, memory: 384 gib
        Node: cn-beijing.171.18.106.75(i-bp6f7txtyecxuauzgk6d) NoSpot, InstanceType: ecs.hfc7.8xlarge, ChargeType: PrePaid
        Node: cn-beijing.171.18.106.120(i-bp6f7txtyecxuauzgk6l) NoSpot, InstanceType: ecs.hfc7.8xlarge, ChargeType: PrePaid
        Node: cn-beijing.171.18.106.71(i-bp6f7txtyecxuauzgk6f) NoSpot, InstanceType: ecs.hfc7.8xlarge, ChargeType: PrePaid
        Node: cn-beijing.171.18.106.123(i-bp6f7txtyecxuauzgk6i) NoSpot, InstanceType: ecs.hfc7.8xlarge, ChargeType: PrePaid
        Node: cn-beijing.171.18.106.78(i-bp6f7txtyecxuauzgk6n) NoSpot, InstanceType: ecs.hfc7.8xlarge, ChargeType: PrePaid
        Node: cn-beijing.171.18.106.248(i-bp6f7txtyecxuauzgk6e) NoSpot, InstanceType: ecs.hfc7.8xlarge, ChargeType: PrePaid  
    Max Potential Saving: CPU: 98880 core hours; Memory: 197760 gib hours
    Potential Saving: CPU: 84754.29 core hours; Memory: 169508.57 gib hours
    Recommendation: no recommendation
    Sum of Deployment Resource Requests: CPU 224.00 cores, Memory 448.00 gibs
    ```

## Supported Flags

| Short Flag | Flag | Description |
| --- | --- | --- |
| `-k` | `--kubeconfig` | The path to the kubeconfig file |
| `-a` | `--access-key-id` | Access Key ID for authentication |
| `-s` | `--access-key-secret` | Access Key Secret for authentication |
| `-H` | `--hibernate-cron` | Cron expression for hibernation schedule, default is `0 21 * * 1-5` |
| `-W` | `--awaken-cron` | Cron expression for awakening schedule, default is `0 8 * * 1-5` |
| `-c` | `--cloud` | Cloud provider's name, default is `acs` |
| `-o` | `--file` | Output file location, default is `cluster-hibernate-saving-estimate.txt` |
| `-f` | `--format` | Output format, default is text, avaiable values are text and json |
