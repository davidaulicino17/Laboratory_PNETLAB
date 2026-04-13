# Laboratorio: Campus Fabric Cisco-Only (LISP Overlay)

Questo repository contiene un laboratorio pratico progettato per esplorare e configurare un'architettura di rete overlay basata esclusivamente su dispositivi Cisco, utilizzando LISP (Locator/ID Separation Protocol).

## 🎯 Obiettivo del Laboratorio

L'obiettivo principale è dimostrare l'efficacia di LISP nel separare l'identità di un endpoint (EID) dalla sua posizione topologica (RLOC), permettendo la mobilità e semplificando l'underlay L3. 

In questo scenario "Cisco-only", osserveremo:
* **Control Plane (LISP MS/MR):** Gestione centralizzata del database di mappatura EID-to-RLOC.
* **Data Plane (LISP-Encapsulation):** Incapsulamento diretto del traffico host-to-host da parte dei nodi Ingress/Egress Tunnel Router (xTR).
* **Underlay OSPF:** Raggiungibilità di base limitata esclusivamente alle interfacce fisiche e di Loopback (RLOC).

## 🛠️ Prerequisiti

* **Piattaforma:** PNetLab / EVE-NG / GNS3
* **Immagini:**
    * Cisco vIOS-L3 (es. `vios-adventerprisek9-m.vmdk.SPA.156-2.T` - Ottimizzata per 512MB RAM)
    * VPCS (Virtual PC Simulator) per gli host di test.

## 🏗️ Topologia e Ruoli

L'infrastruttura è composta da 3 router Cisco vIOS-L3 e 2 host leggeri:

1.  **SPINE-MSMR (Cisco vIOS-L3):** Agisce come Map-Server (MS) e Map-Resolver (MR). È il "cervello" della rete che mantiene il registro delle posizioni degli host.
2.  **EDGE-1 (Cisco vIOS-L3):** Agisce come LISP xTR (ITR/ETR) per il Sito 1. Registra la subnet locale sullo SPINE e incapsula il traffico verso l'esterno.
3.  **EDGE-2 (Cisco vIOS-L3):** Agisce come LISP xTR (ITR/ETR) per il Sito 2.
4.  **PC-A & PC-B (VPCS):** Endpoint simulati per generare traffico e testare il tunnel overlay.

---

## 🗺️ Tabella Indirizzamento IP

### Underlay (RLOC) e Link Fisici

| Dispositivo | Interfaccia | Indirizzo IP | Subnet Mask | Ruolo / Note |
| :--- | :--- | :--- | :--- | :--- |
| **SPINE-MSMR** | Loopback0 | `1.1.1.1` | `255.255.255.255` | MS/MR IP |
| | Gi0/0 | `192.168.1.1` | `255.255.255.252` | Link verso EDGE-1 |
| | Gi0/1 | `192.168.2.1` | `255.255.255.252` | Link verso EDGE-2 |
| **EDGE-1** | Loopback0 | `2.2.2.2` | `255.255.255.255` | RLOC xTR 1 |
| | Gi0/0 | `192.168.1.2` | `255.255.255.252` | Link verso SPINE |
| **EDGE-2** | Loopback0 | `3.3.3.3` | `255.255.255.255` | RLOC xTR 2 |
| | Gi0/0 | `192.168.2.2` | `255.255.255.252` | Link verso SPINE |

### Overlay (EID - Endpoint Identifier)

| Host | Indirizzo IP | Gateway | Ruolo / Note |
| :--- | :--- | :--- | :--- |
| **PC-A** | `10.1.1.10` | `10.1.1.1` (su EDGE-1) | Host Sito 1 |
| **PC-B** | `10.1.1.20` | `10.1.1.1` (su EDGE-2) | Host Sito 2 |

---

## 🚀 Istruzioni Rapide per l'Avvio

1.  Caricare i nodi su PNetLab e collegarli secondo la tabella delle interfacce.
2.  Avviare tutti i nodi.
3.  Applicare la configurazione dell'underlay (OSPF) per garantire la raggiungibilità delle Loopback.
4.  Applicare la configurazione LISP sui 3 nodi.
5.  Effettuare un ping da PC-A a PC-B e verificare le associazioni tramite il comando `show ip lisp map-cache` sugli EDGE.