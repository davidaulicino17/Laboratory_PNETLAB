\# Workbook: Configurazione Private VLAN (PVLAN) su Cisco IOL



\## 1. Scenario

In questo laboratorio simuleremo una rete in cui è necessario applicare una rigida segregazione del traffico a Livello 2, mantenendo però tutti i dispositivi sulla stessa sottorete IP (Livello 3).

\* \*\*Switch Core (L3):\*\* Agisce da default gateway per la rete.

\* \*\*Switch Access (L2):\*\* Dispositivo su cui verranno configurate le policy PVLAN.

\* \*\*Host PC1/PC2:\*\* Inseriti in una VLAN Isolata (non comunicano tra loro).

\* \*\*Host PC3/PC4:\*\* Inseriti in una VLAN Community (comunicano tra loro ma non con altre VLAN).



\*\*Punto Critico:\*\* L'isolamento a livello L2 richiede l'impostazione dello switch in modalità VTP Transparent. È fondamentale configurare correttamente la porta \*\*Promiscuous\*\* per permettere a tutti gli host isolati e in community di raggiungere il gateway condiviso senza infrangere le regole di sicurezza L2.



\---



\## 2. Topologia e Connessioni

Le connessioni fisiche (IOL Interfaces) sono definite come segue:



| Device Locale | Porta Locale | Device Remoto | Porta Remota | Ruolo Porta | Descrizione |

| :--- | :--- | :--- | :--- | :--- | :--- |

| \*\*Access L2\*\* | e0/0 | \*\*Core L3\*\* | e0/0 | Promiscuous | Uplink verso il Gateway |

| \*\*Access L2\*\* | e0/1 | \*\*PC1\*\* | e0/0 | Host (Isolated) | Nodo isolato |

| \*\*Access L2\*\* | e0/2 | \*\*PC2\*\* | e0/0 | Host (Isolated) | Nodo isolato |

| \*\*Access L2\*\* | e0/3 | \*\*PC3\*\* | e0/0 | Host (Community)| Nodo in community |

| \*\*Access L2\*\* | e0/4 | \*\*PC4\*\* | e0/0 | Host (Community)| Nodo in community |



\---



\## 3. Tabella Indirizzamento IP

Tutti i dispositivi si trovano all'interno della stessa sottorete logica L3 (`10.0.0.0/24`).



| Device | Interface | IP Address | Subnet Mask | Default Gateway | VLAN Logica |

| :--- | :--- | :--- | :--- | :--- | :--- |

| \*\*Core L3\*\* | SVI 100 | 10.0.0.254 | /24 | N/A | 100 (Primary) |

| \*\*PC1\*\* | e0/0 | 10.0.0.1 | /24 | 10.0.0.254 | 101 (Isolated) |

| \*\*PC2\*\* | e0/0 | 10.0.0.2 | /24 | 10.0.0.254 | 101 (Isolated) |

| \*\*PC3\*\* | e0/0 | 10.0.0.3 | /24 | 10.0.0.254 | 102 (Community) |

| \*\*PC4\*\* | e0/0 | 10.0.0.4 | /24 | 10.0.0.254 | 102 (Community) |



\---



\## 4. Lab Tasks (Obiettivi)



\### Task 1: Preparazione e Creazione VLAN (Access L2)

1\. Impostare lo switch L2 in modalità \*\*VTP Transparent\*\*.

2\. Creare la VLAN 101 come `isolated`.

3\. Creare la VLAN 102 come `community`.

4\. Creare la VLAN 100 come `primary` e associarle le VLAN 101 e 102.



\### Task 2: Configurazione Porte di Accesso (Access L2)

1\. Configurare le porte `e0/1` ed `e0/2` (PC1 e PC2) come host PVLAN e mapparle alla coppia 100-101 (Primary-Isolated).

2\. Configurare le porte `e0/3` ed `e0/4` (PC3 e PC4) come host PVLAN e mapparle alla coppia 100-102 (Primary-Community).



\### Task 3: Configurazione Uplink e Gateway

1\. Configurare la porta `e0/0` sullo switch L2 come `promiscuous` e mappare la VLAN primaria 100 alle secondarie 101 e 102.

2\. Sul \*\*Core L3\*\*, configurare la SVI 100 (o l'interfaccia fisica routed) con l'IP `10.0.0.254/24` e abilitare la porta.



\### Task 4: Verifica dell'Isolamento

1\. Effettuare un ping da PC1 verso PC2: \*\*Deve fallire\*\* (Isolamento L2).

2\. Effettuare un ping da PC3 verso PC4: \*\*Deve avere successo\*\* (Stessa Community).

3\. Effettuare un ping da PC1 verso PC3: \*\*Deve fallire\*\* (Cross-VLAN).

4\. Effettuare un ping da qualsiasi PC verso 10.0.0.254: \*\*Deve avere successo\*\* (Raggiungibilità del Gateway).



\---



\## 5. Diagramma di Rete (Schema Mermaid)



```mermaid

graph TD

&#x20;   %% Definizione Stili

&#x20;   classDef core fill:#ffcccc,stroke:#cc0000,stroke-width:2px;

&#x20;   classDef access fill:#ffffcc,stroke:#ffcc00,stroke-width:2px;

&#x20;   classDef isolated fill:#ccffcc,stroke:#009900,stroke-width:2px;

&#x20;   classDef community fill:#ccccff,stroke:#0000cc,stroke-width:2px;



&#x20;   %% Nodi Device

&#x20;   Core("Core Switch L3<br/>(Gateway: 10.0.0.254)"):::core

&#x20;   Acc("Access Switch L2<br/>(PVLAN Domain)"):::access

&#x20;   PC1("PC1<br/>10.0.0.1"):::isolated

&#x20;   PC2("PC2<br/>10.0.0.2"):::isolated

&#x20;   PC3("PC3<br/>10.0.0.3"):::community

&#x20;   PC4("PC4<br/>10.0.0.4"):::community



&#x20;   %% Connessioni Promiscuous

&#x20;   Core -- "e0/0 <br/> \[Primary VLAN 100] <br/> e0/0 (Promiscuous)" --> Acc



&#x20;   %% Connessioni Host

&#x20;   Acc -- "e0/1 <br/> \[Isolated 101]" --> PC1

&#x20;   Acc -- "e0/2 <br/> \[Isolated 101]" --> PC2

&#x20;   Acc -- "e0/3 <br/> \[Community 102]" --> PC3

&#x20;   Acc -- "e0/4 <br/> \[Community 102]" --> PC4



&#x20;   %% Subgraphs per raggruppamento logico

&#x20;   subgraph "L3 Routing Domain"

&#x20;       Core

&#x20;   end

&#x20;   subgraph "L2 PVLAN Domain"

&#x20;       Acc

&#x20;       subgraph "VLAN 101 (Isolated)"

&#x20;           PC1

&#x20;           PC2

&#x20;       end

&#x20;       subgraph "VLAN 102 (Community)"

&#x20;           PC3

&#x20;           PC4

&#x20;       end

&#x20;   end

