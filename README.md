# 🚀 Cloud & IoT Edge Showcase: INSANE Virtualization

*Read this in other languages: [Italiano 🇮🇹](README.it.md)*

---

This repository serves as a showcase for my integration and virtualization project of **INSANE** (Integrated aNd Selective Acceleration at the Network Edge), a middleware developed for network acceleration in distributed applications. *Due to confidentiality restrictions, the full source code is not public. This repository contains the architectural presentation and the complete technical report of the project.*

## 🎯 Project Objective
The main objective was the containerization of INSANE-based applications and their subsequent integration within an orchestrator. The project involved the deployment of a demo application and a network daemon (`nsnd`), enabling high-performance communication via shared memory.

## 🛠 Tech Stack
* **Container & Orchestration:** Docker, Kubernetes (Kubeadm, K3s).
* **Networking & CNI:** Calico, Flannel, Multus, Macvlan.
* **OS & Low-level:** Linux (Ubuntu), HugePages (Zero-Copy shared memory).
* **Protocols:** TCP/IP, UDP.

## 🏗 Architecture and Design Choices
* **DaemonSet vs Sidecar Pattern:** For the Kubernetes implementation, I designed the architecture using the DaemonSet pattern instead of the Sidecar. This architectural choice ensures that a specific instance of the INSANE daemon runs on all cluster nodes, optimizing node-level resources and simplifying IP configuration.
* **Advanced Network Management:** To ensure proper Docker container isolation, I moved away from the `--network host` mode in favor of `macvlan` networks, assigning specific IP addresses on the physical network. On Kubernetes, I tested various CNI solutions (Flannel, Multus) before adopting **Calico**, configuring dedicated static IPs for the daemon pods.
* **Zero-Copy & HugePages:** I enabled zero-copy communication between containers by mapping the `/tmp` volumes (for control Unix sockets) and `/dev/hugepages` (for data transfer). This required configuring security flags (e.g., `--privileged` on Docker) and proper resource allocation (explicit request for `hugepages-2Mi`) in the Kubernetes YAML manifests.
* **Configuration Decoupling:** I used Kubernetes **ConfigMaps** to dynamically manage the daemon configuration files (`nsnd.cfg`), eliminating the need to rebuild Docker images for every IP change.

## 📊 Performance Analysis (Virtualization Overhead)
I conducted an extensive testing campaign to evaluate message transmission time (from 512B to 1024B, up to 100,000 packets) by comparing three environments: **Bare Metal, Docker, and Kubernetes**. **Key results:**
1. **Scalability:** The system scales linearly as the load increases, without sudden performance drops.
2. **Virtualization Impact:** The overhead introduced by Docker containers is minimal, offering performance almost identical to Bare Metal.
3. **K8s Overhead:** Kubernetes introduces the most significant latency due to the additional virtualization layer and the use of the Container Network Interface (CNI).
4. **Protocol Behavior:** The TCP protocol is more affected by the latency increase (compared to UDP) due to its native flow control mechanisms.

## 📂 Repository Content
* `Relazione_Virtualizzazione_INSANE.pdf`: Complete technical documentation of the setup, testing, and resolved challenges.
* `Presentazione_Architettura.pptx`: Summary slides on the architecture and benchmark results.
