---
title: rough2
deprecated: false
hidden: false
metadata:
  robots: index
---
# Diagram 2 — OpenShift control-plane architecture in depth

The control plane is the brain of the cluster. It manages the desired state of the cluster, decides where workloads should run, and coordinates recovery when something fails.

In a standard OpenShift Container Platform deployment, the three control-plane nodes each host control-plane components, including an API Server and an etcd member. OpenShift also runs its own APIs, controllers, and Operators on the control plane.

Red Hat Documentation

+1

## 2.1 Internal architecture of the three master nodes

```mermaid

flowchart TB
    ADMIN["Administrator / oc CLI / Console"]
    ALB["API Load Balancer<br/>api.ocp.example.com:6443"]

    ADMIN --> ALB

    subgraph CP["Control Plane"]
      subgraph M1["Master 1 — 10.10.10.11"]
        API1["Kubernetes API Server"]
        ETCD1["etcd member 1"]
        SCH1["Scheduler"]
        CM1["Controller Manager"]
      end

      subgraph M2["Master 2 — 10.10.10.12"]
        API2["Kubernetes API Server"]
        ETCD2["etcd member 2"]
        SCH2["Scheduler"]
        CM2["Controller Manager"]
      end

      subgraph M3["Master 3 — 10.10.10.13"]
        API3["Kubernetes API Server"]
        ETCD3["etcd member 3"]
        SCH3["Scheduler"]
        CM3["Controller Manager"]
      end
    end

    ALB --> API1
    ALB --> API2
    ALB --> API3

    API1 <--> ETCD1
    API2 <--> ETCD2
    API3 <--> ETCD3

    ETCD1 <--> ETCD2
    ETCD2 <--> ETCD3
    ETCD1 <--> ETCD3

    SCH1 --> API1
    SCH2 --> API2
    SCH3 --> API3

    CM1 --> API1
    CM2 --> API2
    CM3 --> API3

    API1 -. "Cluster coordination" .-> API2
    API2 -. "Cluster coordination" .-> API3
```

How to read this diagram:

- The API load balancer distributes incoming API connections to healthy API Server endpoints.
- Each master runs its own control-plane components; the scheduler and controller-manager instances coordinate leader election where applicable.
- Each etcd member participates in a distributed consensus cluster.
- The API Server is the main interface through which Kubernetes clients and controllers interact with cluster state.
- The three masters communicate over the machine network. They do not need to send every application packet through the API Server.

The arrows are logical relationships, not a literal map of every connection. In particular, control-plane components can connect to API endpoints independently, and etcd peer traffic is distinct from API client traffic.

## 2.2 What runs inside a master node?

A master is not just a Linux server running one Kubernetes process. It runs the operating system, node services, control-plane containers, and OpenShift management components.

```mermaid

flowchart TB
    subgraph HOST["Master VM / Physical Server"]
      OS["Red Hat Enterprise Linux CoreOS (RHCOS)"]
      SYSTEMD["systemd"]
      KUBELET["kubelet"]
      CRIO["CRI-O container runtime"]

      subgraph STATIC["Control-plane static Pods"]
        API["Kubernetes API Server"]
        ETCD["etcd"]
        SCHED["Kubernetes Scheduler"]
        CTRL["Kubernetes Controller Manager"]
      end

      subgraph OCP["OpenShift Platform Components"]
        OAPI["OpenShift API Server"]
        OCTRL["OpenShift Controllers"]
        OP["Cluster Operators"]
      end

      OS --> SYSTEMD
      SYSTEMD --> KUBELET
      SYSTEMD --> CRIO
      KUBELET --> CRIO
      CRIO --> STATIC
      API --> OCP
    end
```

### Component responsibilities

| Component                     | What it does                                                       |
| ----------------------------- | ------------------------------------------------------------------ |
| RHCOS                         | Provides the operating system and host services.                   |
| kubelet                       | Ensures the required Pods and containers are running on that node. |
| CRI-O                         | Creates and manages containers.                                    |
| Kubernetes API Server         | Validates API requests and exposes Kubernetes resources.           |
| etcd                          | Stores persistent cluster state.                                   |
| Scheduler                     | Assigns unscheduled Pods to suitable nodes.                        |
| Controller Manager            | Reconciles Kubernetes resources toward their desired state.        |
| OpenShift API and controllers | Manage OpenShift-specific resources such as Routes and Projects.   |
| Cluster Operators             | Manage the lifecycle and health of platform components.            |

