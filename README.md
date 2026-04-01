# MCP企业级AI智能体平台
一个基于MCP协议+云原生架构的私有化AI智能体平台，解决通用大模型无法安全访问内部数据的核心痛点，实现自然语言驱动的企业级业务操作自动化。

## 项目介绍
MCP企业级AI智能体平台是一款基于Anthropic MCP（Model Context Protocol）协议构建的解耦式AI智能体平台，核心设计思想是实现大模型推理层与企业内部业务执行层的安全解耦对接。在保障企业核心数据私有化的前提下，让大模型具备自主规划、执行企业级业务操作的能力，可作为企业私有化AI助手，支持自然语言驱动的内部数据库查询、业务流程执行等场景，无需将敏感数据暴露至公网大模型。

## 核心特性
- 🛡️ 私有化全栈部署：基于K8s+Ollama实现大模型推理、业务服务全私有化部署，全程不触碰公网，保障企业数据安全
- 🧩 MCP协议解耦架构：分离AI推理层（AI Gateway）与业务执行层（MCP Server），新增业务场景无需修改AI核心逻辑
- 📊 Text-to-SQL自主执行：针对HR数据查询等场景，ReAct循环引擎支持多步规划→SQL生成→执行→结果反馈全流程自动化
- ☁️ 云原生高可用：基于Kubernetes构建，具备弹性伸缩、OpenEBS持久化存储、横向扩展特性
- 🚀 GitOps生命周期管理：通过Argo CD实现声明式部署与配置管理，简化平台运维与版本迭代
- 📡 高效异步通信：基于SSE/JSON-RPC 2.0实现AI推理与业务操作的异步交互，提升响应效率

## 技术栈
### 核心技术
- Kubernetes (K8s)：容器编排与高可用运行平台
- Ollama（Qwen 2.5-7B）：本地私有化LLM推理服务，提供智能决策能力
- Golang：AI Gateway/MCP Server核心开发语言
- MCP协议：解耦大模型推理与业务操作的核心通信协议
- MySQL：企业级业务数据存储（如HR人力资源数据）
- OpenEBS：K8s持久化存储，保障数据持久化
- Argo CD：GitOps持续部署工具，实现声明式配置管理
- SSE/JSON-RPC 2.0：AI层与业务层的异步通信协议
- Docker/Containerd：容器化与镜像管理
- Helm：Kubernetes应用包管理

架构图
```mermaid
flowchart TB
    %% 样式定义：纯深色模式高对比度，所有边框、连线、文字全白
    classDef userLayer fill:#1e293b,stroke:#ffffff,stroke-width:2px,color:#ffffff
    classDef gatewayLayer fill:#0f172a,stroke:#ffffff,stroke-width:2px,color:#ffffff
    classDef llmLayer fill:#1e3a5f,stroke:#ffffff,stroke-width:2px,color:#ffffff
    classDef mcpLayer fill:#064e3b,stroke:#ffffff,stroke-width:2px,color:#ffffff
    classDef dataLayer fill:#7f1d1d,stroke:#ffffff,stroke-width:2px,color:#ffffff
    classDef infraLayer fill:#334155,stroke:#ffffff,stroke-width:2px,color:#ffffff

    %% ====================== 1. 用户交互层 ======================
    User[用户] --> WebUI[Web UI 交互层]
    class User,WebUI userLayer

    %% ====================== 2. AI 网关层（MCP Host） ======================
    WebUI --> AIGateway[AI Gateway <br/> Golang 开发 <br/> MCP Host 核心调度]
    class AIGateway gatewayLayer

    %% ====================== 3. LLM 推理层 ======================
    AIGateway <--> Ollama[Ollama <br/> K8s/ai-services 命名空间 <br/> LLM 推理引擎]
    class Ollama llmLayer

    %% ====================== 4. MCP 服务层（工具暴露） ======================
    AIGateway -. MCP 协议(SSE) .-> MCPServer[MCP Server <br/> Golang 开发 <br/> 暴露 read_schema/execute_query 工具]
    class MCPServer mcpLayer

    %% ====================== 5. 数据层 ======================
    MCPServer <--> MySQL[MySQL 8.0 <br/> K8s/ai-services 命名空间 <br/> OpenEBS 持久化 <br/> HR 样本数据]
    class MySQL dataLayer

    %% ====================== 6. 基础设施层（K8s 集群） ======================
    subgraph K8sCluster[Kubernetes 1.32 集群 <br/> 1 Master + 2 Worker]
        direction TB
        Ansible[Ansible <br/> 集群部署] --> K8sMaster[Master 节点 <br/> k8s-node1]
        Ansible --> K8sWorker1[Worker 节点 <br/> k8s-node2]
        Ansible --> K8sWorker2[Worker 节点 <br/> k8s-node3]
        
        ArgoCD[Argo CD <br/> GitOps 部署管控]
        Prometheus[Prometheus/Grafana <br/> 监控告警]
        OpenEBS[OpenEBS <br/> 持久化存储]
        
        %% 基础设施关联业务组件
        OpenEBS --> MySQL
        OpenEBS --> Ollama
        ArgoCD --> AIGateway & Ollama & MCPServer & MySQL
        Prometheus --> AIGateway & Ollama & MCPServer & MySQL
    end
    class K8sCluster,Ansible,ArgoCD,Prometheus,OpenEBS infraLayer

    %% 链路标注：纯白色虚线，深色模式下清晰可见
    linkStyle 4 stroke:#ffffff,stroke-width:1.5px,stroke-dasharray:5,5
    note["MCP 协议核心：解耦 LLM 与数据/工具调用"]
    AIGateway -.-> note
```
