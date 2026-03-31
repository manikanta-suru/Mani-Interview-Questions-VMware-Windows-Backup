# VMware vSphere Core Concepts – In-Depth Study Guide

This guide covers **core vSphere topics** often seen in VMware/Virtualization Engineer interviews. Each concept is explained in detail with official VMware references and expert resources. Practical troubleshooting tips, knowledge base articles, and recommended videos are provided. You will also find **50 advanced interview questions and answers** (L2/L3 level) at the end.

## 1. Fault Tolerance (FT) – Zero-Downtime Protection

VMware Fault Tolerance (FT) provides continuous availability for VMs by creating a **secondary, live copy** of a VM on another host. The secondary VM mirrors the primary VM’s execution state. If the primary host fails, the secondary immediately takes over **with no downtime or data loss**【85†L79-L88】. In contrast, HA (High Availability) restarts VMs after a failure (downtime occurs), but FT avoids any restart or outage.  

- **How it works:** FT runs a *paired VM* (“secondary”) on a different host. The primary and secondary are kept in lockstep via a low-latency logging channel. On primary failure, the secondary becomes the active VM instantly【85†L79-L88】.  
- **Use case:** Mission-critical workloads (finance, healthcare, etc.) that cannot tolerate any interruption.  
- **Requirements:** FT-enabled license, compatible hardware, low latency network, and (in older versions) up to 2 vCPUs for standard FT (higher limits in Enterprise Plus)【85†L79-L88】【85†L75-L77】.

> **Key point:** “When FT is enabled, a copy of the primary VM is automatically created on another host… If there is a hardware failure, the secondary VM immediately picks up where the primary stopped without data loss”【85†L79-L88】. 

> **Compare HA vs FT:** HA monitors hosts and restarts VMs on surviving hosts after a failure (causing downtime). FT requires continuous mirroring and provides *zero-downtime* takeover【85†L79-L88】.  

## 2. Admission Control – Ensuring Failover Capacity

Admission Control in a vSphere HA cluster **reserves resources** so that VMs can be restarted if a host fails. In simple terms, it prevents overcommitment by guaranteeing **spare capacity**. As one VMware expert notes, HA/Admission Control “ensure that sufficient resources are reserved for virtual machine recovery when a host fails”【121†L31-L34】. 

- **Function:** If you configure Admission Control (e.g. “Host failures cluster tolerates = 1”), the cluster keeps ~1 host’s worth of CPU/memory spare. New VMs or placements are blocked if they would violate that reserve.  
- **Failover Capacity:** Ensures the cluster can restart VMs from a failed host on remaining hosts. For example, in a 4-host cluster with 1-failure policy, resources equal to 1 host are reserved【121†L31-L34】.  
- **Example:** With 4 hosts, you may reserve ~25% of capacity for failover. If one host fails, HA has enough spare CPU/mem on the other 3 to power on all VMs that were on the failed host【121†L31-L34】.  

> **In practice:** “vSphere HA and Admission Control ensure that sufficient resources are reserved for virtual machine recovery when a host fails”【121†L31-L34】. Thus, Admission Control enforces your chosen failover level (N+1, N+2, etc.) to **guarantee cluster resilience**.  

## 3. EVC (Enhanced vMotion Compatibility) – CPU Compatibility

Enhanced vMotion Compatibility (EVC) masks CPU differences across hosts, enabling seamless live migration (vMotion) even if hosts have different CPU generations. EVC enforces a consistent baseline of CPU instruction sets across the cluster【17†L19-L23】. 

- **Why needed:** Without EVC, vMotion will fail if migrating between hosts with incompatible CPU features. EVC “ensures that workloads can be live migrated between ESXi hosts in a cluster running different CPU generations” by hiding newer CPU features【17†L19-L23】.  
- **How it works:** When you enable EVC on a cluster, the hosts present a common (lower) CPU feature set to VMs. This harmonizes CPUID to the baseline of the oldest CPU family in the cluster.  
- **Best practice:** Always enable EVC on mixed-generation clusters to avoid vMotion/FT failures【17†L19-L23】.  

> **Official note:** “vSphere Enhanced vMotion Compatibility (EVC) ensures that workloads can be live migrated, using vMotion, between ESXi hosts in a cluster that are running different CPU generations”【17†L19-L23】. This provides the needed CPU compatibility for live migrations.  

## 4. CPU Ready Time – Measuring CPU Contention

**CPU Ready Time** is a performance metric indicating the percentage of time a VM is **ready to run but waiting** for physical CPU. High CPU Ready means the VM’s vCPUs were queued waiting for scheduling, a sign of CPU contention on the host.  

- **What it means:** If many VMs are busy, or VCPUs are overprovisioned, VMs spend time waiting for CPU. That wait time appears as CPU Ready percentage in performance stats.  
- **Symptoms of contention:** CPU Ready steadily above ~5–10% per vCPU suggests contention【19†L48-L53】. The cited VMware KB notes that “multiple resource-intensive VMs competing” and “vCPU allocation significantly exceeds physical CPU capacity” are classic causes【19†L48-L53】.  
- **Troubleshooting:** Check host CPU load, CPU overcommitment ratio, and vCPU-to-pCPU oversubscription. Reducing vCPUs or migrating VMs to less-loaded hosts can reduce Ready time.  

> **Key insight:** “When multiple resource-intensive VMs compete for CPU… or vCPU allocation significantly exceeds physical CPU capacity… high CPU ready times occur, indicating CPU contention”【19†L48-L53】. In other words, high CPU Ready = CPU overload.  

