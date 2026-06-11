# Esercizi Realistici di IPv4 e Subnetting

## Introduzione

In questi esercizi non viene richiesto solamente di eseguire calcoli matematici, ma di ragionare come un tecnico di rete che deve progettare o verificare una configurazione reale.

**Regola d'oro:** Prima prova a risolvere l'esercizio autonomamente. Se incontri difficoltà, apri i suggerimenti. Solo alla fine consulta la soluzione completa.

---

# Livello Base

## Esercizio 1 - Configurazione di una piccola azienda

Una piccola azienda dispone della rete `192.168.10.0/24`.
L'amministratore vuole suddividerla in sottoreti capaci di ospitare almeno **30 dispositivi** ciascuna.

### Domande

1. Quale subnet mask potresti utilizzare?
2. Quante sottoreti vengono create?
3. Quanti host utilizzabili sono disponibili in ogni sottorete?
4. Quali sono gli indirizzi di rete delle prime quattro sottoreti?
5. Qual è il broadcast della seconda sottorete?

<details>
<summary>Suggerimento 1</summary>
Calcola quanti bit host servono per ottenere almeno 30 host utilizzabili.
</details>

<details>
<summary>Suggerimento 2</summary>
Utilizza la formula: <code>2^h - 2</code>
</details>

<details>
<summary>Soluzione</summary>

* **Subnet mask:** `/27` (ovvero `255.255.255.224`)
* **Numero di sottoreti:** `27 - 24 = 3 bit` presi in prestito -> `2^3 = 8` sottoreti.
* **Host per sottorete:** `2^5 - 2 = 30` host.
* **Prime quattro sottoreti:**

| Sottorete | Indirizzo di Rete |
| :---: | :--- |
| 1 | `192.168.10.0` |
| 2 | `192.168.10.32` |
| 3 | `192.168.10.64` |
| 4 | `192.168.10.96` |

* **Broadcast seconda sottorete:** `192.168.10.63`
</details>

---

## Esercizio 2 - Laboratorio scolastico

Un laboratorio deve ospitare:
- 24 PC
- 1 stampante
- 1 access point

La rete disponibile è `192.168.1.0/24`.

### Domande

1. Una subnet `/27` è sufficiente?
2. Quanti host utilizzabili offre?
3. Qual è il margine di crescita?
4. Qual è il primo e l'ultimo host assegnabile?

<details>
<summary>Suggerimento</summary>
Conta il numero totale di dispositivi richiesti (IP necessari) e confrontalo con gli host disponibili in una /27.
</details>

<details>
<summary>Soluzione</summary>

* **Host disponibili:** `2^5 - 2 = 30` host utilizzabili.
* **Dispositivi richiesti:** `24 + 1 + 1 = 26` IP necessari. 
* **La subnet è sufficiente?** Sì.
* **Margine di crescita:** `30 - 26 = 4` host disponibili per il futuro.
* **Primo host:** `192.168.1.1`
* **Ultimo host:** `192.168.1.30` (il `.31` è il Broadcast).
</details>

---

## Esercizio 3 - Verifica di comunicazione

| Dispositivo | IP Configurato |
| :--- | :--- |
| **PC-A** | `192.168.1.45` |
| **PC-B** | `192.168.1.62` |

**Subnet Mask:** `255.255.255.224`

### Domande