Static Pods are especially important: kubelet manages them from manifests on the node, rather than relying on the API Server to create those Pods in the first place. This helps the core control plane start even when the cluster API is not yet available.

## 2.3 How etcd quorum works

etcd is a distributed key-value database. It stores the authoritative persistent state used by Kubernetes, including objects representing workloads, configuration, and cluster resources.

With three etcd members, a majority of two members is needed to maintain quorum.

## Three-member etcd cluster

etcd-1

Healthy

etcd-2

Healthy

etcd-3

Failed

Quorum retained: 2 of 3

The cluster can continue processing etcd operations while the remaining members are healthy and communicating.

| Total members | Majority required | Member failures tolerated |
| ------------- | ----------------- | ------------------------- |
| 1             | 1                 | 0                         |
| 3             | 2                 | 1                         |
| 5             | 3                 | 2                         |

If two of the three members fail, quorum is lost. The cluster cannot safely commit new etcd updates until quorum is restored. Existing workloads may continue running on workers, but management and scheduling operations can be impaired.

Production lesson: Three masters do not mean the cluster can survive any two failures. With a three-member etcd cluster, it can generally tolerate only one member failure.

## 2.4 What happens when you deploy an application?

Suppose you run:

bash

```
oc create deployment frontend --image=nginx
```

The request follows this control-plane sequence:

```mermaid

sequenceDiagram
    participant User as oc CLI
    participant LB as API Load Balancer
    participant API as API Server
    participant ETCD as etcd
    participant S as Scheduler
    participant C as Controller
    participant K as Worker kubelet

    User->>LB: HTTPS request to port 6443
    LB->>API: Forward API request
    API->>API: Authenticate and authorize
    API->>ETCD: Persist desired state
    ETCD-->>API: Commit acknowledged
    API-->>User: Request accepted
    C->>API: Reconcile Deployment and ReplicaSet
    S->>API: Watch for unscheduled Pods
    S->>API: Bind Pod to selected worker
    K->>API: Watch for assigned Pod
    K->>K: Ask CRI-O to start containers
    K->>API: Report Pod status
```

The exact sequence involves asynchronous watches, controller reconciliation, and API updates, so not every action happens in a single linear transaction. The important concept is that the API Server records the desired state, and controllers and node agents work toward making reality match it.

Also, `oc create deployment` does not guarantee the application is immediately ready. The image must be available, a worker must have enough resources, and the Pod must pass its readiness checks.

## 2.5 Control-plane network traffic: which node talks to which?

For a physical/VM installation, these are the important communication paths:

```mermaid

flowchart LR
    ADMIN["Admin workstation"]
    LB["API VIP / Load Balancer"]

    subgraph CP["Control Plane Network"]
      A["Master 1"]
      B["Master 2"]
      C["Master 3"]
    end

    subgraph WORKERS["Worker Network"]
      W1["Worker 1"]
      W2["Worker 2"]
      W3["Worker 3"]
    end

    ADMIN -->|"TCP 6443"| LB
    LB --> A
    LB --> B
    LB --> C

    A <-->|"etcd peer TCP 2380"| B
    B <-->|"etcd peer TCP 2380"| C
    A <-->|"etcd peer TCP 2380"| C

    W1 -->|"API TCP 6443"| LB
    W2 -->|"API TCP 6443"| LB
    W3 -->|"API TCP 6443"| LB

    A -. "Pod scheduling / API state" .-> W1
    B -. "Pod scheduling / API state" .-> W2
    C -. "Pod scheduling / API state" .-> W3
```

The diagram shows logical connections. Actual connections can use direct node addresses or load-balanced endpoints depending on the component and configuration.

| Traffic              | Typical port | Purpose                                       |
| -------------------- | ------------ | --------------------------------------------- |
| Client to API Server | TCP 6443     | Cluster API access                            |
| etcd peer to peer    | TCP 2380     | Replication and consensus                     |
| API Server to etcd   | TCP 2379     | etcd client requests                          |
| Worker to API        | TCP 6443     | Node registration, watches and status updates |
| Node to node overlay | UDP 6081     | Geneve encapsulation for OVN-Kubernetes       |

Use the port requirements for your exact OpenShift release and network plugin when building firewall rules. See [Red Hat OpenShift control-plane architecture](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/architecture/control-plane?utm_source=chatgpt.com).

Red Hat Documentation

+1

## 2.6 What happens if a master fails?

Imagine master-1 suddenly goes offline.

