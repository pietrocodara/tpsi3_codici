# Capitolo 3: Codici numerici in digitale

*Sul libro pagine 137-151.*

Quando un computer deve rappresentare numeri, non esiste un unico modo "giusto" di farlo.
A seconda dell'applicazione — un display, un encoder rotativo, un sistema di barcode industriale
— conviene usare codifiche diverse, ciascuna progettata per minimizzare errori, semplificare
i circuiti o accelerare i calcoli. Un **codice numerico** è la convenzione che stabilisce come
tradurre una cifra decimale (0–9) o un numero binario in una sequenza di bit.

---

## 3.1 Perché esistono tanti codici?

Il binario puro è ottimo per la matematica interna al processore, ma in molte situazioni crea
problemi pratici. Immagina un encoder rotativo meccanico: se più bit cambiano simultaneamente
durante una transizione (cosa che accade in binario puro passando da 3 = `011` a 4 = `100`),
un'imprecisione temporale può far leggere per un istante un valore completamente sbagliato.
Il codice Gray risolve proprio questo: garantisce che tra due valori consecutivi cambi
**sempre e solo un bit**, eliminando ambiguità nelle transizioni.

Analogamente, il BCD nasce per interfacciarsi con display a 7 segmenti: è molto più naturale
codificare separatamente le cifre `5` e `9` come `0101` e `1001` (invece di convertire 59 in
binario `111011`), perché ogni singolo nibble pilota direttamente un display indipendente.

---

## 3.2 Codice BCD (8421)

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
nei sistemi che lavorano con stringhe numeriche (es. EBCDIC).

---

## 3.3 Codice Aiken (2421)

Il codice Aiken usa i pesi **2-4-2-1** invece di 8-4-2-1. La scelta di questi pesi non è
casuale: permette di costruire un codice **autocomplementante**, cioè il complemento a 9 di
una cifra si ottiene semplicemente **invertendo tutti i bit**. Questo semplifica i
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

| Cifra | Aiken  | ↔ | Cifra | Aiken  | 
|:-----:|:------:|:-:|:-----:|:------:|
|   0   | `0000` | ↔ |   9   | `1111` | 
|   1   | `0001` | ↔ |   8   | `1110` | 
|   2   | `0010` | ↔ |   7   | `1101` | 
|   3   | `0011` | ↔ |   6   | `1100` | 
|   4   | `0100` | ↔ |   5   | `1011` | 

**Esempio di codifica:** il numero **36** in Aiken → 3 = `0011`, 6 = `1100` → `0011 1100`.

**Esempio di decodifica:** `1110 0001` → `1110` = 2+4+2+0 = 8, `0001` = 1 → **81**.

---

## 3.4 Codice Eccesso 3 (XS-3)

L'Eccesso-3 si costruisce in modo semplicissimo: si prende il valore decimale, **si aggiunge 3**,
e si codifica il risultato in binario a 4 bit. Quindi:

- 0 → 0+3 = 3 → `0011`
- 5 → 5+3 = 8 → `1000`
- 9 → 9+3 = 12 → `1100`

Anche Eccesso 3 è un codice **autocomplementante**:
il complemento a 9 si ottiene invertendo tutti i bit.

| Decimale |  0   |  1   |  2   |  3   |  4   |  5   |  6   |  7   |  8   |  9   |
|:--------:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
| XS-3     |`0011`|`0100`|`0101`|`0110`|`0111`|`1000`|`1001`|`1010`|`1011`|`1100`|

La tabella seguente mostra le coppie simmetriche:

| Cifra | XS-3   | ↔ | Cifra | XS-3   | 
|:-----:|:------:|:-:|:-----:|:------:|
|   0   | `0011` | ↔ |   9   | `1100` | 
|   1   | `0100` | ↔ |   8   | `1011` | 
|   2   | `0101` | ↔ |   7   | `1010` | 
|   3   | `0110` | ↔ |   6   | `1001` | 
|   4   | `0111` | ↔ |   5   | `1000` |

**Esempio di codifica:** **47** in XS-3 → 4+3=7 → `0111`, 7+3=10 → `1010` → `0111 1010`.

**Esempio di decodifica:** `0101 1001` → 5−3=**2**, 9−3=**6** → **26**.

---

## 3.5 Codice 2-su-5 pesato (63210)

Il codice 2 su 5 che vediamo usa 5 bit con pesi **6-3-2-1-0** e impone il vincolo che esattamente
**2 bit siano a `1`** in ogni parola valida. Questo codice è pesato e permette un semplice controllo d'errore, perché qualsiasi errore su un singolo bit porta il conteggio degli `1` a 1 o 3.

