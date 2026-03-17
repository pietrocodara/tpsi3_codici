# Capitolo 2 — Testo e caratteri: ASCII e UTF‑8

*Sul libro pagine 130-136.*

*Base 64 e URL encoding non sono sul libro.*

## 2.1 Testo e byte non sono la stessa cosa

Quando lavori con un computer devi distinguere due livelli.

- Il **testo** è fatto di caratteri. Un carattere è un concetto astratto: per esempio “A” è “A” indipendentemente da come lo memorizzi.
- In memoria e in rete, i dati sono organizzati in **byte**, cioè gruppi di 8 bit. Con 8 bit esistono 256 combinazioni possibili e, se le interpreti come valori numerici, corrispondono ai numeri da 0 a 255. Questo numero non è automaticamente il “significato” del byte: è solo un modo comodo per descriverlo; il significato dipende da come lo interpretiamo, per esempio come parte di un testo codificato in UTF‑8 oppure come parte di un numero.

Una codifica di caratteri è la regola che trasforma testo in byte e viceversa. In questo capitolo ci concentriamo su ASCII e UTF‑8.

In Python:

- `str` rappresenta testo
- `bytes` rappresenta una sequenza di byte

```python
s = "Ciao"
b = b"Ciao"

print(s, type(s))   # Ciao <class 'str'>
print(b, type(b))   # b'Ciao' <class 'bytes'>
```

Esempio importante: stesso byte, visualizzazioni e interpretazioni diverse.

```python
x = b"\x41"

print("x =", x, "type:", type(x))
# x = b'A' type: <class 'bytes'>

print("x[0] =", x[0], "type:", type(x[0]))
# x[0] = 65 type: <class 'int'>

print("list(x) =", list(x))
# list(x) = [65]

print("hex =", x.hex())
# hex = 41

print("ASCII =", x.decode("ascii"))
# ASCII = A
```


## 2.2 ASCII: l’idea essenziale e a cosa serve

ASCII è una codifica storica che associa numeri a caratteri di base come lettere inglesi, cifre, punteggiatura e alcuni caratteri di controllo. L’ASCII classico usa 7 bit, quindi può rappresentare 128 valori da 0 a 127.

Oggi ASCII compare spesso perché:

- molti protocolli e formati testuali si basano su caratteri semplici
- UTF‑8 mantiene i caratteri ASCII come caso particolare


### 2.2.1 Esempi con Python su ASCII

```python
c = "A"
n = ord(c)

print("c =", c, "type:", type(c))     # c = A type: <class 'str'>
print("n =", n, "type:", type(n))     # n = 65 type: <class 'int'>
print("chr(n) =", chr(n))             # chr(n) = A
```

```python
print(format(ord("A"), "07b"))        # 1000001
print(format(ord("A"), "08b"))        # 01000001
```

```python
s = "Ciao!"
b = s.encode("ascii")

print("b =", b, type(b))              # b = b'Ciao!' <class 'bytes'>
print(b.decode("ascii"))              # Ciao!
```

```python
"è".encode("ascii")  # UnicodeEncodeError
```


## 2.3 ASCII esteso e code page

ASCII copre solo 128 caratteri. Quando si è cercato di scrivere testi con lettere accentate o simboli aggiuntivi, sono nate varie estensioni a 8 bit, cioè con 256 valori possibili.

Il punto importante è questo: non esiste un unico “ASCII esteso” universale. In epoche e sistemi diversi, la stessa sequenza di byte tra 128 e 255 poteva rappresentare caratteri diversi a seconda della tabella usata. Queste tabelle vengono spesso chiamate code page o set di caratteri.

Conseguenza pratica: se un file di testo “vecchio” non dichiara la codifica, lo stesso byte può essere interpretato in modi diversi su computer diversi.

## 2.4 Unicode e code point

Unicode è un grande repertorio di caratteri. Per ogni carattere Unicode definisce un numero chiamato **code point**.

Un code point è quindi un identificatore numerico del carattere. Spesso viene scritto in esadecimale con la forma `U+....`.

Esempi di code point:

