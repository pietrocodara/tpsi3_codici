# Capitolo 3: Codici numerici in digitale

Quando un computer deve rappresentare numeri, non esiste un unico modo "giusto" di farlo.
A seconda dell'applicazione — un display, un encoder rotativo, un sistema di barcode industriale
— conviene usare codifiche diverse, ciascuna progettata per minimizzare errori, semplificare
i circuiti o accelerare i calcoli. Un **codice numerico** è la convenzione che stabilisce come
tradurre una cifra decimale (0–9) o un numero binario in una sequenza di bit.

---

## 3.1 Perché esistono tanti codici?

Il binario puro è ottimo per la matematica interna al processore, ma in molte situazioni crea
problemi pratici. Immagina un encoder rotativo meccanico: se due bit cambiano simultaneamente
durante una transizione (cosa che accade in binario puro passando da 3 = `011` a 4 = `100`),
un'imprecisione temporale può far leggere per un istante un valore completamente sbagliato.
Il codice Gray risolve proprio questo: garantisce che tra due valori consecutivi cambi
**sempre e solo un bit**, eliminando ambiguità nelle transizioni.

Analogamente, il BCD nasce per interfacciarsi con display a 7 segmenti: è molto più naturale
codificare separatamente le cifre `5` e `9` come `0101` e `1001` (invece di convertire 59 in
binario `111011`), perché ogni singolo nibble pilota direttamente un display indipendente.

---

## 3.2 Codice BCD 8421

Il BCD (Binary Coded Decimal) è il codice decimale più diffuso. Ogni cifra decimale viene
codificata con 4 bit usando i pesi **8-4-2-1**, ovvero le normali potenze del 2. Solo le
combinazioni da `0000` a `1001` sono valide: le restanti 6 (`1010`–`1111`) sono **codici
proibiti** e, se compaiono, indicano un errore.

La decodifica è immediata: `0110` vale 0·8 + 1·4 + 1·2 + 0·1 = **6**.

| Decimale |  0   |  1   |  2   |  3   |  4   |  5   |  6   |  7   |  8   |  9   |
|:--------:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
| BCD 8421 |`0000`|`0001`|`0010`|`0011`|`0100`|`0101`|`0110`|`0111`|`1000`|`1001`|

Per codificare un numero multi-cifra come **59**, si codifica ogni cifra separatamente:

> 5 → `0101`, 9 → `1001`  ⟹  **59 in BCD = `0101 1001`**

Per decodificare `0011 0111`: si spezza in nibble → `0011` = 3, `0111` = 7 → **37**.

Il BCD viene usato nei PLC industriali, nelle calcolatrici e ovunque l'interfaccia utente
lavori in decimale.

### BCD Packed e Unpacked

Quando si memorizza un numero BCD multi-cifra in memoria, esistono due convenzioni principali
su come disporre i nibble all'interno dei byte.

**BCD Packed** (compatto): si inseriscono **due cifre BCD per byte**, una nel nibble alto
(bit 7–4) e una nel nibble basso (bit 3–0). È il formato più efficiente in termini di spazio.

> Esempio: il numero **59** occupa **1 solo byte** → `0101 1001` (5 nel nibble alto, 9 in
> quello basso).
> Il numero **273** occupa **2 byte** → `0000 0010` `0111 0011`
> (il primo byte contiene solo 2 nel nibble basso, con il nibble alto a zero; il secondo
> contiene 7 e 3).

**BCD Unpacked** (non compatto): ogni cifra BCD occupa un **byte intero**, con i 4 bit utili
nel nibble basso e il nibble alto posto a zero (oppure, in alcuni sistemi come ASCII/EBCDIC,
riempito con una zona fissa tipo `0011` per ASCII).

> Esempio: il numero **59** occupa **2 byte** → `0000 0101` `0000 1001`

| Formato      | Byte usati per "59" | Nibble alto | Efficienza |
|:-------------|:-------------------:|:-----------:|:----------:|
| BCD Packed   | 1                   | Cifra BCD   | Alta       |
| BCD Unpacked | 2                   | `0000`      | Bassa      |

Il formato **packed** è preferito nei sistemi embedded e nei PLC dove la memoria è limitata.
Il formato **unpacked** semplifica la manipolazione software cifra per cifra ed è più comune
nei sistemi che lavorano con stringhe numeriche (es. COBOL, EBCDIC).

---

## 3.3 Codice Aiken (2421)

