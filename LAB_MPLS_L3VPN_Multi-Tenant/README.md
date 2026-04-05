# Network Design: MPLS L3VPN Multi-Tenant (OSPF, LDP & MP-BGP)

> **Autore:** David Aulicino
> **Livello:** Intermedio / Avanzato
> **Simulatore:** EVE-NG / GNS3 con immagini Cisco IOL L3

---

## Descrizione del Laboratorio

Questo laboratorio costruisce un'infrastruttura **Service Provider Core** scalabile che fornisce servizi di connettività isolati tramite VRF (Virtual Routing and Forwarding). L'obiettivo didattico è comprendere, configurare e verificare una rete MPLS L3VPN completa, dalla distribuzione delle label nel core fino all'isolamento del traffico tra clienti distinti.

---

## Architettura a Tre Livelli

### 1. Underlay (Core Fisico - Livello IGP/LDP)
OSPF (Area 0) fornisce il routing dell'infrastruttura e garantisce la raggiungibilità tra le interfacce Loopback dei nodi Provider. LDP (Label Distribution Protocol) distribuisce le etichette di trasporto MPLS attraverso il core.

### 2. Overlay (Control Plane VPN - Livello MP-BGP)
MP-BGP con peering iBGP via Loopback utilizza l'address-family **VPNv4** per annunciare le rotte dei clienti complete di Route Distinguisher (RD) e Label VPN.

### 3. Accesso Cliente (PE-CE - Route Statiche)
I link PE-CE usano subnet /30 dedicate. Il PE ha una route statica verso la LAN del cliente (192.168.10.0/24) con next-hop l'IP del CE sul link /30. La route statica viene redistribuita in MP-BGP.

---

## Piano VRF (Tenant)

| Cliente | VRF | Route Distinguisher | Route Target Import | Route Target Export |
|---------|-----|--------------------|--------------------|---------------------|
| Cliente A | VRF_A | 100:1 | 100:1 | 100:1 |
| Cliente B | VRF_B | 200:1 | 200:1 | 200:1 |

---

## Piano di Indirizzamento

### Infrastruttura SP Core

| Dispositivo | Interfaccia | IP / Mask | Scopo |
|-------------|-------------|-----------|-------|
| P-1 | Loopback0 | 10.200.1.1/32 | Router-ID, OSPF, LDP |
| P-1 | Ethernet0/0 | 10.100.1.1/30 | Link P2P verso PE-1 |
| P-1 | Ethernet0/1 | 10.100.2.1/30 | Link P2P verso PE-2 |
| PE-1 | Loopback0 | 10.200.2.1/32 | Router-ID, iBGP Peering |
| PE-1 | Ethernet0/0 | 10.100.1.2/30 | Link Underlay verso P-1 |
| PE-2 | Loopback0 | 10.200.2.2/32 | Router-ID, iBGP Peering |
| PE-2 | Ethernet0/0 | 10.100.2.2/30 | Link Underlay verso P-1 |

### Link PE-CE (subnet /30 dedicate)

| Link | PE Interfaccia | IP PE | IP CE | VRF |
|------|---------------|-------|-------|-----|
| PE-1 ↔ CE-A1 | Ethernet0/1 | 10.1.1.1/30 | 10.1.1.2/30 | VRF_A |
| PE-1 ↔ CE-B1 | Ethernet0/2 | 10.1.2.1/30 | 10.1.2.2/30 | VRF_B |
| PE-2 ↔ CE-A2 | Ethernet0/1 | 10.1.3.1/30 | 10.1.3.2/30 | VRF_A |
| PE-2 ↔ CE-B2 | Ethernet0/2 | 10.1.4.1/30 | 10.1.4.2/30 | VRF_B |

### LAN Clienti (su Ethernet0/1 dei CE)

| CE | Interfaccia LAN | IP LAN | Sito |
|----|----------------|--------|------|
| CE-A1 | Ethernet0/1 | 192.168.10.1/24 | Cliente A, Roma |
| CE-A2 | Ethernet0/1 | 192.168.10.2/24 | Cliente A, Milano |
| CE-B1 | Ethernet0/1 | 192.168.10.1/24 | Cliente B, Napoli |
| CE-B2 | Ethernet0/1 | 192.168.10.2/24 | Cliente B, Torino |

> **Overlapping IP intenzionale:** CE-A1 e CE-B1 hanno entrambi 192.168.10.1/24 sulla LAN. Le VRF mantengono tabelle di routing completamente separate — l'overlap non crea conflitti.

---

## Cablaggio Fisico

| Da | Interfaccia | A | Interfaccia | Scopo |
|----|-------------|---|-------------|-------|
| PE-1 | eth0/0 | P-1 | eth0/0 | Link Underlay SP |
| PE-2 | eth0/0 | P-1 | eth0/1 | Link Underlay SP |
| CE-A1 | eth0/0 | PE-1 | eth0/1 | Accesso VRF_A Sito 1 |
| CE-B1 | eth0/0 | PE-1 | eth0/2 | Accesso VRF_B Sito 1 |
| CE-A2 | eth0/0 | PE-2 | eth0/1 | Accesso VRF_A Sito 2 |
| CE-B2 | eth0/0 | PE-2 | eth0/2 | Accesso VRF_B Sito 2 |

---

## Route Statiche PE (logica)

| PE | VRF | Destinazione | Next-Hop | Significato |
|----|-----|-------------|----------|-------------|
| PE-1 | VRF_A | 192.168.10.0/24 | 10.1.1.2 | LAN Roma via CE-A1 |
| PE-1 | VRF_B | 192.168.10.0/24 | 10.1.2.2 | LAN Napoli via CE-B1 |
| PE-2 | VRF_A | 192.168.10.0/24 | 10.1.3.2 | LAN Milano via CE-A2 |
| PE-2 | VRF_B | 192.168.10.0/24 | 10.1.4.2 | LAN Torino via CE-B2 |

---

## Bill of Materials (BoM)

| Qty | Tipo | Ruolo | Nome |
|-----|------|-------|------|
| 2 | Cisco IOL L3 | Provider Edge (PE) | PE-1, PE-2 |
| 1 | Cisco IOL L3 | Provider Core (P) | P-1 |
| 4 | Cisco IOL L3 | Customer Edge | CE-A1, CE-A2, CE-B1, CE-B2 |

---

## Obiettivi di Verifica

1. **Connettività Intranet:** `CE-A1 (192.168.10.1) → CE-A2 (192.168.10.2)` e stesso per Cliente B.
2. **Isolamento tra VRF:** Ping da CE-A1 verso la LAN del Cliente B deve fallire.
3. **Overlap IP senza conflitti:** CE-A1 e CE-B1 con stessa LAN coesistono su PE-1 senza problemi.
4. **Verifica Label MPLS:** Traceroute CE-A1 → CE-A2 mostra doppia label MPLS su P-1.

---

## Concetti Chiave

- **VRF:** Istanze di routing virtuali e isolate su un singolo router fisico.
- **Route Distinguisher (RD):** Rende univoco un prefisso IPv4 nel piano BGP VPNv4.
- **Route Target (RT):** Controlla import/export tra VRF — il meccanismo dell'isolamento.
- **LDP:** Distribuisce le Transport Label nel core, accoppiato con OSPF.
- **Doppia label MPLS:** Label esterna (Transport, LDP) + label interna (VPN, MP-BGP).
- **PHP:** P-1 rimuove la Transport Label prima di consegnare il pacchetto al PE egress.

---

### Diagramma di Rete
![Topology Diagram](Diagram.png)