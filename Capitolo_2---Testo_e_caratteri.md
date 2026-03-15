# Capitolo 2 — Testo e caratteri: ASCII e UTF‑8

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

