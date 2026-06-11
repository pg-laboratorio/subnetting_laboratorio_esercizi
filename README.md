# Esercizi IPv4 e Subnetting

Questo modulo è focalizzato sull'indirizzamento IPv4 e sulle tecniche di **Subnetting**. L'obiettivo è fornire un metodo chiaro e replicabile per risolvere qualsiasi tipologia di esercizio o scenario reale legato alle reti IP.

## Materiale

* **[Esercizi_IPv4_Subnetting Svolti](./subnetting_esercizi_svolti.md)** — *Documento completo con le regole, le formule e gli esercizi commentati.*
* **[Esercizi_IPv4_Subnetting Casi Reali](./subnetting_esercizi_casi_reali.md)** — *Documento che contiene esercizi applicati a casi reali, ideali per mettersi alla prova con scenari pratici e troubleshooting.*

## Cosa impariamo a fare
In ogni esercizio applichiamo un pattern standard a step che ti aiuterà a non perderti durante i calcoli. 

### **Analisi IPv4**:
1. Identificare lo Scopo
2. Identificare la Subnet Mask
3. Numero di Host Disponibili
4. Indirizzo di Rete
5. Indirizzo Broadcast

### **Subnetting**:
* Analisi strutturale dei bit e conteggi (confronto tra le due maschere).  
1. Numero di bit della sottorete (Subnet Bits)
2. Numero di sottoreti che verranno create (Subnets Created)
3. Numero di bit dell'host (Host Bits)
4. Numero di host per sottorete (Hosts per Subnet)
* Calcolo degli indirizzi specifici per la sottorete in cui si trova l'IP.  
5. Indirizzo di rete dell'IP corrente (Network Address)
6. Primo host nella rete dell'IP corrente (First Host)
7. Indirizzo di broadcast nella rete (Broadcast Address)
8. Ultimo host nella rete dell'IP corrente (Last Host)

## Consiglio:
**Tabella delle potenze di 2:** Tieni sempre a mente (o scrivi su un foglio) le potenze di 2 fino a $2^8$ (128, 64, 32, 16, 8, 4, 2, 1).
