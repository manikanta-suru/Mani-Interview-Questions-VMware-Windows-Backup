# 🔒 Understanding VMware Port Numbers 🔒

A concise guide to essential ports used in VMware environments like vSphere, vCenter, ESXi, vMotion, and vSAN.

---

## 🖥️ vSphere and vCenter Ports

| Port | Description |
|------|-------------|
| **443**  | Enables HTTPS communication for vCenter, ESXi Host Client, and Web Console. |
| **5480** | VMware Appliance Management Interface (VAMI). |
| **902**  | vSphere Client to ESXi hosts (VM console). |
| **903**  | VMware Remote Console (VMRC). |
| **9443** | vSphere Web Client for vCenter HTTPS access. |

---

## 🔧 ESXi Ports

| Port | Description |
|------|-------------|
| **22**   | SSH access for ESXi management. |
| **80**   | HTTP redirection to HTTPS (port 443). |
| **123**  | NTP (Network Time Protocol) for time sync. |
| **2049** | NFS traffic for datastores. |
| **5988/5989** | CIM monitoring for hardware health. |

---

## 🚀 VMotion Port

| Port | Description |
|------|-------------|
| **8000** | VMotion traffic between ESXi hosts. |

---

## 💾 vSAN Ports

| Port | Description |
|------|-------------|
| **2233 (TCP/UDP)** | vSAN cluster communication. |
| **2049** | vSAN NFS traffic. |

---

## 🔌 Additional Services

| Port | Description |
|------|-------------|
| **3260** | iSCSI storage communication. |
| **389**  | LDAP for directory services. |
| **636**  | Secure LDAP (LDAPS). |
| **53**   | DNS for name resolution. |

---

> ⚙️ *Keep this port guide handy when configuring firewall rules or troubleshooting VMware environments.*

---

📘 Inspired by real-world production configurations  
🧠 Maintained by [manikanta](https://github.com/manikanta-suru)
