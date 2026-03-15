# Point-Summary
系统方向的面试准备

如果是 **K8s 基础设施** 岗，准备要分成 4 条线：
1. **Kubernetes 原理**
2. **云原生基础设施能力**
3. **工程开发能力**
4. **项目与排障表达**

# 一、先明确岗位在考什么
通常不是只问会不会用 `kubectl`，而是会看你是否理解：
- **集群怎么工作**
- **控制面怎么设计**
- **节点/网络/存储怎么接入**
- **怎么做平台化、自动化、可观测、稳定性治理**
- **能不能写控制器、Operator、Webhook、平台工具链**
所以面试重点通常是：
- Kubernetes 核心原理
- 容器运行时 / CNI / CSI / Ingress
- 调度、弹性、升级、容灾
- Go 开发能力
- Linux / 网络 / 系统基础
- 线上问题分析能力

# 二、核心知识点怎么准备
## 1）Kubernetes 基础原理
至少要能讲清：
### 核心对象
- Pod、Deployment、StatefulSet、DaemonSet、Job、CronJob
- Service、Endpoints、Ingress
- ConfigMap、Secret
- PV / PVC / StorageClass
- Namespace、Label、Annotation、Taint/Toleration、Affinity
### 控制面组件
- `kube-apiserver`
- `etcd`
- `kube-scheduler`
- `kube-controller-manager`
- `cloud-controller-manager`
### 节点组件
- `kubelet`
- `kube-proxy`
- container runtime（containerd / CRI）

*   **架构设计**：API Server, etcd, Controller Manager, Scheduler, Kubelet, Kube-proxy 的各自职责和交互流程。
*   **API Machinery（必考源码）**：深入理解 **Informer 机制**。熟练掌握 Reflector、DeltaFIFO、LocalStore、Indexer、WorkQueue 的工作流。
*   **etcd 原理**：Raft 共识算法、Watch 机制底层实现、MVCC 多版本并发控制机制、etcd 的脑裂处理。
*   **调度器 (kube-scheduler)**：Predicates（预选）和 Priorities（优选）过程。如何编写自定义调度器（Scheduler Framework）。
*   **三大接口 (CRI, CNI, CSI)**：
    *   **CNI (网络)**：Flannel 和 Calico 的原理及区别（IPIP, VXLAN, BGP 模式）。
    *   **CSI (存储)**：PV/PVC/StorageClass 绑定流程，Attach/Detach 机制。
    *   **CRI (运行时)**：Kubelet 如何通过 CRI 呼叫 containerd/docker 启动 Pod 的全链路。

### 必会原理题
- Pod 创建全过程
- Deployment 滚动更新原理
- Service 如何转发到 Pod
- kube-proxy 的 iptables/ipvs 模式区别
- 调度器如何筛选和打分
- etcd 为什么重要，如何保证一致性
- HPA / VPA / Cluster Autoscaler 分别做什么
- StatefulSet 为什么适合有状态服务
- DaemonSet 为什么适合节点级组件

## 2）容器和 Linux 基础
- 容器本质是什么：**namespace + cgroups + rootfs**
- Docker 和 containerd 区别
- 镜像分层原理
- 容器启动流程
- Linux 进程、文件系统、网络命名空间
- cgroup 如何限制 CPU / Memory
- OOMKilled 的常见原因
- liveness/readiness/startup probe 区别
*   **容器核心技术**：`Namespaces`（隔离机制：PID, Mount, Network 等）和 `Cgroups`（资源限制：CPU, 内存等）。
*   **Linux 网络**：TCP/IP 协议栈、`iptables` / `IPVS` 原理（极度重要，与 Kube-proxy 直接相关）、`veth pair`、网桥（Bridge）。最近几年非常爱考 **eBPF/Cilium** 基础。
*   **Linux 存储**：OverlayFS 等联合文件系统原理。

## 3）网络
- Pod 网络模型：**每个 Pod 独立 IP，Pod 间直接通信**
- CNI 是什么，常见插件：
  - Calico
  - Flannel
  - Cilium
- Service 的 ClusterIP / NodePort / LoadBalancer
- Ingress 和 Ingress Controller
- CoreDNS 工作机制
- NetworkPolicy 怎么做网络隔离
- kube-proxy、iptables、ipvs
- 东西向 / 南北向流量
### 高频问题
- 一个请求从 Ingress 到 Pod 的路径是什么？
- Service 为什么能负载均衡？
- Calico 和 Flannel 区别？
- 排查 Pod 网络不通你怎么做？

## 4）存储
- PV / PVC / StorageClass 的关系
- CSI 是什么
- 本地盘、网络盘、分布式存储区别
- StatefulSet + PVC 的使用场景
- 数据持久化、快照、扩容
- 常见存储故障：挂载失败、权限问题、IO 瓶颈

## 5）K8s 扩展开发
- CRD 是什么
- Operator 是什么
- Controller 模式是什么
- Informer / Reflector / WorkQueue
- Reconcile loop
- Finalizer
- Admission Webhook
- Mutating / Validating Webhook
### 至少要能讲清
> “如何开发一个 Operator 管理某类业务资源”

如果会 Go，建议一定要补：
- `client-go`
- `controller-runtime`
- Kubebuilder 基本用法