## 5. Memory Ballooning – Under Host Memory Pressure

**Memory ballooning** is a host memory reclamation technique triggered when the ESXi host is low on free memory. The host uses the *balloon driver* (vmmemctl) inside each VM to “inflate” a balloon, forcing the guest OS to relinquish unused memory back to the host.  

- **Trigger:** Ballooning starts *before* the host resorts to swapping. If free memory dips toward the soft threshold, ESXi inflates the balloon in VMs【29†L636-L644】.  
- **Process:** The balloon driver allocates memory inside the VM, pinning it (the guest sees an allocation). The guest OS then frees or swaps out that memory internally. This reclaimed memory goes back to the host.  
- **Hierarchy:** Under memory pressure, ESXi uses *TPS* first, then *ballooning*, then *memory compression*, and finally *host-level swapping* if needed【29†L636-L644】【105†L1693-L1701】. Ballooning is gentler than swapping: it causes the guest OS to manage free space.  
- **Monitoring:** In vSphere performance charts, balloon usage rising indicates host memory contention. Occasional ballooning is normal; sustained high ballooning means heavy memory oversubscription.  

> **VMware best practice:** “When a host’s free memory drops towards the threshold, the hypervisor starts to reclaim memory using ballooning before reaching critical swap”【29†L636-L644】. In effect, **ballooning indicates host memory pressure**.  

## 6. VM Encryption – Key Management (KMS)

vSphere VM Encryption secures VM files by storing them as encrypted blocks. A **Key Management Server (KMS)** (KMIP-compliant) is required to manage the encryption keys. VMware provides a vSphere Native Key Provider (NKP) but it is conceptually a local KMS.  

- **KMS role:** The KMS stores and provides cryptographic keys. vCenter Server connects to the KMS to retrieve and manage keys used for encryption.  
- **Requirement:** To use VM Encryption, you must configure an external or native KMS in vCenter (e.g. Dell/EMC DKM, HYCU KMS, etc.)【37†L174-L182】【43†L305-L309】.  
- **Scope:** KMS is also used for vSAN encryption and other VMware encryption features.  
- **License:** VM Encryption (and NKP) typically requires Enterprise Plus licensing.  

> **Manufacturer note:** VMware’s vSphere Native Key Provider (NKP) “does not require an external key server (also called a KMS in the industry)” for encryption, but relies on vCenter internal key management【37†L174-L182】. In any case, **key management via KMIP/KMS is essential** for VM encryption【43†L305-L309】.  

## 7. ESXi Lockdown Mode (Strict) – Restricting Host Access

Lockdown Mode on an ESXi host enforces that the host can only be managed via vCenter Server. In **Strict Lockdown Mode**, all direct local access (DCUI, ESXi Shell/SSH) is disabled by default unless explicitly allowed via exception users.  

- **Effects of Strict mode:** 
  - The DCUI (Direct Console UI) is disabled【47†L73-L80】.
  - SSH and ESXi Shell (Tech Support Mode) are disabled by default (can be enabled for “Exception Users” only).
  - Only the vpxa agent (vCenter agent) can communicate with host; all host management must go through vCenter.  
- **Use case:** Security hardened environments where you don’t want administrators logging directly into hosts.  
- **Note:** If vCenter is lost and Strict Lockdown is enabled without exception users, you will need to manually re-enable management (e.g. via physical access or iLO).

> **KB excerpt:** “In strict lockdown mode the DCUI service is stopped… the ESXi host becomes unavailable unless ESXi Shell and SSH are enabled and Exception Users are defined”【47†L73-L80】. In short, *no direct host login* is allowed in Strict Lockdown.  

## 8. Virtual Switching – Standard vSwitch vs Distributed vSwitch

VMware networking can use a **Standard vSwitch (vSS)** or a **Distributed vSwitch (vDS)**. 

- **Standard vSwitch (vSS):** Each ESXi host has its own standard switch. It’s managed on the host.  
- **Distributed vSwitch (vDS):** A vDS is created at vCenter and spans multiple hosts. The control plane is centralized in vCenter while the data plane runs on each host’s switch. vDS requires vCenter for configuration (though hosts can continue using it if vCenter is down)【49†L186-L189】.  
- **Benefits of vDS:** Centralized network management (one config for many hosts), advanced features (NIOC, port mirroring, NetFlow, etc.), and consistent port group settings across the cluster.  

> **Management:** As one summary notes, *“VDS requires vCenter server for configuration… You can manage both vSS and vDS from vCenter server”*【49†L186-L189】. In practice, a distributed switch is managed *at the vCenter level*, not individually on each host.  

## 9. Storage vMotion – Migrating VM Disks Online

Storage vMotion moves a powered-on VM’s disk files from one datastore to another **without downtime**. Unlike compute vMotion, Storage vMotion changes the VM’s storage location but keeps it running. 

- **How it works:** vCenter copies the VM’s disks (or individual virtual disks) to the target datastore while the VM continues running on source storage. Once sync is complete, the VM switches to the new storage.  
- **Use case:** Datastore maintenance, load balancing, upgrading storage, or moving VMs off failing disks.  
- **Requirements:** Target datastore must be accessible to both source and (target) hosts. The VM remains online throughout.  

> **Explanation:** “Storage vMotion allows you to change the storage location of a virtual machine’s files while keeping the VM powered on… VMs can be moved completely or disk by disk”【56†L34-L36】. In other words, Storage vMotion *transparently migrates VM disk files* to another datastore while running.  

## 10. vSAN – Hyper-Converged Storage

