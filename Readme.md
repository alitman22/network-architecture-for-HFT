# Low Latency Network Architecture for High-Frequency Trading (HFT)

This repository documents a network and systems architecture designed to meet the stringent demands of a High-Frequency Trading (HFT) firm. The entire design is engineered for sub-millisecond latency, absolute redundancy to eliminate single points of failure (SPoF), and adherence to strict regulatory and security standards.

---

## 📜 Overview

This architecture is built on a foundation of "defense-in-depth" security and multi-layered redundancy. Every component, from the physical servers and storage to the network fabric and security appliances, is deployed in a high-availability configuration. A sophisticated dual-fabric, multi-tier storage solution ensures data integrity and performance for every workload. The primary network fabric is a fully meshed, non-blocking Ethernet design utilizing LACP for maximum throughput, while a dedicated Fibre Channel SAN provides legacy block storage.

---

## 🗺️ Network Topology

[![network-diagram-HFT.drawio (2).png](https://kb.stellaramc.ir/uploads/images/gallery/2025-08/scaled-1680-/network-diagram-hft-drawio-2.png)](https://kb.stellaramc.ir/uploads/images/gallery/2025-08/network-diagram-hft-drawio-2.png)

---

## ⚖️ Regulatory Compliance &amp; Security Posture

This design explicitly addresses common financial industry regulations by embedding security and availability into its core.

- **No Single Point of Failure (SPoF):** Every critical path is fully redundant. This includes dual firewalls, dual core switches (Nexus vPC), stacked management switches (StackWise), and multi-path server/storage connectivity across separate network fabrics.
- **Segregation of Traffic:** **15 distinct VLANs** are implemented to logically isolate market data feeds, trading execution traffic, storage traffic (iSCSI/NFS), vMotion, management, and backup traffic. This prevents traffic contention and enhances security.
- **Hardened Attack Surface:** All Virtual Machines are hardened according to the **OWASP `server_level2` security guidelines**. The infrastructure has **no honeypots** or unnecessary services, presenting a minimal and hardened attack surface.
- **High Availability by Design:**
    - **FortiGate &amp; pfSense:** Deployed in Active-Passive HA clusters.
    - **Cisco Nexus:** Deployed as a Virtual Port Channel (vPC) domain.
    - **Cisco 3750:** Deployed as a resilient StackWise unit for the Out-of-Band Management network.

---

## 🛠️ Engineering Philosophy &amp; The Resulting List of Materials (LOM)

This infrastructure is not merely a collection of high-end components; it is the product of a single, unified architectural vision designed to solve complex business problems under a specific set of real-world constraints. The resulting List of Materials (LOM) was meticulously crafted, with every decision guided by the following core principles.

### A Holistic, Single-Architect Design

As the sole architect, I designed this entire Line of Business (LOB) infrastructure from the ground up. This ensured a cohesive and deeply integrated system, eliminating the communication gaps and conflicting priorities that often arise in multi-team projects. Every choice, from the network fabric to the storage layout and virtualization layer, was made with a full understanding of its impact on the rest of the stack. This unified vision is the key to the system's stability and performance.

### Pragmatic Sourcing in a Constrained Market

The creation of the LOM began with a rigorous analysis of the **Iranian technology market**. Rather than designing in a vacuum, the strategy was to build the most powerful and resilient system possible using hardware with **strong local distribution channels**. This pragmatic approach delivers several key business advantages:

- **Cost-Effectiveness &amp; TCO:** Prioritizing available hardware significantly lowers the Total Cost of Ownership (TCO).
- **Supply Chain Resilience:** It guarantees immediate access to **spare parts**, drastically reducing downtime risk.
- **Predictable Expansion:** It provides a clear and reliable roadmap for future expansion, as the availability of identical or compatible components is assured.

### Fanatical Dedication to Compatibility

A primary mandate of this project was to achieve flawless, end-to-end compatibility. Each component in the LOM was meticulously cross-referenced and validated to ensure perfect synergy across every layer of the technology stack:

- **Hardware &amp; Firmware:** Verifying that the specific server models (HP), switch operating systems (Cisco NX-OS), and NIC/HBA firmware versions were fully supported and interoperable.
- **Virtualization:** Ensuring deep compatibility with our chosen hypervisor, VMware vSphere, to unlock advanced features like vSAN, vMotion, and Distributed Switching without issue.
- **Operating Systems:** Guaranteeing that our chosen guest OSs (FreeBSD for pfSense/TrueNAS, Linux for applications) had stable, high-performance drivers for the underlying virtual hardware.

This "full-stack" compatibility check ensures the system operates as a single, harmonious unit, free from the subtle performance issues and instability that plague less integrated designs.

### Optimizing for Proven Performance over Bleeding-Edge

In the high-stakes world of HFT, **predictable stability is just as critical as raw speed**. Therefore, this architecture deliberately favors "battle-tested" components known for their rock-solid stability and deterministic low-latency performance. By choosing hardware that is proven in the field, we eliminate the unknown variables and potential "Day Zero" bugs associated with brand-new technologies. This design choice represents a conscious trade-off, prioritizing **unwavering reliability and consistent high performance**—the cornerstones of a successful trading infrastructure.

## ⚡ Core Technologies &amp; Implementation

### 🖥️ Compute &amp; Virtualization

- **Primary Compute Cluster:** A cluster of **HP ProLiant DL360 G10** servers provides the core processing power for trading applications (C1-C7), ClickHouse Compute Cluster nodes (CH1-CH2), and quantitative analysis (Q1).
- **Virtualization Platform:** VMware vSphere is deployed across the cluster, managed by a vCenter Server instance.

### 💾 Multi-Tier Storage Fabric

A dual-fabric storage architecture is in place to provide optimal performance and reliability for different data types.

- **Hot Tier (Hyper-Converged):**
    
    
    - **Platform:** A high-performance **VMware vSAN** cluster running on the DL360 G10 compute nodes.
    - **Capacity:** Each node contains **7 x 1.92TB Samsung PM1643A SAS SSDs**. These are presented to ESXi in software **RAID10** volumes for maximum performance and resilience.
    - **Cache:** Each node utilizes **1 x 1.92TB Intel 4600 Series NVMe SSD** as a high-speed cache tier.
- **Nearline Tier (Ethernet Fabric):**
    
    
    - **Platform:** A high-availability **TrueNAS Core** for secondary backups, logs, and ISO storage.
    - **Hot Backup Pool:** A **DL380 G9 SFF** chassis with **24 x 1TB SSDs** in a **RAIDz2** pool, configured with multiple LUNs for fast backup/restore operations.
    - **Capacity Pool:** A **DL380 G9 LFF** chassis with **12 x 18TB HDDs** in multiple pools (**RAIDz2, RAIDz, Stripe, RAID10**) for semi-cold backups and log archival.
- **Cold Tier (Fibre Channel SAN):**
    
    
    - **Platform:** An **EMC VNX5200** with a disk enclosure, connected via a dedicated **8G Fibre Channel SAN**.
    - **Configuration:** The array is configured with **32 x 1.8TB 10K RPM HDDs** in a **RAID5** pool, used for long-term, immutable archival of backups to meet compliance requirements.

### 🔒 Security Appliances &amp; Policy Enforcement

- **Perimeter Firewall (FortiGate):**
    - A cluster of two **FortiGate 100F** appliances in Active-Passive mode provides edge security, threat prevention, and VPN services.
- **Internal Segmentation Firewall (pfSense):**
    - A cluster of two custom-built firewalls on **HP DL360 G9** servers, providing robust inter-VLAN routing and access control.
    - **CPU:** 2 x Intel Xeon E5-2630v3 per node
    - **RAM:** 64GB DDR4 per node
    - **NICs:** On-board 4x1G ports for management/HA and 2 x dual-port **HP 560 10G SFP+** cards for data plane traffic.

### 🌐 Network Fabric &amp; Connectivity

- **End-to-End Link Aggregation (LACP):** To maximize bandwidth and resilience, LACP bonds are consistently implemented across the entire stack: 
    - **pfSense (FreeBSD):** LAGG interfaces are used for the **LAN (LACP)**, the internal **HA sync (Fail-over)**, and the external **WAN uplinks (Active-Passive)**.
    - **Cisco Nexus N3K Switches:** Port-channels are used for the `4x10G` vPC peer-link and for all server-facing connections.
    - **VMware ESXi:** vSphere Distributed Switches (VDS) on all hosts utilize LACP to bond the uplinks from the server NICs.
    - **TrueNAS Core:** All data plane network interfaces are configured in a resilient LACP bond.

---

## 🔬 Design Rationale

- **Perimeter (FortiGate Layer):** In an HFT context, the FortiGate 100F provides essential security without compromising performance, inspecting traffic at line rate.
- **Internal Segmentation (pfSense Layer):** The powerful custom pfSense cluster creates an ultra-secure enclave for the trading systems. By inspecting inter-VLAN traffic, it protects critical applications from any potential lateral movement within the network.
- **Core Data Plane (Cisco Nexus vPC):** For HFT, every microsecond counts. The Nexus vPC architecture provides a non-blocking, active-active Layer 2 fabric. This design ensures the lowest possible latency between servers and for market data delivery.
- **Server Connectivity:** Each HP DL360 G10 server is equipped with **two dual-port 10G SFP+ NICs (4 ports total)**. These ports are bundled in LACP port-channels connecting to **both** Nexus switches. This vPC-based mesh provides **40Gbps of active, usable throughput** per server and ensures that a failure of an entire core switch will not impact server connectivity.