Il codice Aiken usa i pesi **2-4-2-1** invece di 8-4-2-1. La scelta di questi pesi non è
casuale: permette di costruire un codice **autocomplementante**, cioè il complemento a 9 di
una cifra si ottiene semplicemente **invertendo tutti i bit**. Questo semplifica molto i
circuiti per la sottrazione decimale, perché non serve una logica separata per calcolare
il complemento.

La proprietà di autocomplementazione impone che per i valori 5–9 si scelga la codifica "alta"
(che inizia con `1`), anche quando esisterebbe una codifica più bassa con gli stessi pesi.
Ad esempio, il 5 potrebbe essere `0101` (0+4+0+1=5), ma il suo NOT sarebbe `1010`=7, rompendo
la simmetria. Si usa invece `1011` (2+0+2+1=5), il cui NOT è `0100`=4 ✓.

| Decimale |  0   |  1   |  2   |  3   |  4   |  5   |  6   |  7   |  8   |  9   |
|:--------:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
| Aiken    |`0000`|`0001`|`0010`|`0011`|`0100`|`1011`|`1100`|`1101`|`1110`|`1111`|

La tabella seguente mostra la proprietà di autocomplementazione: ogni coppia
(cifra, complemento a 9) è affiancata per evidenziare che i rispettivi codici sono NOT
bit-a-bit l'uno dell'altro.

| Cifra | Aiken  | ↔ | Cifra | Aiken  | NOT coincide? |
|:-----:|:------:|:-:|:-----:|:------:|:-------------:|
|   0   | `0000` | ↔ |   9   | `1111` | ✓             |
|   1   | `0001` | ↔ |   8   | `1110` | ✓             |
|   2   | `0010` | ↔ |   7   | `1101` | ✓             |
|   3   | `0011` | ↔ |   6   | `1100` | ✓             |
|   4   | `0100` | ↔ |   5   | `1011` | ✓             |

**Esempio di codifica:** il numero **36** in Aiken → 3 = `0011`, 6 = `1100` → `0011 1100`.

**Esempio di decodifica:** `1110 0001` → `1110` = 2+4+2+0 = 8, `0001` = 1 → **81**.

---

## 3.4 Codice XS-3 (Eccesso 3)

L'Eccesso-3 si costruisce in modo semplicissimo: si prende il valore decimale, **si aggiunge 3**,
e si codifica il risultato in binario a 4 bit. Quindi:

- 0 → 0+3 = 3 → `0011`
- 5 → 5+3 = 8 → `1000`
- 9 → 9+3 = 12 → `1100`

Proprio perché è BCD spostato di 3, è anch'esso un codice **autocomplementante**:
il complemento a 9 si ottiene invertendo tutti i bit.

| Decimale |  0   |  1   |  2   |  3   |  4   |  5   |  6   |  7   |  8   |  9   |
|:--------:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
| XS-3     |`0011`|`0100`|`0101`|`0110`|`0111`|`1000`|`1001`|`1010`|`1011`|`1100`|

La tabella seguente mostra le coppie simmetriche:

| Cifra | XS-3   | ↔ | Cifra | XS-3   | NOT coincide? |
|:-----:|:------:|:-:|:-----:|:------:|:-------------:|
|   0   | `0011` | ↔ |   9   | `1100` | ✓             |
|   1   | `0100` | ↔ |   8   | `1011` | ✓             |
|   2   | `0101` | ↔ |   7   | `1010` | ✓             |
|   3   | `0110` | ↔ |   6   | `1001` | ✓             |
|   4   | `0111` | ↔ |   5   | `1000` | ✓             |

**Esempio di codifica:** **47** in XS-3 → 4+3=7 → `0111`, 7+3=10 → `1010` → `0111 1010`.

**Esempio di decodifica:** `0101 1001` → 5−3=**2**, 9−3=**6** → **26**.

---

## 3.5 Codice 63210 (2-su-5 pesato)

Il codice 63210 usa 5 bit con pesi **6-3-2-1-0** e impone il vincolo che esattamente
**2 bit siano a `1`** in ogni parola valida. Questa combinazione lo rende unico: è al tempo
stesso **pesato** (il valore decimale si ottiene sommando i pesi delle due posizioni a `1`
per le cifre 1–9) e **self-checking** (qualsiasi errore su un singolo bit porta il conteggio
degli `1` a 1 o 3, immediatamente riconoscibile).