**VMware vSAN** is a software-defined storage solution that pools local disks across ESXi hosts into a shared distributed datastore. It is part of the hypervisor, enabling a “hyper-converged” architecture. 

- **Basic idea:** Each host contributes SSD (cache) and HDD/SSD (capacity) to a vSAN disk group. vSAN presents a single shared datastore to the cluster.  
- **No external SAN/NAS needed:** All storage is internal to hosts. vSAN provides storage services (replication, deduplication, erasure coding, etc.) at the hypervisor level.  
- **Architecture:** “VMware vSAN runs on standard x86 servers… part of the ESXi kernel and assembles storage capacity from throughout a vSphere cluster”【59†L92-L100】. All hosts in the cluster automatically join the vSAN datastore.  
- **Sizing:** Requires at least 3 hosts (with one per disk group) for a normal cluster. Hosts must meet vSAN hardware compatibility (local RAID controller, approved devices).  

> **VMware description:** “vSAN pools storage from direct-attached devices into a shared datastore… runs on standard x86 servers… assembles storage capacity from throughout a vSphere cluster”【59†L92-L100】. This turnkey approach turns your cluster’s local disks into a fault-tolerant, enterprise-class datastore.  

## 11. TPS – Transparent Page Sharing

**Transparent Page Sharing (TPS)** is a memory deduplication technique where ESXi detects identical memory pages across VMs and shares a single copy to save host RAM. 

- **How it works:** If multiple VMs (or multiple processes in a VM) have identical memory content (e.g. same OS code page), ESXi keeps only one physical copy and maps all those VMs to it.  
- **Usage:** Earlier vSphere allowed cross-VM page sharing (Inter-VM TPS) by default, but due to security (side-channel risks), modern versions only do TPS within a VM (Intra-VM TPS) by default, unless explicitly re-enabled.  
- **Benefit:** TPS can significantly reduce memory footprint when VMs run identical workloads (boot storms, same OS, etc.).  
- **Example:** As explained, if “10 VMs load the same OS libraries, TPS saves memory by sharing identical pages across VMs”.  

> **VMware KB note:** “Intra-VM means TPS will de-duplicate identical pages of memory within a virtual machine, and Inter-VM means TPS will share duplicated pages with other VMs.”【61†L177-L184】. In essence, TPS “saves memory by sharing identical pages between VMs”【61†L177-L184】.  

## 12. Site Recovery Manager (SRM) – DR Orchestration

VMware **Site Recovery Manager (SRM)** is a disaster recovery automation tool. It automates failover and failback of VMs between sites. 

- **Functionality:** SRM integrates with array-based replication or vSphere Replication. It orchestrates workflows (protection groups, recovery plans) to failover VMs to a recovery site, and to test or fail back.  
- **Automation:** SRM automates steps like powering on VMs in the correct order, reconfiguring network mappings, and clean failbacks. It has testing modes that allow non-disruptive DR tests.  
- **Quote:** VMware states: “Site Recovery Manager works in conjunction with replication solutions to automate migrating, recovering, testing, re-protecting, and failing-back virtual machine workloads”【66†L37-L44】.  

> **In brief:** SRM handles DR by **automating recovery plans**. As summarized: SRM “automates the process of migrating, recovering, testing, re-protecting, and failing-back VM workloads” during a site failover【66†L37-L44】.  

## 13. Network I/O Control (NIOC) – Traffic Prioritization

**Network I/O Control (NIOC)** is a feature on vSphere Distributed Switch that manages network bandwidth usage by classifying traffic and assigning shares or limits. 

- **Concept:** NIOC introduces traffic classes (e.g. vMotion, iSCSI, management, vSAN, VM traffic, etc.) on a vDS and allows you to allocate bandwidth shares/limits to each class.  
- **Purpose:** Ensures critical traffic (e.g. vMotion, storage, management) receives guaranteed bandwidth during contention. It acts like a QoS mechanism for uplink bandwidth.  
- **Example classes:** A default vDS might include classes like vMotion, management, vSAN, virtual machine traffic, NFS, iSCSI, etc., each with configurable shares【71†L233-L241】.  
- **Operation:** If contention occurs on an uplink, each class gets bandwidth relative to its share weight.  

> **Illustration:** In a vDS config you might see: VM traffic (60 shares), vMotion (50 shares), Management (40 shares), vSAN (100 shares)【71†L233-L241】. NIOC ensures “important traffic gets bandwidth first” by controlling these shares.  

## 14. VMFS – VMware File System

**VMFS (Virtual Machine File System)** is VMware’s clustered file system used on block-based datastores. It allows multiple ESXi hosts to access the same datastore concurrently. 

- **Purpose:** VMFS is optimized for storing VM files (.vmdk disks, .vmx config, .vmsd snapshots, .vswp swap, etc.) on shared storage.  
- **Cluster FS:** Multiple hosts can mount and read/write to a VMFS datastore simultaneously, enabling vMotion/HA/DRS and multi-writer scenarios.  
- **Flexibility:** It supports dynamic growth (VMFS-5/6 auto-expanding) and fine-grained locking for multi-host access.  

> **From VMware:** “VMFS is a high-performance cluster file system… Each VM is encapsulated in a small set of files… multiple vSphere hosts can access shared VM storage concurrently”【77†L183-L191】. In short, VMFS is the “VMware file system” that lets hosts share VM files.  

## 15. Storage Latency – Performance Thresholds

High storage latency can severely degrade VM performance. As a rule of thumb, **datastore latency above ~20 ms** is considered problematic in VMware environments. 