Con i pesi 6-3-2-1-0 non esiste alcuna coppia di bit la cui somma dia 0, quindi lo zero
non può rispettare il vincolo dato dai pesi: viene convenzionalmente codificato con la combinazione `00110` (2+1=3), l'unica rimasta disponibile dopo aver assegnato tutte le altre cifre. Per le cifre **1–9 invece la
somma dei pesi corrisponde esattamente al valore decimale**, rendendo la decodifica hardware
molto semplice.

| Decimale | Codice  | Pesi attivi (6·b₄ + 3·b₃ + 2·b₂ + 1·b₁ + 0·b₀) |
|:--------:|:-------:|:------------------------------------------------:|
|    0     | `00110` | 2 + 1                                            |
|    1     | `00011` | 1 + 0                                            |
|    2     | `00101` | 2 + 0                                            |
|    3     | `01001` | 3 + 0                                            |
|    4     | `01010` | 3 + 1                                            |
|    5     | `01100` | 3 + 2                                            |
|    6     | `10001` | 6 + 0                                            |
|    7     | `10010` | 6 + 1                                            |
|    8     | `10100` | 6 + 2                                            |
|    9     | `11000` | 6 + 3                                            |

**Esempio di codifica:** il numero **47** → 4 = `01010`, 7 = `10010` → `01010 10010`.

**Esempio di decodifica:** `01100` → 3+2 = **5**; `10010` → 6+1 = **7** → **57**.

**Esempio di rilevazione errore:** arriva `10110` — conta degli `1`: tre → **errore rilevato**.

---

## 3.6 Codice Gray

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
| 2 bit | `0`, `1` \| `1`,`0` → con prefissi: **`00`, `01`, `11`, `10`** |
| 3 bit | `00`,`01`,`11`,`10` \| `10`,`11`,`01`,`00` → **`000`,`001`,`011`,`010`,`110`,`111`,`101`,`100`** |
| 4 bit | Lista da 3 bit con prefisso `0`, poi riflessa con prefisso `1` (16 valori totali) |

Il risultato a 4 bit, confrontato al binario puro:

| Decimale | Gray   | Binario |
|:--------:|:------:|:-------:|
|    0     | `0000` | `0000`  |
|    1     | `0001` | `0001`  |
|    2     | `0011` | `0010`  |
|    3     | `0010` | `0011`  |
|    4     | `0110` | `0100`  |
|    5     | `0111` | `0101`  |
|    6     | `0101` | `0110`  |
|    7     | `0100` | `0111`  |
|    8     | `1100` | `1000`  |
|    9     | `1101` | `1001`  |
|   10     | `1111` | `1010`  |
|   11     | `1110` | `1011`  |
|   12     | `1010` | `1100`  |
|   13     | `1011` | `1101`  |
|   14     | `1001` | `1110`  |
|   15     | `1000` | `1111`  |

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

## 3.7 Codice Eccesso 3 Riflesso (XS-3)

L'Eccesso 3 riflesso è una variante dell'Eccesso-3 che garantisce che tra due cifre consecutive
cambi **sempre e solo 1 bit**, esattamente come nel codice Gray. Per questo motivo è detto
anche *Eccesso 3 Gray*.

### Come si costruisce

Avendo già a disposizione la tabella del codice Gray, la costruzione è immediata: si prendono
i codici Gray corrispondenti ai valori 3–12, ovvero i 10 valori che si ottengono aggiungendo 3
alle cifre 0–9. In altre parole:

1. Prendiamo la tabella del codice Gray a 4 bit (già vista nella sezione precedente).
2. Individuiamo le righe corrispondenti ai valori **3, 4, 5, 6, 7, 8, 9, 10, 11, 12**.
3. Assegnamo le codifiche rispettivamente alle cifre decimali **0, 1, 2, 3, 4, 5, 6, 7, 8, 9**.

Il codice Gray del valore (decimale + 3) diventa quindi il codice Eccesso 3 Riflesso di quella cifra.

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

### Altro modo di costruire Eccesso 3 Riflesso

Un altro modo per costruire il codice Eccesso 3 Riflesso è partire dal codice Eccesso 3.
Ho tutti i dati. Il meccanismo è chiaro: si prende la parola XS-3 a 4 bit e si applica XOR con se stessa shiftata di 1 a destra (esattamente la formula binario→Gray). Ecco il sottoparagrafo riscritto correttamente, con la tabella di verifica completa:

