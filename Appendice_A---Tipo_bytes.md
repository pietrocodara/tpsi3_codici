# Il tipo `bytes` in Python

Il tipo `bytes` in Python 3 rappresenta una sequenza immutabile di valori interi compresi tra 0 e 255. Serve per lavorare con dati binari grezzi, come file, immagini, stream di rete, contenuti letti da socket o testo codificato in UTF-8.

***

## A cosa serve

Userai `bytes` quando lavori con:

- File binari, ad esempio immagini, PDF, archivi `.zip`.
- Dati ricevuti dalla rete, per esempio tramite socket o richieste HTTP.
- Conversioni tra testo e rappresentazione binaria.
- Protocolli, formati file e strutture dati a basso livello.

In pratica, `str` rappresenta testo leggibile, mentre `bytes` rappresenta dati binari.

***

## Come si crea

### Con un letterale

```python
b = b"ciao"
print(b)        # b'ciao'
print(type(b))  # <class 'bytes'>
```

### Con il costruttore `bytes()`

```python
b1 = bytes("ciao", encoding="utf-8")
b2 = bytes([65, 66, 67])
b3 = bytes(4)

print(b1)  # b'ciao'
print(b2)  # b'ABC'
print(b3)  # b'\x00\x00\x00\x00'
```

***

## Caratteristiche principali

- È immutabile: una volta creato, non puoi cambiare i suoi elementi.
- Ogni elemento è un numero intero tra 0 e 255.
- Supporta indicizzazione e slicing.
- Quando accedi a un singolo elemento, ottieni un intero.

```python
b = b"Python"

print(b[0])    # 80
print(b [realpython](https://realpython.com/python-markitdown/))    # 121
print(b[1:4])  # b'yth'
```

***

## Differenza tra `str` e `bytes`

```python
s = "ciao"
b = b"ciao"

print(type(s))  # <class 'str'>
print(type(b))  # <class 'bytes'>
```

- `str` contiene caratteri Unicode.
- `bytes` contiene valori binari.
- Non puoi mescolarli direttamente.

Questo, ad esempio, genera errore:

```python
"ciao" + b"ciao"
```

***

## Conversione tra stringhe e bytes

### Da stringa a bytes: `encode()`

```python
testo = "ciao"
dati = testo.encode("utf-8")

print(dati)         # b'ciao'
print(type(dati))   # <class 'bytes'>
```

### Da bytes a stringa: `decode()`

```python
dati = b"ciao"
testo = dati.decode("utf-8")

print(testo)        # ciao
print(type(testo))  # <class 'str'>
```


***

## Operazioni che puoi fare

### Concatenazione

```python
a = b"Hello"
b = b" World"

print(a + b)  # b'Hello World'
```

### Ripetizione

```python
print(b"ab" * 3)  # b'ababab'
```

### Controllo appartenenza

```python
b = b"Python"

print(b"Py" in b)   # True
print(b"zz" in b)   # False
```

### Lunghezza

```python
print(len(b"ciao"))  # 4
```

### Iterazione

```python
for x in b"ABC":
    print(x)
```

Output:

```python
65
66
67
```

***

## Metodi più utili

Molti metodi di `bytes` assomigliano a quelli delle stringhe, ma lavorano su dati binari.

### Ricerca e controllo

#### `find()`

```python
b = b"banana"
print(b.find(b"na"))  # 2
```

#### `startswith()` e `endswith()`

```python
b = b"document.pdf"
print(b.startswith(b"doc"))  # True
print(b.endswith(b".pdf"))   # True
```

***

### Divisione e unione

#### `split()`

```python
b = b"a,b,c"
print(b.split(b","))  # [b'a', b'b', b'c']
```

#### `join()`

```python
parti = [b"a", b"b", b"c"]
print(b"-".join(parti))  # b'a-b-c'
```

***

### Sostituzione

#### `replace()`

```python
b = b"ciao mondo"
print(b.replace(b"mondo", b"Python"))  # b'ciao Python'
```

***

### Rimozione spazi o caratteri