- `A` ha code point `U+0041`
- `è` ha code point `U+00E8`
- `€` ha code point `U+20AC`

Questi numeri non sono ancora byte “da salvare in un file”: dicono solo quale carattere intendiamo. Per ottenere i byte serve una codifica come UTF‑8.

Esempi con Python per vedere i code point:

```python
print(ord("A"))           # 65
print(hex(ord("A")))      # 0x41

print(ord("è"))           # 232
print(hex(ord("è")))      # 0xe8

print(ord("€"))           # 8364
print(hex(ord("€")))      # 0x20ac
```

Esempio inverso: da code point a carattere.

```python
print(chr(0x41))          # A
print(chr(0x00E8))        # è
print(chr(0x20AC))        # €
```


## 2.5 UTF‑8: cosa fa e perché è ovunque

UTF‑8 è una codifica che trasforma testo Unicode in byte. È molto usata perché rappresenta i caratteri ASCII con 1 byte identico ad ASCII, e rappresenta gli altri caratteri usando 2, 3 o 4 byte.

### 2.5.1 Come riconoscere la lunghezza guardando i byte

In UTF‑8, il primo byte indica la lunghezza della sequenza.

- `0xxxxxxx` → 1 byte
- `110xxxxx` → inizio sequenza di 2 byte
- `1110xxxx` → inizio sequenza di 3 byte
- `11110xxx` → inizio sequenza di 4 byte
- I byte successivi iniziano con `10xxxxxx`


### 2.5.2 Esempi con Python: vedere i byte reali

```python
def show_utf8(s: str):
    b = s.encode("utf-8")
    print("Testo:", s, "type:", type(s))
    print("Bytes:", b, "type:", type(b))
    print("len caratteri:", len(s), "len byte:", len(b))
    print("byte (int):", list(b))
    print("byte (hex):", [f"{x:02x}" for x in b])
    print()

show_utf8("A")
show_utf8("è")
show_utf8("€")
show_utf8("Ciao è €")
```


### 2.5.3 Caratteri e byte possono dare lunghezze diverse

Questo esempio mostra che `len` su una `str` conta i caratteri, mentre `len` sui `bytes` conta i byte, e le due cose non coincidono quando ci sono caratteri non ASCII.

```python
s = "è"
b = s.encode("utf-8")

print(len(s), type(s))   # 1 <class 'str'>
print(len(b), type(b))   # 2 <class 'bytes'>
```


## 2.6 Esercizi

1) Scegli 5 caratteri (almeno 2 non ASCII) e per ciascuno scrivi:
    - il code point in decimale con `ord`
    - il code point in esadecimale con `hex(ord(...))`
2) Per gli stessi caratteri, stampa i byte UTF‑8 in esadecimale con:
    - `list(c.encode("utf-8"))`
    - `[f"{x:02x}" for x in c.encode("utf-8")]`
3) Decodifica questi byte e spiega perché non sono ASCII.
```python
data = bytes([0xC3, 0xA8])
print(data.decode("utf-8"))
```


## 2.7 Strumenti Python da ricordare

- `type(x)` per capire se stai lavorando con `str`, `bytes`, `int`
- `ord(c)` per ottenere il code point del carattere
- `chr(n)` per ottenere il carattere dal code point
- `s.encode("utf-8")` per passare da testo a byte
- `b.decode("utf-8")` per passare da byte a testo
- `b.hex()` per vedere i byte in esadecimale
- `list(b)` per vedere i byte come numeri 0–255

## 2.8 Codifiche testuali avanzate

Abbiamo visto come ASCII e UTF-8 trasformino caratteri in byte (e viceversa). Ma spesso dobbiamo inviare questi byte su canali che accettano **solo testo puro**, come email, URL web o configurazioni JSON. Qui entrano in gioco le **codifiche testuali avanzate**: prendono sequenze di byte e le rappresentano come stringhe ASCII "sicure", pronte per la trasmissione.

Queste codifiche **non sono cifrature** (non nascondono i dati), ma semplice "rappresentazioni alternative". Sono essenziali per protocolli reali: embed di immagini in HTML, dati binari in API REST, o parametri speciali nei link.