```mermaid

flowchart TD
    FAIL["Master 1 fails"]
    ETCD["Two etcd members remain"]
    API["Load balancer detects unhealthy API endpoint"]
    REQ["New API requests"]
    WORK["Existing worker workloads"]

    FAIL --> ETCD
    FAIL --> API
    ETCD -->|"Quorum retained"| REQ
    API -->|"Route to healthy API endpoints"| REQ
    FAIL -. "Does not automatically stop all running Pods" .-> WORK
```

In a healthy three-master cluster:

1. The API load balancer stops sending new connections to the failed API endpoint after its health checks detect the failure.
2. The two remaining etcd members retain quorum.
3. The surviving control-plane components continue handling requests and reconciling resources.
4. Existing application Pods on healthy workers can generally continue serving traffic.
5. If a workload actually needs to move, Kubernetes must detect the relevant failure and schedule replacement Pods where possible.

A master failure does not automatically mean that all application traffic stops. However, recovery depends on which component failed, how quickly health checks react, and whether the remaining infrastructure has sufficient capacity.

## 2.7 Commands to inspect the control plane

Run these from an authenticated administrative workstation:

bash

```
# List control-plane nodes
oc get nodes -l node-role.kubernetes.io/control-plane

# Check cluster operators
oc get clusteroperators

# Check the API endpoint and cluster version
oc whoami --show-server
oc get clusterversion

# Inspect control-plane Pods
oc get pods -n openshift-kube-apiserver -o wide
oc get pods -n openshift-etcd -o wide
oc get pods -n openshift-kube-scheduler -o wide
oc get pods -n openshift-kube-controller-manager -o wide

# Check etcd health status through the operator
oc get co etcd
```

Some commands require elevated privileges, and Pod names and placement can vary across releases. Do not restart etcd or delete control-plane Pods as a troubleshooting shortcut.

## Diagram 2 recap

The key distinction is:

- API Server: accepts and processes cluster API requests.
- etcd: durably stores cluster state and uses quorum to agree on updates.
- Scheduler: selects a node for each new Pod.
- Controllers: continually reconcile desired and actual state.
- kubelet and CRI-O: turn the assigned workload into running containers.

These are the control mechanisms. The application data path—how browser requests and Pod-to-Pod packets travel—is a separate network path, which we will examine in the following diagrams.

Next: Diagram 3 — DNS, API VIP, load balancers, and the bootstrap process. We'll trace how `api.ocp.example.com` resolves, how the load balancer selects a master, how a new cluster bootstraps, and how DNS and health checks affect cluster availability.

## Diagram 1 — Complete OpenShift control-plane architecture

