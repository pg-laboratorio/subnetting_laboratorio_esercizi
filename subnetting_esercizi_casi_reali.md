# Esercizi Realistici di IPv4 e Subnetting

## Introduzione

In questi esercizi non viene richiesto solamente di eseguire calcoli matematici, ma di ragionare come un tecnico di rete che deve progettare o verificare una configurazione reale.

**Regola d'oro:** Prima prova a risolvere l'esercizio autonomamente. Se incontri difficoltà, apri i suggerimenti. Solo alla fine consulta la soluzione completa.

---

# Livello Base

## Esercizio 1 - Configurazione di una piccola azienda

Una piccola azienda dispone della rete `192.168.10.0/24`.
L'amministratore vuole suddividerla in sottoreti capaci di ospitare almeno **29 dispositivi** ciascuna, oltre all'interfaccia del router che farà da gateway per ogni sottorete.

### Domande

1. Quale subnet mask potresti utilizzare?
2. Quante sottoreti vengono create?
3. Quanti host utilizzabili sono disponibili in ogni sottorete?
4. Quali sono gli indirizzi di rete delle prime quattro sottoreti?
5. Qual è il broadcast della seconda sottorete?

<details>
<summary>Suggerimento 1</summary>

Conta gli indirizzi davvero necessari: 29 dispositivi + 1 gateway = 30. Poi calcola quanti bit host servono per ottenere almeno 30 host utilizzabili.
</details>

<details>
<summary>Suggerimento 2</summary>

Utilizza la formula $2^h - 2$, poi trova le sottoreti con il metodo del salto.
</details>

<details>
<summary>Soluzione</summary>

* **Indirizzi necessari:** $29 + 1 \text{ (gateway)} = 30$.
* **Subnet mask:** `/27` (ovvero `255.255.255.224`), perché $2^5 - 2 = 30$ è sufficiente, mentre una `/28` offrirebbe solo $2^4 - 2 = 14$ host.
* **Numero di sottoreti:** $27 - 24 = 3$ bit presi in prestito → $2^3 = 8$ sottoreti.
* **Host per sottorete:** $2^5 - 2 = 30$ host.
* **Prime quattro sottoreti** (salto $256 - 224 = 32$):

| Sottorete | Indirizzo di Rete |
| :---: | :--- |
| 1 | `192.168.10.0` |
| 2 | `192.168.10.32` |
| 3 | `192.168.10.64` |
| 4 | `192.168.10.96` |

* **Broadcast seconda sottorete:** `192.168.10.63` (la terza sottorete inizia a `.64`, quindi $64 - 1 = 63$).
</details>

---

## Esercizio 2 - Laboratorio scolastico

Un laboratorio deve ospitare:
- 24 PC
- 1 stampante
- 1 access point

La rete disponibile è `192.168.1.0/24` e il laboratorio utilizzerà la **prima** sottorete `/27`.

### Domande

1. Una subnet `/27` è sufficiente?
2. Quanti host utilizzabili offre?
3. Qual è il margine di crescita?
4. Qual è il primo e l'ultimo host assegnabile?

<details>
<summary>Suggerimento</summary>

Conta il numero totale di indirizzi IP necessari e confrontalo con gli host disponibili in una `/27`. Attenzione: c'è un dispositivo che non compare nell'elenco ma che ha sempre bisogno di un IP nella sottorete…
</details>

<details>
<summary>Soluzione</summary>

* **Host disponibili:** $2^5 - 2 = 30$ host utilizzabili.
* **IP necessari:** $24 \text{ (PC)} + 1 \text{ (stampante)} + 1 \text{ (access point)} + 1 \text{ (gateway)} = 27$. Il gateway, cioè l'interfaccia del router, è l'indirizzo che si dimentica più spesso: senza di lui i PC non escono dal laboratorio.
* **La subnet è sufficiente?** Sì.
* **Margine di crescita:** $30 - 27 = 3$ host disponibili per il futuro.
* **Primo host:** `192.168.1.1` (per convenzione viene spesso assegnato al gateway).
* **Ultimo host:** `192.168.1.30` (il `.31` è il broadcast).
</details>

---

## Esercizio 3 - Verifica di comunicazione

| Dispositivo | IP Configurato |
| :--- | :--- |
| **PC-A** | `192.168.1.100` |
| **PC-B** | `192.168.1.126` |

**Subnet Mask:** `255.255.255.192`

### Domande

