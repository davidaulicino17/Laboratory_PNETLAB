# Workbook: Advanced Routing - Manipulation & Redistribution

## 1. Scenario
In questo laboratorio simuleremo una rete Enterprise durante una fusione aziendale. La rete è composta da tre domini di routing principali:
* **Dominio OSPF (Area 0):** La rete "Core" esistente.
* **Dominio EIGRP (AS 100):** La rete della nuova filiale acquisita.
* **Dominio BGP (AS 65001):** Connessione verso un partner esterno/ISP.

**Punto Critico:** I router **R1** e **R2** fungono da punti di ridistribuzione tra OSPF ed EIGRP per garantire la ridondanza. Senza le corrette manipolazioni (Route-Maps, Tags, AD), si verificheranno routing sub-ottimale o loop.

---

## 2. Topologia e Connessioni
Le connessioni fisiche (IOL Interfaces) sono definite come segue:

| Device A | Int A | Device B | Int B | Subnet | Descrizione |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **R1** | e0/0 | **R3** | e0/0 | 10.1.13.0/24 | Segmento OSPF |
| **R2** | e0/0 | **R3** | e0/1 | 10.1.23.0/24 | Segmento OSPF |
| **R1** | e0/1 | **R4** | e0/0 | 10.1.14.0/24 | Segmento EIGRP |
| **R2** | e0/1 | **R4** | e0/1 | 10.1.24.0/24 | Segmento EIGRP |
| **R1** | e0/2 | **R5** | e0/0 | 203.0.113.0/30 | Link eBGP verso ISP |
| **R1** | e0/3 | **R2** | e0/3 | 10.1.12.0/24 | Link Inter-AS / Backdoor |

---

## 3. Tabella Indirizzamento IP
Tutti i dispositivi utilizzano una Loopback0 per l'ID del router.

| Device | Interface | IP Address | Subnet Mask | Protocollo |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | Lo0 | 1.1.1.1 | /32 | OSPF/EIGRP |
| | e0/0 | 10.1.13.1 | /24 | OSPF Area 0 |
| | e0/1 | 10.1.14.1 | /24 | EIGRP 100 |
| | e0/2 | 203.0.113.1 | /30 | BGP (AS 65000) |
| | e0/3 | 10.1.12.1 | /24 | Backdoor |
| **R2** | Lo0 | 2.2.2.2 | /32 | OSPF/EIGRP |
| | e0/0 | 10.1.23.2 | /24 | OSPF Area 0 |
| | e0/1 | 10.1.24.2 | /24 | EIGRP 100 |
| | e0/3 | 10.1.12.2 | /24 | Backdoor |
| **R3** | Lo0 | 3.3.3.3 | /32 | OSPF |
| | e0/0 | 10.1.13.3 | /24 | OSPF Area 0 |
| | e0/1 | 10.1.23.3 | /24 | OSPF Area 0 |
| **R4** | Lo0 | 4.4.4.4 | /32 | EIGRP |
| | e0/0 | 10.1.14.4 | /24 | EIGRP 100 |
| | e0/1 | 10.1.24.4 | /24 | EIGRP 100 |
| **R5** | Lo0 | 5.5.5.5 | /32 | BGP |
| | e0/0 | 203.0.113.2 | /30 | BGP (AS 65001) |

---

## 4. Lab Tasks (Obiettivi)

### Task 1: Configurazione Base e IGP
1.  Configurare IP e Loopback su tutti i router.
2.  Configurare **OSPF Area 0** su R1, R2, R3.
3.  Configurare **EIGRP AS 100** su R1, R2, R4.
4.  Verificare che R1 e R2 abbiano rotte da entrambi i domini.

### Task 2: Configurazione BGP
1.  Configurare eBGP tra R1 (AS 65000) e R5 (AS 65001).
2.  Su R5, creare e annunciare la Loopback 100 (172.16.0.1/24).
3.  Verificare la ricezione della rotta su R1.

### Task 3: Redistribuzione "Naive" (Creating the Loop)
1.  Eseguire Mutual Redistribution (OSPF <-> EIGRP) su R1 e R2.
2.  Ridistribuire BGP in OSPF su R1.
3.  Analizzare instabilità o percorsi sub-ottimali verso 172.16.0.0/24 da R4.

### Task 4: Fix con Route-Map e Tagging
1.  Prevenire il rientro delle rotte ridistribuite usando il **Route-Tagging**.
2.  Esempio: Taggare le rotte OSPF->EIGRP su R1 e negarle su R2 (EIGRP->OSPF).
3.  Verificare la stabilità con traceroute simmetrici.

### Task 5: Manipolazione del Percorso
1.  Forzare il traffico in uscita da R4 verso R5 a passare preferibilmente per R2 (simulando saturazione del link R1-R4).
2.  Utilizzare **Offset-list** o modificare il **Delay**.

---

## 5. Diagramma di Rete (Schema Mermaid)

```mermaid
graph TD
    %% Definizione Stili
    classDef bgp fill:#ffcccc,stroke:#cc0000,stroke-width:2px;
    classDef core fill:#ffffcc,stroke:#ffcc00,stroke-width:2px;
    classDef eigrp fill:#ccffcc,stroke:#009900,stroke-width:2px;
    classDef ospf fill:#ccccff,stroke:#0000cc,stroke-width:2px;

    %% Nodi Router
    R5("R5 - ISP<br/>(AS 65001)"):::bgp
    R1("R1 - Border/Core<br/>(Redistribution)"):::core
    R2("R2 - Core<br/>(Redistribution)"):::core
    R3("R3 - Internal OSPF<br/>(Area 0)"):::ospf
    R4("R4 - Internal EIGRP<br/>(AS 100)"):::eigrp

    %% Connessioni BGP
    R5 -- "e0/0 <br/> [203.0.113.0/30] <br/> e0/2" --> R1

    %% Connessioni Core Backdoor
    R1 -- "e0/3 <br/> [10.1.12.0/24] <br/> e0/3" --> R2

    %% Connessioni OSPF
    R1 -- "e0/0 <br/> [10.1.13.0/24] <br/> e0/0" --> R3
    R2 -- "e0/0 <br/> [10.1.23.0/24] <br/> e0/1" --> R3

    %% Connessioni EIGRP
    R1 -- "e0/1 <br/> [10.1.14.0/24] <br/> e0/0" --> R4
    R2 -- "e0/1 <br/> [10.1.24.0/24] <br/> e0/1" --> R4

    %% Subgraphs per raggruppamento logico
    subgraph "ISP Domain"
        R5
    end
    subgraph "Enterprise Core"
        R1
        R2
    end
    subgraph "OSPF Domain"
        R3
    end
    subgraph "EIGRP Domain"
        R4
    end