#### `strip()`, `lstrip()`, `rstrip()`

```python
b = b"  ciao  "
print(b.strip())   # b'ciao'
```

***

### Cambio maiuscole/minuscole

#### `upper()`, `lower()`, `capitalize()`, `title()`

```python
b = b"hello world"
print(b.upper())      # b'HELLO WORLD'
print(b.capitalize()) # b'Hello world'
```

Questi metodi sono pensati soprattutto per contenuti ASCII.

***

### Conversione esadecimale

#### `hex()`

```python
b = b"\xff\x10\x00"
print(b.hex())  # ff1000
```

#### `bytes.fromhex()`

```python
b = bytes.fromhex("ff1000")
print(b)  # b'\xff\x10\x00'
```

Questo è molto utile per ispezionare dati binari in formato leggibile.

***

## Conversione tra bytes e numeri

### Da bytes a intero

```python
b = b"\x01\x00"
n = int.from_bytes(b, byteorder="big")
print(n)  # 256
```

### Da intero a bytes

```python
n = 256
b = n.to_bytes(2, byteorder="big")
print(b)  # b'\x01\x00'
```

***

## Lettura di file binari

Per leggere un file binario devi usare la modalità `"rb"`:

```python
with open("immagine.jpg", "rb") as f:
    dati = f.read()

print(type(dati))  # <class 'bytes'>
```

Per scrivere bytes su file usi `"wb"`:

```python
with open("copia.bin", "wb") as f:
    f.write(b"\x00\x01\x02")
```

***

## Esempi pratici

### 1. Codificare una stringa prima di inviarla

```python
messaggio = "ciao".encode("utf-8")
```

### 2. Decodificare dati ricevuti

```python
risposta = b"ok"
print(risposta.decode("utf-8"))
```

### 3. Analizzare un header binario

```python
header = b"\x00\x10"
lunghezza = int.from_bytes(header, "big")
print(lunghezza)  # 16
```

### 4. Leggere un file a blocchi

```python
with open("video.mp4", "rb") as f:
    while chunk := f.read(1024):
        print(len(chunk))
```

***

## Limite importante: immutabilità

Questo non funziona:

```python
b = b"ciao"
b[0] = 65
```

Perché `bytes` è immutabile.

Se ti serve modificare il contenuto byte per byte, usa `bytearray`:

```python
ba = bytearray(b"ciao")
ba[0] = ord("C")

print(ba)  # bytearray(b'Ciao')
```

***

## `bytes` e `bytearray`

| Tipo | Mutabile | Uso tipico |
|---|---|---|
| `bytes` | No | Dati binari da leggere o passare così come sono |
| `bytearray` | Sì | Dati binari da modificare |

***

## Metodi principali di `bytes`

Ecco una panoramica rapida:

| Metodo | Cosa fa |
|---|---|
| `decode()` | Converte bytes in stringa |
| `find()` | Cerca una sottosequenza |
| `replace()` | Sostituisce bytes con altri bytes |
| `split()` | Divide i dati |
| `join()` | Unisce più sequenze |
| `startswith()` | Controlla il prefisso |
| `endswith()` | Controlla il suffisso |
| `strip()` | Rimuove byte iniziali/finali |
| `upper()` | Converte in maiuscolo |
| `lower()` | Converte in minuscolo |
| `hex()` | Restituisce la rappresentazione esadecimale |
| `fromhex()` | Crea bytes da una stringa esadecimale |

***

## Mini esempio completo

```python
testo = "Python"
dati = testo.encode("utf-8")

print(dati)              # b'Python'
print(dati[0])           # 80
print(dati[:3])          # b'Pyt'
print(dati.decode())     # Python
print(dati.hex())        # 507974686f6e
print(dati.replace(b"Py", b"My"))  # b'Mython'
```

***

## Quando usare `bytes`

Usa `bytes` quando:

- leggi o scrivi file binari;
- lavori con dati di rete;
- devi codificare o decodificare testo;
- analizzi formati binari;
- scambi dati con librerie o API che lavorano a basso livello.

Usa `str` quando lavori con testo umano leggibile.


