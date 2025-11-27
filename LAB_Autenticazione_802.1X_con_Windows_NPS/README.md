# LAB: Autenticazione 802.1X con Windows NPS

Questo repository documenta un laboratorio di sicurezza di rete focalizzato sul controllo degli accessi alla LAN (NAC). Il progetto dimostra come implementare l'autenticazione **802.1X basata su porte** per proteggere l'accesso alla rete cablata, utilizzando un'infrastruttura Microsoft come server di autenticazione.

Il laboratorio è realizzato su piattaforma **PNetLab** integrando il mondo Windows Server con lo switching Cisco.

## 🎯 Obiettivi del Design

1.  **Sicurezza dell'Accesso:** Impedire l'accesso non autorizzato alla rete fisica configurando le porte dello switch per richiedere l'autenticazione prima di passare traffico.
2.  **Autenticazione Centralizzata:** Utilizzare il protocollo **RADIUS** per centralizzare le policy di accesso e le credenziali utenti su un server dedicato.
3.  **Integrazione Multi-Vendor:** Configurare l'interoperabilità tra uno switch Cisco (Network Access Server) e un server Microsoft NPS (RADIUS Server).
4.  **Validazione Client:** Testare il processo di autenticazione lato client (Supplicant) utilizzando una macchina Windows.

## 🏗️ Topologia e Tecnologie

L'infrastruttura è composta da tre ruoli logici fondamentali (architettura AAA):

### 1. Authentication Server (Il "Cervello")
* **Nodo:** Windows Server 2012 R2.
* **Ruolo:** Esegue il servizio **NPS (Network Policy Server)**.
* **Funzione:** Riceve le richieste RADIUS dallo switch, verifica le credenziali (User/Password o Certificati) nel database locale (o AD) e invia la risposta (Access-Accept/Reject).

### 2. Authenticator (Il "Poliziotto")
* **Nodo:** Switch Cisco IOL L2.
* **Ruolo:** Network Access Server (NAS).
* **Funzione:** Blocca fisicamente la porta del client. Inoltra le credenziali EAP del client al server RADIUS incapsulate in pacchetti RADIUS. Sblocca la porta solo dopo aver ricevuto l'OK dal server.

### 3. Supplicant (Chi chiede accesso)
* **Nodo:** Windows Client (es. Tiny10 o XP).
* **Ruolo:** Client di rete.
* **Funzione:** Invia le credenziali o il certificato quando rileva che la porta dello switch richiede autenticazione 802.1X.

## 🛠️ Software e Immagini Utilizzate

| Ruolo | Immagine PNetLab | Note |
| :--- | :--- | :--- |
| **Switch (Authenticator)** | Cisco IOL (L2) | `i86bi_linux-l2-adventerprisek9` |
| **RADIUS Server** | Windows Server 2012 R2 | `winserver-2012R2` (ID 1745) con ruolo NPS |
| **Client (Supplicant)** | Windows 10 Tiny / XP | `win-tiny10` (ID 1740) o `win-xp` |

## 🚀 Scenari di Test

Il laboratorio copre i seguenti scenari di validazione:
1.  **Configurazione Base:** Setup dell'indirizzamento IP di management e raggiungibilità tra Switch e Server.
2.  **Configurazione RADIUS:** Definizione del client RADIUS (lo switch) e della Shared Secret sul server NPS.
3.  **Abilitazione 802.1X:** Attivazione globale (`dot1x system-auth-control`) e per interfaccia sullo switch.
4.  **Test di Accesso:**
    * **Successo:** Un client con credenziali valide ottiene l'accesso alla rete (la porta passa in stato `authorized`).
    * **Fallimento:** Un client non configurato o con credenziali errate rimane isolato (la porta resta in stato `unauthorized`).

---

## 🗺️ Diagramma di Rete

![Network Diagram](Diagram.png)