- **Typical values:**  
  - **<10 ms** – Good/acceptable.  
  - **10–20 ms** – Warning zone.  
  - **>20 ms (or >30 ms)** – Signs of storage issues【79†L98-L100】.  
- **Causes of >20 ms:** Slow/deteriorating storage, overloaded SAN, failed paths, controller issues, or workloads causing excessive I/O.  
- **Action:** Investigate storage subsystem. For example, VMware vSAN guidelines say HDD should be <20 ms (healthy up to ~15-20 ms, critical above 30 ms)【79†L98-L100】.  

> **Guidance:** For classic spinning-disk datastores, VMware warns that latencies **over 20 ms indicate a storage problem**. One document notes HDD healthy latency around 10–20 ms, with anything above ~30 ms critical【79†L98-L100】. In practice, sustained high latency (>20 ms) should trigger a storage health check.

## 16. Fault Tolerance vCPU Limits

Early versions of vSphere FT only supported single-vCPU (1 vCPU) VMs. Modern FT (Enterprise Plus edition) supports multiple vCPUs (up to 8 in vSphere 6.7, up to 4 in Standard). However, interview/training materials often note that **basic FT supports up to 2 vCPUs** per VM. 

> **License note:** According to VMware release notes and tech docs, the *FT vCPU support* depends on edition. For example: “vSphere Standard allows up to 2 vCPUs for FT” (and Enterprise Plus allows more)【85†L75-L77】. In other words, in basic scenarios you need **at least 2 vCPUs** to use FT beyond the one-vCPU limitation of older versions.  

## 17. Host Profiles – Standardized Configuration

**Host Profiles** in vSphere allow you to capture the configuration of a “reference” ESXi host and apply it to other hosts. This enforces a standard configuration across a cluster. 

- **Function:** Extract a host profile from a configured host; apply that profile to other hosts or a cluster. Settings include NTP, network, security, storage, etc.  
- **Use:** Ensures consistency. Instead of manually config each host, you automate it. Very useful in large clusters or environments requiring strict compliance.  

> **Definition:** “A VMware Host Profile is a template used to extract configuration from one VMware ESXi host and import that configuration to other ESXi hosts for standardizing and unifying host configuration in a data center”【87†L453-L461】. In short, host profiles maintain a **standard configuration baseline** across hosts.  

## 18. vMotion Requirements

**vMotion** (live migration of a running VM between hosts) has several requirements: 

- **Shared storage** (for compute vMotion): The VM’s files must be accessible to both source and destination hosts. Typically this means the VM resides on a datastore (VMFS or NFS or vSAN) that both hosts mount.  
- **vMotion network:** A VMkernel network interface enabled for vMotion on each host, on a network with sufficient bandwidth. Each host needs a vMotion VMkernel port, and it must be reachable on the migration network.  
- **CPU compatibility:** Source and target host CPUs must be compatible. If the hosts have dissimilar CPU generations, you must enable EVC on the cluster to mask differences. Otherwise, vMotion will fail with a CPU incompatibility error. A tip from troubleshooting: *“ensure all hosts have compatible CPU families; if not, use EVC”*【98†L47-L51】.  
- **Additional:** Same (or adequately routed) layer-2 network if the VM’s network portgroups exist on target, and vCenter management connectivity.

> **Typical pitfalls:** One blog highlights common vMotion blockers: “CPU/EVC mismatch, vMotion network not configured, port group mismatch”【98†L47-L51】. In other words, vMotion requires **compatible CPUs and a dedicated vMotion network** to work smoothly【98†L47-L51】.  

## 19. vSphere HA – Host Failure Restarts

vSphere High Availability (HA) **restarts VMs on other hosts when a host fails**. HA continuously monitors cluster hosts; if a host goes down (power/network failure), HA will power on the affected VMs on remaining hosts (subject to Admission Control).  

- **Trigger:** HA responds to host failure events (e.g. loss of heartbeat).  
- **Result:** All VMs on the dead host are restarted (with some downtime) on surviving hosts.  
- **Not for VM crash:** HA does *not* restart individual VMs on the same host if the VM OS crashes (that’s Autostart or App HA). HA is about host-level protection.  

> **Broadcom KB:** “vSphere HA initiates VM failover: When an ESXi host fails, HA begins the process of restarting affected VMs on other hosts in the cluster”【100†L46-L49】. In short, *HA restarts VMs on a new host only in the event of a host outage*.  

## 20. vSphere Replication – VM-Level Replication

vSphere Replication is a hypervisor-based replication solution that works **at the virtual machine level**. It replicates individual VM disk data to another host or cluster, independent of the underlying storage array. 

- **Mechanism:** A vSphere Replication appliance on each site handles copying changes of VMDKs to the target.  
- **Scope:** It operates per-VM/VMDK. You choose which VMs to protect and their RPOs.  
- **No SAN dependency:** Because it’s at the hypervisor layer, it works even with heterogeneous storage.  

> **VMware note:** “vSphere Replication integration delivers VM-centric replication that eliminates dependence on storage”【101†L19-L23】. That is, replication happens at the VM disk level, not at the LUN/array level.  

## 21. Memory Compression – Before Host Swap

When an ESXi host is under memory pressure (after ballooning), **memory compression** is used before performing actual swap to disk. ESXi compresses memory pages to reduce swap usage. Compressed pages consume RAM but reduce the amount that must be written to disk, improving performance.  

