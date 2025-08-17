# AWS FortiGate NGFW — Secure VPC Egress & Web Filtering

A secure AWS VPC was designed and deployed with a FortiGate-VM (NGFW) acting as the egress gateway for a private Windows VM. All outbound traffic from the private subnet is routed through FortiGate, where granular firewall rules and a Web Filter UTM profile enforce category-based access control (e.g., blocking “Shopping”).

<img width="800" height="267" alt="image" src="https://github.com/user-attachments/assets/80f83abc-568f-4770-aedc-d804ff72d9a7" />


---

## What This Proves
- **Segmentation & Egress Control**: Private workloads have no direct internet exposure; all traffic hairpins through FortiGate.
- **Inbound Access via NAT**: RDP to the Windows VM is enabled only through a FortiGate Virtual IP (VIP), not via a public IP on the VM.
- **UTM Enforcement**: A custom Web Filter profile blocks selected content categories while allowing business traffic.

---

## Cloud Architecture
- **VPC** with a **public subnet** (FortiGate WAN ENI) and a **private subnet** (FortiGate LAN ENI + Windows VM).
- **Internet Gateway** attached to the VPC.
- **Route Tables**
  - Public subnet: default route → **IGW**.
  - Private subnet: default route → **FortiGate LAN ENI** (instance-based gateway).
- **EC2 Instances**
  - **FortiGate-VM** with dual ENIs (WAN/LAN). **Source/Destination Check disabled** on both ENIs.
  - **Windows Server/Client VM** in the private subnet (no public IP).

---

## FortiGate Configuration (High Level)
1. **Interfaces**
   - `port1` (WAN) in public subnet; `port2` (LAN) in private subnet.
2. **Static Routes**
   - Default route via `port1` (internet).
3. **VIP for RDP**
   - VIP maps `PublicIP:3389` → `WindowsPrivateIP:3389`.
4. **Firewall Policies**
   - **WAN → LAN (RDP)**: `ACCEPT`, Service `RDP`, using VIP, logging enabled.
   - **LAN → WAN (HTTP/HTTPS)**: `ACCEPT`, NAT **enabled**, Security Profiles:
     - **Web Filter**: profile blocking **Shopping** category (example).
     - **SSL Inspection**: *certificate-inspection* (domain-category filtering without full TLS MITM).
   - (Optional) **Deny/No-UTM** for other protocols (e.g., ICMP) to demonstrate least privilege.
5. **Web Filter Profile**
   - Create profile (e.g., `TSTWebFilter`), set action **Block** for targeted categories; log violations.

---

## Validation & Testing
- **Outbound control**: Browsing to allowed sites succeeds; attempts to access blocked categories return a FortiGuard block page.
- **Inbound access**: RDP to the VM works only via the FortiGate VIP.
- **Protocol restriction**: Non-permitted protocols (e.g., ICMP) are dropped.
- **Logging**: Verify policy hits and web filter events in FortiGate logs.

---

## Troubleshooting Tips
- **No internet from private subnet**: Confirm the private subnet’s route table points 0.0.0.0/0 to the **FortiGate LAN ENI** and that FortiGate has a default route out its WAN interface.
- **RDP failing**: Check VIP mapping, security group(s), and the **WAN→LAN** policy order.
- **Web filter not triggering**: Ensure the **LAN→WAN** policy has the Web Filter profile attached and SSL inspection is set to **certificate-inspection**.

---

## Repository Structure