***

### Altro modo di costruire Eccesso 3 Riflesso

Un altro modo per costruire il codice Eccesso 3 Riflesso è partire dal codice Eccesso 3.
Si prende la parola di codice Eccesso 3 a 4 bit e si applica l'operazione **XOR tra la parola e se stessa shiftata di 1 posizione a destra**.

La tabella seguente mostra la derivazione per tutte le cifre:

| Decimale | XS-3   | XS-3 >> 1 | XS-3 Rifl. (XOR) |
|:--------:|:------:|:---------:|:------------------:|
|    0     | `0011` |   `0001`  |       `0010`       |
|    1     | `0100` |   `0010`  |       `0110`       |
|    2     | `0101` |   `0010`  |       `0111`       |
|    3     | `0110` |   `0011`  |       `0101`       |
|    4     | `0111` |   `0011`  |       `0100`       |
|    5     | `1000` |   `0100`  |       `1100`       |
|    6     | `1001` |   `0100`  |       `1101`       |
|    7     | `1010` |   `0101`  |       `1111`       |
|    8     | `1011` |   `0101`  |       `1110`       |
|    9     | `1100` |   `0110`  |       `1010`       |

I risultati coincidono esattamente con la tabella costruita tramite il codice Gray nella sezione precedente ✓. 


**Esempio di codifica:** il numero **28** in Eccesso 3 Riflesso:
- 2 → `0111`, 8 → `1110` → **`0111 1110`**

**Esempio di decodifica:** `0101 1100` → `0101` = 3, `1100` = 5 → **35**.

---

## 3.8 Codice 1 su n

Il codice **1 su n** (o *one-hot*) utilizza **n bit** per codificare **n simboli distinti**, con il vincolo che in ogni parola valida esattamente **1 bit sia a `1`** e tutti gli altri siano a `0`. Non si tratta di un codice pesato: ogni bit corrisponde direttamente a uno e un solo stato o simbolo.

Il vantaggio principale è la **semplicità del decodificatore hardware**: riconoscere quale simbolo è attivo significa semplicemente individuare quale linea è alta, senza alcuna logica combinatoria aggiuntiva. La verifica di errore è altrettanto immediata: qualsiasi parola con zero o più di un bit a `1` è invalida.

| Simbolo | Codice 1-su-4 |
|:-------:|:-------------:|
|    0    |    `0001`     |
|    1    |    `0010`     |
|    2    |    `0100`     |
|    3    |    `1000`     |

### Applicazioni tipiche

Il codice one-hot è usato in tutti i contesti dove ogni simbolo corrisponde a **una linea fisica dedicata**: decoder, display a segmenti multipli, tastiere a matrice, e sistemi di selezione canale dove ogni bit abilita direttamente un componente hardware senza necessità di logica intermedia.

**Esempio:** un sistema con 4 canali da selezionare usa 4 bit one-hot. Per attivare il canale 2: `0100`. Una parola come `0101` (due bit a `1`) è immediatamente riconoscibile come errore.

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
nelle reti (Internet è big endian per convenzione), e da molti formati di file
standard.

In **little endian** è invece il byte meno significativo (LSB, *Least Significant Byte*) a
occupare l'indirizzo più basso. Lo stesso numero `0x12345678` si troverebbe in memoria come
`78 56 34 12`. Può sembrare controintuitivo, ma ha un vantaggio pratico: leggendo
dall'indirizzo base si ottiene subito la parte meno significativa, il che semplifica alcune
operazioni aritmetiche su numeri di lunghezza variabile. È la convenzione adottata da Intel
x86/x64, ARM (in modalità predefinita) e dalla grande maggioranza dei processori moderni general-purpose.

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

Preferisco scrivere il codice direttamente — è sufficientemente chiaro da non richiedere esecuzione preventiva. Ecco il programma completo:

***

## 3.10 Programma Python sui codici numerici

Il programma seguente raccoglie in funzioni separate tutte le codifiche viste nel capitolo. Ogni funzione riceve una cifra decimale (0–9) e restituisce la stringa di bit corrispondente; il menu principale permette di esplorare i codici in modo interattivo.