# Encoding binari in Python: `codecs` e Base64

Oltre agli encoding testuali, in Python puoi usare anche trasformazioni per dati binari come Base64, Hex e alcune codifiche storiche o utilitarie. Queste servono a convertire dati `bytes` in formati più facili da trasportare, salvare o leggere.

***

## 1. Uso di `codecs`

Il modulo `codecs` permette di applicare alcune trasformazioni direttamente ai dati.

```python
import codecs

dati = b"ciao mondo"
```

### Base64

```python
codecs.encode(dati, "base64")
codecs.decode(b"Y2lhbyBtb25kbw==", "base64")
```

### Hex

```python
codecs.encode(dati, "hex")
codecs.decode(b"6369616f206d6f6e646f", "hex")
```

### Compressione con zlib

```python
codecs.encode(dati, "zlib")
codecs.decode(codecs.encode(dati, "zlib"), "zlib")
```

### Compressione con bz2

```python
codecs.encode(dati, "bz2")
codecs.decode(codecs.encode(dati, "bz2"), "bz2")
```

### ROT13

```python
codecs.encode("ciao", "rot_13")
```

### Quoted-Printable

```python
codecs.encode(dati, "quopri")
```

### UU encoding

```python
codecs.encode(dati, "uu_codec")
```

***

## 2. Uso di `base64`

Per Base64, il modulo `base64` è in genere più comodo e più chiaro da usare.

```python
import base64

dati = b"ciao mondo"
```

### Base64 classico

```python
base64.b64encode(dati)
base64.b64decode(b"Y2lhbyBtb25kbw==")
```

### Base16

```python
base64.b16encode(dati)
base64.b16decode(b"6369616F206D6F6E646F")
```

### Base32

```python
base64.b32encode(dati)
base64.b32decode(b"MNQXI2DJNZTSA5DFON2A====")
```

### URL-safe Base64

```python
base64.urlsafe_b64encode(dati)
base64.urlsafe_b64decode(base64.urlsafe_b64encode(dati))
```

***

## 3. Esempi rapidi completi

### Convertire bytes in Base64

```python
import base64

dati = b"ciao mondo"
codificati = base64.b64encode(dati)

print(codificati)  # b'Y2lhbyBtb25kbw=='
```

### Tornare da Base64 a bytes

```python
import base64

b64 = b"Y2lhbyBtb25kbw=="
originale = base64.b64decode(b64)

print(originale)  # b'ciao mondo'
```

### Convertire bytes in hex

```python
import codecs

dati = b"ABC"
hexdata = codecs.encode(dati, "hex")

print(hexdata)  # b'414243'
```

### Tornare da hex a bytes

```python
import codecs

hexdata = b"414243"
dati = codecs.decode(hexdata, "hex")

print(dati)  # b'ABC'
```

***

## 4. A cosa servono

- `base64`: per trasportare dati binari in testo ASCII, ad esempio in JSON, email, token, HTML.
- `hex`: per visualizzare o serializzare bytes in forma leggibile.
- `zlib` e `bz2`: per comprimere dati.
- `rot_13`: per trasformazioni testuali semplici.
- `quopri`: per contenuti email.
- `uu_codec`: formato storico, oggi poco usato.

***

## 5. Mini tabella utile

| Trasformazione | Modulo | Input | Output |
|---|---|---|---|
| Base64 | `base64` / `codecs` | `bytes` | `bytes` ASCII |
| Hex | `codecs` | `bytes` | `bytes` ASCII |
| Base16 | `base64` | `bytes` | `bytes` ASCII |
| Base32 | `base64` | `bytes` | `bytes` ASCII |
| URL-safe Base64 | `base64` | `bytes` | `bytes` ASCII |
| zlib | `codecs` | `bytes` | `bytes` compressi |
| bz2 | `codecs` | `bytes` | `bytes` compressi |

***

## 6. Nota importante

Base64 e Hex **non cifrano** i dati. Li trasformano soltanto in una rappresentazione diversa. Se qualcuno riceve quei dati, può decodificarli facilmente.