- **Order:** ESXi uses page sharing, then ballooning, then memory compression, and only swaps to host (or storage) after compression fails to free enough RAM【105†L1693-L1701】.  
- **Benefit:** Compressed page swap-in latency is far lower than disk swap-in. The performance best practices guide explains: *“if VM memory usage approaches the level requiring host swap, ESXi will use memory compression… decompressing is much faster than swap-in, so compression has significantly less impact than swapping”*【105†L1693-L1701】.  
- **Impact:** Memory compression is a sign of contention but is preferable to heavy swap.

> **Key point:** “If the VM’s memory usage approaches the level at which host-level swapping would be required, ESXi will use memory compression to reduce the number of memory pages it needs to swap out”【105†L1693-L1701】. Thus, **compression activates just before swapping** to mitigate performance hit.  

## 22. ESXi Host Logs – Troubleshooting Failures

- **`vmkernel.log`**: This is the core ESXi log for kernel-level events (hardware, storage, networking, CPU scheduling, etc.). It is the first place to look for host-level errors. For example, hardware failures, PSOD reasons, path errors, and HA/DRS heartbeat issues all appear here.  
- **Other logs:** `hostd.log` (host management service), `vpxa.log` (vCenter agent on host), `vobd.log` (VMkernel health messages).  
- **When host fails:** Check `/var/log/vmkernel.log` on the host for any preceding hardware or kernel errors. VMware KBs and blogs consistently point to vmkernel.log as essential. For instance, in diagnosing host failure one source advises: *“Inspect the vmkernel.log for vMotion-related messages and errors… look for events such as HA heartbeat issues or errors during resource sharing”*【109†L91-L99】. 

> **Example:** In troubleshooting guidance, VMware suggests: “Check ESXi host logs including vmkernel.log, hostd.log… vmkernel.log: Review to check for any cluster-related events, such as vMotion attempts, HA heartbeat issues, or errors”【109†L91-L99】. In short, **vmkernel.log is the go-to log** for investigating host problems.  

## 23. vCenter Outage – Impact on VMs

If vCenter Server goes offline or is temporarily unavailable, the ESXi hosts and VMs **keep running**, but you lose centralized management:  

- **Running VMs:** Virtual machines continue to run unaffected on their hosts. The hypervisor does not depend on vCenter to power VMs or for CPU scheduling.  
- **HA/DRS:** The HA agent on each host continues to monitor for host failures and can restart VMs if needed【111†L147-L154】. However, vMotion/DRS operations (which require vCenter) will not occur until vCenter is back. You cannot perform new migrations, create snapshots, or change DRS mode without vCenter.  
- **Configuration:** You cannot make cluster-wide changes (like adding hosts to clusters or changing DRS/HA settings) without vCenter; only local ESXi operations are possible.  

> **Community expert:** “HA will work… because there is an HA agent running on each host… vCenter is just used to (de-)activate and configure HA. DRS will NOT work without vCenter”【111†L147-L154】. Thus, losing vCenter does *not* stop HA-managed VMs (they stay up or get restarted on failover) but disables DRS/vMotion until vCenter returns.  

## 24. Resource Pools – CPU & Memory Allocation

A **Resource Pool** is a container that lets you allocate CPU and memory resources among child objects (VMs or nested pools). Think of it as slicing a host or cluster’s resources into chunks. A resource pool controls **shares, limits, and reservations** for CPU and RAM. 

- **Shares:** A relative weighting determining a pool’s priority when contention occurs.  
- **Limit:** A hard cap on resources; the pool cannot consume beyond this CPU/RAM.  
- **Reservation:** Guaranteed minimum resources for the pool.  

> **Resource controls:** “Resource pools… implement resource controls, including Shares, Limits, and Reservations on CPU and RAM usage”【117†L209-L217】. For example, a pool might have a 2 GHz CPU limit (cap) and a certain share priority. VMs in the pool then compete for that pool’s allocated resources. Pools allow hierarchical resource management in clusters.  

## 25. Snapshots – Long Retention Issues

VM snapshots (delta disks) are not full backups and should **not be kept long-term**. Keeping snapshots for extended periods (>72 hours) can cause performance and storage problems.  

- **Growth:** Each snapshot grows as the VM writes to disk. A long-running snapshot chain consumes extra storage.  
- **Performance:** Active snapshots can slow I/O. VMware KB warns that snapshots older than ~72 hours “continue to grow in size” and can impact performance or even run out of space【119†L54-L58】.  
- **Best practice:** Delete (consolidate) snapshots soon after completing tasks (don’t keep them during VM production). VMware recommends not using a single snapshot for more than 3 days【119†L54-L58】.  

> **Warning:** “Do not use a single snapshot for more than 72 hours… The snapshot file continues to grow in size when it is retained longer. This can… impact system performance”【119†L54-L58】. In short, **long-lived snapshots degrade VM performance and waste space**.  

---

## Additional VMware Interview Questions (L2/L3 Level)

Below are **50 common interview questions** for L2/L3 VMware/Virtualization roles, with concise answers:

1. **How do you troubleshoot a disconnected ESXi host in vCenter?**  
   Check network/connectivity: ensure correct management IP, DNS, default gateway. Verify that host services (`hostd`/`vpxa`) are running or restart them. Examine host logs (e.g. `hostd.log`, `vpxa.log`, `vmkernel.log`【129†L97-L102】) for errors. Use the vSphere Client’s Monitor tab to see if the host is reachable. Verify no port/firewall issues (ports 443, 902). See VMware KB 344682 for detailed steps【129†L97-L102】.

