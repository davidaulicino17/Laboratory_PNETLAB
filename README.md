# Laboratori di Networking PNetLab

Benvenuti nel mio repository di laboratori di networking.

Questo spazio serve come raccolta personale e pubblica di topologie e configurazioni di rete che ho costruito e testato. L'obiettivo principale è esplorare, validare e documentare architetture di rete complesse, con un focus specifico sul **Network Design**, la **scalabilità** e l'**interoperabilità** dei protocolli.

Tutti i laboratori sono realizzati e testati utilizzando la piattaforma di emulazione **PNetLab**.

## 🔬 Laboratori Principali

Man mano che aggiungerò laboratori, questo elenco crescerà. I laboratori attualmente documentati includono:

* **Design OSPF Scalabile (Single-Area vs. Multi-Area):** Un'analisi comparativa sull'impatto della progettazione delle aree OSPF sulla stabilità e l'efficienza della LSDB.
* **Fabric L3 Spine-Leaf (BGP & OSPF):** Progettazione di una fabric L3 (architettura Clos) per Data Center, utilizzando OSPF come underlay e BGP per la distribuzione delle rotte.
* **Fabric Datacenter Greenfield con MP-BGP EVPN:** Implementazione di una moderna architettura Data Center da zero (Greenfield), utilizzando MP-BGP EVPN per la gestione della control plane e l'overlay di rete.
* **Integrazione Brownfield-Greenfield:** Studio di scenari di migrazione e interconnessione tra infrastrutture legacy esistenti (Brownfield) e nuove architetture di rete (Greenfield).
* **Nexus vPC Domain Setup con LACP:** Configurazione di un dominio vPC (Virtual Port Channel) su switch Cisco Nexus per garantire alta affidabilità (HA) e aggregazione di banda tramite LACP.
* **Estensione L2 con VXLAN Statico:** Implementazione di un'estensione L2 tra due siti (Data Center Interconnect) utilizzando tunnel VXLAN statici.

## ⚠️ Disclaimer

Questi laboratori sono creati a scopo didattico, di test e di validazione del design. Le configurazioni sono ottimizzate per un ambiente di emulazione e potrebbero non essere direttamente applicabili alla produzione senza un'adeguata revisione.
