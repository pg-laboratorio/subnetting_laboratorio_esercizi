# Esercizi IPv4 e Subnetting

Questo documento contiene la risoluzione completa di esercizi di subnetting, analizzati e risolti punto per punto secondo i requisiti fondamentali di analisi di rete.

---

# Sezione 1: IPv4 Basics

Partendo da un indirizzo IPv4 qualunque andremo a rispondere alle domande:

1. Identificare lo Scopo
2. Identificare la Subnet Mask
3. Numero di Host Disponibili
4. Indirizzo di Rete
5. Indirizzo Broadcast



>### Ricorda: Ambito di Instradamento IPv4 (tabella riassuntiva)
>
>| Ambito di Rete | Intervallo IP / CIDR | Regola Visiva di Riconoscimento | Instradabile su Internet? | Destinazione e Caso d'Uso Tipico |
>| :--- | :--- | :--- | :--- | :--- |
>| **Privato (Classe A)** | `10.0.0.0 - 10.255.255.255`<br>`10.0.0.0/8` | Inizia sempre con **`10.`** | **NO** | Reti locali (LAN) aziendali di grandi dimensioni e infrastrutture VPC in cloud. |
>| **Privato (Classe B)** | `172.16.0.0 - 172.31.255.255`<br>`172.16.0.0/12` | Inizia con **`172.`** e il secondo ottetto è compreso **tra 16 e 31**. | **NO** | Reti aziendali interne e collegamenti VPN site-to-site. |
>| **Privato (Classe C)** | `192.168.0.0 - 192.168.255.255`<br>`192.168.0.0/16` | Inizia sempre con **`192.168.`** | **NO** | Reti domestiche, router commerciali (SOHO) e hotspot Wi-Fi. |
>| **Loopback** | `127.0.0.0 - 127.255.255.255`<br>`127.0.0.0/8` | Inizia sempre con **`127.`** (es. `127.0.0.1`). | **NO** | Interfaccia locale (*Local Host*); usata per testare servizi sulla propria macchina senza inviare dati all'esterno. |
>| **APIPA / Link-Local** | `169.254.0.0 - 169.254.255.255`<br>`169.254.0.0/16` | Inizia sempre con **`169.254.`**. | **NO** | Auto-assegnato dal sistema operativo in caso di fallimento o assenza di un server DHCP (segnala un problema di connettività locale). |
>| **Host su Rete Locale** | `0.0.0.0 - 0.255.255.255`<br>`0.0.0.0/8` | Il primo ottetto è **`0.`**. | **NO** | Riservato per la comunicazione con l'host locale sulla rete corrente (RFC 1122). |
>| **Non Specificato** | `0.0.0.0`<br>`0.0.0.0/32` | È esattamente l'indirizzo **`0.0.0.0`**. | **NO** | Usato temporaneamente dai dispositivi all'avvio prima di ottenere un IP valido o per definire la rotta di default (*default route*) nel routing. |
>| **Multicast (Classe D)** | `224.0.0.0 - 239.255.255.255`<br>`224.0.0.0/4` | Il primo ottetto è compreso **tra 224 e 239**. | **NO** *(Solo su reti abilitate)* | Invio di un singolo flusso di dati replicato verso un gruppo specifico di host iscritti (es. streaming live IPTV o protocolli di routing). |
>| **Broadcast Limitato** | `255.255.255.255`<br>`255.255.255.255/32` | È esattamente l'indirizzo **`255.255.255.255`**. | **NO** | Pacchetto inviato contemporaneamente a tutti gli host appartenenti alla stessa rete locale fisica (es. richiesta *DHCP DISCOVER*). |
>| **Sperimentale (Classe E)** | `240.0.0.0 - 255.255.255.254`<br>`240.0.0.0/4` | Il primo ottetto è compreso **tra 240 e 255**. | **NO** | Blocco di indirizzi interamente riservato ad attività di ricerca, scopi sperimentali e usi futuri. |
>| **Pubblico** | *Tutti i blocchi non speciali/privati* | Non rientra in nessuna delle regole o eccezioni precedenti. | **SÌ** | Identifica in modo univoco e globale un dispositivo (server web, router di confine) esposto direttamente su Internet. |
>
> ---
> **Nota su Routing e NAT:** Gli indirizzi contrassegnati con **Instradabile su Internet: NO** non vengono propagati sulla rete pubblica e vengono bloccati dai router di frontiera dei provider Internet (ISP). 
> * Gli **indirizzi privati (RFC 1918)** possono accedere a Internet solo se il router della LAN implementa il meccanismo di **NAT (Network Address Translation)**, che mappa l'IP privato interno su un IP pubblico valido.
> * Gli altri indirizzi speciali (come Loopback o APIPA) non vengono NATtati e sono confinati all'host locale o al segmento fisico di rete.