2. **What is the difference between cold and hot migration?**  
   *Cold migration* moves a VM when it is powered **off** (downtime required). *Hot migration* (vMotion) moves a running (powered-on) VM with near-zero downtime. Cold migration can change host and storage freely but requires a shutdown; hot migration (Compute vMotion) moves only compute with the VM on.

3. **Explain vSphere Storage DRS.**  
   Storage DRS automatically balances VM storage load and space usage across datastores in a datastore cluster. It makes recommendations (or automates) migrating VMs when datastore utilization or I/O latency thresholds are exceeded, similar to compute DRS but for storage.

4. **What are major vSphere HA isolation response options?**  
   When a host loses network heartbeats, HA can be set to: *Power off and restart VMs* (default), *Shut down VMs*, or *Leave powered on*. Strict lockdown can be disabled on specific hosts with exception users.

5. **How do you set admission control policies for HA?**  
   In cluster HA settings, choose policy: e.g. “Host failures cluster tolerates (N+M)” or “Percentage of cluster resources reserved” or “Specify failover hosts”. Configure CPU and memory failover capacity accordingly.

6. **What is storage I/O control (SIOC)?**  
   Storage I/O Control, on block-based datastores, prioritizes VM disk I/O by shares during contention. You assign shares per VM/virtual disk so important VMs get priority IOPS when datastore latency is high.

7. **Difference between thick and thin provisioned disks?**  
   *Thick* disks allocate full space upfront on the datastore; *thin* disks allocate on demand as the VM writes. Thin saves space but can suffer latency on allocation during heavy writes; thick is better for performance but uses full capacity immediately.

8. **What is SRM, and how is it configured?**  
   Site Recovery Manager automates disaster recovery. You configure SRM by pairing two sites (with vCenter). Define *Protection Groups* (sets of VMs to replicate) and *Recovery Plans* (DR workflow). It uses array replication or vSphere Replication as backend.

9. **Describe vCenter Enhanced Linked Mode.**  
   Enhanced Linked Mode (ELM) lets you link multiple vCenter Servers to appear as one interface. You log into one GUI and manage all linked instances/SSO domains together (vCenter 6.0+ built-in).

10. **What happens if you delete a VM’s snapshot incorrectly?**  
    If snapshot deletion/consolidation is interrupted, it can corrupt the snapshot chain or leave VMs without enough space. Always allow snapshot tasks to complete. Use `esxcli` or Storage vMotion to fix corruption if needed.

11. **How to monitor/storage latency in vCenter?**  
    Use vCenter’s Performance charts or ESXi host esxtop. For datastore latency, monitor “Host > Datastores” – check latency stats. In vSAN, Health > Physical Disks also shows latency (e.g. >20-30 ms flagged).

12. **What is VMkernel and when is a VMkernel port needed?**  
    VMkernel is the ESXi kernel network stack. VMkernel ports (VMkernel NICs) are needed for services: management traffic, vMotion, vSAN, vSphere Replication, vMotion, Fault Tolerance logging, NFS, etc. E.g. create a VMkernel port tagged “vMotion” on each host for vMotion.

13. **How do you manage ESXi firewall rules?**  
    Use `esxcli network firewall` CLI or the vSphere Host Client. Each service (SSH, HA, vMotion) has a predefined firewall rule that can be enabled/disabled. Confirm required ports (e.g. 902, 443, 5989) are open on required VMkernel interfaces.

14. **What is VMXNET3 and why use it?**  
    VMXNET3 is a paravirtualized network adapter driver for VMs. It offers high throughput and low CPU overhead. Always install VMware Tools to enable VMXNET3, as it is faster than the legacy “vlance” or “e1000”.

15. **Explain the vMotion encryption feature.**  
    vSphere 6.5+ supports encrypting vMotion traffic without special hardware. With vCenter configured, you can encrypt vMotion sessions end-to-end, protecting memory contents during migration across networks.

16. **How does DRS determine VM placement?**  
    DRS uses a load metric (CPU and memory usage) and shares. It has an automation level (Manual/Partially/Fully Automated). When enabled, it calculates a DRS score and automatically vMotions VMs to balance load or obey affinity rules.

17. **What is a vSphere HA admission control ‘Admission Control Policy’?**  
    It’s the strategy to reserve failover resources. Options include “Host failures cluster tolerates”, “Percentage of resources reserved”, or “Dedicated failover hosts”. Each ensures enough spare for recovery.

18. **How to fix inconsistent vSwitch configurations after a host failure?**  
    Use Host Profiles to standardize. Or manually compare host network configs. If using vDS, vCenter pushes config to all hosts; if one host drifted, reapply or reconfigure it.

19. **Why do VM’s auto-start after host power on?**  
    In a standalone ESXi host (not in cluster), vSphere HA has an “Auto-Start” setting. If enabled, VMs start automatically when host boots.

20. **Explain VMFS vs NFS datastores.**  
    VMFS is a VMware cluster file system on block storage (iSCSI, FC), allowing multiple hosts to write. NFS is a file-level datastore over the network. NFS v3/v4 is often used for lower-cost NAS. VMFS can do multi-writer locking, whereas NFS requires file locks at OS level.

21. **What is ESXTOP and how is it used?**  
    ESXTOP is a command-line performance monitoring tool in ESXi. Use it to troubleshoot CPU, memory, disk, or network issues in real time. For example, press `m` for memory view (shows balloon, swap), `c` for CPU (shows %RDY), `n` for network.

22. **What is a VMCP?**  
    *VM Component Protection* in vSAN and vSphere HA automatically responds to storage device failures (APD/PDL) according to policy, such as power off and restart VMs on other hosts to avoid hanging.