Con i pesi 6-3-2-1-0 non esiste alcuna coppia di bit la cui somma dia 0, quindi lo zero è
l'unico caso convenzionale: viene codificato con la combinazione `00110` (2+1=3), l'unica
rimasta disponibile dopo aver assegnato tutte le altre cifre. Per le cifre **1–9 invece la
somma dei pesi corrisponde esattamente al valore decimale**, rendendo la decodifica hardware
molto semplice.

| Decimale | Codice  | Pesi attivi (6·b₄ + 3·b₃ + 2·b₂ + 1·b₁ + 0·b₀) | Somma |
|:--------:|:-------:|:------------------------------------------------:|:-----:|
|    0     | `00110` | 2 + 1                                            | 3 (conv.) |
|    1     | `00011` | 1 + 0                                            | 1 ✓  |
|    2     | `00101` | 2 + 0                                            | 2 ✓  |
|    3     | `01001` | 3 + 0                                            | 3 ✓  |
|    4     | `01010` | 3 + 1                                            | 4 ✓  |
|    5     | `01100` | 3 + 2                                            | 5 ✓  |
|    6     | `10001` | 6 + 0                                            | 6 ✓  |
|    7     | `10010` | 6 + 1                                            | 7 ✓  |
|    8     | `10100` | 6 + 2                                            | 8 ✓  |
|    9     | `11000` | 6 + 3                                            | 9 ✓  |

**Esempio di codifica:** il numero **47** → 4 = `01010`, 7 = `10010` → `01010 10010`.

**Esempio di decodifica:** `01100` → 3+2 = **5**; `10010` → 6+1 = **7** → **57**.

**Esempio di rilevazione errore:** arriva `10110` — conta degli `1`: tre → **errore rilevato**.

---

## 3.6 Codice Gray: costruzione e conversioni

Il codice Gray è il più elegante dal punto di vista logico. La sua proprietà fondamentale è
che tra due valori **consecutivi** cambia sempre e solo **1 bit**. Questo elimina le ambiguità
di lettura negli encoder rotativi, nei potenziometri digitali e in qualsiasi sistema fisico
dove le transizioni non sono mai perfettamente istantanee.

### Come si costruisce il codice Gray

La costruzione è **riflessiva** e si può eseguire a mano per qualsiasi numero di bit:

1. Con **1 bit**: si parte da `0`, `1`.
2. Con **n bit**: si prende la lista a n−1 bit, la si copia al contrario (riflessa), si
   aggiunge il prefisso `0` ai valori originali e il prefisso `1` ai valori rispecchiati.

Passo per passo fino a 4 bit:

| Passo | Lista risultante |
|:-----:|:-----------------|
| 1 bit | `0`, `1` |
| 2 bit | `00`, `01` \| riflesso `01`,`00` → con prefissi: **`00`, `01`, `11`, `10`** |
| 3 bit | `000`,`001`,`011`,`010` \| riflesso → **`000`,`001`,`011`,`010`,`110`,`111`,`101`,`100`** |
| 4 bit | Lista da 3 bit con prefisso `0`, poi riflessa con prefisso `1` (16 valori totali) |

Il risultato a 4 bit, confrontato al binario puro:

| Decimale | Binario | Gray   | Bit cambiati dalla riga precedente |
|:--------:|:-------:|:------:|:----------------------------------:|
|    0     | `0000`  | `0000` | —                                  |
|    1     | `0001`  | `0001` | bit 0                              |
|    2     | `0010`  | `0011` | bit 1                              |
|    3     | `0011`  | `0010` | bit 0                              |
|    4     | `0100`  | `0110` | bit 2                              |
|    5     | `0101`  | `0111` | bit 0                              |
|    6     | `0110`  | `0101` | bit 1                              |
|    7     | `0111`  | `0100` | bit 0                              |
|    8     | `1000`  | `1100` | bit 3                              |
|    9     | `1001`  | `1101` | bit 0                              |

Nel binario puro il passaggio 7→8 cambia **4 bit** contemporaneamente (`0111`→`1000`);
in Gray cambia solo il bit 3 (`0100`→`1100`).

### Conversione binario → Gray

Dato un numero binario $B = b_{n-1}b_{n-2}\ldots b_1 b_0$, il codice Gray
$G = g_{n-1}g_{n-2}\ldots g_1 g_0$ si calcola così:

- Il bit più significativo rimane uguale: $g_{n-1} = b_{n-1}$
- Ogni altro bit è lo XOR tra il bit corrente e quello immediatamente superiore:
  $g_i = b_{i+1} \oplus b_i$