```mermaid
flowchart TB
    ADMIN["Administrator / oc CLI / Web Console / CI-CD"]
    DNS["DNS: api.ocp.example.com"]
    LB["API Load Balancer / API VIP :6443"]

    ADMIN --> DNS --> LB

    subgraph CP["OPENSHIFT CONTROL PLANE — 3 MASTER NODES"]
        direction TB

        subgraph M1["MASTER 1 — 10.10.10.11"]
            direction TB
            OS1["RHCOS / Linux Kernel / systemd"]
            K1["kubelet"]
            R1["CRI-O"]
            A1["Kubernetes API Server"]
            E1["etcd Member 1"]
            S1["Kubernetes Scheduler"]
            C1["Kubernetes Controller Manager"]
            OS1 --> K1
            OS1 --> R1
            K1 --> R1
            K1 -. "Static Pod manifests" .-> A1
            K1 -.-> E1
            K1 -.-> S1
            K1 -.-> C1
        end

        subgraph M2["MASTER 2 — 10.10.10.12"]
            direction TB
            OS2["RHCOS / Linux Kernel / systemd"]
            K2["kubelet"]
            R2["CRI-O"]
            A2["Kubernetes API Server"]
            E2["etcd Member 2"]
            S2["Kubernetes Scheduler"]
            C2["Kubernetes Controller Manager"]
            OS2 --> K2
            OS2 --> R2
            K2 --> R2
            K2 -.-> A2
            K2 -.-> E2
            K2 -.-> S2
            K2 -.-> C2
        end

        subgraph M3["MASTER 3 — 10.10.10.13"]
            direction TB
            OS3["RHCOS / Linux Kernel / systemd"]
            K3["kubelet"]
            R3["CRI-O"]
            A3["Kubernetes API Server"]
            E3["etcd Member 3"]
            S3["Kubernetes Scheduler"]
            C3["Kubernetes Controller Manager"]
            OS3 --> K3
            OS3 --> R3
            K3 --> R3
            K3 -.-> A3
            K3 -.-> E3
            K3 -.-> S3
            K3 -.-> C3
        end
    end

    LB --> A1
    LB --> A2
    LB --> A3

    A1 <--> E1
    A2 <--> E2
    A3 <--> E3

    E1 <-->|"Raft / TCP 2380"| E2
    E2 <-->|"Raft / TCP 2380"| E3
    E1 <-->|"Raft / TCP 2380"| E3

    A1 <-. "API watches / coordination" .-> A2
    A2 <-. "API watches / coordination" .-> A3

    subgraph OCP["OPENSHIFT PLATFORM SERVICES"]
        OAPI["OpenShift API Server"]
        OAUTH["OAuth / Authentication"]
        ROUTE["Ingress Operator / Router"]
        NET["Cluster Network Operator"]
        MCO["Machine Config Operator"]
        CVO["Cluster Version Operator"]
        ETCDOP["etcd Operator"]
        OBS["Monitoring / Alerting"]
    end

    A1 <--> OAPI
    A2 <--> OAUTH
    A3 <--> ROUTE
    A1 <--> NET
    A2 <--> MCO
    A3 <--> CVO
    A1 <--> ETCDOP
    A2 <--> OBS

    subgraph WORKERS["WORKER NODES"]
        W1["Worker 1<br/>kubelet + CRI-O + Pods"]
        W2["Worker 2<br/>kubelet + CRI-O + Pods"]
        W3["Worker 3<br/>kubelet + CRI-O + Pods"]
    end

    W1 -->|"HTTPS API :6443"| LB
    W2 -->|"HTTPS API :6443"| LB
    W3 -->|"HTTPS API :6443"| LB

    classDef api fill:#dbeafe,stroke:#2563eb,color:#111827
    classDef db fill:#dcfce7,stroke:#15803d,color:#111827
    classDef platform fill:#f3e8ff,stroke:#7e22ce,color:#111827
    class A1,A2,A3 api
    class E1,E2,E3 db
    class OAPI,OAUTH,ROUTE,NET,MCO,CVO,ETCDOP,OBS platform
```

## Diagram 2 — Deep internal architecture of a master node

```mermaid
flowchart TB
    subgraph HOST["MASTER NODE — RHCOS"]
        direction TB

        subgraph OS["OPERATING SYSTEM LAYER"]
            KERNEL["Linux Kernel"]
            SYSTEMD["systemd"]
            NETDEV["Host Networking / NIC / Routes"]
            DISK["Persistent Disk / Filesystem"]
            CERT["Certificates / Keys / Trust"]
        end

        subgraph NODE["NODE AGENT LAYER"]
            KUBELET["kubelet"]
            CRIO["CRI-O Runtime"]
            CNI["CNI / OVN-Kubernetes Integration"]
        end

        subgraph STATIC["STATIC PODS — MANAGED BY KUBELET"]
            API["kube-apiserver"]
            ETCD["etcd"]
            SCHED["kube-scheduler"]
            KCM["kube-controller-manager"]
        end

        subgraph PLATFORM["OPENSHIFT-MANAGED PODS AND SERVICES"]
            OAPI["OpenShift API Server"]
            OAUTH["OAuth API / Authentication"]
            CVO["Cluster Version Operator"]
            ETCDOP["etcd Operator"]
            MCO["Machine Config Operator"]
            CNO["Cluster Network Operator"]
            INGRESS["Ingress Operator / Router"]
            MON["Monitoring Components"]
        end

        KERNEL --> SYSTEMD
        SYSTEMD --> KUBELET
        SYSTEMD --> CRIO
        NETDEV <--> KERNEL
        DISK <--> KERNEL
        CERT --> API
        CERT --> ETCD

        KUBELET -->|"Pod lifecycle"| CRIO
        KUBELET -->|"Read local manifests"| STATIC
        CRIO --> API
        CRIO --> ETCD
        CRIO --> SCHED
        CRIO --> KCM
        CRIO --> PLATFORM

        API <--> ETCD
        API <--> SCHED
        API <--> KCM
        API <--> PLATFORM

        ETCD <--> DISK
        API <--> CERT
        KUBELET <--> NETDEV
        CNI <--> NETDEV
    end

    LB["External API Load Balancer"]
    WORKERS["Worker Nodes"]
    PEERS["Other Control-plane Nodes"]

    LB -->|"TCP 6443"| API
    API <-->|"etcd client TCP 2379"| ETCD
    ETCD <-->|"etcd peer TCP 2380"| PEERS
    API <-->|"Node API traffic"| WORKERS
```