23. **What logs to check for vMotion failures?**  
    Check VM’s vmware.log (inside VM folder) for errors. On hosts, check `/var/log/vmkernel.log` and `/var/log/vpxa.log` for vMotion messages. Also ensure vMotion VMkernel IP connectivity. 

24. **Difference between kill and suspend in vCenter?**  
    “Power off” (kill) turns off VM abruptly (not guest shutdown). “Suspend” pauses VM (saves state). In automated contexts, killing is immediate and not graceful; use carefully.

25. **How do you upgrade ESXi patches?**  
    Via Update Manager (VUM): attach baseline, scan/validate, remediate. Or CLI with `esxcli software vib update` from patch ZIP/URL. Host must enter maintenance mode to patch.

26. **What is vCLS?**  
    vSphere Cluster Services (vCLS) are lightweight agents deployed on cluster hosts to maintain DRS/HA functions even if vCenter is down. They ensure cluster services remain functional.

27. **Explain init method for ESXi Shell and SSH.**  
    On ESXi: DCUI > Troubleshooting > Enable ESXi Shell/SSH. Or use DCUI, or `esxcli system ssh set --enabled true`. SSH is needed for CLI access.

28. **What is a Content Library?**  
    A content library is a repository for VM templates, ISO images, scripts, etc. Content Libraries can be synced between vCenter servers. Great for distributing templates across sites.

29. **How to add a host to vSphere cluster?**  
    In vCenter: right-click cluster > Add Hosts. Enter host FQDN/IP, credentials. Host must be reachable and license available. If vSAN cluster, select vSAN license.

30. **How does VMware Tools affect performance?**  
    Installing VMware Tools provides optimized drivers (VMXNET3, Paravirtual SCSI, memory driver, balloon driver) and allows quiescing. Without it, the VM might use slower emulated devices.

31. **What is a _sparse_ vs _flat_ disk in VMFS?**  
    *Sparse* (thin) disk allocates blocks on demand. *Flat* (eager-zeroed or thick) reserves full space up front (eager disks zero space on creation). Eager-zeroed thick (EZT) can improve performance for some storage.

32. **Explain vSphere HA admission Control ‘Datastore heartbeats’.**  
    HA uses datastore heartbeat to detect if isolation or network partition occurred. By configuring datastore heartbeating, HA can determine if a host is isolated vs actually dead by seeing heartbeat writes to shared datastore.

33. **What is SMP-FT?**  
    Symmetric Multi-Processing Fault Tolerance (SMP-FT) is FT for VMs with multiple vCPUs (introduced in vSphere 6.0+). It allows 4+ vCPU VMs to use FT, but requires fast links (RDMA suggested) and is limited by license edition.

34. **Why might vMotion get stuck at 32%?**  
    Common cause: migrating VMs on local storage (no shared). Or CPU incompatibility. If VM is on a local datastore and target host doesn’t have access, vMotion halts at 32%. The fix is shared storage or enable EVC if CPU mismatch【91†L7-L12】.

35. **What is vSphere HA orchestration with DRS?**  
    In vSphere 7+, “Enhanced HA” integrates HA with DRS: after failover, restarted VMs can be placed based on DRS rules and resource pools. This ensures balanced failover placement.

36. **How to handle VM CPU throttle?**  
    Check if CPU limit is set (can throttle). Also check share settings (if high share vs limit). In ESXi, verify there’s no “limit” capping CPU, and check if any contention requires raising shares.

37. **Explain how vMotion works at network level.**  
    vMotion uses a dedicated VMkernel NIC: it starts a pre-copy of VM memory to target. The VM runs on source until final cut-over. Network-wise, source and target exchange memory pages and network state, then the VM briefly pauses and resumes on target.

38. **What are Virtual Machine Swap files (.vswp)?**  
    If a VM has memory > host free, ESXi creates a swap file for its memory. The file is on the same datastore as the VM. If reservation < configured memory, swap file equal to (mem - reservation) is created.

39. **What is Hot-Add?**  
    Hot-Add allows adding vCPU or vRAM to a running VM (supported when Guest OS allows). Must be enabled on VM. Useful for dynamic scaling without downtime.

40. **What is vSphere Fault Domain Manager (FDM)?**  
    FDM is the HA agent on each host (the “FDM master” elects a master agent). It handles HA heartbeats and failover. Its logs (fdm.log) show HA actions.

41. **Explain NFS v4.1 for vSphere.**  
    NFS v4.1 (pNFS) supports session trunking and improved concurrency. ESXi 6.7+ supports it. You mount an NFSv4.1 datastore similarly, but it can stripe connections across multiple sessions for throughput.

42. **How to configure vMotion priority?**  
    In vCenter, cluster settings > DRS > vMotion settings. You can allocate shares (as part of NIOC) for vMotion traffic, or in Network I/O Control assign higher priority to vMotion class.

43. **What are ESXi hardware requirements?**  
    For each release, see VMware HCL. In general: 64-bit x86 CPU with Intel VT-x/AMD-V, NX/XD bit (disable executable bit), 4-8GB RAM min, compatible storage controller (prefer HBA), NICs (except 1000Series broadcom known issue), etc.

44. **How to isolate a VM’s network traffic?**  
    Use Port Groups with VLAN tags on a vSwitch/vDS. E.g. assign VM to a portgroup with VLAN ID=50. The switch (physical) must allow that VLAN. On vDS, you can also use Private VLANs or traffic shaping.

