本仓库用于管理 GX10 本地服务器上的 k3s（Kubernetes）集群配置。

在不影响现有生产容器业务的前提下，引入 k3s 作为测试与扩展平台，用于新业务验证及后续演进。

---

## 设计原则

- 不迁移、不改动现有 Docker / NPM / 生产业务
- k3s 独立运行，仅承载测试及新业务
- 所有 Kubernetes 配置文件通过 Git 统一管理
- 集群节点只负责运行，不作为配置编辑环境

---

## 当前状态

- GX10 运行单节点 k3s（control-plane + worker）
- 已完成：
  - 命名空间划分（dev / infra / prod）
  - dev 环境资源限制（LimitRange / ResourceQuota）
  - 应用部署、Service 通信、PVC 持久化验证
- 通过本地 kubectl 远程管理集群

---

## 目录结构

k3s/
├── namespaces/        # 命名空间定义
│   ├── dev.yaml
│   ├── infra.yaml
│   └── prod.yaml
│
├── dev/               # 测试环境
│   ├── baseline/      # 环境级基础配置（配额、限制）
│   └── apps/          # 测试业务
│       ├── hello/
│       └── apache/
│
├── infra/             # 基础设施组件（预留）
├── prod/              # 生产环境（预留）
└── README.md


⸻

使用规划

1. 测试与验证（当前阶段）
	•	在 dev 命名空间中部署测试应用
	•	验证 Deployment / Service / PVC 的实际行为
	•	验证资源限制、调度与重建机制
	•	不承载任何现有生产流量

⸻

2. 节点扩展（后续）
	•	计划将公司老电脑作为 worker node 加入集群
	•	仍保持单 control-plane 架构
	•	通过调度策略提升整体资源利用率

⸻

3. 按需演进
	•	根据业务实际情况决定：
	•	是否迁移至 k3s
	•	是否独立命名空间
	•	是否需要节点亲和或资源隔离
	•	不强制所有业务迁移，支持长期混合运行模式

⸻

管理方式
	•	Kubernetes YAML 文件在本地或 Git 中维护
	•	使用 kubeconfig 在本地（Mac）通过 kubectl 管理集群
	•	不直接在服务器节点上编辑配置文件
	•	集群结构稳定后，将为相关成员提供 dev 环境的 kubeconfig，用于测试与开发操作

⸻

目标

在不影响现有系统运行的前提下：
	•	实现测试环境与生产环境隔离
	•	提升硬件资源利用率
	•	为后续业务扩展和集群演进预留空间

---

