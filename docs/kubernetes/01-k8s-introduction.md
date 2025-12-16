# Kubernetes Introduction & Architecture

## What is Kubernetes?

**Kubernetes** (also known as **k8s** or "kube") is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications.

Think of Docker as a way to create and run a single container (like a single ship), while **Kubernetes is the crane and harbor master** responsible for managing a fleet of ships (containers) across multiple docks (servers).

### A Brief History
*   **Origin:** Developed by Google engineers.
*   **Inspiration:** Based on Google's internal cluster management system called **Borg**, which they used for over a decade to run services like Gmail and YouTube.
*   **Open Source:** Donated to the Cloud Native Computing Foundation (CNCF) in 2015.

---

## Why Do We Need It? (Key Features)

*   **⚡ Self-healing:** restarts containers that fail, replaces and reschedules containers when nodes die.
*   **⚖️ Automated Rollouts & Rollbacks:** You can describe the desired state for your deployed containers, and it can change the actual state to the desired state at a controlled rate.
*   **🧱 Service Discovery & Load Balancing:** If a container is high traffic, Kubernetes is able to load balance and distribute the network traffic so that the deployment is stable.
*   **📦 Storage Orchestration:** Automatically mount a storage system of your choice (local storage, cloud providers).

---

## Kubernetes Architecture

A Kubernetes cluster consists of a set of worker machines, called **nodes**, that run containerized applications. Every cluster has at least one worker node. The **Control Plane** manages the worker nodes and the Pods in the cluster.

```mermaid
graph TD
    subgraph CP [Control Plane (Master Node)]
        API[API Server]
        ETCD[(etcd)]
        SCH[Scheduler]
        CM[Controller Manager]
        CCM[Cloud Controller<br>Manager]
        
        API <--> ETCD
        API <--> SCH
        API <--> CM
        API <--> CCM
    end

    subgraph WN1 [Worker Node 1]
        Kubelet1[Kubelet]
        Proxy1[Kube-proxy]
        Pod1[Pod]
        
        Kubelet1 <--> API
        Proxy1 -.-> Pod1
    end

    subgraph WN2 [Worker Node 2]
        Kubelet2[Kubelet]
        Proxy2[Kube-proxy]
        Pod2[Pod]
        
        Kubelet2 <--> API
        Proxy2 -.-> Pod2
    end
    
    style CP fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
    style WN1 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    style WN2 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
```

### 1. Control Plane Component (The Brain)
The control plane makes global decisions about the cluster (for example, scheduling), as well as detecting and responding to cluster events.

*   **kube-apiserver:** The front end of the control plane. It exposes the Kubernetes API. All tools (kubectl, dashboard) talk to this.
*   **etcd:** Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data. **This is the single source of truth.**
*   **kube-scheduler:** Watches for newly created Pods with no assigned node, and selects a node for them to run on.
*   **kube-controller-manager:** Runs controller processes (like Node Controller, Job Controller). It ensures the "Current State" matches the "Desired State".

### 2. Node Components (The Workers)
Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

*   **kubelet:** An agent that runs on each node in the cluster. It ensures that containers are running in a Pod.
*   **kube-proxy:** A network proxy running on each node. It maintains network rules that allow network communication to your Pods.
*   **Container Runtime:** The software that is responsible for running containers (e.g., Docker, containerd, CRI-O).

---

> [!NOTE]
> We will dive deeper into each of these components in the upcoming sections.