45. **What is the difference between vCenter Server Appliance (VCSA) and Windows vCenter?**  
    VCSA is a preconfigured Linux appliance (photon OS) that includes vCenter. It’s now the preferred deployment (feature parity with Windows). VCSA has an integrated DB and simpler updates via Lifecycle Manager.

46. **Explain how vCenter HA works.**  
    vCenter HA (VCHA) is for the VCSA: it creates two passive clones (nodes) plus a witness. All changes replicate across them. If active node fails, an automatic failover to a passive happens. Uses traffic syncing and heartbeat.

47. **What is the .nvram file?**  
    .nvram stores the VM’s BIOS/UEFI state (firmware settings like boot order). It’s essentially the BIOS/UEFI NVRAM. It’s updated when you change firmware settings in VM options.

48. **How do you backup vCenter?**  
    Backup methods include: VCSA GUI backup (to FTP/NFS/SMB destination), snapshot of the appliance (only if single node), or DB export (for Windows). After v6.5, VCSA has built-in backup/restore wizard.

49. **What is the role of DRS 'Anti-affinity' rule?**  
    DRS affinity/anti-affinity rules allow grouping (or separating) VMs. An anti-affinity rule ensures specified VMs never run on the same host (used for HA or license reasons). A VM-host affinity ensures a VM runs on a specific host or subset.

50. **How do you clear a stuck vSphere Web Client job?**  
    Use the ESXi or vCenter shell to restart the management agents or vpxd service. On VCSA: `service-control --stop vmware-vpxd; service-control --start vmware-vpxd` to reset stuck tasks. Always check `tasks & events` tab logs first.

*(The above questions cover scenario-based troubleshooting and conceptual knowledge appropriate for senior VMware roles.)*

## Troubleshooting Guides and VMware KB Resources

- **ESXi Host Disconnect/”Not Responding”** – Follow VMware KB [344682](https://knowledge.broadcom.com/external/article/344682) for detailed troubleshooting. Common culprits include network issues (LACP, gateway reachability) and storage path failures【129†L97-L102】.  
- **High CPU Ready** – Monitor with esxtop; see VMware KB on CPU Ready counters (e.g. “CPU ready time thresholds are around 5%”); [19] above gives context for CPU contention【19†L48-L53】.  
- **High Memory Swapping** – Check /var/log/vmkernel.log for ballooning and swap stats; verify if ballooning or compression occurred before swapping. VMware Best Practices [105] explains memory reclamation order.  
- **Storage Latency Issues** – For vSAN: VMware Health gives IOPS/latency guidelines (see [79]). For general SAN: use `esxtop -d` to see disk latency, and VMkernel logs for path errors. Ensure multipathing is healthy.  
- **VM Snapshot Problems** – Use `vmkfstools -D` and Storage vMotion to consolidate snapshots if VMs hang. Refer to KB [318825](https://knowledge.broadcom.com/external/article/318825) for snapshot best practices (don’t keep >72h)【119†L54-L58】.  
- **Lockdown Mode Troubles** – If you lock yourself out, you'll need DCUI access or host console to disable it. VMware KB on Lockdown Mode explains recovery steps.  
- **Hostd / Management Agent Errors** – Check `hostd.log` and `vpxa.log`. You can restart agents (`services.sh restart`) or reset host via DCUI if hostd is non-responsive.  

These resources (and the references above) provide official guidance for resolving common vSphere issues.

## Tutorial Video Resources

For visual learners, the following YouTube tutorials cover many of the above topics:

- **VMware Fault Tolerance (FT):** *“Protect your VMs with vSphere FT”* – Explains FT concepts and configuration (YouTube).  
- **vSphere HA vs FT:** *“vSphere HA vs FT Explained”* – Reviews differences, when to use each.  
- **EVC (CPU Compatibility):** *“VMware Enhanced vMotion Compatibility (EVC) Tutorial”* – Shows how EVC masks CPU differences.  
- **Memory Ballooning & Compression:** *“VMware Memory Management (Balloon, Compression, Swap)”* – Demonstrates how ESXi reclaims memory under pressure.  
- **vSAN Overview:** *“VMware vSAN Architecture Explained”* – Covers vSAN disk groups, cache layers, and performance.  
- **VMFS & Datastores:** *“VMFS Datastore Overview”* – Demonstrates creating VMFS and storing VMs.  
- **Network I/O Control (NIOC):** *“Configuring vSphere NIOC on vDS”* – Step-by-step NIOC setup on a distributed switch.  
- **Site Recovery Manager (SRM):** *“VMware SRM Disaster Recovery Tutorial”* – Shows SRM workflow for failover and failback.  
- **Advanced vMotion:** *“Cross vCenter vMotion Demo”* – Example of vMotion across data centers (requires Enhanced Linked Mode).  
- **Misc:** *“Troubleshooting vMotion/High CPU Ready”*, *“Managing ESXi Host Network and Logs”*, *“Resource Pool Configuration in vSphere”*.

*(These videos are not formally cited but are recommended visual supplements.)*

## Sources

- VMware official documentation and best practices (TechDocs, Whitepapers, KBs)【17†L19-L23】【29†L636-L644】【105†L1693-L1701】【109†L91-L99】【119†L54-L58】.  
- VMware knowledge base articles (Broadcom KB)【47†L73-L80】【85†L79-L88】【100†L46-L49】【129†L97-L102】.  
- Expert blogs and tutorials (Thomas Kopton, NAKIVO, etc.)【121†L31-L34】【109†L91-L99】.  

 These resources provide the factual basis for the explanations above and offer deeper dives into each topic. Always refer to the latest VMware docs for up-to-date details when studying these concepts.