## 6）可观测与稳定性
- Prometheus + Alertmanager + Grafana
- 常见指标：
  - CPU
  - Memory
  - QPS
  - 延迟
  - 错误率
  - Pod 重启次数
- 日志采集：EFK / Loki
- 链路追踪：Jaeger / OpenTelemetry
- 集群稳定性治理：
  - 资源配额
  - 限流
  - PDB
  - 优雅终止
  - 驱逐机制
  - 节点故障迁移
*   **Operator 开发**：熟练使用 `Kubebuilder` 或 `Operator SDK` 编写 CRD 和 Controller。能说清楚 Reconcile 循环的设计哲学（声明式 API vs 命令式 API）。
*   **Service Mesh**：Istio 的核心组件，Envoy 流量劫持原理（iptables NAT）。
*   **可观测性**：Prometheus 架构、PromQL 核心语法、如何设计 Custom Metrics（自定义指标）并结合 HPA 做弹性伸缩。

# 三、开发岗还会考什么
## 1）Go 语言
- goroutine / channel
- context
- map 并发安全
- slice / defer / interface
- error 处理
- 锁、原子操作
- GC 基础
- HTTP / gRPC 基础
- 单元测试、benchmark
*   **并发模型**：Goroutine 调度机制（GMP 模型）、Channel 的底层实现与使用场景（有缓冲/无缓冲、死锁场景）。
*   **内存管理**：垃圾回收机制（三色标记法、混合写屏障）、内存逃逸分析。
*   **核心包原理**：`Context` 的底层原理与级联取消、`sync` 包（Mutex、RWMutex、WaitGroup、Map 底层实现）。
*   **高阶特性**：反射（Reflection）、泛型（Generics）、Interface 的底层结构（eface/iface）。

## 2）数据结构与工程能力
不一定像算法岗那么难，但建议准备：
- 链表、哈希、堆、队列
- 排序、二分
- 并发模型
- 设计一个任务调度器 / 限流器 / 重试机制
- 高可用系统设计

# 四、项目经验怎么准备
面试里最拉开差距的是 **项目讲解能力**。
你最好准备 2~3 个项目，每个项目按这个结构讲：
## 项目回答模板
1. **背景**：业务问题是什么
2. **目标**：为什么要做
3. **架构**：你设计了什么方案
4. **关键难点**：调度、网络、性能、稳定性、升级、兼容性
5. **你的贡献**：你具体写了什么
6. **结果**：收益指标
7. **复盘**：有什么不足，如何优化

### 特别适合讲的项目
- K8s 集群搭建 / 升级 / 多集群管理
- Operator 开发
- 发布系统 / 弹性伸缩系统
- 监控告警平台
- 容器网络 / 服务治理 / Service Mesh
- 存储接入
- CI/CD 平台
- 集群稳定性治理、成本优化

# 五、面试官最爱问的高频题
可以重点刷这些：
## Kubernetes
- Pod 创建流程
- Deployment 和 StatefulSet 区别
- Service 实现原理
- kube-proxy 工作机制
- HPA 原理
- 调度器工作流程
- etcd 为什么用 Raft
- CRD 和 Operator 区别
- Finalizer 有什么用
- Admission Webhook 应用场景
## 排障
- Pod 一直 Pending 怎么查
- Pod CrashLoopBackOff 怎么查
- Service 不通怎么查
- 节点 NotReady 怎么查
- CoreDNS 异常怎么查
- 镜像拉取失败怎么查
- 滚动发布卡住怎么查
- etcd 空间满了怎么办
## 系统设计
- 设计一个多租户 K8s 平台
- 设计一个灰度发布系统
- 设计一个弹性伸缩平台
- 设计一个 Operator 管理数据库集群
- 设计集群监控与告警体系
*   **场景排错题**：
    *   "一个 Pod 一直处于 Pending 状态，你如何排查？"
    *   "Pod 出现 OOMKilled，但 Node 内存还有空余，为什么？"
    *   "集群内两个 Pod 无法通信，请描述你的排查思路（从网络插件到 iptables 规则）。"
*   **系统设计题**：
    *   "如何设计一个支持 1 万个 Node 的大规模 K8s 集群？"（考察 APIServer 性能瓶颈、etcd 拆分、Kube-API 缓存等）。
    *   "如何设计一个多租户的 K8s 平台？"（考察 vcluster, RBAC, NetworkPolicy, ResourceQuota）。
*   **算法与数据结构**：一般要求 LeetCode 中等难度（Medium）。多刷树、链表、图、动态规划以及与并发相关的算法题。


# 一个很实用的面试回答原则
回答尽量用这个结构：
**结论 -> 原理 -> 例子 -> 排障/优化**
比如：
> “Pod Pending 通常先看调度失败原因。原理上是 scheduler 没找到符合条件的节点，常见原因包括资源不足、污点、亲和性约束、PVC 未绑定。我会先 `kubectl describe pod` 看 Events，再看 node 状态和 PVC 状态。”

这样会显得你很像真正做过。


# 十、建议

如果你的目标是拿到这类岗位，最低要做到：
- **能讲清 K8s 核心原理**
- **能讲 1 个你真正做过的基础设施项目**
- **能用 Go 写基本控制器逻辑**
- **能回答常见线上故障排查**