---

## Esercizio 1.1: Analisi di 192.168.10.133/27

### 1. Identificare lo Scopo
* **Classificazione:** Classe C (Maschera nativa `/24`).
* **Ambito di Rete:** Privato. Appartiene al blocco normato dall'RFC 1918 (inizia con il prefisso fisso `192.168.`). È valido esclusivamente all'interno di una rete locale (LAN) e non risulta instradabile direttamente su Internet senza l'ausilio del NAT.

### 2. Identificare la Subnet Mask
Il prefisso CIDR `/27` indica che i primi 27 bit della maschera sono impostati a logico `1` (parte di rete) e i restanti 5 bit sono impostati a `0` (parte host).
* **Rappresentazione Binaria:** `11111111.11111111.11111111.11100000`
* **Conversione in Decimale Puntato:** `255.255.255.224` (ottenuto convertendo l'ultimo ottetto: $128 + 64 + 32 = 224$).

### 3. Numero di Host Disponibili
Il numero di host utilizzabili dipende dai 5 bit dedicati ai dispositivi ($h = 5$). Si applica la formula standard $2^h - 2$:
* **Calcolo:** $2^5 - 2 = 32 - 2 = 30$
* **Risultato:** **30 host utilizzabili** all'interno di questa sottorete (vengono sottratti 2 indirizzi per escludere l'ID di rete e l'indirizzo di broadcast).

### 4. Indirizzo di Rete
L'indirizzo si ricava applicando l'operazione logica AND bit-a-bit tra l'indirizzo IP del dispositivo e la subnet mask calcolata. L'operazione è discriminante nell'ultimo ottetto ($133 \text{ AND } 224$):

```text
10000101  (Ultimo ottetto IP: .133)
   AND
11100000  (Ultimo ottetto Mask: .224)
-------------------------------------
10000000  (Risultato decimale: .128)
```

* **Risultato:** `192.168.10.128`

### 5. Indirizzo Broadcast
L'indirizzo di broadcast si ottiene partendo dal valore binario dell'indirizzo di rete e convertendo a `1` tutti i 5 bit della parte host:
* **Calcolo ultimo ottetto:** Il blocco `10000000` diventa `10011111`. In decimale corrisponde a $128 + 16 + 8 + 4 + 2 + 1 = 159$.
* **Risultato:** `192.168.10.159`

---

## Esercizio 1.2: Analisi di 172.16.43.100/22

### 1. Identificare lo Scopo
* **Classificazione:** Classe B (Maschera nativa `/16`).
* **Ambito di Rete:** Privato. Fa parte del blocco RFC 1918 dedicato alle medie e grandi aziende, poiché inizia con `172.` ed il secondo ottetto ($16$) si colloca nell'intervallo protetto tra 16 e 31. È isolato dal traffico Internet globale.

### 2. Identificare la Subnet Mask
Il prefisso CIDR `/22` definisce una maschera con 22 bit di rete bloccati a `1` e 10 bit d'host allocati a `0`.
* **Rappresentazione Binaria:** `11111111.11111111.11111100.00000000`
* **Conversione in Decimale Puntato:** `255.255.252.0` (il terzo ottetto `11111100` equivale a $128 + 64 + 32 + 16 + 8 + 4 = 252$).

### 3. Numero di Host Disponibili
Con 10 bit a disposizione per indirizzare le macchine ($h = 10$), applichiamo la formula matematica $2^h - 2$:
* **Calcolo:** $2^{10} - 2 = 1024 - 2 = 1022$
* **Risultato:** **1022 host utilizzabili** configurabili sui dispositivi della subnet.

### 4. Indirizzo di Rete
Applichiamo l'operazione logica AND bit-a-bit. La variazione e il calcolo si concentrano nel terzo ottetto dell'indirizzo ($43 \text{ AND } 252$), mentre il quarto ottetto si azzera automaticamente:

```text
00101011  (Terzo ottetto IP: .43)
   AND
11111100  (Terzo ottetto Mask: .252)
-------------------------------------
00101000  (Risultato decimale: .40)
```

* **Risultato:** `172.16.40.0`

### 5. Indirizzo Broadcast
Prendiamo l'indirizzo di rete e configuriamo a valore logico `1` tutti i 10 bit finali assegnati agli host (ovvero gli ultimi 2 bit del terzo ottetto e tutti gli 8 bit del quarto ottetto):
* **Terzo ottetto binario:** `00101000` diventa `00101011` (decimale $43$).
* **Quarto ottetto binario:** `00000000` diventa `11111111` (decimale $255$).
* **Risultato:** `172.16.43.255`

---

## Esercizio 1.3: Analisi di 200.1.1.70/26

### 1. Identificare lo Scopo
* **Classificazione:** Classe C (Maschera nativa `/24`).
* **Ambito di Rete:** Pubblico. Non rientrando in alcuna categoria di indirizzi privati o speciali, questo IP è registrato in modo univoco a livello globale. È direttamente raggiungibile e instradabile sulla rete Internet globale.


### 2. Identificare la Subnet Mask
Il prefisso CIDR `/26` indica che i primi 26 bit dell'indirizzo sono dedicati alla componente di rete (impostati a `1`) e i restanti 6 bit sono assegnati alla componente host (impostati a `0`).
* **Rappresentazione Binaria:** `11111111.11111111.11111111.11000000`
* **Conversione in Decimale Puntato:** `255.255.255.192` (l'ultimo ottetto modificato presenta i primi due bit accesi: $128 + 64 = 192$).

### 3. Numero di Host Disponibili
Il numero di indirizzi assegnabili all'interno di ciascuna sottorete dipende dai 6 bit host rimasti liberi ($h = 6$). Applichiamo la formula $2^h - 2$:
* **Calcolo:** $2^6 - 2 = 64 - 2 = 62$
* **Risultato:** **62 host utilizzabili** (escludendo l'indirizzo iniziale di rete e l'indirizzo finale di broadcast).

### 4. Indirizzo di Rete
L'indirizzo si ottiene effettuando l'operazione logica AND bit-a-bit tra l'IP e la maschera di sottorete. L'ultimo ottetto dell'IP vale $70$, mentre quello della maschera vale $192$:

```text
01000110  (Ultimo ottetto IP: .70)
   AND
11000000  (Ultimo ottetto Mask: .192)
-------------------------------------
01000000  (Risultato decimale: .64)
```

* **Risultato:** `200.1.1.64`

### 5. Indirizzo Broadcast
L'indirizzo di broadcast si calcola prendendo la forma binaria della rete e impostando a `1` tutti i 6 bit della parte host:
* **Calcolo ultimo ottetto:** Il blocco della rete `01000000` diventa `01111111`. Convertito in decimale corrisponde a: $64 + 32 + 16 + 8 + 4 + 2 + 1 = 127$.
* **Risultato:** `200.1.1.127`



---
---

# Sezione 2: Subnetting 

In questa tipologia di esercizi viene fornito un indirizzo IP di partenza, la sua subnet mask originale (la rete madre) e una nuova subnet mask (più restrittiva).

Per comodità possiamo individuare due parti: 

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


---

## Esercizio 2.1: 
* **Indirizzo IP:** `172.16.68.230`
* **Subnet Mask Originale:** `255.255.0.0` (Notazione CIDR: `/16`)
* **Subnet Mask Nuova:** `255.255.240.0` (Notazione CIDR: `/20`)

### Analisi dei Bit e Conteggi (confronto tra le due maschere)

Convertiamo le due maschere in binario per evidenziare il cambiamento strutturale:
* **Mask Originale (/16):** `11111111.11111111.00000000.00000000`
* **Mask Nuova (/20):** `11111111.11111111.11110000.00000000`

#### 1. Numero di bit della sottorete (Subnet Bits)
È il numero di bit che sono stati "presi in prestito" dalla vecchia parte host per creare le nuove sottoreti.
* **Formula:** `(Bit della Nuova Mask) - (Bit della Mask Originale)`
* **Calcolo:** 20 bit - 16 bit = 4 bit
* **Risultato:** **4 bit** (i 4 bit "accesi" a `1` nel terzo ottetto della nuova maschera).

#### 2. Numero di sottoreti che verranno create (Subnets Created)
Il numero di combinazioni logiche ottenibili con i bit presi in prestito ($s$).
* **Formula:** $2^s$ *(dove s = bit di sottorete)*
* **Calcolo:** $2^4 = 16$
* **Risultato:** **16 sottoreti totali** create all'interno dello spazio della rete principale.

#### 3. Numero di bit dell'host (Host Bits)
Il numero di bit rimasti impostati a `0` nella nuova subnet mask, dedicati all'indirizzamento dei dispositivi.
* **Formula:** `(Bit Totali IPv4) - (Bit della Nuova Mask)`
* **Calcolo:** 32 bit - 20 bit = 12 bit
* **Risultato:** **12 bit** (4 zeri rimasti nel terzo ottetto + 8 zeri del quarto ottetto).

#### 4. Numero di host per sottorete (Hosts per Subnet)
Il numero di indirizzi IP reali che si possono assegnare ai dispositivi in ogni singola nuova sottorete.
* **Formula:** $2^h - 2$ *(dove h = bit dell'host)*
* **Calcolo:** $2^{12} - 2 = 4096 - 2 = 4094$
* **Risultato:** **4094 host utilizzabili** per ogni sottorete.


### Calcolo degli Indirizzi (Subnet Specifica)

Utilizziamo l'IP di partenza (`172.16.68.230`) e la **Nuova Subnet Mask** (`255.255.240.0`) per isolare i parametri della sottorete specifica in cui risiede questo host.

#### 5. Indirizzo di rete dell'IP corrente (Network Address)
Si esegue l'operazione logica AND bit-a-bit. I primi due ottetti rimangono invariati (AND con 255) e l'ultimo si azzera (AND con 0). Sviluppiamo il calcolo binario sul terzo ottetto (68 AND 240):

```text
01000100  (Terzo ottetto IP: .68)
   AND
11110000  (Terzo ottetto Nuova Mask: .240)
------------------------------------------
01000000  (Risultato in decimale: .64)
```
* **Risultato:** `172.16.64.0`

#### 6. Primo host nella rete dell'IP corrente (First Host)
È l'indirizzo immediatamente successivo all'indirizzo di rete, ottenuto incrementando di 1 l'ultimo bit della parte host.
* **Formula:** `(Indirizzo di Rete) + 1`
* **Calcolo:** `172.16.64.0 + 1`
* **Risultato:** `172.16.64.1`

#### 7. Indirizzo di broadcast nella rete (Broadcast Address)
Si ottiene prendendo l'indirizzo di rete in binario e impostando a 1 tutti i 12 bit della parte host (gli ultimi 4 bit del terzo ottetto e tutti gli 8 del quarto).
* **Terzo ottetto binario:** `01000000` diventa `01001111` (decimale: $64 + 8 + 4 + 2 + 1 = 79$).
* **Quarto ottetto binario:** `00000000` diventa `11111111` (decimale: $255$).
* **Risultato:** `172.16.79.255`

#### 8. Ultimo host nella rete dell'IP corrente (Last Host)
È l'indirizzo immediatamente precedente all'indirizzo di broadcast della sottorete.
* **Formula:** `(Indirizzo di Broadcast) - 1`
* **Calcolo:** `172.16.79.255 - 1`
* **Risultato:** `172.16.79.254`

---

## Esercizio 2.2:

* **Indirizzo IP Dato:** `192.168.1.185`
* **Subnet Mask Originale:** `255.255.255.192` (Notazione CIDR: `/26`)
* **Subnet Mask Nuova:** `255.255.255.240` (Notazione CIDR: `/28`)

---

#### Analisi dei Bit e Conteggi
Convertiamo l'ultimo ottetto delle due maschere in binario per evidenziare la transizione strutturale (i primi tre ottetti restano invariati a `255`):
* **Mask Originale (/26):** `11111111.11111111.11111111.11000000`
* **Mask Nuova (/28):** `11111111.11111111.11111111.11110000`

#### 1. Numero di bit della sottorete (Subnet Bits)
Indica quanti bit sono stati "presi in prestito" dalla vecchia parte host della maschera `/26` per creare le nuove sottoreti con la maschera `/28`.
* **Formula:** `(Bit della Nuova Mask) - (Bit della Mask Originale)`
* **Calcolo:** $28 \text{ bit} - 26 \text{ bit} = 2 \text{ bit}$
* **Risultato:** **2 bit** (i due bit aggiuntivi accesi a `1` nel quarto ottetto).

#### 2. Numero di sottoreti che verranno create (Subnets Created)
Il numero di nuove sottoreti più piccole ricavate all'interno della subnet `/26` di partenza, combinando i 2 bit presi in prestito ($s$).
* **Formula:** $2^s$ *(dove s = bit di sottorete)*
* **Calcolo:** $2^2 = 4$
* **Risultato:** **4 sottoreti totali** create all'interno del segmento originale.

#### 3. Numero di bit dell'host (Host Bits)
Il numero di bit rimasti impostati a `0` nella nuova maschera `/28`, responsabili dell'assegnazione degli IP ai dispositivi.
* **Formula:** `(Bit Totali IPv4) - (Bit della Nuova Mask)`
* **Calcolo:** $32 \text{ bit} - 28 \text{ bit} = 4 \text{ bit}$
* **Risultato:** **4 bit** (gli ultimi 4 zeri rimasti nel quarto ottetto).

#### 4. Numero di host per sottorete (Hosts per Subnet)
Il numero di indirizzi IP reali e assegnabili alle interfacce dei dispositivi in ogni nuova sottorete.
* **Formula:** $2^h - 2$ *(dove h = bit dell'host)*
* **Calcolo:** $2^4 - 2 = 16 - 2 = 14$
* **Risultato:** **14 host utilizzabili** per ciascuna sottorete (escludendo l'ID di rete e il broadcast).

---

#### Calcolo degli Indirizzi (Subnet Specifica)
Utilizziamo l'IP di partenza (`192.168.1.185`) e la **Nuova Subnet Mask** (`255.255.255.240`) per isolare i parametri della sottorete specifica in cui risiede questo host.

#### 5. Indirizzo di rete dell'IP corrente (Network Address)
Si esegue l'operazione logica AND bit-a-bit tra il quarto ottetto dell'IP ($185$) e quello della nuova maschera ($240$). I primi tre ottetti non subiscono variazioni.

```text
10111001  (Ultimo ottetto IP: .185)
   AND
11110000  (Ultimo ottetto Nuova Mask: .240)
------------------------------------------
10110000  (Risultato in decimale: 128 + 32 + 16 = 176)
```
* **Risultato:** `192.168.1.176`

#### 6. Primo host nella rete dell'IP corrente (First Host)
È il primo indirizzo IP utilizzabile per un host, ottenuto incrementando di 1 l'indirizzo di rete.
* **Formula:** `(Indirizzo di Rete) + 1`
* **Calcolo:** `192.168.1.176 + 1`
* **Risultato:** `192.168.1.177`

#### 7. Indirizzo di broadcast nella rete (Broadcast Address)
Si calcola mantenendo intatta la parte di rete nel quarto ottetto (`1011`) e impostando a `1` tutti i restanti 4 bit dedicati all'host (`1111`).
* **Calcolo ultimo ottetto binario:** `10110000` diventa `10111111`. In decimale: $176 + 15 = 191$.
* **Risultato:** `192.168.1.191`

#### 8. Ultimo host nella rete dell'IP corrente (Last Host)
È l'ultimo indirizzo IP valido assegnabile a un dispositivo, immediatamente precedente al broadcast.
* **Formula:** `(Indirizzo di Broadcast) - 1`
* **Calcolo:** `192.168.1.191 - 1`
* **Risultato:** `192.168.1.190`

---
---
> # Come capire se due IP si trovano sulla stessa rete?
> 
> Per determinare se due indirizzi IP appartengono alla stessa rete (o sottorete) non basta guardare i numeri a occhio: **è indispensabile conoscere la Subnet Mask**. 
> 
> La regola d'oro è la seguente:
> **Due indirizzi IP si trovano sulla stessa rete solo e soltanto se, applicando la stessa Subnet Mask ad entrambi, si ottiene l'esatto identico Indirizzo di Rete (Network ID).**
> 
> 
> ### Passaggi risolutivi:
> 
> 1. **Prendi i due IP e la Subnet Mask** comune (es. fornita dal router o dall'esercizio).
> 2. **Calcola l'Indirizzo di Rete** del primo IP usando l'operazione logica **AND** (o la regola del salto).
> 3. **Calcola l'Indirizzo di Rete** del secondo IP con la stessa maschera.
>    * **Se i due risultati coincidono:** I dispositivi sono nella stessa rete e possono comunicare direttamente senza router.
>    * **Se i due risultati differiscono:** I dispositivi sono su reti diverse e hanno bisogno di un router (Gateway) per comunicare.
> 
> ---
> 
> ### Caso 1: Maschere Standard
> Con maschere "piene" come `/8` (`255.0.0.0`), `/16` (`255.255.0.0`) o `/24` (`255.255.255.0`), il calcolo è visivo.
> 
> * **IP A:** `192.168.1.45`  
> * **IP B:** `192.168.1.130`  
> * **Subnet Mask:** `255.255.255.0` (`/24`)
> 
> **Analisi visiva:** La maschera `/24` blocca i primi 3 ottetti.
> * Rete di IP A: `192.168.1.0`
> * Rete di IP B: `192.168.1.0`
> * **Risultato:** I primi tre ottetti (`192.168.1.`) sono identici. **STESSA RETE.**
> 
> Se l'IP B fosse stato `192.168.2.130`, l'indirizzo di rete sarebbe diventato `192.168.2.0` (diverso da `192.168.1.0`), quindi sarebbero stati su **RETI DIVERSE**.
> 
> ---
> 
> ### Caso 2: Maschere "Classless" o Frazionate
> Quando la maschera taglia un ottetto a metà (es. `/26`, `/27`, `/28`), serve il calcolo matematico.
> 
> * **IP A:** `192.168.1.185`  
> * **IP B:** `192.168.1.178`  
> * **Subnet Mask:** `255.255.255.240` (`/28`)
> 
> **Calcolo veloce (Metodo del "Salto" o *Magic Number*):**
> 1. Prendi l'ottetto interessato della maschera: `240`.
> 2. Trova la dimensione del blocco (Salto): $256 - 240 = 16$.
> 3. Le sottoreti viaggiano di 16 in 16: `0, 16, 32, ..., 160, 176, 192, 208...`
> 4. Posiziona i due IP nei blocchi:
>    * L'ultimo ottetto di IP A è **185**: si trova nel blocco che inizia a **176** (poiché il successivo è 192). Rete: `192.168.1.176`
>    * L'ultimo ottetto di IP B è **178**: si trova anche lui nel blocco che inizia a **176**. Rete: `192.168.1.176`
> 
> **Risultato:** Entrambi gli IP generano la rete `192.168.1.176`. **STESSA RETE.**
> 
> #### Cosa succede se cambiamo un IP?
> * **IP C:** `192.168.1.195` (con la stessa maschera `/28`).
> * Il numero **195** supera il confine di 192 ed entra nel blocco successivo (`192` fino a `207`).
> * La sua rete sarà `192.168.1.192`.
> * **Risultato:** IP A (`.185`) e IP C (`.195`) si trovano su **RETI DIVERSE**.
> 
> ---
> 
> ### Attenzione
> A volte nei quiz potresti trovare una domanda trabocchetto come questa:
> *"L'IP 10.0.0.5 e l'IP 10.0.0.6 sono sulla stessa rete?"*
> 
> La risposta corretta è sempre: **"Non si può stabilire senza conoscere la Subnet Mask"**. 
> * Se la mask fosse `/30` (`255.255.255.252`), i blocchi andrebbero di 4 in 4 (`0, 4, 8...`). `.5` sarebbe nella rete `10.0.0.4`, mentre `.6` sarebbe nella stessa rete `10.0.0.4` (**Stessa rete**).
> * Se la mask fosse `/31` (`255.255.255.254`), i blocchi andrebbero di 2 in 2 (`0, 2, 4, 6...`). `.5` sarebbe nella rete `10.0.0.4`, mentre `.6` sarebbe nella rete `10.0.0.6` (**Reti diverse**).