**Esempio — converti 6 (= `0110`) in Gray:**

| Bit | Calcolo             | Risultato |
|:---:|:-------------------:|:---------:|
| g₃  | = b₃ = 0            | `0`       |
| g₂  | = b₃ ⊕ b₂ = 0 ⊕ 1  | `1`       |
| g₁  | = b₂ ⊕ b₁ = 1 ⊕ 1  | `0`       |
| g₀  | = b₁ ⊕ b₀ = 1 ⊕ 0  | `1`       |

**Risultato: 6 in Gray = `0101`** ✓

### Conversione Gray → binario

Il processo inverso è a cascata: il bit più significativo è uguale, poi ogni bit binario
si ottiene facendo XOR tra il bit di Gray corrente e il bit binario **già calcolato** alla
posizione superiore:

- $b_{n-1} = g_{n-1}$
- $b_i = g_i \oplus b_{i+1}$

**Esempio — decodifica il codice Gray `1101`:**

| Bit | Calcolo             | Risultato |
|:---:|:-------------------:|:---------:|
| b₃  | = g₃ = 1            | `1`       |
| b₂  | = g₂ ⊕ b₃ = 1 ⊕ 1  | `0`       |
| b₁  | = g₁ ⊕ b₂ = 0 ⊕ 0  | `0`       |
| b₀  | = g₀ ⊕ b₁ = 1 ⊕ 0  | `1`       |

**Risultato: `1001` = 9** ✓

In Python, la conversione binario→Gray si ottiene con una sola riga:
```python
gray = n ^ (n >> 1)
```

---

## 3.7 Codice XS-3 Riflesso (Eccesso 3 Gray)

L'XS-3 riflesso è una variante dell'Eccesso-3 che combina due proprietà in una sola: è
**autocomplementante** come l'XS-3 classico, ma in più garantisce che tra due cifre consecutive
cambi **sempre e solo 1 bit**, esattamente come il codice Gray. Per questo motivo è detto
anche *XS-3 Gray* o *Excess-3 Gray*.

### Come si costruisce

Avendo già a disposizione la tabella del codice Gray, la costruzione è immediata: si prendono
i codici Gray corrispondenti ai valori 3–12, ovvero i 10 valori che si ottengono aggiungendo 3
alle cifre 0–9, esattamente come nell'XS-3 classico. In altre parole:

1. Prendi la tabella del codice Gray a 4 bit (già vista nella sezione precedente).
2. Individua le righe corrispondenti ai valori **3, 4, 5, 6, 7, 8, 9, 10, 11, 12**.
3. Assegnale rispettivamente alle cifre decimali **0, 1, 2, 3, 4, 5, 6, 7, 8, 9**.

Il codice Gray del valore (decimale + 3) diventa il codice XS-3 riflesso di quella cifra.

| Decimale | Decimale + 3 | Gray di (dec+3) | XS-3 Riflesso |
|:--------:|:------------:|:---------------:|:-------------:|
|    0     |      3       |     `0010`      |    `0010`     |
|    1     |      4       |     `0110`      |    `0110`     |
|    2     |      5       |     `0111`      |    `0111`     |
|    3     |      6       |     `0101`      |    `0101`     |
|    4     |      7       |     `0100`      |    `0100`     |
|    5     |      8       |     `1100`      |    `1100`     |
|    6     |      9       |     `1101`      |    `1101`     |
|    7     |     10       |     `1111`      |    `1111`     |
|    8     |     11       |     `1110`      |    `1110`     |
|    9     |     12       |     `1010`      |    `1010`     |

### Proprietà di autocomplementazione

Come nell'XS-3 classico e in Aiken, il complemento a 9 di una cifra si ottiene invertendo
tutti i bit. Le coppie simmetriche:

| Cifra | XS-3 Riflesso | ↔ | Cifra | XS-3 Riflesso | NOT coincide? |
|:-----:|:-------------:|:-:|:-----:|:-------------:|:-------------:|
|   0   |    `0010`     | ↔ |   9   |    `1101`     | ✓             |
|   1   |    `0110`     | ↔ |   8   |    `1001`     | ✓             |
|   2   |    `0111`     | ↔ |   7   |    `1000`     | ✓             |
|   3   |    `0101`     | ↔ |   6   |    `1010`     | ✓             |
|   4   |    `0100`     | ↔ |   5   |    `1011`     | ✓             |