### 2.8.1 Base64: da binario a testo ASCII sicuro

**Base64** è la più comune. Prende gruppi di **3 byte** (24 bit), li divide in **4 gruppi da 6 bit**, e mappa ogni gruppo su uno di **64 caratteri ASCII sicuri**: A-Z, a-z, 0-9, +, /. 

- Se i byte non sono multipli di 3, aggiunge **padding** con `=` alla fine.
- Ogni 3 byte diventano 4 caratteri: **aumento del 33%** in volume.
- Usa solo caratteri stampabili, perfetti per email (MIME), HTML o JSON.

**Esempio pratico con Python** (riprendendo `b"Ciao!"` dal paragrafo 2.5):

```python
import base64

dati = b"Ciao!"      # 5 byte (da UTF-8)
codifica = base64.b64encode(dati)
print(codifica)      # b'Q2lhbw=='
testo_base64 = codifica.decode('ascii')
print(testo_base64)  # Q2lhbw==
decod = base64.b64decode(testo_base64)
print(decod)         # b'Ciao!'
print(decod.decode('utf-8'))  # Ciao!
```

**Prova tu**: Codifica i byte di `€` (UTF-8: `b'\xe2\x82\xac'`). Risultato? `4q2D` (no padding).

**Variante**: `base64.urlsafe_b64encode()` sostituisce `+` con `-` e `/` con `_` per URL sicuri.

### 2.8.2 URL Encoding (Percent-Encoding): caratteri speciali nei link

Gli URL non tollerano spazi, &, ?, ò o caratteri non-ASCII: li **rompono**. **URL encoding** (o percent-encoding) li trasforma in **%HH**, dove HH sono due cifre esadecimali (00-FF).

- Spazio → `%20`
- `?` → `%3F`
- ò → `%C3%B2` (i suoi 2 byte UTF-8)
- È **standard RFC 3986**, usata in query string (`?nome=citt%C3%A0`), path o form HTML.

**Esempio pratico con Python**:

```python
import urllib.parse

testo = "città? & ò"
cod = urllib.parse.quote(testo)           # Encode
print(cod)     # citt%C3%A0%3F%20%26%20%C3%B2
dec = urllib.parse.unquote(cod)           # Decode
print(dec)     # città? & ò

# Per soli parametri query (non path)
print(urllib.parse.quote_plus("a b"))     # a+b (spazio → +)
```

**URL completo**:
```
https://esempio.com/cerca?query=citt%C3%A0&filtro=alta%20qualit%C3%A0
```
Senza encoding, il browser interpreta `&filtro` come parametro separato!

### 2.8.3 Altre codifiche utili per testo

- **Quoted-Printable** (per email MIME): Caratteri non-stampa → `=HH` (es. `à` → `=C3=A0`). Simile a URL, ma con `=` invece di `%`.
  
```python
import quopri
b = b"ciao\xc3\xa0"  # ò in UTF-8? No, à
qp = quopri.encodestring(b)
print(qp.decode())  # Y2lhbz0K (righe con = terminano)
```

- **Hex (già visto)**: `b.hex()` dà byte come stringa esadecimale (es. `b"A"` → `"41"`). Utile per debug.

### 2.8.4 Confronto rapido

| Codifica | Quando usarla | Esempio: "à" (UTF-8: C3 A0) | Aumento dimensione |
|----------|---------------|------------------------------|-------------------|
| Base64  | Binario in email/JSON | `w6E=` | +33% | 
| URL     | Parametri web | `%C3%A0` | +200% (per char) |
| Hex     | Debug | `c3a0` | +100% |
| Quoted-Printable | Email testuale | `=C3=A0` | Minimo |

**Nota**: Per dati sensibili, usa **crittografia** (capitoli futuri), non queste!

## 2.9 Esercizi aggiuntivi

1. Codifica `"Hello ò€"` in Base64 e URL. Confronta le stringhe resultanti.
2. Crea un URL con `query="spazio & accenti"`. Quali %HH vedi?
3. Prendi byte casuali (`os.urandom(10)`), codifica in Base64, decodifica: tornano uguali?