## 🧠 VMware vCenter Converter Standalone – Pre-requisites

---

### ✅ 1. Source Machine Requirements (Physical / VM)
- Machine should be powered ON and accessible  
- Supported OS (Windows / Linux)  
- Admin/root credentials required  

**Disable (temporarily):**
- Antivirus  
- Firewall (or allow required ports)  

---

### ✅ 2. Destination Requirements
- VMware ESXi host OR VMware vCenter Server  

**Ensure sufficient:**
- CPU  
- RAM  
- Storage (very important)  

---

### ✅ 3. Network Connectivity
- Source ↔ Converter ↔ Destination must be reachable  

**Required ports:**
- 443 (HTTPS)  
- 902 (ESXi communication)  

- Proper DNS / IP resolution  

---

### ✅ 4. Disk & Storage Check
- Clean up unnecessary data (reduce migration time)  

**Ensure:**
- No disk errors  
- Enough free space on destination  

---

### ✅ 5. Converter Installation
- Install Converter Standalone on:
  - Source machine OR  
  - Separate jump server  

---

### ✅ 6. Permissions
- Source: Local admin / root access  
- Destination: vCenter / ESXi admin access  

---

### ✅ 7. VM Planning (Very Important)

**Decide:**
- VM name  
- Datastore  
- Network (VLAN / Port Group)  

**Disk Type:**
- Thin Provisioning  
- Thick Provisioning  

---

### ⚠️ Best Practices (Real-time)

- Take backup before migration  
- Schedule during low usage time  
- Stop heavy services (DB / apps) if needed  
- Check time sync (NTP)  

---

### 🟢 One-line Summary
👉 Ensure connectivity, credentials, storage, and proper planning before starting VMware Converter migration.
