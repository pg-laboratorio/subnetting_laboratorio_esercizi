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
>| **Sperimentale (Classe E)** | `240.0.0.0 - 255.255.255.254`<br>`240.0.0.0/4` | Il primo ottetto è compreso **tra 240 e 255** (escluso l'indirizzo `255.255.255.255`, vedi riga precedente). | **NO** | Blocco di indirizzi interamente riservato ad attività di ricerca, scopi sperimentali e usi futuri. |
>| **Shared Address Space (CGNAT)** | `100.64.0.0 - 100.127.255.255`<br>`100.64.0.0/10` | Inizia con **`100.`** e il secondo ottetto è compreso **tra 64 e 127**. | **NO** | Riservato ai provider per il *Carrier-Grade NAT* (RFC 6598): può comparire come IP WAN del router di casa quando il provider condivide un unico IP pubblico tra più clienti. |
>| **Pubblico** | *Tutti i blocchi non speciali/privati* | Non rientra in nessuna delle regole o eccezioni precedenti. | **SÌ** | Identifica in modo univoco e globale un dispositivo (server web, router di confine) esposto direttamente su Internet. |
>
> ---
> **Nota su Routing e NAT:** Gli indirizzi contrassegnati con **Instradabile su Internet: NO** non vengono propagati sulla rete pubblica e vengono bloccati dai router di frontiera dei provider Internet (ISP). 
> * Gli **indirizzi privati (RFC 1918)** possono accedere a Internet solo se il router della LAN implementa il meccanismo di **NAT (Network Address Translation)**, che mappa l'IP privato interno su un IP pubblico valido.
> * Gli altri indirizzi speciali (come Loopback o APIPA) non vengono NATtati e sono confinati all'host locale o al segmento fisico di rete.
>
> **Nota sulle classi:** la suddivisione in Classi A, B, C, D, E (*classful*) è la nomenclatura storica di IPv4, abbandonata nel 1993 con l'introduzione del **CIDR** (*classless*). Oggi la lunghezza della parte di rete è indicata solo dal prefisso (`/8`, `/27`...), ma i nomi delle classi si usano ancora per indicare rapidamente gli intervalli: per questo compaiono negli esercizi come "classificazione storica".


>### Ricorda: due metodi per calcolare Indirizzo di Rete e Broadcast
>
>**Metodo 1 — AND bit-a-bit (il metodo "ufficiale")**
>
>Si scrivono in binario IP e maschera e si esegue l'AND bit per bit. Per il broadcast si portano a `1` tutti i bit della parte host. È il metodo che usano davvero i dispositivi, ed è utile per capire *perché* funziona.
>
>**Metodo 2 — Il "salto" (*Magic Number*), il metodo veloce**
>
>1. Individua l'**ottetto interessante**: quello in cui la maschera non vale né `255` né `0`.
>2. Calcola il **salto**: $256 - \text{(valore della maschera in quell'ottetto)}$.
>3. Le sottoreti, in quell'ottetto, partono da tutti i **multipli del salto**: `0, salto, 2·salto, ...`
>4. **Indirizzo di rete:** nell'ottetto interessante metti il multiplo del salto più grande che non supera il valore dell'IP; gli ottetti a sinistra restano uguali all'IP, quelli a destra diventano `0`.
>5. **Broadcast:** nell'ottetto interessante metti *(multiplo successivo − 1)*; gli ottetti a destra diventano `255`.
>
>| Maschera nell'ottetto | 128 | 192 | 224 | 240 | 248 | 252 | 254 |
>| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
>| **Salto** | 128 | 64 | 32 | 16 | 8 | 4 | 2 |
>
>Negli esercizi seguenti useremo l'AND come metodo principale e il salto come **verifica rapida**.


---

## Esercizio 1.1: Analisi di 192.168.10.133/27

### 1. Identificare lo Scopo
* **Classificazione (storica):** Classe C (maschera nativa `/24`, oggi sostituita dal prefisso CIDR indicato).
* **Ambito di Rete:** Privato. Appartiene al blocco normato dall'RFC 1918 (inizia con il prefisso fisso `192.168.`). È valido esclusivamente all'interno di una rete locale (LAN) e non risulta instradabile direttamente su Internet senza l'ausilio del NAT.

### 2. Identificare la Subnet Mask
Il prefisso CIDR `/27` indica che i primi 27 bit della maschera sono impostati a logico `1` (parte di rete) e i restanti 5 bit sono impostati a `0` (parte host).
* **Rappresentazione Binaria:** `11111111.11111111.11111111.11100000`
* **Conversione in Decimale Puntato:** `255.255.255.224` (ottenuto convertendo l'ultimo ottetto: $128 + 64 + 32 = 224$).

### 3. Numero di Host Disponibili
Il numero di host utilizzabili dipende dai 5 bit dedicati ai dispositivi ($h = 5$). Si applica la formula standard $2^h - 2$:
* **Calcolo:** $2^5 - 2 = 32 - 2 = 30$
* **Risultato:** **30 host utilizzabili** all'interno di questa sottorete (vengono sottratti 2 indirizzi per escludere l'ID di rete e l'indirizzo di broadcast).

> **Eccezioni alla formula $2^h - 2$:** la formula vale per le maschere fino a `/30`. Con `/31` ($h = 1$) la formula darebbe $0$ host, ma l'RFC 3021 consente di usare entrambi gli indirizzi sui collegamenti punto-punto tra router (non esistono né indirizzo di rete né broadcast). Con `/32` ($h = 0$) si identifica un singolo host (usato ad esempio nelle rotte verso un solo dispositivo o sulle interfacce di loopback dei router).

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

> **Verifica con il salto:** maschera `224` nel quarto ottetto → salto $256 - 224 = 32$. Multipli: `0, 32, 64, 96, 128, 160...`. Il valore `133` cade tra `128` e `160`: rete `.128`, broadcast $160 - 1 = 159$. ✔

---

## Esercizio 1.2: Analisi di 172.16.43.100/22

### 1. Identificare lo Scopo
* **Classificazione (storica):** Classe B (maschera nativa `/16`, oggi sostituita dal prefisso CIDR indicato).
* **Ambito di Rete:** Privato. Fa parte del blocco privato definito dall'RFC 1918 (`172.16.0.0/12`), poiché inizia con `172.` e il secondo ottetto ($16$) è compreso tra 16 e 31. È isolato dal traffico Internet globale.

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

> **Verifica con il salto:** qui l'ottetto interessante è il **terzo** (maschera `252`) → salto $256 - 252 = 4$. Multipli: `..., 36, 40, 44, ...`. Il valore `43` cade tra `40` e `44`: rete `172.16.40.0` (quarto ottetto a `0`), broadcast `172.16.43.255` ($44 - 1 = 43$ nel terzo ottetto, `255` nel quarto). ✔

---

## Esercizio 1.3: Analisi di 200.1.1.70/26

### 1. Identificare lo Scopo
* **Classificazione (storica):** Classe C (maschera nativa `/24`, oggi sostituita dal prefisso CIDR indicato).
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

> **Verifica con il salto:** maschera `192` nel quarto ottetto → salto $256 - 192 = 64$. Multipli: `0, 64, 128, 192`. Il valore `70` cade tra `64` e `128`: rete `.64`, broadcast $128 - 1 = 127$. ✔



---
---

# Sezione 2: Subnetting

In questa tipologia di esercizi viene fornito un indirizzo IP di partenza, la sua subnet mask originale (la rete madre) e una nuova subnet mask (più restrittiva).

Per comodità possiamo individuare un passo preliminare e due parti:

**Passo preliminare: individuare la rete madre.** Con la maschera **originale** si calcola la rete di partenza, cioè lo spazio di indirizzi che verrà suddiviso. Tutte le nuove sottoreti stanno *dentro* questa rete.

**Analisi strutturale dei bit e conteggi** (confronto tra le due maschere):
1. Numero di bit della sottorete (Subnet Bits)
2. Numero di sottoreti che verranno create (Subnets Created)
3. Numero di bit dell'host (Host Bits)
4. Numero di host per sottorete (Hosts per Subnet)

**Calcolo degli indirizzi specifici** per la sottorete in cui si trova l'IP:

5. Indirizzo di rete dell'IP corrente (Network Address)
6. Primo host nella rete dell'IP corrente (First Host)
7. Indirizzo di broadcast nella rete (Broadcast Address)
8. Ultimo host nella rete dell'IP corrente (Last Host)


---

## Esercizio 2.1: Subnetting di 172.16.68.230 (da /16 a /20)

* **Indirizzo IP:** `172.16.68.230`
* **Subnet Mask Originale:** `255.255.0.0` (Notazione CIDR: `/16`)
* **Subnet Mask Nuova:** `255.255.240.0` (Notazione CIDR: `/20`)

### Passo preliminare: la rete madre

Con la maschera originale `/16` i primi due ottetti restano invariati e gli altri si azzerano: la rete madre è **`172.16.0.0/16`** (da `172.16.0.0` a `172.16.255.255`). È questo lo spazio che verrà suddiviso in sottoreti `/20`.

### Analisi dei Bit e Conteggi (confronto tra le due maschere)

Convertiamo le due maschere in binario per evidenziare il cambiamento strutturale:
* **Mask Originale (/16):** `11111111.11111111.00000000.00000000`
* **Mask Nuova (/20):** `11111111.11111111.11110000.00000000`

#### 1. Numero di bit della sottorete (Subnet Bits)
È il numero di bit che sono stati "presi in prestito" dalla vecchia parte host per creare le nuove sottoreti.
* **Formula:** `(Bit della Nuova Mask) - (Bit della Mask Originale)`
* **Calcolo:** $20 - 16 = 4$ bit
* **Risultato:** **4 bit** (i 4 bit "accesi" a `1` nel terzo ottetto della nuova maschera).

#### 2. Numero di sottoreti che verranno create (Subnets Created)
Il numero di combinazioni logiche ottenibili con i bit presi in prestito ($s$).
* **Formula:** $2^s$ *(dove s = bit di sottorete)*
* **Calcolo:** $2^4 = 16$
* **Risultato:** **16 sottoreti totali** create all'interno della rete madre.

Con il salto ($256 - 240 = 16$ nel terzo ottetto) le 16 sottoreti sono:

`172.16.0.0` · `172.16.16.0` · `172.16.32.0` · `172.16.48.0` · **`172.16.64.0`** · `172.16.80.0` · ... · `172.16.224.0` · `172.16.240.0`

L'IP `172.16.68.230` cade nella **quinta** sottorete (`172.16.64.0/20`), evidenziata in grassetto.

#### 3. Numero di bit dell'host (Host Bits)
Il numero di bit rimasti impostati a `0` nella nuova subnet mask, dedicati all'indirizzamento dei dispositivi.
* **Formula:** `(Bit Totali IPv4) - (Bit della Nuova Mask)`
* **Calcolo:** $32 - 20 = 12$ bit
* **Risultato:** **12 bit** (4 zeri rimasti nel terzo ottetto + 8 zeri del quarto ottetto).

#### 4. Numero di host per sottorete (Hosts per Subnet)
Il numero di indirizzi IP reali che si possono assegnare ai dispositivi in ogni singola nuova sottorete.
* **Formula:** $2^h - 2$ *(dove h = bit dell'host)*
* **Calcolo:** $2^{12} - 2 = 4096 - 2 = 4094$
* **Risultato:** **4094 host utilizzabili** per ogni sottorete.

### Calcolo degli Indirizzi (Subnet Specifica)

Utilizziamo l'IP di partenza (`172.16.68.230`) e la **Nuova Subnet Mask** (`255.255.240.0`) per isolare i parametri della sottorete specifica in cui risiede questo host.

#### 5. Indirizzo di rete dell'IP corrente (Network Address)

**Metodo 1 — AND bit-a-bit.** I primi due ottetti rimangono invariati (AND con 255) e l'ultimo si azzera (AND con 0). Sviluppiamo il calcolo binario sul terzo ottetto ($68 \text{ AND } 240$):

```text
01000100  (Terzo ottetto IP: .68)
   AND
11110000  (Terzo ottetto Nuova Mask: .240)
------------------------------------------
01000000  (Risultato in decimale: .64)
```

**Metodo 2 — Salto.** Ottetto interessante: il terzo (maschera `240`) → salto $256 - 240 = 16$. Multipli: `..., 48, 64, 80, ...`. Il valore `68` cade tra `64` e `80`, quindi nel terzo ottetto mettiamo `64` e il quarto diventa `0`.

Con questa maschera il salto è molto più rapido: non serve convertire nulla in binario.

* **Risultato:** `172.16.64.0`

#### 6. Primo host nella rete dell'IP corrente (First Host)
È l'indirizzo immediatamente successivo all'indirizzo di rete, ottenuto incrementando di 1 l'ultimo bit della parte host.
* **Formula:** `(Indirizzo di Rete) + 1`
* **Calcolo:** `172.16.64.0 + 1`
* **Risultato:** `172.16.64.1`

#### 7. Indirizzo di broadcast nella rete (Broadcast Address)

**Metodo 1 — Bit host a 1.** Si prende l'indirizzo di rete in binario e si impostano a 1 tutti i 12 bit della parte host (gli ultimi 4 bit del terzo ottetto e tutti gli 8 del quarto).
* **Terzo ottetto binario:** `01000000` diventa `01001111` (decimale: $64 + 8 + 4 + 2 + 1 = 79$).
* **Quarto ottetto binario:** `00000000` diventa `11111111` (decimale: $255$).

**Metodo 2 — Salto.** La sottorete successiva inizia a `80`, quindi nel terzo ottetto mettiamo $80 - 1 = 79$ e il quarto diventa `255`.

* **Risultato:** `172.16.79.255`

#### 8. Ultimo host nella rete dell'IP corrente (Last Host)
È l'indirizzo immediatamente precedente all'indirizzo di broadcast della sottorete.
* **Formula:** `(Indirizzo di Broadcast) - 1`
* **Calcolo:** `172.16.79.255 - 1`
* **Risultato:** `172.16.79.254`

---

## Esercizio 2.2: Subnetting di 192.168.1.185 (da /26 a /28)

* **Indirizzo IP:** `192.168.1.185`
* **Subnet Mask Originale:** `255.255.255.192` (Notazione CIDR: `/26`)
* **Subnet Mask Nuova:** `255.255.255.240` (Notazione CIDR: `/28`)

### Passo preliminare: la rete madre

Attenzione: qui la rete madre **non** è `192.168.1.0`. Applichiamo la maschera **originale** `/26` all'IP ($185 \text{ AND } 192$):

```text
10111001  (Ultimo ottetto IP: .185)
   AND
11000000  (Ultimo ottetto Mask Originale: .192)
-----------------------------------------------
10000000  (Risultato in decimale: .128)
```

La rete madre è **`192.168.1.128/26`** e va da `192.168.1.128` a `192.168.1.191` (verifica con il salto: $256 - 192 = 64$, il valore `185` cade tra `128` e `192`). Le nuove sottoreti `/28` vanno cercate **solo dentro questo intervallo**.

### Analisi dei Bit e Conteggi (confronto tra le due maschere)

Convertiamo le due maschere in binario per evidenziare il cambiamento strutturale (i primi tre ottetti restano invariati a `255`):
* **Mask Originale (/26):** `11111111.11111111.11111111.11000000`
* **Mask Nuova (/28):** `11111111.11111111.11111111.11110000`

#### 1. Numero di bit della sottorete (Subnet Bits)
Indica quanti bit sono stati "presi in prestito" dalla vecchia parte host della maschera `/26` per creare le nuove sottoreti con la maschera `/28`.
* **Formula:** `(Bit della Nuova Mask) - (Bit della Mask Originale)`
* **Calcolo:** $28 - 26 = 2$ bit
* **Risultato:** **2 bit** (i due bit aggiuntivi accesi a `1` nel quarto ottetto).

#### 2. Numero di sottoreti che verranno create (Subnets Created)
Il numero di nuove sottoreti più piccole ricavate all'interno della rete madre `/26`, combinando i 2 bit presi in prestito ($s$).
* **Formula:** $2^s$ *(dove s = bit di sottorete)*
* **Calcolo:** $2^2 = 4$
* **Risultato:** **4 sottoreti totali** create all'interno della rete madre.

Con il salto ($256 - 240 = 16$) le 4 sottoreti, partendo dall'inizio della rete madre (`.128`), sono:

| Sottorete | Indirizzo di Rete | Intervallo |
| :---: | :--- | :--- |
| 1 | `192.168.1.128/28` | `.128 – .143` |
| 2 | `192.168.1.144/28` | `.144 – .159` |
| 3 | `192.168.1.160/28` | `.160 – .175` |
| **4** | **`192.168.1.176/28`** | **`.176 – .191`** ← contiene `.185` |

> **Errore tipico:** elencare le sottoreti partendo da `192.168.1.0` (`.0, .16, .32...`). Quelle sottoreti esistono, ma **non** appartengono alla rete madre `192.168.1.128/26`: i 2 bit presi in prestito generano solo 4 sottoreti, tutte comprese tra `.128` e `.191`.

#### 3. Numero di bit dell'host (Host Bits)
Il numero di bit rimasti impostati a `0` nella nuova maschera `/28`, responsabili dell'assegnazione degli IP ai dispositivi.
* **Formula:** `(Bit Totali IPv4) - (Bit della Nuova Mask)`
* **Calcolo:** $32 - 28 = 4$ bit
* **Risultato:** **4 bit** (gli ultimi 4 zeri rimasti nel quarto ottetto).

#### 4. Numero di host per sottorete (Hosts per Subnet)
Il numero di indirizzi IP reali e assegnabili alle interfacce dei dispositivi in ogni nuova sottorete.
* **Formula:** $2^h - 2$ *(dove h = bit dell'host)*
* **Calcolo:** $2^4 - 2 = 16 - 2 = 14$
* **Risultato:** **14 host utilizzabili** per ciascuna sottorete (escludendo l'ID di rete e il broadcast).

### Calcolo degli Indirizzi (Subnet Specifica)

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
* **Risultato:** `192.168.1.176` (coincide con la sottorete 4 della tabella).

#### 6. Primo host nella rete dell'IP corrente (First Host)
È il primo indirizzo IP utilizzabile per un host, ottenuto incrementando di 1 l'indirizzo di rete.
* **Formula:** `(Indirizzo di Rete) + 1`
* **Calcolo:** `192.168.1.176 + 1`
* **Risultato:** `192.168.1.177`

#### 7. Indirizzo di broadcast nella rete (Broadcast Address)
Si calcola mantenendo intatta la parte di rete nel quarto ottetto (`1011`) e impostando a `1` tutti i restanti 4 bit dedicati all'host (`1111`).
* **Calcolo ultimo ottetto binario:** `10110000` diventa `10111111`. In decimale: $176 + 15 = 191$.
* **Verifica con il salto:** la sottorete successiva inizierebbe a $176 + 16 = 192$, quindi il broadcast è $192 - 1 = 191$.
* **Risultato:** `192.168.1.191`

> Nota: `.191` è anche il broadcast della rete madre `/26`. Non è un caso: l'ultima sottorete termina sempre dove termina la rete madre.

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
> **Calcolo veloce (Metodo del "Salto" o *Magic Number*, vedi il riquadro all'inizio della Sezione 1):**
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
> Osserva questa domanda:
> *"L'IP 10.0.0.5 e l'IP 10.0.0.6 sono sulla stessa rete?"*
> 
> La risposta corretta è sempre: **"Non si può stabilire senza conoscere la Subnet Mask"**. 
> * Se la mask fosse `/30` (`255.255.255.252`), i blocchi andrebbero di 4 in 4 (`0, 4, 8...`). `.5` sarebbe nella rete `10.0.0.4`, mentre `.6` sarebbe nella stessa rete `10.0.0.4` (**Stessa rete**).
> * Se la mask fosse `/31` (`255.255.255.254`), i blocchi andrebbero di 2 in 2 (`0, 2, 4, 6...`). `.5` sarebbe nella rete `10.0.0.4`, mentre `.6` sarebbe nella rete `10.0.0.6` (**Reti diverse**).
>
> *Nota:* l'esempio con la `/31` serve solo a mostrare come cambiano i blocchi. Una `/31` non si usa per una LAN con PC: ha solo 2 indirizzi ed è riservata ai collegamenti punto-punto tra router (vedi le eccezioni alla formula $2^h - 2$ nell'Esercizio 1.1).