1. Qual è la rete di PC-A?
2. Qual è la rete di PC-B?
3. Sono nella stessa rete?
4. Possono comunicare direttamente (senza l'uso di un router)?

<details>
<summary>Suggerimento</summary>

Applica l'operazione logica AND bit-a-bit tra l'ultimo ottetto degli IP e l'ultimo ottetto della Subnet Mask per trovare gli indirizzi di rete. Puoi verificare il risultato con il metodo del salto.
</details>

<details>
<summary>Soluzione</summary>

Eseguiamo l'operazione AND tra l'ultimo ottetto degli IP e la maschera (`192` in binario è `11000000`).

**Rete PC-A:**
```text
100 = 01100100  AND
192 = 11000000
----------------
64  = 01000000  -> Rete: 192.168.1.64
```

**Rete PC-B:**
```text
126 = 01111110  AND
192 = 11000000
----------------
64  = 01000000  -> Rete: 192.168.1.64
```

**Verifica con il salto:** $256 - 192 = 64$, quindi le sottoreti sono `.0`, `.64`, `.128`, `.192`. Sia `100` sia `126` cadono tra `64` e `128`.

* **Risultato:** Si trovano nella **STESSA RETE** (`192.168.1.64/26`, da `.64` a `.127`).
* **Comunicazione:** Sì, possono comunicare direttamente tramite lo switch locale, senza bisogno di un router/gateway.
* **Osservazione:** `.126` è l'**ultimo host** utilizzabile della sottorete, perché `.127` è il broadcast. Un PC configurato con `.128` sarebbe già in un'altra rete.
</details>

---

# Livello Intermedio

## Esercizio 4 - Uffici di un'azienda

Rete aziendale disponibile: `172.16.0.0/16`.
Devono essere create **almeno 20 sottoreti** per vari reparti e sedi future.

### Domande

1. Quanti bit bisogna prendere in prestito dalla parte host?
2. Quale nuova subnet mask occorre utilizzare?
3. Quante sottoreti si ottengono realmente?
4. Quanti host sono disponibili per ciascuna sottorete?

<details>
<summary>Suggerimento</summary>

Trova la prima potenza di 2 che sia maggiore o uguale a 20.
</details>

<details>
<summary>Soluzione</summary>

* $2^4 = 16$ (non basta, ci servono 20 reti).
* $2^5 = 32$ (sufficiente).
* **Bit presi in prestito:** 5 bit.
* **Nuova mask:** $16 + 5 = 21$ → `/21` → `255.255.248.0`
* **Sottoreti ottenute:** 32 sottoreti in totale.
* **Host disponibili:** restano $32 - 21 = 11$ bit per gli host → $2^{11} - 2 = 2046$ host per sottorete.
</details>

---

## Esercizio 5 - Piano di indirizzamento completo

Il reparto IT deve documentare il piano di indirizzamento di una sede. La rete `192.168.20.0/24` viene suddivisa in sottoreti con maschera `/26`.

### Domande

1. Quanti bit vengono presi in prestito e quante sottoreti si ottengono?
2. Completa la tabella con **tutte** le sottoreti: indirizzo di rete, primo host, ultimo host, broadcast.
3. A quale sottorete appartiene l'host `192.168.20.150`?
4. Un tecnico vuole assegnare a una stampante l'indirizzo `192.168.20.191/26`. È una scelta corretta?

<details>
<summary>Suggerimento</summary>

Usa il metodo del salto: l'ultimo ottetto della maschera `/26` vale `192`, quindi le sottoreti avanzano di $256 - 192 = 64$ in 64. Ogni sottorete termina con il broadcast, che è sempre l'indirizzo immediatamente precedente alla rete successiva.
</details>

<details>
<summary>Soluzione</summary>

* **Bit presi in prestito:** $26 - 24 = 2$ → $2^2 = 4$ sottoreti, ciascuna con $2^6 - 2 = 62$ host utilizzabili.
* **Tabella completa** (salto = 64):

| Sottorete | Indirizzo di Rete | Primo Host | Ultimo Host | Broadcast |
| :---: | :--- | :--- | :--- | :--- |
| 1 | `192.168.20.0` | `192.168.20.1` | `192.168.20.62` | `192.168.20.63` |
| 2 | `192.168.20.64` | `192.168.20.65` | `192.168.20.126` | `192.168.20.127` |
| 3 | `192.168.20.128` | `192.168.20.129` | `192.168.20.190` | `192.168.20.191` |
| 4 | `192.168.20.192` | `192.168.20.193` | `192.168.20.254` | `192.168.20.255` |

* **Host `192.168.20.150`:** `150` è compreso tra `128` e `191`, quindi appartiene alla **sottorete 3** (`192.168.20.128/26`).
* **Indirizzo `192.168.20.191/26`:** **No.** È l'indirizzo di **broadcast** della sottorete 3 e non può essere assegnato a nessun dispositivo. Il sistema operativo della stampante (o il router) rifiuterebbe la configurazione, oppure la stampante non funzionerebbe correttamente. L'ultimo indirizzo assegnabile in quella sottorete è `192.168.20.190`.
</details>

---

## Esercizio 6 - Stampanti di rete

In una sede aziendale di grandi dimensioni, tre stampanti sono configurate con la subnet mask `255.255.252.0` (`/22`).

| Stampante | IP |
| :---: | :--- |
| **S1** | `10.20.33.15` |
| **S2** | `10.20.35.200` |
| **S3** | `10.20.36.10` |

### Domande

1. Quale rete appartiene a S1?
2. Quale rete appartiene a S2?
3. Quale rete appartiene a S3?
4. Quali dispositivi comunicano direttamente tra loro e quali richiedono un router?

<details>
<summary>Suggerimento</summary>

Questa volta l'ottetto "interessante" non è l'ultimo: la maschera vale `252` nel **terzo** ottetto e `0` nel quarto. Il quarto ottetto dell'IP quindi non conta (AND con 0 dà sempre 0). Con il salto: $256 - 252 = 4$, le reti avanzano di 4 in 4 nel terzo ottetto. Non farti ingannare da quali IP "sembrano" vicini!
</details>

<details>
<summary>Soluzione</summary>

L'ottetto interessante è il terzo: la maschera vale `252` (`11111100` in binario). Calcoliamo la rete per ciascuna stampante tramite l'operazione AND sul terzo ottetto; il quarto ottetto della rete diventa `0`.

**Rete S1:**
```text
33  = 00100001  AND
252 = 11111100
----------------
32  = 00100000  -> Rete S1: 10.20.32.0
```

**Rete S2:**
```text
35  = 00100011  AND
252 = 11111100
----------------
32  = 00100000  -> Rete S2: 10.20.32.0
```

**Rete S3:**
```text
36  = 00100100  AND
252 = 11111100
----------------
36  = 00100100  -> Rete S3: 10.20.36.0
```

**Verifica con il salto:** con salto 4 le reti nel terzo ottetto sono `..., 28, 32, 36, 40, ...`. I valori `33` e `35` cadono nel blocco `32`–`35`, mentre `36` apre il blocco successivo.

* **Comunicazione diretta:** `S1 <-> S2`: condividono la rete `10.20.32.0/22` (da `10.20.32.0` a `10.20.35.255`), anche se il terzo ottetto è diverso (`33` e `35`).
* **Richiedono Router:** S3 appartiene alla rete `10.20.36.0/22`, quindi i flussi verso S1 o S2 devono essere instradati da un gateway (router).
* **La trappola:** S2 (`10.20.35.200`) e S3 (`10.20.36.10`) *sembrano* vicinissimi, ma sono in reti diverse; S1 e S2 sembrano più lontani, ma sono nella stessa rete. Senza calcolo, a occhio si sbaglia.
</details>

---

# Livello Avanzato

## Esercizio 7 - Collegamento Point-to-Point tra Router

La tua azienda ha appena aperto una nuova sede. Devi collegare in modo diretto ed esclusivo il router della sede principale (**Router-A**) al router della nuova filiale (**Router-B**) tramite un link dedicato in fibra.

L'amministratore ha riservato il blocco `10.255.255.0/24` da utilizzare per i collegamenti dell'infrastruttura. L'obiettivo è sprecare il minor numero possibile di indirizzi IP per questo singolo collegamento tra i due router.

### Domande

1. Quale subnet mask (in notazione CIDR e decimale puntato) rappresenta la "scelta perfetta" per un collegamento punto-punto?
2. Quanti host utilizzabili fornisce esattamente questa maschera?
3. Utilizzando la primissima sottorete disponibile partendo da `10.255.255.0`, quali saranno l'indirizzo di rete, gli IP da configurare sulle interfacce dei due router e l'indirizzo di broadcast?
4. Qual è l'indirizzo di rete della *seconda* sottorete utile se dovessi collegare in futuro un **Router-C**?

<details>
<summary>Suggerimento 1</summary>

Per collegare due router tra loro ti servono esattamente 2 indirizzi IP validi. Quale potenza di 2, sottratta di 2 (Rete e Broadcast), ti dà come risultato esattamente 2?
</details>

<details>
<summary>Suggerimento 2</summary>

Calcola a ritroso: se ti servono solo 2 bit per la parte host, quanti bit ti rimangono per la parte di rete su un totale di 32?
</details>

<details>
<summary>Soluzione</summary>

1. **Subnet Mask ideale:** `/30` (in decimale puntato corrisponde a `255.255.255.252`).
2. **Host utilizzabili:** lasciando $32 - 30 = 2$ bit per gli host, la formula dà $2^2 - 2 = 2$ host. Lo spazio esatto e perfetto per due interfacce router, senza sprecare nessun indirizzo extra!
3. **Parametri della prima sottorete (Link Router-A <-> Router-B):**
   * **Network:** `10.255.255.0`
   * **IP Router-A:** `10.255.255.1`
   * **IP Router-B:** `10.255.255.2`
   * **Broadcast:** `10.255.255.3`
4. **Seconda sottorete (Link futuri):**
   * Il *Magic Number* (salto) è $256 - 252 = 4$. Le reti viaggiano di 4 in 4.
   * La rete successiva disponibile inizierà quindi da **`10.255.255.4/30`** (con IP validi `.5` e `.6`, e broadcast `.7`).

*Nota tecnica:* Sebbene oggi i router moderni supportino anche le maschere `/31` (RFC 3021) appositamente per i link punto-punto (eliminando rete e broadcast), la `/30` rimane lo standard didattico e operativo più richiesto e universale.
</details>

---

## Esercizio 8 - Problema di connettività (Troubleshooting)

Un tecnico riceve questo ticket: *"Due PC dello stesso ufficio, collegati allo stesso switch, non riescono a comunicare tra loro."*

Controllando la configurazione IP trova questi valori:

| Dispositivo | IP | Subnet Mask |
| :--- | :--- | :--- |
| **PC-1** | `10.10.10.34` | `/27` |
| **PC-2** | `10.10.10.62` | `/28` |

### Domande

1. Calcola la rete di appartenenza di PC-1 e di PC-2, ciascuno con **la propria** subnet mask.
2. PC-1 considera PC-2 parte della propria rete? E PC-2 considera PC-1 parte della propria rete?
3. Che cosa succede quando PC-1 invia un `ping` a PC-2?
4. Come si risolve il problema?

<details>
<summary>Suggerimento</summary>

Ogni host decide se una destinazione è "locale" usando **solo la propria** subnet mask. Calcola l'intervallo di indirizzi che ciascun PC ritiene locale e verifica se l'altro PC vi rientra.
</details>

<details>
<summary>Soluzione</summary>

**Rete PC-1** (`/27` = `255.255.255.224`, ultimo ottetto `11100000`):
```text
34  = 00100010  AND
224 = 11100000
----------------
32  = 00100000  -> Rete PC-1: 10.10.10.32/27 (da .32 a .63)
```

**Rete PC-2** (`/28` = `255.255.255.240`, ultimo ottetto `11110000`):
```text
62  = 00111110  AND
240 = 11110000
----------------
48  = 00110000  -> Rete PC-2: 10.10.10.48/28 (da .48 a .63)
```

* **Punto di vista di PC-1:** l'indirizzo `.62` rientra nel suo intervallo `.32 – .63`, quindi PC-1 considera PC-2 **locale** e gli invia i pacchetti direttamente (tramite ARP).
* **Punto di vista di PC-2:** l'indirizzo `.34` **non** rientra nel suo intervallo `.48 – .63`, quindi PC-2 considera PC-1 **remoto** e prova a inviargli le risposte tramite il gateway.
* **Cosa succede al ping:** la richiesta arriva a PC-2, ma la risposta prende un'altra strada. Se il gateway non esiste o non è raggiungibile dalla rete di PC-2, la risposta si perde e il ping fallisce. Anche quando la risposta arriva passando dal router, la comunicazione è **asimmetrica** e fragile: è un tipico problema difficile da diagnosticare senza calcolare le reti.
* **Soluzione:** correggere la subnet mask di PC-2 in `/27` (`255.255.255.224`). Con la stessa maschera, entrambi i PC appartengono alla rete `10.10.10.32/27` e comunicano direttamente tramite lo switch.

> **Nota per il tecnico:** se i due PC avessero avuto entrambi la maschera `/27`, la subnet mask **non** sarebbe stata la causa del guasto: in quel caso si dovrebbero indagare altre cause (firewall attivo sui PC, cavi o porte dello switch, VLAN diverse). Calcolare le reti serve proprio a escludere o confermare rapidamente questa ipotesi.
</details>

---

## Esercizio 9 - Progettazione rete scolastica complessa

Sei il tecnico incaricato di progettare la rete per un istituto.
Rete disponibile: `192.168.0.0/23` (che include lo spazio da `192.168.0.0` a `192.168.1.255`).

| Reparto | Dispositivi da collegare |
| :--- | :---: |
| **Wi-Fi Ospiti** | 100 |
| **Laboratorio** | 60 |
| **Segreteria** | 20 |
| **Aula Docenti** | 15 |

Ogni reparto avrà una propria sottorete con un'interfaccia del router come gateway.

### Obiettivi

1. Realizzare il piano di indirizzamento utilizzando il **VLSM**.
2. Ridurre al minimo lo spreco di indirizzi.
3. Indicare quali porzioni di rete rimangono libere per le espansioni future.

<details>
<summary>Suggerimento 1</summary>

Nel VLSM devi sempre ordinare i reparti dal fabbisogno più grande a quello più piccolo. Questo serve a prevenire la frammentazione dello spazio IP e a evitare che le reti più grandi si sovrappongano ai confini di quelle più piccole.
</details>

<details>
<summary>Suggerimento 2</summary>

Come negli esercizi 1 e 2: ogni sottorete ha bisogno di un indirizzo in più per il **gateway**. Calcola il fabbisogno reale di ogni reparto prima di scegliere la maschera.
</details>

<details>
<summary>Soluzione</summary>

**Fabbisogno reale** (dispositivi + 1 gateway) e maschera scelta, in ordine decrescente:

| Reparto | Dispositivi | + Gateway | Maschera | Host disponibili | Margine |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Wi-Fi Ospiti** | 100 | 101 | `/25` | 126 | 25 |
| **Laboratorio** | 60 | 61 | `/26` | 62 | **1** |
| **Segreteria** | 20 | 21 | `/27` | 30 | 9 |
| **Aula Docenti** | 15 | 16 | `/27` | 30 | 14 |

Allocazione ordinata decrescente:

1. **Wi-Fi Ospiti** → `/25` (126 host)
   * **Network:** `192.168.0.0/25`
   * **Gateway:** `192.168.0.1`
   * **Broadcast:** `192.168.0.127`

2. **Laboratorio** → `/26` (62 host)
   * **Network:** `192.168.0.128/26`
   * **Gateway:** `192.168.0.129`
   * **Broadcast:** `192.168.0.191`

3. **Segreteria** → `/27` (30 host)
   * **Network:** `192.168.0.192/27`
   * **Gateway:** `192.168.0.193`
   * **Broadcast:** `192.168.0.223`

4. **Aula Docenti** → `/27` (30 host)
   * **Network:** `192.168.0.224/27`
   * **Gateway:** `192.168.0.225`
   * **Broadcast:** `192.168.0.255`
   * *Nota:* senza contare il gateway (15 indirizzi) si potrebbe pensare a una `/28`, ma una `/28` offre solo 14 host: non basta nemmeno per i soli dispositivi.

**Spazio rimasto (future espansioni):**
L'intera progettazione ha consumato esattamente lo spazio del blocco `192.168.0.0/24`.
L'intera metà superiore della rete assegnata, ovvero il blocco **`192.168.1.0/24`**, rimane completamente intatta e disponibile per future aule, laboratori o servizi. Ricorda che la rete assegnata è una `/23`: hai a disposizione due blocchi `/24` interi (`192.168.0.x` e `192.168.1.x`).

> **Riflessione da tecnico:** il piano rispetta i requisiti, ma il **Laboratorio ha un margine di un solo indirizzo**: basta aggiungere due PC per esaurire la sottorete. Anche il **Wi-Fi Ospiti** merita attenzione: gli ospiti entrano ed escono di continuo e ogni dispositivo occupa un lease DHCP per un certo tempo, quindi 100 utenti contemporanei possono richiedere molti più di 100 indirizzi nell'arco della giornata. Avendo libera un'intera `/24`, sarebbe ragionevole assegnare una rete più ampia a questi due reparti. "Minimo spreco" non significa "nessun margine".
</details>