## Diagram 3 — API Server request lifecycle, authentication, authorization, admission, and persistence

```mermaid
sequenceDiagram
    autonumber
    participant U as Administrator / oc
    participant LB as API Load Balancer
    participant API as Kubernetes API Server
    participant AUTH as Authentication
    participant RBAC as Authorization / RBAC
    participant ADM as Admission Plugins
    participant STORE as API Storage Layer
    participant E as etcd Cluster
    participant C as Controllers
    participant W as Watch Clients

    U->>LB: HTTPS request :6443
    LB->>API: Forward to healthy API endpoint
    API->>AUTH: Authenticate identity
    AUTH-->>API: User / ServiceAccount identity

    API->>RBAC: Check requested verb and resource
    RBAC-->>API: Allow or deny

    alt Authorization denied
        API-->>U: 403 Forbidden
    else Authorized
        API->>ADM: Validate and mutate request
        ADM-->>API: Approved object or rejection

        alt Admission rejected
            API-->>U: Rejection response
        else Admission approved
            API->>STORE: Process resource operation
            STORE->>E: Persist state change
            E-->>STORE: Commit acknowledgment
            STORE-->>API: Operation result
            API-->>U: Success response

            E-->>W: Committed state becomes observable
            W-->>C: Resource change / watch event
            C->>API: Reconcile related resources
        end
    end
```

## Diagram 4 — Three-member etcd architecture and Raft consensus

```mermaid
flowchart TB
    subgraph CLIENTS["ETCD CLIENTS"]
        A1["API Server — Master 1"]
        A2["API Server — Master 2"]
        A3["API Server — Master 3"]
    end

    subgraph ETCD["THREE-MEMBER ETCD CLUSTER"]
        E1["etcd-1<br/>Master 1"]
        E2["etcd-2<br/>Master 2"]
        E3["etcd-3<br/>Master 3"]
    end

    A1 -->|"Client API :2379"| E1
    A2 -->|"Client API :2379"| E2
    A3 -->|"Client API :2379"| E3

    E1 <-->|"Raft messages / replication :2380"| E2
    E2 <-->|"Raft messages / replication :2380"| E3
    E1 <-->|"Raft messages / replication :2380"| E3

    E1 -. "Leader election / log replication" .-> E2
    E1 -.-> E3

    subgraph LOGIC["CONSENSUS LOGIC"]
        LEADER["Raft Leader"]
        LOG["Replicated Log"]
        COMMIT["Commit after majority agreement"]
        STATE["Consistent committed cluster state"]
        LEADER --> LOG --> COMMIT --> STATE
    end

    E1 --> LOGIC
    E2 --> LOGIC
    E3 --> LOGIC
```

## Diagram 5 — Interactive etcd quorum and failure scenarios

```mermaid
flowchart TD
    START["Three voting members<br/>etcd-1, etcd-2, etcd-3"]
    Q["Quorum = floor(N / 2) + 1 = 2"]
    HEALTHY{"How many members remain healthy?"}

    FULL["3 of 3 healthy"]
    ONE["2 of 3 healthy"]
    LOST["1 of 3 healthy"]
    NONE["0 of 3 healthy"]

    NORMAL["Quorum available<br/>Normal consensus operation"]
    DEGRADED["Quorum available<br/>One member failure tolerated"]
    NOQUORUM["Quorum lost<br/>Cannot commit new etcd writes"]
    OUTAGE["Persistent cluster state unavailable"]

    START --> Q --> HEALTHY

    HEALTHY -->|"3"| FULL --> NORMAL
    HEALTHY -->|"2"| ONE --> DEGRADED
    HEALTHY -->|"1"| LOST --> NOQUORUM
    HEALTHY -->|"0"| NONE --> OUTAGE

    NORMAL --> REC["Monitor health and latency"]
    DEGRADED --> REC
    NOQUORUM --> RESTORE["Restore member connectivity or recover failed members"]
    OUTAGE --> RESTORE
    RESTORE --> CHECK["Verify etcd health and quorum"]
    CHECK --> NORMAL
```

## Diagram 6 — Deployment creation: complete control-plane-to-worker sequence

