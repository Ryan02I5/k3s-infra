# k3s 集群平台配置说明（k3s-infra）

本仓库用于管理公司本地服务器（以 GX10 为核心）的 k3s（Kubernetes）集群平台级配置。

在不影响现有 Docker 生产容器与人工 SSH 使用方式的前提下，引入 k3s 作为资源隔离、测试与扩展平台，用于新业务验证，并为后续逐步演进做好准备。

---

## 设计原则

不迁移、不改动现有 Docker / NPM / 生产容器业务  
k3s 与现有容器并行运行  
本仓库只维护平台级配置，不维护具体业务  
业务 YAML 由各成员自行在本地维护  
所有 Kubernetes 配置通过 Git 统一管理  
不在服务器节点上直接手工修改集群配置  

---

## 当前集群状态

GX10 同时承担 control-plane 与 worker 角色，并作为 GPU / 核心节点  
已有额外 worker 节点加入集群（老机器）  
节点通过 label 与 taint 区分角色（GPU / 普通节点）  

---

## 命名空间规划

sandbox  
用于学习、试验、Demo  
资源配额严格，防止误操作影响整体环境  

dev  
用于功能验证与测试  
面向熟悉 Kubernetes 的成员  

ai  
用于 AI / 推理相关工作  
资源配额较大，仅 GPU 节点可调度  

infra  
集群基础组件命名空间  
仅由平台维护者管理  

prod  
生产命名空间预留  
当前不承载业务  

---

## 资源管理策略

LimitRange  
用于约束单个 Pod / 容器的资源使用  
自动注入默认 requests 与 limits  
限制单容器最大 CPU / 内存，防止节点被拖死  

ResourceQuota  
用于约束命名空间整体资源上限  
限制 CPU、内存、Pod 数量、PVC 数量、存储总量  
命名空间之间资源互不影响  

---

## 节点调度策略

GPU 节点（GX10）  
标记 gpu=true  
通过 taint 保护，避免普通 Pod 默认调度  

普通 Pod  
默认调度到 worker 节点  

AI / GPU Pod  
通过 nodeSelector 与 tolerations 显式调度到 GX10  

---

## 仓库目录结构

```text
k3s-infra/
├── namespaces/
│   ├── sandbox.yaml
│   ├── dev.yaml
│   ├── ai.yaml
│   ├── infra.yaml
│   └── prod.yaml
│
├── baseline/
│   ├── sandbox/
│   │   ├── limitrange.yaml
│   │   └── quota.yaml
│   ├── dev/
│   │   ├── limitrange.yaml
│   │   └── quota.yaml
│   ├── ai/
│   │   ├── limitrange.yaml
│   │   └── quota.yaml
│   └── prod/
│
├── infra/
└── README.md


⸻

使用方式说明

本仓库不存放任何具体业务 YAML
业务由使用者在自己本地维护 Deployment / Pod / Service / PVC
通过 kubeconfig 连接集群并指定命名空间运行
平台只负责命名空间、资源限制与调度隔离

⸻

使用阶段规划

当前阶段
在 sandbox / dev 中进行学习与验证
ai 命名空间用于 GPU / 推理测试
不承载任何生产流量

后续演进
根据业务成熟度逐步迁移到 k3s
支持 Docker 与 k3s 的长期混合运行
后续可引入更细粒度 RBAC、独立 control-plane 与 CI/CD

⸻

管理约定

平台配置仅由维护者修改并提交 Git
使用者不直接修改平台仓库
集群稳定后，将向成员发放 sandbox / dev 的 kubeconfig

