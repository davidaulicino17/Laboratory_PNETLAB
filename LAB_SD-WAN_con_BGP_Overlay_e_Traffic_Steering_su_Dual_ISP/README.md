# Lab: SD-WAN con BGP Overlay e Traffic Steering su Dual ISP

## Obiettivo del Laboratorio
Questo laboratorio è progettato per lo studio della certificazione Cisco CCDE. Simula un'architettura Enterprise Site-to-Site (Main e Branch) utilizzando due connessioni ISP (Active/Active) fornite da un router Cisco IOL centrale. L'obiettivo è configurare un overlay dinamico tramite tunnel IPsec e BGP, implementando politiche di SD-WAN per il traffic steering basato sulla qualità del link (SLA).

## Immagini da Utilizzare
* **FortiGate (FGT):** FortiOS 7.x (es. `fortinet-FGT-v7.2.x`)
* **Cisco IOL:** L3 Router (es. `i86bi-linux-l3-adventerprisek9`)
* **VPC / Linux:** Qualsiasi immagine PC leggera per i test LAN.

## Numero Nodi
* 2x FortiGate (FGT-Main, FGT-Branch)
* 1x Cisco IOL (IOL-Internet)
* 2x VPC (PC-Main, PC-Branch)

## Tabella Cablaggio

| Nodo Origine | Interfaccia PNET | Nome Logico FortiOS | Nodo Destinazione | Interfaccia Dest. | Scopo |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **FGT-Main** | mgmt | mgmt | Cloud/Switch | N/A | Management (DHCP) |
| **FGT-Main** | e1 | port1 | IOL-Internet | e0/0 | ISP 1 Underlay |
| **FGT-Main** | e2 | port2 | IOL-Internet | e0/1 | ISP 2 Underlay |
| **FGT-Main** | e3 | port3 | PC-Main | eth0 | LAN Main |
| **FGT-Branch** | mgmt | mgmt | Cloud/Switch | N/A | Management (DHCP) |
| **FGT-Branch** | e1 | port1 | IOL-Internet | e0/2 | ISP 1 Underlay |
| **FGT-Branch** | e2 | port2 | IOL-Internet | e0/3 | ISP 2 Underlay |
| **FGT-Branch** | e3 | port3 | PC-Branch | eth0 | LAN Branch |

## Tabella Indirizzamento IP

| Dispositivo | Interfaccia | Indirizzo IP / Subnet | Descrizione |
| :--- | :--- | :--- | :--- |
| **IOL-Internet** | e0/0 | 192.0.2.1/30 | Gateway ISP1 Main |
| **IOL-Internet** | e0/1 | 198.51.100.1/30 | Gateway ISP2 Main |
| **IOL-Internet** | e0/2 | 203.0.113.1/30 | Gateway ISP1 Branch |
| **IOL-Internet** | e0/3 | 100.64.0.1/30 | Gateway ISP2 Branch |
| **IOL-Internet** | Loopback0 | 8.8.8.8/32 | Test IP (Internet DNS) |
| **FGT-Main** | port1 | 192.0.2.2/30 | Public IP ISP1 |
| **FGT-Main** | port2 | 198.51.100.2/30 | Public IP ISP2 |
| **FGT-Main** | port3 | 10.10.10.254/24 | Gateway LAN Main |
| **FGT-Main** | tun-isp1 | 172.16.1.1/30 | Overlay IPsec via ISP1 |
| **FGT-Main** | tun-isp2 | 172.16.2.1/30 | Overlay IPsec via ISP2 |
| **FGT-Branch** | port1 | 203.0.113.2/30 | Public IP ISP1 |
| **FGT-Branch** | port2 | 100.64.0.2/30 | Public IP ISP2 |
| **FGT-Branch** | port3 | 10.20.20.254/24 | Gateway LAN Branch |
| **FGT-Branch** | tun-isp1 | 172.16.1.2/30 | Overlay IPsec via ISP1 |
| **FGT-Branch** | tun-isp2 | 172.16.2.2/30 | Overlay IPsec via ISP2 |

## Network Diagram

![Network Diagram](Diagram.png)