```mermaid
sequenceDiagram
    autonumber
    participant U as oc CLI
    participant LB as API Load Balancer
    participant API as Kubernetes API Server
    participant E as etcd
    participant DC as Deployment Controller
    participant RC as ReplicaSet Controller
    participant S as Scheduler
    participant K as Worker kubelet
    participant CRI as CRI-O
    participant REG as Container Registry

    U->>LB: Create Deployment frontend
    LB->>API: Forward authenticated request
    API->>API: Authenticate / authorize / admission
    API->>E: Persist Deployment
    E-->>API: Commit acknowledgment
    API-->>U: Deployment accepted

    DC->>API: Watch Deployment changes
    DC->>API: Create ReplicaSet
    API->>E: Persist ReplicaSet
    E-->>API: Commit

    RC->>API: Observe ReplicaSet
    RC->>API: Create Pod objects
    API->>E: Persist Pod objects
    E-->>API: Commit

    S->>API: Observe unscheduled Pods
    S->>S: Evaluate resources and constraints
    S->>API: Bind Pod to selected worker
    API->>E: Persist scheduling assignment
    E-->>API: Commit

    K->>API: Watch assigned Pods
    API-->>K: Pod specification
    K->>CRI: Ensure container is running
    CRI->>REG: Pull image if required
    REG-->>CRI: Image layers
    CRI->>CRI: Create and start container
    K->>API: Report Pod status and readiness

    API-->>DC: Updated resource status
    DC->>API: Reconcile Deployment status
    API-->>U: Updated state visible to oc
```

## Diagram 7 — Desired state reconciliation and self-healing

```mermaid
flowchart TD
    USER["Administrator creates Deployment"]
    API["API Server"]
    ETCD["etcd: persisted desired state"]
    DC["Deployment controller"]
    RS["ReplicaSet controller"]
    POD["Pod objects"]
    SCHED["Scheduler"]
    NODE["Selected worker"]
    KUBELET["kubelet"]
    CRIO["CRI-O"]
    APP["Application container"]
    STATUS["Observed status and readiness"]

    USER --> API --> ETCD
    ETCD --> DC
    DC -->|"Desired replica count"| RS
    RS -->|"Create missing Pods"| POD
    POD --> SCHED
    SCHED --> NODE
    NODE --> KUBELET
    KUBELET --> CRIO
    CRIO --> APP
    APP --> STATUS
    STATUS --> API
    API --> DC

    FAIL["Pod crashes or disappears"]
    FAIL --> STATUS
    STATUS --> DC
    DC -->|"Replica deficit detected"| RS
    RS -->|"Create replacement Pod"| POD
```

## Diagram 8 — Scheduler internals and Pod placement

```mermaid
flowchart TD
    P["Unscheduled Pod"]
    QUEUE["Scheduling Queue"]
    FILTER["Filter nodes against constraints"]
    RES["CPU / Memory / Pod capacity"]
    AFF["Affinity / Anti-affinity"]
    TAINT["Taints / Tolerations"]
    SEL["Node Selector / Node Affinity"]
    VOL["Volume and topology constraints"]
    FIT{"Any feasible nodes?"}
    FAIL["Pod remains Pending<br/>Scheduling events emitted"]
    SCORE["Score feasible nodes"]
    PICK["Select best candidate"]
    BIND["Bind Pod through API"]
    ETCD["Persist assignment"]
    KUBELET["Target node kubelet"]
    CRI["CRI-O starts containers"]
    STATUS["Pod status reported"]

    P --> QUEUE --> FILTER
    FILTER --> RES
    FILTER --> AFF
    FILTER --> TAINT
    FILTER --> SEL
    FILTER --> VOL
    RES --> FIT
    AFF --> FIT
    TAINT --> FIT
    SEL --> FIT
    VOL --> FIT

    FIT -->|"No"| FAIL
    FIT -->|"Yes"| SCORE --> PICK --> BIND --> ETCD
    ETCD --> KUBELET --> CRI --> STATUS
```

## Diagram 9 — Leader election and controller-manager high availability

