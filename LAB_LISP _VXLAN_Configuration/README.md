# Laboratorio: Campus Fabric Multi-Vendor (LISP & VXLAN)

Questo repository contiene un laboratorio progettato per analizzare, testare e configurare un'architettura di rete overlay basata sulla separazione tra Control Plane e Data Plane, ispirata ai design SD-Access.

## 🎯 Obiettivo del Laboratorio

Comprendere e dimostrare, attraverso test pratici, l'interazione tra LISP e VXLAN in un ambiente multi-vendor:

* **Control Plane (LISP):** Utilizzo di LISP per la registrazione e la risoluzione dinamica degli Endpoint Identifier (EID) verso i Routing Locator (RLOC), eliminando la necessità di propagare le rotte host nell'underlay.
* **Data Plane (VXLAN):** Utilizzo di VXLAN per l'incapsulamento e il trasporto del traffico Layer 2 su un'infrastruttura Layer 3.
* **Interoperabilità:** Dimostrare la compatibilità tra Cisco (Map-Server/Map-Resolver) e Arista (VTEP/xTR).

## 🛠️ Prerequisiti

* **Piattaforma:** PNetLab / EVE-NG / GNS3
* **Immagini:** * Cisco CSR1000v (es. `csr1000v-universalk9.16.12.05.qcow2` - Min. 3GB RAM)
  * Arista vEOS (Min. 2GB RAM)
  * VPCS / Alpine Linux (Host leggeri)
  *Disabilitare da vEOS lo Zerotouch tramite "zerotouch cancel" ed attendere il reboot
* **Nodi Totali:** 5 (1 Spine, 2 Edge, 2 Host)

## 🔬 Struttura del Laboratorio

Il laboratorio è diviso in tre fasi sequenziali per costruire l'architettura dal basso verso l'alto.

### Fase 1: La Rete Underlay (Trasporto base)
Configurazione della connettività IP di base tra le interfacce fisiche e le Loopback (RLOC) degli apparati utilizzando OSPF in Area 0. L'obiettivo è garantire la raggiungibilità IP pura senza trasportare i dati degli host.

### Fase 2: Il Control Plane (LISP)
Configurazione del Cisco CSR1000v come Map-Server e Map-Resolver. Gli switch Arista Edge verranno configurati per interrogare lo Spine per scoprire la posizione (RLOC) degli host di destinazione.

### Fase 3: Il Data Plane (VXLAN)
Configurazione delle interfacce VXLAN (VTEP) sugli switch Arista per estendere il Layer 2 (VLAN 10) tra gli host PC-A e PC-B, sfruttando l'incapsulamento dinamico guidato dal database LISP.

---

## 🔌 Piano di Cablaggio

| Dispositivo A | Interfaccia A | Dispositivo B | Interfaccia B | Scopo |
| :--- | :--- | :--- | :--- | :--- |
| **SPINE-MSMR** | Gi1 | **EDGE-1** | Et1 | Link Underlay 1 |
| **SPINE-MSMR** | Gi2 | **EDGE-2** | Et1 | Link Underlay 2 |
| **EDGE-1** | Et2 | **PC-A** | eth0 | Link Accesso Host A |
| **EDGE-2** | Et2 | **PC-B** | eth0 | Link Accesso Host B |

---

## 🗺️ Tabella Indirizzamento IP

| Dispositivo | Interfaccia | Indirizzo IP | Subnet Mask | Ruolo |
| :--- | :--- | :--- | :--- | :--- |
| **SPINE-MSMR (Cisco)** | Loopback0 | `1.1.1.1` | `255.255.255.255` | LISP MS/MR |
| | Gi1 | `192.168.1.1` | `255.255.255.252` | Link Underlay |
| | Gi2 | `192.168.2.1` | `255.255.255.252` | Link Underlay |
| **EDGE-1 (Arista)** | Loopback0 | `2.2.2.2` | `255.255.255.255` | RLOC / VTEP |
| | Et1 | `192.168.1.2` | `255.255.255.252` | Link Underlay |
| **EDGE-2 (Arista)** | Loopback0 | `3.3.3.3` | `255.255.255.255` | RLOC / VTEP |
| | Et1 | `192.168.2.2` | `255.255.255.252` | Link Underlay |
| **PC-A** | eth0 | `10.1.1.10` | `255.255.255.0` | EID (Host) |
| **PC-B** | eth0 | `10.1.1.20` | `255.255.255.0` | EID (Host) |

*(Gateway Anycast per gli Host: 10.1.1.1)*

---

## 💡 Conclusioni di Design

* **Scalabilità:** Delegando la mappatura degli endpoint al database LISP centrale, gli switch di accesso (Edge) e la rete underlay non hanno bisogno di memorizzare l'intera tabella di routing degli host.
* **Mobilità:** Se un host si sposta da EDGE-1 a EDGE-2, l'IP dell'host (EID) rimane invariato; viene semplicemente aggiornata la mappatura LISP sul Map-Server associandola al nuovo RLOC.
* **Flessibilità:** L'uso di VXLAN permette di estendere domini di broadcast Layer 2 sopra una rete instradata Layer 3 ad alte prestazioni.

---

## Diagramma di Rete

! (Diagram.png)