1. Qual è la rete di PC-A?
2. Qual è la rete di PC-B?
3. Sono nella stessa rete?
4. Possono comunicare direttamente (senza l'uso di un router)?

<details>
<summary>Suggerimento</summary>
Applica l'operazione logica AND bit-a-bit tra l'ultimo ottetto degli IP e l'ultimo ottetto della Subnet Mask per trovare gli indirizzi di rete.
</details>

<details>
<summary>Soluzione</summary>

Eseguiamo l'operazione AND tra l'ultimo ottetto degli IP e la maschera (`224` in binario è `11100000`).

**Rete PC-A:**
```text
45  = 00101101  AND
224 = 11100000
----------------
32  = 00100000  -> Rete: 192.168.1.32
```

**Rete PC-B:**
```text
62  = 00111110  AND
224 = 11100000
----------------
32  = 00100000  -> Rete: 192.168.1.32
```

* **Risultato:** Si trovano nella **STESSA RETE**.
* **Comunicazione:** Sì, possono comunicare direttamente tramite lo switch locale, senza bisogno di un router/gateway.
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

* `2^4 = 16` (Non basta, ci servono 20 reti).
* `2^5 = 32` (Perfetto).
* **Bit presi in prestito:** `5 bit`.
* **Nuova mask:** `/16 + 5 = /21` -> `255.255.248.0`
* **Sottoreti ottenute:** `32 sottoreti` in totale.
* **Host disponibili:** Ci restano 11 bit per gli host (`32 - 21 = 11`). -> `2^11 - 2 = 2046` host per sottorete.
</details>

---

## Esercizio 5 - Analisi di una rete esistente

Un server aziendale possiede il seguente indirizzo IP: `172.16.68.230/20`

### Domande

1. Determina la Subnet Mask estesa (decimale puntato).
2. Calcola l'indirizzo di Rete.
3. Calcola il Broadcast.
4. Determina il primo e l'ultimo host assegnabile.
5. Quanti host utilizzabili ci sono in questa subnet?

<details>
<summary>Suggerimento</summary>
Converti la maschera /20 in decimale puntato. Esegui l'operazione AND bit-a-bit sul terzo ottetto per isolare la porzione di rete.
</details>

<details>
<summary>Soluzione</summary>

* **Mask:** `255.255.240.0` (il terzo ottetto `240` in binario è `11110000`).

Calcoliamo l'indirizzo di rete con l'operazione AND sul terzo ottetto (tra `68` e `240`):
```text
68  = 01000100  AND
240 = 11110000
----------------
64  = 01000000
```

* **Network:** `172.16.64.0`
* **Broadcast:** Impostando a 1 tutti i bit della parte host (gli ultimi 4 bit del terzo ottetto e tutti gli 8 del quarto) otteniamo `172.16.79.255`.
* **Primo host:** `172.16.64.1`
* **Ultimo host:** `172.16.79.254`
* **Host totali:** `2^12 - 2 = 4094`
</details>

---

## Esercizio 6 - Stampanti di rete

Tre stampanti sono configurate con la subnet mask `255.255.255.240` (`/28`).

| Stampante | IP |
| :---: | :--- |
| **S1** | `192.168.50.178` |
| **S2** | `192.168.50.185` |
| **S3** | `192.168.50.194` |

### Domande

1. Quale rete appartiene a S1?
2. Quale rete appartiene a S2?
3. Quale rete appartiene a S3?
4. Quali dispositivi comunicano direttamente tra loro e quali richiedono un router?

<details>
<summary>Soluzione</summary>

L'ultimo ottetto della maschera è `240` (`11110000` in binario). Calcoliamo la rete per ciascuna stampante tramite l'operazione AND.

**Rete S1:**
```text
178 = 10110010  AND
240 = 11110000
----------------
176 = 10110000  -> Rete S1: 192.168.50.176
```

**Rete S2:**
```text
185 = 10111001  AND
240 = 11110000
----------------
176 = 10110000  -> Rete S2: 192.168.50.176
```

**Rete S3:**
```text
194 = 11000010  AND
240 = 11110000
----------------
192 = 11000000  -> Rete S3: 192.168.50.192
```

* **Comunicazione diretta:** `S1 <-> S2` (condividono lo stesso indirizzo di rete `192.168.50.176`).
* **Richiedono Router:** S3 appartiene a una subnet diversa (`.192`), quindi i flussi verso S1 o S2 devono essere ruotati da un gateway.
</details>

---

# Livello Avanzato

## Esercizio 7 - Progettazione rete aziendale (VLSM)

Rete principale: `192.168.100.0/24`

| Reparto | Host richiesti |
| :--- | :--- |
| **Produzione** | 80 |
| **Amministrazione** | 50 |
| **IT** | 20 |

### Domande

1. Assegna le subnet mask più adeguate usando il VLSM.
2. Determina Network e Broadcast per ogni reparto.

<details>
<summary>Suggerimento</summary>
Nel VLSM devi <strong>sempre</strong> ordinare i reparti dal fabbisogno più grande a quello più piccolo prima di assegnare le reti.
</details>

<details>
<summary>Soluzione</summary>

1. **Produzione (80 host richiesti)**
* Subnet ideale: `/25` (offre fino a 126 host).
* **Network:** `192.168.100.0/25`
* **Broadcast:** `192.168.100.127`

2. **Amministrazione (50 host richiesti)**
* Subnet ideale: `/26` (offre fino a 62 host).
* **Network:** `192.168.100.128/26`
* **Broadcast:** `192.168.100.191`

3. **IT (20 host richiesti)**
* Subnet ideale: `/27` (offre fino a 30 host).
* **Network:** `192.168.100.192/27`
* **Broadcast:** `192.168.100.223`
</details>

---

## Esercizio 8 - Problema di connettività (Troubleshooting)

Un tecnico riceve questo ticket: *"Due PC non riescono a comunicare tra loro."*

| Dispositivo | IP | Subnet Mask |
| :--- | :--- | :--- |
| **PC-1** | `10.10.10.34` | `/27` |
| **PC-2** | `10.10.10.62` | `/27` |

### Domande

1. Calcola la rete di appartenenza di PC-1 e PC-2.
2. I dispositivi si trovano nella stessa subnet? 
3. L'errore di comunicazione potrebbe essere causato da una configurazione errata della Subnet Mask?

<details>
<summary>Soluzione</summary>

La subnet mask `/27` corrisponde a `255.255.255.224` (ultimo ottetto: `11100000`). Eseguiamo l'AND logico per i due IP.

**Rete PC-1:**
```text
34  = 00100010  AND
224 = 11100000
----------------
32  = 00100000  -> Rete PC-1: 10.10.10.32
```

**Rete PC-2:**
```text
62  = 00111110  AND
224 = 11100000
----------------
32  = 00100000  -> Rete PC-2: 10.10.10.32
```

* **Esito:** Generano esattamente lo stesso indirizzo di rete (`10.10.10.32`).
* **Risposta:** Si trovano nella stessa subnet. La subnet mask inserita **è corretta** per entrambi e non è la causa del problema. Il tecnico dovrà indagare su altre cause (es. Firewall attivo, problemi di cablaggio, porte disabilitate sullo switch).
</details>

---

# Challenge

## Esercizio 9 - Progettazione rete scolastica complessa

Sei il tecnico incaricato di progettare la rete per un istituto.
Rete disponibile: `192.168.0.0/23` (che include lo spazio da `192.168.0.0` a `192.168.1.255`).

| Reparto | Fabbisogno Host |
| :--- | :--- |
| **Wi-Fi Ospiti** | 100 |
| **Laboratorio** | 60 |
| **Segreteria** | 20 |
| **Aula Docenti** | 15 |

### Obiettivi

1. Realizzare il piano di indirizzamento utilizzando il **VLSM**.
2. Ridurre al minimo lo spreco di indirizzi.
3. Indicare quali porzioni di rete rimangono libere per le espansioni future.

<details>
<summary>Suggerimento</summary>
Ordina per dimensione decrescente. Ricorda che la rete assegnata è una <code>/23</code>, il che significa che hai a disposizione ben due blocchi di classe C interi (la <code>0.x</code> e la <code>1.x</code>).
</details>

<details>
<summary>Soluzione</summary>

Allocazione ordinata decrescente:

1. **Wi-Fi Ospiti (100 host)** -> Serve una `/25` (126 host)
* **Network:** `192.168.0.0/25`
* Broadcast: `192.168.0.127`

2. **Laboratorio (60 host)** -> Serve una `/26` (62 host)
* **Network:** `192.168.0.128/26`
* Broadcast: `192.168.0.191`

3. **Segreteria (20 host)** -> Serve una `/27` (30 host)
* **Network:** `192.168.0.192/27`
* Broadcast: `192.168.0.223`

4. **Aula Docenti (15 host)** -> Serve una `/27` (30 host) *Nota: una /28 darebbe solo 14 host, non sufficienti.*
* **Network:** `192.168.0.224/27`
* Broadcast: `192.168.0.255`

**Spazio Rimasto (Future Espansioni):**
L'intera progettazione ha consumato esattamente lo spazio del blocco `192.168.0.0/24`. 
L'intera metà superiore della nostra rete nativa, ovvero il blocco **`192.168.1.0/24`**, rimane completamente intatta e disponibile per future aule, laboratori o servizi!
</details>
