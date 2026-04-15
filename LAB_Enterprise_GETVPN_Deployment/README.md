# Laboratorio PNETLab: Enterprise GETVPN Deployment

Questo laboratorio simula un'implementazione professionale di **Group Encrypted Transport VPN (GETVPN)** su un'infrastruttura di trasporto multi-hop (simulazione MPLS/WAN). L'obiettivo principale è dimostrare l'**Header Preservation**, permettendo la cifratura del traffico tra filiali senza l'uso di tunnel point-to-point (VTI/GRE), mantenendo l'instradamento basato sugli IP originali.

## 1. Descrizione del Laboratorio
Il cuore della soluzione è il protocollo **GDOI (Group Domain of Interpretation)**. Il Key Server (KS) agisce come autorità centrale per la distribuzione delle chiavi di sicurezza (TEK e KEK), mentre i router di filiale (Group Members) cifrano i dati mantenendo l'header IP esterno identico a quello interno. Questo permette al Core del Service Provider di instradare i pacchetti cifrati come se fossero normale traffico IP.

## 2. Componenti del Laboratorio
- **Nodi Totali:** 6 Router (vIOS) + Host di test.
- **Key Server (KS):** Gestisce le policy e le chiavi.
- **Core Network (C1, C2):** Fornisce la raggiungibilità IP tra tutti i nodi (simulazione Backbone).
- **Group Members (GM1, GM2, GM3):** Router di bordo delle filiali che eseguono la cifratura.

## 3. Tabella Cablaggio (Wiring)

| Nodo Sorgente | Interfaccia | Nodo Destinazione | Interfaccia | Network |
| :--- | :--- | :--- | :--- | :--- |
| **KS** | Gi0/0 | **Core-1** | Gi0/1 | 192.168.100.0/24 |
| **Core-1** | Gi0/0 | **Core-2** | Gi0/0 | 10.0.0.0/30 |
| **Core-1** | Gi0/2 | **GM-2** | Gi0/0 | 10.0.0.8/30 |
| **Core-2** | Gi0/1 | **GM-1** | Gi0/0 | 10.0.0.4/30 |
| **Core-2** | Gi0/2 | **GM-3** | Gi0/0 | 10.0.0.12/30 |

## 4. Tabella Indirizzamento IP (Loopback & LAN)

| Nodo | Loopback0 (Router ID) | LAN Interface | LAN Network |
| :--- | :--- | :--- | :--- |
| **KS** | 1.1.1.1/32 | - | - |
| **GM-1** | 101.101.101.101/32 | Gi0/1 | 172.16.1.1/24 |
| **GM-2** | 102.102.102.102/32 | Gi0/1 | 172.16.2.1/24 |
| **GM-3** | 103.103.103.103/32 | Gi0/1 | 172.16.3.1/24 |

## 5. Network Diagram (Mermaid)

```mermaid
graph TD
    subgraph "Management & Control"
        KS[Key Server - 1.1.1.1]
    end

    subgraph "Service Provider Core (OSPF)"
        C1((Core-1)) <--> C2((Core-2))
    end

    subgraph "Remote Branches (Group Members)"
        GM1[GM-1 / Site A]
        GM2[GM-2 / Site B]
        GM3[GM-3 / Site C]
    end

    KS --- C1
    C1 --- GM2
    C2 --- GM1
    C2 --- GM3

    style KS fill:#f96,stroke:#333,stroke-width:2px
    style C1 fill:#ddd,stroke:#333
    style C2 fill:#ddd,stroke:#333
    style GM1 fill:#bbf,stroke:#333
    style GM2 fill:#bbf,stroke:#333
    style GM3 fill:#bbf,stroke:#333