> **Nota:** nel passaggio da 4 a 5 cambia **1 solo bit** (proprietà Gray), ma il codice
> salta dall'intervallo basso (`0100`) a quello alto (`1100`). Questo "salto" è strutturale
> ed è la stessa rottura di simmetria visibile in Aiken: è il prezzo da pagare per avere
> contemporaneamente la proprietà Gray e quella di autocomplementazione.

**Esempio di codifica:** il numero **28** in XS-3 riflesso:
- 2 → `0111`, 8 → `1110` → **`0111 1110`**

**Esempio di decodifica:** `0101 1100` → `0101` = 3, `1100` = 5 → **35**.

---

## 3.9 Ordine dei byte: Big Endian e Little Endian

Quando un numero occupa più di un byte in memoria — come un intero a 32 bit o un numero BCD
packed con più cifre — sorge una domanda pratica: **in quale ordine si dispongono i byte?**
Il byte più significativo (quello con il peso maggiore) va all'indirizzo più basso o più alto?
Le due convenzioni opposte si chiamano **big endian** e **little endian**.

In **big endian** il byte più significativo (MSB, *Most Significant Byte*) viene scritto
per primo, all'indirizzo di memoria più basso. È la convenzione "naturale" per chi legge da
sinistra a destra: il numero `0x12345678` si trova in memoria esattamente come lo si scriverebbe
su carta — `12 34 56 78` agli indirizzi consecutivi. È usata dalle architetture Motorola 68k,
dai processori di rete (Internet è big endian per convenzione), e da molti formati di file
standard.

In **little endian** è invece il byte meno significativo (LSB, *Least Significant Byte*) a
occupare l'indirizzo più basso. Lo stesso numero `0x12345678` si troverebbe in memoria come
`78 56 34 12`. Può sembrare controintuitivo, ma ha un vantaggio pratico: leggendo
dall'indirizzo base si ottiene subito la parte meno significativa, il che semplifica alcune
operazioni aritmetiche su numeri di lunghezza variabile. È la convenzione adottata da Intel
x86/x64, ARM (in modalità predefinita) e dalla grande maggioranza dei processori moderni per
uso general-purpose.

| Convenzione   | Byte all'indirizzo più basso | Esempio: `0x12345678` in memoria | Usata da                       |
|:--------------|:----------------------------:|:--------------------------------:|:-------------------------------|
| Big Endian    | Più significativo (MSB)      | `12` `34` `56` `78`              | Motorola, reti TCP/IP, PowerPC |
| Little Endian | Meno significativo (LSB)     | `78` `56` `34` `12`              | Intel x86/x64, ARM (default)   |

Il problema diventa concreto ogni volta che si scambiano dati tra sistemi diversi: un file
scritto da un sistema big endian e letto da uno little endian produrrà valori completamente
errati se non si applica una conversione esplicita. Per questo i protocolli di rete definiscono
un **network byte order** (big endian) come standard, e le librerie di sistema forniscono
funzioni come `htons()` / `ntohl()` per convertire tra l'ordine dell'host e quello di rete.
Nel contesto dei codici BCD packed, l'endianness determina in quale byte si trovano le cifre
più significative di un numero multi-byte: una questione tutt'altro che trascurabile nei
sistemi industriali e nei PLC che comunicano tra architetture eterogenee.

---

## 3.10 Riepilogo

| Codice          | Bit | Come si costruisce                              | Proprietà chiave                               | Applicazione tipica                     |
|:----------------|:---:|:------------------------------------------------|:-----------------------------------------------|:----------------------------------------|
| BCD 8421        | 4   | Binario diretto, solo 0–9 validi                | Pesato standard; packed/unpacked               | Display 7 seg., PLC, calcolatrici       |
| Aiken 2421      | 4   | Pesato 2-4-2-1, codifica alta per 5–9           | Autocomplementante                             | Didattica, calcolatori storici          |
| XS-3            | 4   | BCD + 3 in binario                              | Autocomplementante                             | Aritmetica BCD hardware                 |
| 63210           | 5   | Pesi 6-3-2-1-0, esattamente 2 bit a `1`         | Pesato (1–9) + self-checking                   | Barcode, sistemi industriali            |
| Gray            | n   | Costruzione riflessiva, XOR tra bit adiacenti   | 1 bit cambia per transizione                   | Encoder rotativi, CNC, robotica         |
| XS-3 Riflesso   | 4   | Gray del valore (dec+3)                         | 1 bit per transizione     | Sistemi misti display/encoder           |