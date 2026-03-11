# Capitolo 1 — Fondamenti: come si rappresenta l’informazione

## 1.1 Un’idea semplice: la stessa informazione si può rappresentare in modi diversi

Per “codificare” intendiamo scegliere una regola per rappresentare un’informazione usando una sequenza di simboli presi da un insieme, che chiameremo **alfabeto**.

Esempi noti:

- **Codice Morse**: rappresenta lettere e numeri con sequenze di punti e linee (e pause).
- **DNA**: rappresenta informazione come stringhe su un alfabeto di 4 simboli (A, C, G, T).
- **Braille**: rappresenta caratteri tramite configurazioni di puntini in rilievo.

In informatica facciamo la stessa cosa, ma scegliendo rappresentazioni adatte a memorizzazione e trasmissione digitale.

## 1.2 Perché in informatica si usa spesso il binario

I dispositivi elettronici distinguono in modo affidabile due stati fisici. Per questo è comodo usare un alfabeto con due simboli, che indichiamo con 0 e 1.

Da qui derivano i concetti di **bit** e **byte**, che useremo in tutta la dispensa.

## 1.3 Alfabeto, simbolo, stringa

- **Alfabeto**: insieme finito di simboli distinti. Esempi: $\{0,1\}$, $\{A,B,\dots,Z\}$.
- **Simbolo**: un elemento dell’alfabeto (per esempio `1` nell’alfabeto binario).
- **Stringa**: sequenza finita di simboli di un alfabeto (per esempio `010011`).

Esempio: se l’alfabeto è $\{0,1\}$, allora `010011` è una stringa su quell’alfabeto.

## 1.4 Codifica, codice, parola di codice

- **Codifica (encoding)**: la regola che associa a ogni simbolo (o gruppo di simboli) della sorgente una stringa di simboli dell’alfabeto scelto per rappresentarlo.
- **Parola di codice (codeword)**: una singola stringa usata come rappresentazione.
- **Codice (code)**: l’insieme di tutte le parole di codice ammesse.

Esempio: voglio rappresentare 4 simboli $\{A,B,C,D\}$ usando bit.

- Codifica: $A \mapsto 00$, $B \mapsto 01$, $C \mapsto 10$, $D \mapsto 11$
- Parole di codice: `00`, `01`, `10`, `11`
- Codice: $\{00,01,10,11\}$


## 1.5 Lunghezza fissa e lunghezza variabile

Una codifica può assegnare parole tutte della stessa lunghezza oppure di lunghezze diverse.

### Codifica a lunghezza fissa

Tutte le parole di codice hanno la stessa lunghezza $L$. Questo permette di leggere e separare il messaggio a gruppi uguali di $L$ simboli, cioè “a blocchi”, perché ogni gruppo corrisponde a una parola di codice.

Esempio con lunghezza fissa 3 bit: rappresento 8 simboli $\{S_0,\dots,S_7\}$ con tutte le stringhe di 3 bit:

- $S_0 \mapsto 000$
- $S_1 \mapsto 001$
- …
- $S_7 \mapsto 111$


### Codifica a lunghezza variabile

Le parole di codice possono avere lunghezze diverse. Questo può ridurre la lunghezza totale del messaggio quando alcuni simboli compaiono molto spesso, perché a quei simboli si possono assegnare parole più corte, mentre ai simboli rari si assegnano parole più lunghe.

Esempio: $A \mapsto 0$, $B \mapsto 10$, $C \mapsto 110$, $D \mapsto 111$.

## 1.6 Quanti bit servono per rappresentare $C$ simboli

Con $n$ bit posso formare $2^n$ sequenze diverse.

Per rappresentare $C$ simboli distinti in binario serve un numero minimo di bit $n$ tale che $2^n \ge C$. In forma compatta si scrive $n = \lceil \log_2(C) \rceil$.

Esempio: per 10 simboli servono almeno 4 bit, perché $2^3 = 8$ non basta e $2^4 = 16$ basta.

## 1.7 Ridondanza

Consideriamo una codifica binaria a lunghezza fissa $n$ che rappresenta un alfabeto sorgente con $C$ simboli distinti.

Con $n$ bit esistono $2^n$ parole possibili; se la codifica usa solo $C$ di queste parole, allora:

- il codice è **non ridondante** quando $C = 2^n$;
- il codice è **ridondante** quando $C < 2^n$, perché esistono parole di lunghezza $n$ che non rappresentano nessun simbolo.

Un modo equivalente per dirlo è questo: una codifica binaria a lunghezza fissa è non ridondante quando usa il numero minimo di bit e usa tutte le combinazioni possibili; se invece usa più combinazioni di quante servono (lasciandone alcune inutilizzate), introduce ridondanza.

### Esempio di codice non ridondante

Rappresento 4 simboli con $n=2$ bit:

- $C = 4$ e $2^n = 2^2 = 4$
- uso tutte le parole: $\{00,01,10,11\}$

Qui il codice è non ridondante.

### Esempio di codice ridondante

Rappresento 10 simboli con $n=4$ bit:

- $C = 10$ e $2^n = 2^4 = 16$
- restano 6 parole che non rappresentano nessun simbolo

Qui il codice è ridondante.

La ridondanza non è sempre un difetto: spesso viene introdotta apposta perché rende più facile accorgersi degli errori e, in alcuni casi, anche correggerli.

## 1.8 Mini-glossario

- **Bit**: un simbolo dell’alfabeto $\{0,1\}$.
- **Byte**: gruppo di 8 bit.
- **Carattere**: unità di testo (lettera, cifra, segno…).


## 1.9 Esercizi

1) Hai un alfabeto di 26 simboli (per esempio le lettere A–Z). Qual è il numero minimo di bit $n$ per rappresentarli con una codifica a lunghezza fissa?
2) Considera una codifica binaria a lunghezza fissa di $n=3$ bit. Quanti simboli può rappresentare al massimo senza ridondanza? La codifica è ridondante se rappresenta solo 5 simboli?
3) Progetta una codifica a lunghezza fissa per 6 simboli usando il minimo numero di bit possibile. Elenca le parole che useresti e quante parole restano inutilizzate.
