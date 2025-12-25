# k3s 集群配置说明（k3s-infra）

本仓库用于管理 GX10 本地服务器上的 k3s（Kubernetes）集群基础配置。

在不影响现有生产 Docker 容器业务的前提下，引入 k3s 作为资源隔离、测试与扩展平台，用于新业务验证，并为后续逐步演进做好准备。

---

## 设计原则

- 不迁移、不改动现有 Docker / NPM / 生产业务
- k3s 与现有容器并行运行，仅承载测试与新业务
- 本仓库只维护平台级配置，不维护具体业务
- 所有 Kubernetes 配置通过 Git 统一管理
- 集群节点只负责运行，不作为配置编辑环境

---

## 当前状态

- GX10 运行 k3s（当前为 control-plane + worker）
- 已有额外 worker 节点加入集群
- 已完成：
  - 命名空间划分（dev / sandbox / infra / prod）
  - dev / sandbox 基线资源限制（LimitRange / ResourceQuota）
  - 基础调度策略与节点标签（GPU / worker 区分）
  - 应用部署、Service 通信、PVC 持久化验证
- 通过本地 Mac 使用 kubectl + kubeconfig 远程管理集群

---

## 命名空间约定

- sandbox  
  学习、试验、Demo 使用  
  资源配额较小，防止误操作影响整体环境

- dev  
  熟悉 Kubernetes 的成员使用  
  用于功能验证、测试版本运行

- infra  
  集群基础组件命名空间  
  由维护者统一管理，普通使用者只读或不可见

- prod  
  预留生产命名空间  
  当前不承载业务

---

## 仓库目录结构

k3s-infra/
├── namespaces/        # 命名空间定义（平台级）
│   ├── dev.yaml
│   ├── sandbox.yaml
│   ├── infra.yaml
│   └── prod.yaml
│
├── baseline/          # 各环境资源基线（平台级）
│   ├── dev/
│   │   ├── limitrange.yaml
│   │   └── quota.yaml
│   ├── sandbox/
│   │   ├── limitrange.yaml
│   │   └── quota.yaml
│   └── prod/          # 预留
│
├── infra/             # 基础设施组件（预留）
└── README.md

---

## 使用方式说明

- 本仓库不存放具体业务 YAML
- 业务由各成员在自己本地维护：
  - Deployment / Service / PVC 等
  - 通过 kubeconfig 连接集群进行调度
- 平台只负责：
  - 命名空间管理
  - 资源配额与限制
  - 调度与隔离策略

---

## 使用规划

### 1. 当前阶段：测试与验证

- 在 sandbox / dev 中部署测试应用
- 验证资源限制、调度、重建机制
- 不承载现有生产流量

### 2. 节点扩展（进行中 / 后续）

- GX10 作为 GPU / 核心节点
- 其他机器作为 worker 节点
- 通过调度与标签提升整体资源利用率

### 3. 按需演进

- 根据业务情况逐步迁移到 k3s
- 支持长期混合运行模式（Docker + k3s）
- 为后续集中管理与稳定性提升预留空间

---

## 管理方式

- 平台配置由维护者统一修改并提交 Git
- 使用 kubeconfig 在本地管理集群
- 不直接在服务器节点上手动修改配置
- 集群结构稳定后，将为相关成员提供 dev / sandbox 的 kubeconfig