```mermaid
flowchart TB
    API["Kubernetes API Server"]
    LEASE["Coordination Lease / Election State"]

    subgraph SCHED["SCHEDULER INSTANCES"]
        S1["Scheduler — Master 1"]
        S2["Scheduler — Master 2"]
        S3["Scheduler — Master 3"]
    end

    subgraph CTRL["CONTROLLER-MANAGER INSTANCES"]
        C1["Controller Manager — Master 1"]
        C2["Controller Manager — Master 2"]
        C3["Controller Manager — Master 3"]
    end

    API <--> LEASE

    S1 <--> LEASE
    S2 <--> LEASE
    S3 <--> LEASE

    C1 <--> LEASE
    C2 <--> LEASE
    C3 <--> LEASE

    LEADER_S["Active scheduler leader"]
    LEADER_C["Active controller-manager leader"]

    S1 -. "Election result" .-> LEADER_S
    S2 -. "Election result" .-> LEADER_S
    S3 -. "Election result" .-> LEADER_S

    C1 -. "Election result" .-> LEADER_C
    C2 -. "Election result" .-> LEADER_C
    C3 -. "Election result" .-> LEADER_C

    LEADER_S --> WORK["Schedule pending Pods"]
    LEADER_C --> RECON["Run active controller loops"]

    FAIL["Current leader fails"]
    FAIL --> NEW["Lease expires or leadership is released"]
    NEW --> ELECTION["Another eligible instance acquires leadership"]
    ELECTION --> LEADER_S
    ELECTION --> LEADER_C
```

## Diagram 10 — OpenShift Operators and platform lifecycle management

```mermaid
flowchart TB
    ADMIN["Administrator / Cluster Configuration"]
    API["Kubernetes API Server"]
    ETCD["etcd Persistent State"]

    subgraph OPERATORS["OPENSHIFT OPERATOR FRAMEWORK"]
        CVO["Cluster Version Operator"]
        MCO["Machine Config Operator"]
        CNO["Cluster Network Operator"]
        AUTH["Authentication Operator"]
        ING["Ingress Operator"]
        ETCDOP["etcd Operator"]
        KAPI["kube-apiserver Operator"]
    end

    subgraph TARGETS["MANAGED PLATFORM COMPONENTS"]
        RELEASE["Release Payload / Version State"]
        MACHINES["Node Configuration / OS Updates"]
        NETWORK["OVN-Kubernetes / Network Configuration"]
        OAUTH["OAuth / Authentication Components"]
        ROUTER["Ingress Controller / Routers"]
        ETCDCL["etcd Cluster"]
        APIS["Kubernetes API Server"]
    end

    ADMIN --> API --> ETCD

    API --> CVO
    API --> MCO
    API --> CNO
    API --> AUTH
    API --> ING
    API --> ETCDOP
    API --> KAPI

    CVO --> RELEASE
    MCO --> MACHINES
    CNO --> NETWORK
    AUTH --> OAUTH
    ING --> ROUTER
    ETCDOP --> ETCDCL
    KAPI --> APIS

    RELEASE -. "Observed version and health" .-> CVO
    MACHINES -. "Node configuration status" .-> MCO
    NETWORK -. "Network health" .-> CNO
    OAUTH -. "Authentication health" .-> AUTH
    ROUTER -. "Ingress health" .-> ING
    ETCDCL -. "etcd health" .-> ETCDOP
    APIS -. "API health" .-> KAPI
```

## Diagram 11 — API traffic, etcd traffic, node traffic, and Pod data plane

```mermaid
flowchart TB
    ADMIN["Admin Workstation / CI"]
    VIP["API VIP :6443"]

    subgraph CP["CONTROL PLANE"]
        A1["Master 1 API Server"]
        A2["Master 2 API Server"]
        A3["Master 3 API Server"]
        E1["etcd-1"]
        E2["etcd-2"]
        E3["etcd-3"]
    end

    subgraph WORKERS["WORKER NODES"]
        W1["Worker 1"]
        W2["Worker 2"]
        W3["Worker 3"]
    end

    subgraph PODNET["POD NETWORK"]
        P1["Application Pod A"]
        P2["Application Pod B"]
        P3["Application Pod C"]
    end

    ADMIN -->|"TCP 6443"| VIP
    VIP --> A1
    VIP --> A2
    VIP --> A3

    A1 -->|"TCP 2379"| E1
    A2 -->|"TCP 2379"| E2
    A3 -->|"TCP 2379"| E3

    E1 <-->|"TCP 2380"| E2
    E2 <-->|"TCP 2380"| E3
    E1 <-->|"TCP 2380"| E3

    W1 -->|"API HTTPS :6443"| VIP
    W2 -->|"API HTTPS :6443"| VIP
    W3 -->|"API HTTPS :6443"| VIP

    W1 --- P1
    W2 --- P2
    W3 --- P3

    P1 <-->|"OVN-Kubernetes overlay / routing"| P2
    P2 <-->|"OVN-Kubernetes overlay / routing"| P3
    P1 <-->|"Pod networking"| P3
```

## Diagram 12 — Master failure and recovery

