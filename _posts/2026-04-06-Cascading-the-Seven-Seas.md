---
layout: post
title: Cascading the Seven Seas RITSEC CTF
comments: true
categories: [Web, Writeups]
---

Llevaba tiempo sin subir nada al blog y me pareció que este reto del [RITSEC CTF](https://ctfd.ritsec.club/) merecía la pena tener un writeup.

# Overview

Es un reto de web, que mucha gente sacó de primeras (imagino que la IA no tiene mucho problema para resolverlo). El caso es que el autor ([sy1vi3](https://x.com/rebane2001)), se ha programado un procesador x86 completo en CSS, y se nos carga la página con un programa que implementa un quiz.

<br>
![Image]({{ site.baseurl }}/images/posts/2026-04-06-Cascading-the-Seven-Seas-001.png){:width="600px"}


# Explotación

Dado que el programa implementa un emulador de x86, podemos interpretar los bytes hardcodeados como tales:

```css
@property --m0 {
  syntax: "<integer>";
  initial-value: 204;
  inherits: true;
}
@property --m1 {
  syntax: "<integer>";
  initial-value: 144;
  inherits: true;
}
@property --m2 {
  syntax: "<integer>";
  initial-value: 144;
  inherits: true;
}
[...]
@property --m256 {
  syntax: "<integer>";
  initial-value: 86;
  inherits: true;
}
@property --m257 {
  syntax: "<integer>";
  initial-value: 85;
  inherits: true;
}
@property --m258 {
  syntax: "<integer>";
  initial-value: 137;
  inherits: true;
}
@property --m259 {
  syntax: "<integer>";
  initial-value: 229;
  inherits: true;
}
```

Esto nos deja un flujo como:

- En `0x301` (`769`) inicializa varios punteros de funciones/estado. Podemos verlo ya que IP se inicializa ahí:

```css
@property --IP {
  syntax: "<integer>";
  initial-value: 769;
  inherits: true;
}
```

- Entra al main en 0x248;
- Imprime el banner y las preguntas;
- Lee una respuesta;
- Comprueba longitud;
- Valida con una tabla;
- Repite para las 3 preguntas.

Para simplificar el análisis podemos convertir la memoria a un archivo .bin y analizarlo como un binario x86. Podemos hacerlo con un script en python:

```python
import re

html = open("full.html", "r", encoding="utf-8").read()

pairs = re.findall(
    r'@property\s+--m(\d+)\s*\{[^}]*?initial-value:\s*(\d+);',
    html,
    re.S
)

mem = {int(idx): int(val) for idx, val in pairs}
size = max(mem) + 1

data = bytes(mem.get(i, 0) for i in range(size))

with open("cssvm.bin", "wb") as f:
    f.write(data)

print(f"extraídos {len(mem)} bytes definidos")
print(f"binario escrito en cssvm.bin con tamaño {len(data)} bytes")
```

Y podemos decompilarlo rápidamente con:

```bash
objdump -D -Mintel -b binary -m i8086 cssvm.bin | less
```


Las cadenas en memoria y las longitudes que exige el programa para las tres preguntas son:

1. `Which ocean is the largest?` - 7
2. `Name an aquatic mammal` - 5
3. `What's the flag?` - 32

Para validar las preguntas se itera una tabla de entradas de 8 bytes. Cada entrada son 4 words de 16 bits:

```
indice a
indice b
indice c
valor_objetivo t
```

La condición que se comprueba es:

```
respuesta[a] ^ (respuesta[b] + respuesta[c]) == t
```

Con las tablas ya extraídas, resolverlo es bastante directo. En mi caso tiré de un pequeño backtracking con propagación de dominios, restringiendo además el alfabeto a los caracteres que realmente permite el teclado del reto:

```
0-9
A-Z
_
{}
```

Un solver sencillo sería algo así:

```python
ALPHABET = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ_{}"
CHARSET = set(map(ord, ALPHABET))

def propagate(domains, constraints):
    changed = True
    while changed:
        changed = False

        for a, b, c, t in constraints:
            da, db, dc = domains[a], domains[b], domains[c]

            poss_a = {((x + y) ^ t) & 0xFF for x in db for y in dc}
            poss_b = {((t ^ x) - y) & 0xFF for x in da for y in dc}
            poss_c = {((t ^ x) - y) & 0xFF for x in da for y in db}

            na = da & poss_a
            nb = db & poss_b
            nc = dc & poss_c

            if not na or not nb or not nc:
                return None

            if na != da:
                domains[a] = na
                changed = True
            if nb != db:
                domains[b] = nb
                changed = True
            if nc != dc:
                domains[c] = nc
                changed = True

    return domains

def solve_with_domains(constraints, domains):
    domains = propagate(domains, constraints)
    if domains is None:
        return None

    if all(len(d) == 1 for d in domains):
        return "".join(chr(next(iter(d))) for d in domains)

    idx = min(
        (i for i, d in enumerate(domains) if len(d) > 1),
        key=lambda i: len(domains[i])
    )

    for v in sorted(domains[idx]):
        new_domains = [set(d) for d in domains]
        new_domains[idx] = {v}
        res = solve_with_domains(constraints, new_domains)
        if res is not None:
            return res

    return None

def solve(constraints, length, fixed=None):
    domains = [set(CHARSET) for _ in range(length)]

    if fixed:
        for i, ch in fixed.items():
            domains[i] = {ord(ch)}

    return solve_with_domains(constraints, domains)

print(solve(Q1_TABLE, 7))
print(solve(Q2_TABLE, 5))
print(solve(Q3_TABLE, 32, fixed={0: "R", 1: "S", 2: "{", 31: "}"}))
```

Y así obtendremos la flag! Un reto bastante chulo, espero que os haya gustado!