```python
# ─────────────────────────────────────────────
#  Codici numerici – Capitolo 3
#  Ogni funzione accetta una cifra intera 0-9
#  e restituisce una stringa di bit es. "0101"
# ─────────────────────────────────────────────

def bcd(d):
    """BCD 8421: binario diretto su 4 bit."""
    return format(d, '04b')

def aiken(d):
    """Aiken 2421: pesi 2-4-2-1, autocomplementante."""
    tabella = {
        0: "0000", 1: "0001", 2: "0010", 3: "0011", 4: "0100",
        5: "1011", 6: "1100", 7: "1101", 8: "1110", 9: "1111"
    }
    return tabella[d]

def xs3(d):
    """Eccesso 3: si aggiunge 3 e si codifica in binario su 4 bit."""
    return format(d + 3, '04b')

def xs3_riflesso(d):
    """Eccesso 3 Riflesso: XOR tra XS-3 e XS-3 shiftato di 1 (= Gray del valore d+3)."""
    valore = d + 3
    gray = valore ^ (valore >> 1)
    return format(gray, '04b')

def due_su_cinque(d):
    """Codice 2-su-5 pesato 63210: esattamente 2 bit a 1."""
    tabella = {
        0: "00110", 1: "00011", 2: "00101", 3: "01001", 4: "01010",
        5: "01100", 6: "10001", 7: "10010", 8: "10100", 9: "11000"
    }
    return tabella[d]

def gray(d):
    """Codice Gray a 4 bit: binario XOR shiftato di 1."""
    return format(d ^ (d >> 1), '04b')

def one_hot(d):
    """Codice 1-su-10: bit in posizione d a 1, tutti gli altri a 0."""
    return format(1 << d, '010b')


# ─────────────────────────────────────────────
#  Stampa tutte le codifiche per una cifra
# ─────────────────────────────────────────────

def stampa_tutte(d):
    print(f"\n  Cifra decimale : {d}")
    print(f"  BCD 8421       : {bcd(d)}")
    print(f"  Aiken 2421     : {aiken(d)}")
    print(f"  Eccesso 3      : {xs3(d)}")
    print(f"  XS-3 Riflesso  : {xs3_riflesso(d)}")
    print(f"  2-su-5 (63210) : {due_su_cinque(d)}")
    print(f"  Gray           : {gray(d)}")
    print(f"  One-hot (1/10) : {one_hot(d)}")


# ─────────────────────────────────────────────
#  Menu principale
# ─────────────────────────────────────────────

def leggi_cifra():
    while True:
        s = input("Inserisci una cifra decimale (0-9): ")
        if s.isdigit() and 0 <= int(s) <= 9:
            return int(s)
        print("⚠ Valore non valido. Inserisci un numero tra 0 e 9.")

CODICI = {
    "1": ("BCD 8421",       bcd),
    "2": ("Aiken 2421",     aiken),
    "3": ("Eccesso 3",      xs3),
    "4": ("XS-3 Riflesso",  xs3_riflesso),
    "5": ("2-su-5 (63210)", due_su_cinque),
    "6": ("Gray",           gray),
    "7": ("One-hot",        one_hot),
}

def menu():
    cifra = leggi_cifra()

    while True:
        print()
        print(f"╔══════════════════════════════╗")
        print(f"║  Cifra corrente: {cifra}           ║")
        print(f"╠══════════════════════════════╣")
        print(f"║  0. Tutte le codifiche       ║")
        for k, (nome, _) in CODICI.items():
            print(f"║  {k}. {nome:<25}║")
        print(f"║  8. Cambia cifra             ║")
        print(f"║  9. Esci                     ║")
        print(f"╚══════════════════════════════╝")

        scelta = input("  Scelta: ").strip()

        if scelta == "0":
            stampa_tutte(cifra)
        elif scelta in CODICI:
            nome, funzione = CODICI[scelta]
            print(f"\n  {nome}: {funzione(cifra)}")
        elif scelta == "8":
            cifra = leggi_cifra()
        elif scelta == "9":
            print("\n  Arrivederci!\n")
            break
        else:
            print("  ⚠ Scelta non valida.")

if __name__ == "__main__":
    print("\n╔══════════════════════════════╗")
    print("║   Codici numerici – Cap. 3   ║")
    print("╚══════════════════════════════╝")
    menu()
```
Alcuni commenti sulle scelte progettuali:

- `xs3_riflesso` usa esattamente la formula $\text{XS-3} \oplus (\text{XS-3} \gg 1)$ appena vista nel 3.7, non una tabella hardcoded;
- `gray` usa la formula `n ^ (n >> 1)` già citata nel 3.6;
- `one_hot` usa 10 bit (1 per ogni cifra decimale);
- Aiken e 2-su-5 usano tabelle esplicite.