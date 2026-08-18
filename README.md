# Enterprise-Exchange-DAG-Architecture
high-availability, disaster-recovery, windows-server-core, exchange-server, proxmox
# 🚀 Enterprise Exchange 2019 DAG Architecture on Windows Server Core

**Author:** İsmail Bostan 

## 📌 Executive Summary
In highly regulated sectors (Finance, Healthcare, Heavy Industry), maintaining on-premise "air-gapped" or hybrid communication infrastructures is a strict compliance requirement. This project demonstrates the deployment of a **Microsoft Exchange 2019 Database Availability Group (DAG)** cluster utilizing **Windows Server Core**. 

By consciously excluding the Windows Desktop Experience (GUI), this architecture drastically optimizes hardware resource consumption (RAM/CPU footprint) while maintaining 100% feature parity. Furthermore, this underlying architecture shares the exact core topology and command-line management structure as Microsoft's next-generation **Exchange Server Subscription Edition (SE)**, rendering this deployment highly future-proof.

---

## 🏗️ Architecture & Lab Environment
The entire topology is virtualized on a Proxmox VE cluster. 

*   **Hypervisor:** Proxmox VE
*   **Domain Controller / FSW:** Windows Server 2022 (GUI) - IP: 192.168.1.xxx
*   **Exchange Node 1 (EXCH01):** Windows Server 2022 Core - 8GB RAM, 4 vCPU - IP: 192.168.1.xxx
*   **Exchange Node 2 (EXCH02):** Windows Server 2022 Core - 8GB RAM, 4 vCPU - IP: 192.168.1.xxx
*   **DAG Cluster IP:** 192.168.1.xxx

![EXCH01 Specs](ex1vm spec.png)
![EXCH02 Specs](ex2vm spec.png)

---

## 🛠️ Phase 1: Infrastructure & Prerequisites

### 1. OS Deployment & Domain Join
Windows Server Core environments are managed exclusively via command line. Initial configuration (Static IP, Hostname, and Domain Join) was executed using the `sconfig` utility.

![Domain Join EXCH01](ex1vm domain join.png)
![Domain Join EXCH02](ex2vm domain join.png)

> **📝 Technical Note: Disabling IPv6 for DNS Resolution Conflicts**
> Microsoft officially states that completely disabling IPv6 via the Windows Registry in an Exchange Server architecture is an unsupported scenario. However, unbinding IPv6 strictly at the Network Interface Card (NIC) level using `Disable-NetAdapterBinding` is entirely safe and highly recommended. This approach effectively resolves DNS priority conflicts during the initial domain join process without compromising underlying Exchange Server dependencies.

### 2. Windows Features & Component Installation
Exchange 2019 dependencies, including IIS, Failover Clustering, and RSAT tools, were deployed silently via PowerShell across all nodes.

![Windows Features](ex1vm features.png)

### 3. Silent Prerequisite Packages
Due to the lack of a GUI wrapper on Server Core, Unified Communications Managed API (UCMA 4.0) and Visual C++ Redistributables were extracted and installed using silent `/quiet` MSI execution parameters.

![VCRedist Install](ex1vm vcredist.png)
![UCMA Install](ex1vm UCMA install.png)

---

## ⚙️ Phase 2: Active Directory Schema Extension
Exchange requires structural modifications to the Active Directory schema. This was executed centrally using the `Setup.exe` command-line parameters.

*   `/PrepareSchema`
*   `/PrepareAD`
*   `/PrepareAllDomains`

![AD Schema Prep](ex1vm setup1.png)
![AD Prep](ex1vm setup2.png)
![Domain Prep](ex1vm setup3.png)

---

## 🚀 Phase 3: Unattended Exchange Server Installation
Server Core necessitates unattended (silent) installations. The Mailbox Role and foundational services were deployed automatically without GUI interaction.

```powershell
.\Setup.exe /Mode:Install /Role:Mailbox /IAcceptExchangeServerLicenseTerms_DiagnosticDataON