```mermaid
flowchart TD
    FAIL["Master 1 fails"]
    LB["API load balancer detects failed backend"]
    E1["etcd-1 unavailable"]
    E23["etcd-2 and etcd-3 remain"]
    Q{"Do remaining etcd members have quorum?"}

    API["Surviving API Servers"]
    CTRL["Controllers and scheduler"]
    WORK["Existing Pods on healthy workers"]
    REPLACE["Replacement Pods when needed"]
    APP["Applications continue if their dependencies remain healthy"]

    FAIL --> LB
    FAIL --> E1
    E1 --> E23
    E23 --> Q

    Q -->|"Yes: 2 of 3"| API
    API --> CTRL
    CTRL --> REPLACE
    REPLACE --> APP

    FAIL -. "Does not directly terminate every running Pod" .-> WORK
    WORK --> APP

    Q -->|"No majority"| OUTAGE["New persistent state updates impaired"]
    OUTAGE --> RECOVERY["Restore quorum using supported recovery procedures"]
    RECOVERY --> VERIFY["Verify etcd, API, Operators and nodes"]
    VERIFY --> API
```

## Diagram 13 — Troubleshooting decision tree

```mermaid
flowchart TD
    START["OpenShift cluster or application issue"]
    API{"Can clients access the API?"}
    NODES{"Are control-plane nodes Ready?"}
    ETCD{"Is etcd healthy with quorum?"}
    OPS{"Are critical Cluster Operators healthy?"}
    PODS{"Are application Pods scheduled and ready?"}

    LB["Check DNS / VIP / Load Balancer / TCP 6443"]
    HOST["Check RHCOS / kubelet / host resources"]
    DB["Check etcd member health / latency / peer connectivity"]
    OP["Inspect Operator conditions and component logs"]
    SCHED["Check scheduler events, requests and node constraints"]
    APP["Check image pulls, storage, probes and application logs"]
    NET["Check Services, Routes, DNS and OVN-Kubernetes"]

    START --> API
    API -->|"No"| LB
    API -->|"Yes"| NODES
    NODES -->|"No"| HOST
    NODES -->|"Yes"| ETCD
    ETCD -->|"No"| DB
    ETCD -->|"Yes"| OPS
    OPS -->|"No"| OP
    OPS -->|"Yes"| PODS
    PODS -->|"No"| SCHED
    PODS -->|"Yes, but unreachable"| NET

    LB --> VERIFY["Verify recovery"]
    HOST --> VERIFY
    DB --> VERIFY
    OP --> VERIFY
    SCHED --> APP
    APP --> VERIFY
    NET --> VERIFY
```

## Diagram 14 — Complete end-to-end architecture: management plane versus application data plane

```mermaid
flowchart TB
    subgraph MANAGEMENT["MANAGEMENT PLANE"]
        ADMIN["Administrator / CI-CD / Console"]
        VIP["API Load Balancer :6443"]

        subgraph CONTROL["Highly Available Control Plane"]
            API["Kubernetes API Servers"]
            ETCD["Three-member etcd / Raft Consensus"]
            SCHED["Scheduler"]
            CTRL["Kubernetes Controllers"]
            OPS["OpenShift Operators"]
        end

        ADMIN --> VIP --> API
        API <--> ETCD
        API <--> SCHED
        API <--> CTRL
        API <--> OPS
    end

    subgraph EXECUTION["WORKLOAD EXECUTION PLANE"]
        W1["Worker 1<br/>kubelet + CRI-O"]
        W2["Worker 2<br/>kubelet + CRI-O"]
        W3["Worker 3<br/>kubelet + CRI-O"]

        P1["Frontend Pod"]
        P2["Backend Pod"]
        P3["Database Pod"]

        W1 --> P1
        W2 --> P2
        W3 --> P3
    end

    API <-->|"Pod specs / status / watches"| W1
    API <-->|"Pod specs / status / watches"| W2
    API <-->|"Pod specs / status / watches"| W3

    subgraph APPLICATION["APPLICATION DATA PATH"]
        CLIENT["External Client"]
        ROUTER["OpenShift Ingress / Router"]
        SVC["Kubernetes Service"]
        APP["Frontend / Backend"]
        DB["Database / Persistent Storage"]

        CLIENT --> ROUTER
        ROUTER --> SVC
        SVC --> APP
        APP --> DB
    end

    P1 <--> P2
    P2 <--> P3

    OPS -. "Manage platform configuration" .-> EXECUTION
    CTRL -. "Reconcile workload state" .-> EXECUTION
```
