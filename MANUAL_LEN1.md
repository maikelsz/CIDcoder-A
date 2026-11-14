# Manual de LEN-1 — CID 201-A

> **LEN-1** (*Lenguaje Ensamblador 1*) es el ensamblador nativo de la CID 201-A. Este manual describe el ensamblador que implementa **CIDcoder-A** y se apoya en la pestaña **Ayuda** del editor y en el manual histórico de la CID 201-A. Cuando el manual histórico y el emulador difieren, la diferencia se documenta en la sección 12.

## 1. La máquina

La CID 201-A es una minicomputadora cubana de 1971, diseñada por el Centro de Investigación Digital (CID). Es una máquina de **12 bits**, de una dirección y operación en serie, con memoria de núcleos de ferrita.

| Elemento | Valor |
|---|---|
| Longitud de palabra | 12 bits |
| Memoria | 4096 palabras, 32 páginas × 128 palabras |
| Direcciones | Octal: `0000`–`7777` |
| Reloj | 1 MHz |
| Ciclo de memoria | 1,5 µs |
| Registros | AC (12), EN (1), RM (12), CP (12), RI (5), RD (7), RP (5), RIN (12) |
| Instrucciones | 59 |
| Interrupciones | No tiene; E/S por sondeo (*polling*) |

Los números se escriben en **octal**: cada dígito octal son 3 bits y cuatro dígitos octales representan los 12 bits. La máquina usa **complemento a dos**. A nivel de 12 bits, el rango con signo es −2048 a +2047; los programas pueden construir aritmética de 24 bits o punto flotante por software.

Los bits se numeran de 0 a 11, de izquierda a derecha: el bit 0 es el más significativo y también el bit de signo.

## 2. Sintaxis del fuente

```text
/ comentario
ETIQUETA, INSTRUCCION      / etiqueta + instrucción (coma obligatoria)
* direccion                / directiva de origen
$                          / fin del fuente (opcional)
ETIQUETA, dato1 dato2 ...  / datos con coma
ETIQUETA dato1 dato2 ...   / datos sin coma (coma opcional)
TEXTO, "HOLA"              / cadena ITA2 (1 carácter por palabra)
TEXTO2, 'HOLA'             / cadena empaquetada (2 caracteres por palabra)
```

| Elemento | Regla |
|---|---|
| `/` | Inicia un comentario hasta el final de la línea. |
| `ETIQUETA,` | Empieza por letra o `_`, no distingue mayúsculas/minúsculas, no puede repetirse ni coincidir con una palabra clave. La coma es **obligatoria** cuando la etiqueta va seguida de una instrucción; en una línea de datos es opcional (`A, 0005` o `A 0005`). |
| `* ADDR` | Fija la dirección de ensamblado, en octal por defecto. La **primera** directiva `*` fija el punto de entrada (CP); sin ninguna, el programa arranca en `0000`. |
| `$` | Fin del fuente. Es opcional: sin `$`, el ensamblador termina al final del archivo. Lo que haya después se ignora. |
| Datos | Una línea sin instrucción puede contener números o etiquetas; cada token produce una palabra de 12 bits. |
| Cadenas | `"TEXTO"` genera palabras ITA2 con 1 carácter por palabra; `'TEXTO'` las empaqueta a 2 caracteres por palabra. La cadena debe ser el único dato de la línea y añade el terminador. |

### 2.1 Números

La base por defecto es **octal**. Valores válidos: `0000`–`7777` octal (0–4095 decimal).

| Forma | Interpretación |
|---|---|
| `47` | Octal por defecto. |
| `060` | Octal clásico. |
| `0o60` | Octal explícito. |
| `0d48` | Decimal explícito. |
| `0x30` | Hexadecimal. |
| `-5` | Negativo (complemento a dos, 12 bits). |
| `+2` / `-3` | Desplazamiento relativo en `VEA`/`VAS`; base decimal por defecto. |
| `PARM+1` | Etiqueta más desplazamiento; base decimal por defecto. |
| `PARM+0o10` | Desplazamiento octal explícito. |

### 2.2 Punto de entrada

La primera directiva `*` fija la dirección desde la que arranca el ejecutor (`CP`). Si el fuente no tiene ninguna, el punto de entrada es `0000`. El ejemplo mínimo `*0200` hace que el programa se cargue y ejecute desde la página 2.

## 3. Memoria y direccionamiento

La máquina tiene **4096 palabras** de 12 bits, numeradas de `0000` a `7777` octal. Una dirección se divide en **5 bits de página** (32 páginas, `00`–`37`) y **7 bits de posición** (128 palabras por página, `000`–`177`).

### 3.1 Alcance directo de las IRM

Una IRM dispone de 7 bits de dirección más 1 bit de página. Sin indirección, un operando solo puede estar en:

- **Página cero** (`0000`–`0177`): accesible desde cualquier página.
- **Página actual**: la página donde está la propia instrucción.

Si el operando está en otra página, el ensamblador lo rechaza:

```text
dirección fuera de alcance directo desde PC=0200: 0400 → 0400
```

Esto vale también para el **salto**: `VEA 0400` desde `PC=0200` (página 1) es ilegal porque `0400` está en la página 2.

> **Página 0 en programas generados por LEAL:** el compilador coloca en `0000`–`0177` el bootstrap (`0000: VEA I 0001`), los escalares, los datos del runtime y la tabla de punteros. El vector `0001` apunta al cuerpo. En LEN-1 escrito a mano usted decide qué poner en página 0; recuerde que es el único lugar común a todas las páginas y el recurso más escaso.

### 3.2 Indirección

La indirección (`I` o `@`) usa una celda como puntero: la dirección de la instrucción señala la celda (**que debe estar en página 0 o en la página actual**) y el contenido de esa celda es la dirección efectiva de 12 bits, que puede apuntar a cualquier página.

```text
VEA I PTR      / salta a [PTR]
DEL I IDX      / deposita AC en [[IDX]]
SUM I PTR      / suma [[PTR]] a AC
VEA @PTR       / forma equivalente a VEA I PTR
```

La indirección no se combina con el salto relativo: `VEA I +2` da «indirección no se combina con relativo».

### 3.3 Solapamientos

El ensamblador rechaza cualquier escritura sobre una dirección ya escrita (por ejemplo, dos `*ADDR` incompatibles o un array que pisa una instrucción). El error indica la línea LEN-1 de cada lado.

Para *overlays* intencionales (código automodificable, bootstraps), añada el comentario `/ @@overlap-ok@@` en cualquier parte del fuente y el chequeo se desactiva para todo el programa.

## 4. Instrucciones que referencian a la memoria (IRM)

Formato general de 12 bits:

| Bits | 0–2 | 3 | 4 | 5–11 |
|---|---|---|---|---|
| Campo | código de operación | indirecto | página | dirección relativa |

- **Código de operación:** 3 bits.
- **Indirección:** 0 = directo, 1 = indirecto (requiere un ciclo de memoria adicional).
- **Página:** 0 = página cero (`0000`–`0177`), 1 = página actual (definida por RP/CP).
- **Dirección relativa:** 7 bits (0–127).

| Octal base | Mnemónico | Operación |
|---|---|---|
| `0xxx` | `YLO M` | `AC = AC & [M]` (AND lógico) |
| `1xxx` | `SUM M` | `AC = AC + [M]` (suma en complemento a dos; puede modificar EN) |
| `2xxx` | `ISC M` | `[M] = [M] + 1`; si el resultado es `0000`, salta la instrucción siguiente |
| `3xxx` | `DEL M` | `[M] = AC`; `AC = 0` |
| `4xxx` | `VAS M` | `[M] = CP + 1` (dirección de retorno); `CP = M + 1` |
| `5xxx` | `VEA M` | `CP = M` |
| `6xxx` | — | Microinstrucciones de E/S |
| `7xxx` | — | Microinstrucciones de cálculo |

### 4.1 Acarreo y desbordamiento

En la suma (`SUM`), el **Enlace (EN)** captura el **acarreo sin signo** que sale del bit 12 de la suma de 13 bits: `EN = 1` si `AC + [M] > 7777`, y `EN = 0` en caso contrario. No es una bandera de desbordamiento con signo; es el acarreo que permite encadenar palabras en aritmética de doble precisión.

| Operación | AC resultante | EN |
|---|---|---|
| `2000 + 2000` | `4000` | 0 (no hay acarreo) |
| `7777 + 0001` | `0000` | 1 (acarreo) |
| `7777 + 7777` | `7776` | 1 (acarreo) |

Si se necesita detectar desbordamiento con signo, el software debe comparar los signos de los operandos y del resultado; el hardware de `SUM` no lo hace por sí solo.

### 4.2 Subrutinas con VAS

`VAS RUT` guarda la dirección de retorno (`CP+1`) en la palabra `RUT` y salta a `RUT+1`. Por eso la primera palabra de la subrutina es el **slot de retorno** y el código empieza en `RUT+1`; para volver se usa `VEA I RUT`, con la celda `RUT` en página 0 o en la página actual.

```text
*0200
        VAS SUBRUT
        ALT
SUBRUT, 0000          / slot de retorno
        LAC
        SUM A
        DEL RES
        VEA I SUBRUT   / retorno
A,      0005
RES,    0000
$
```

### 4.3 Saltos relativos

`VEA +2` salta a `PC+2`. `VAS -3` usa `PC−3` como slot de retorno y continúa en `PC−2`. Solo funciona en modo directo, no con `I`.

## 5. Microinstrucciones

Las microinstrucciones se codifican en los últimos 7 u 8 bits. El código de operación `111` (7) permite combinar varias microinstrucciones del mismo grupo en una sola instrucción (OR de sus códigos binarios). En el listado aparecen como dos o más instrucciones en una misma línea.

**Formato:** `111 xxx xxx xxx` (opcode 7 + 9 bits de función). Bits significativos: `LAC` (0o200), `SAN` (0o100), `SAC` (0o040), `SED` (0o020), `SLT` (0o010), `ALT` (0o004), `ORI` (0o002).

### 5.1 Grupo 0 — Limpieza

| Octal | Mnemónico | Operación |
|---|---|---|
| `7200` | `LAC` | `AC = 0` |
| `7100` | `LEN` | `EN = 0` |

> **LAC tiene dos codificaciones.** El manual histórico lo lista en el Grupo 0 como `7200` y en el Grupo 2 como `7600`; ambas son correctas y ejecutan la misma operación (`AC = 0`). `7200` es la forma del Grupo 0 y se combina con `LEN`, `CAC`, `CEN` e `IAC`; `7600` es la forma del Grupo 2 y sirve de base para las combinaciones con saltos (`7700 LAC SAN`, `7710 LAC SAP`, `7770 LAC SAP SAD SEC`…). El ensamblador emite `7200` para el mnemónico `LAC` aislado y compone las combinaciones con los códigos de salto; `7600` no se escribe como mnemónico, pero es una palabra válida y el emulador la ejecuta como `LAC`.
| `7040` | `CAC` | `AC = ~AC` |
| `7020` | `CEN` | `EN = ~EN` |
| `7010` | `IAC` | `AC = AC + 1`; si AC pasa de `7777` a `0000`, `EN = ~EN` |
| `7000` | `NOP` | Sin operación |

Las micros del grupo 0 se pueden combinar: el ensamblador suma sus códigos y las ejecuta en el orden de la palabra, **LAC → LEN → CAC → CEN → IAC**. `NOP` puede añadirse sin cambiar el resultado.

| Octal | Combinación | Operación resultante |
|---|---|---|
| `7050` | `CAC IAC` | `AC = ~AC + 1` (complemento a dos) |
| `7060` | `CAC CEN` | `AC = ~AC`; `EN = ~EN` |
| `7120` | `LEN CEN` | `EN = 1` |
| `7210` | `LAC IAC` | `AC = 1` |
| `7220` | `LAC CEN` | `AC = 0`; `EN = ~EN` |
| `7240` | `LAC CAC` | `AC = 7777` (todos unos) |
| `7300` | `LAC LEN` | `AC = 0`; `EN = 0` |
| `7310` | `LAC LEN IAC` | `AC = 1`; `EN = 0` |
| `7320` | `LAC LEN CEN` | `AC = 0`; `EN = 1` |
| `7330` | `LAC LEN CEN IAC` | `AC = 1`; `EN = 1` |
| `7340` | `LAC LEN CAC` | `AC = 7777`; `EN = 0` |
| `7350` | `LAC LEN CAC IAC` | `AC = 0`; `EN = 1` (el acarreo de 7777+1 complementa EN) |
| `7360` | `LAC LEN CAC CEN` | `AC = 7777`; `EN = 1` |
| `7370` | `LAC LEN CAC CEN IAC` | `AC = 0`; `EN = 0` |

> **IAC y el enlace:** en cualquier combinación con `IAC`, si AC pasa de `7777` a `0000` el acarreo **complementa** EN (no lo fuerza a 1). Por eso `7350` termina con EN=1 y `7370` con EN=0.

### 5.2 Grupo 1 — Rotaciones

La rotación actúa sobre el par (AC, EN) como un registro único de 13 bits. Los bits que salen del AC por desbordamiento pasan al EN y se realimentan por el lado contrario.

| Derecha | Izquierda | Bits |
|---|---|---|
| `RD1` | `RI12` | 1 |
| `RD2` | `RI11` | 2 |
| `RD3` | `RI10` | 3 |
| `RD4` | `RI9` | 4 |
| `RD5` | `RI8` | 5 |
| `RD6` | `RI7` | 6 |
| `RD7` | `RI6` | 7 |
| `RD8` | `RI5` | 8 |
| `RD9` | `RI4` | 9 |
| `RD10` | `RI3` | 10 |
| `RD11` | `RI2` | 11 |
| `RD12` | `RI1` | 12 |

> **Sin espacio:** las rotaciones se escriben `RD1`, `RI12`, etc. `RI 1` no es una rotación: se interpreta como la etiqueta `RI` con el dato `1`.

### 5.3 Grupo 2 — Saltos por *skip*

Los saltos condicionales se evalúan sobre el estado del AC y del EN. Si la condición se cumple, se **omite la instrucción siguiente** (`CP = CP + 1`). Para saltar lejos se usa `SLT` seguido de `VEA`.

| Octal | Mnemónico | Condición | Grupo |
|---|---|---|---|
| `7500` | `SAN` | `AC < 0` | OR |
| `7440` | `SAC` | `AC == 0` | OR |
| `7420` | `SED` | `EN == 1` | OR |
| `7510` | `SAP` | `AC > 0` | AND |
| `7450` | `SAD` | `AC != 0` | AND |
| `7430` | `SEC` | `EN == 0` | AND |
| `7410` | `SLT` | Incondicional | especial |
| `7404` | `ALT` | Detiene | especial |
| `7402` | `ORI` | `AC = AC | RIN` | especial |

**Reglas de combinación:**

- Saltos OR entre sí: `SAN SAC SED`.
- Saltos AND entre sí: `SAP SAD SEC`.
- Nunca mezclar OR con AND.
- `SLT` no se combina.
- `ALT` no se combina con `ORI` ni `SLT`.
- `ORI` solo con saltos OR; no con AND, `ALT` ni `SLT`.
- No se combinan dos rotaciones ni grupos distintos (por ejemplo `LAC SUM` o `LAC RI3`). `LAC` solo se combina con el grupo de saltos/especiales (no con rotaciones); `LEN` solo con una rotación compatible.
- No se combinan microinstrucciones de E/S de periféricos distintos (por ejemplo `LIP TLA`).
- `TAU` y `TAD` son *stubs* (`AC + 0`): no hay registro externo real. `TUA` y `TDA` aún no están implementados.
- `LAC` con cualquier salto: el salto se evalúa sobre el **AC anterior** (orden 1 del salto, orden 2 de `LAC`). `LAC SAC` salta si el AC anterior era 0; `LAC SED` según EN; `LAC SLT` salta siempre; `LAC SAN/SAP/SAD` según el AC anterior.

**Combinaciones documentadas y verificadas en el core:**

| Octal | Mnemónico | Efecto | Grupo |
|---|---|---|---|
| `7460` | `SAC SED` | Salta si `AC == 0` **o** `EN != 0` | OR |
| `7520` | `SAN SED` | Salta si `AC < 0` **o** `EN != 0` | OR |
| `7540` | `SAN SAC` | Salta si `AC < 0` **o** `AC == 0` | OR |
| `7470` | `SAD SEC` | Salta si `AC != 0` **y** `EN == 0` | AND |
| `7530` | `SAP SEC` | Salta si `AC > 0` **y** `EN == 0` | AND |
| `7550` | `SAP SAD` | Salta si `AC > 0` **y** `AC != 0` | AND |
| `7570` | `SAP SAD SEC` | Salta si `AC > 0` **y** `AC != 0` **y** `EN == 0` | AND |
| `7700` | `LAC SAN` | `0 → AC`; salta si el AC anterior `< 0` | OR+E |
| `7710` | `LAC SAP` | `0 → AC`; salta si el AC anterior `> 0` | AND+E |
| `7650` | `LAC SAD` | `0 → AC`; salta si el AC anterior `!= 0` | AND+E |
| `7640` | `LAC SAC` | `0 → AC`; salta si el AC anterior `== 0` | OR+E |
| `7630` | `LAC SEC` | `0 → AC`; salta si `EN == 0` | AND+E |
| `7620` | `LAC SED` | `0 → AC`; salta si `EN != 0` | OR+E |
| `7610` | `LAC SLT` | `0 → AC` y salta siempre | especial |
| `7670` | `LAC SAD SEC` | `0 → AC`; salta si el AC anterior `!= 0` **y** `EN == 0` | AND+E |
| `7770` | `LAC SAP SAD SEC` | `0 → AC`; salta si el AC anterior `> 0` **y** `EN == 0` | AND+E |
| `7504` | `SAN ALT` | Alto si `AC >= 0`; si no, salta y continúa | especial |
| `7444` | `SAC ALT` | Alto si `AC != 0`; si no, salta y continúa | especial |
| `7424` | `SED ALT` | Alto si `EN == 0`; si no, salta y continúa | especial |
| `7514` | `SAP ALT` | Alto si `AC <= 0`; si no, salta y continúa | especial |
| `7454` | `SAD ALT` | Alto si `AC == 0`; si no, salta y continúa | especial |
| `7434` | `SEC ALT` | Alto si `EN != 0`; si no, salta y continúa | especial |
| `7502` | `SAN ORI` | OR con interruptores si `AC >= 0`; si no, salta y continúa | especial |
| `7442` | `SAC ORI` | OR con interruptores si `AC != 0`; si no, salta y continúa | especial |
| `7422` | `SED ORI` | OR con interruptores si `EN == 0`; si no, salta y continúa | especial |

> **El octal 7770:** incluye `LAC` y las tres condiciones AND (`SAP SAD SEC`). `LAC SAD SEC` sin `SAP` es `7670`. Si un manual muestra 7770 como «LAC SAD SEC», su mnemónico debe incluir `SAP` para coincidir con el octal y con el efecto (`AC > 0`).

### 5.4 E/S — Grupo 6 (opcode 110)

El código de operación `110` (6) agrupa las microinstrucciones de E/S. La Ayuda del editor lo llama «Grupo 3 — E/S por periférico» por el campo de grupo del microcódigo; aquí se usa **Grupo 6** por el opcode, que es el nombre del manual histórico. Ambos nombres designan el mismo conjunto.

El sistema es de **sondeo** (*polling*): el programa verifica periódicamente el estado del periférico. Toda instrucción que transfiere AC a un registro de salida **limpia AC**; toda instrucción que transfiere un registro de entrada al AC **suma** al AC.

#### Perforador / impresora

| Octal | Mnemónico | Operación |
|---|---|---|
| `6110` | `LIP` | Arranca motor / limpia el indicador de perforación. |
| `6120` | `TAP` | `AC → registro`; `AC = 0`. |
| `6130` | `TAP LIP` | `AC → registro`; `AC = 0`; limpia el indicador de perforación. |
| `6140` | `SIP` | Salta si el indicador de perforación está listo. |

#### Lector de cinta

| Octal | Mnemónico | Operación |
|---|---|---|
| `6202` | `PAL` | Detiene el motor. |
| `6204` | `COL` | Inicia el motor. |
| `6210` | `LIL` | Limpia el indicador de lectura. |
| `6220` | `TLA` | `AC + registro lector → AC`; si hay acarreo, `EN = 1`. |
| `6230` | `TLA LIL` | Igual que `TLA` y además limpia el indicador de lectura. |
| `6240` | `SIL` | Salta si hay carácter. |

> **LIL:** deja el indicador de lectura en 0. Un carácter que ya estuviera en el buffer no lo vuelve a activar de inmediato; el indicador se pone a 1 cuando el lector entrega un carácter nuevo.

#### Teclado decimal

| Octal | Mnemónico | Operación |
|---|---|---|
| `6411` | `LIF` | Limpia el indicador de fracción. |
| `6412` | `LIA` | Limpia el indicador ALM. |
| `6414` | `LID` | Limpia el indicador de dígito. |
| `6420` | `TRA` | `AC + registro de dígitos → AC`; si hay acarreo, `EN = 1`. |
| `6434` | `TRA LID` | Igual que `TRA` y además limpia el indicador de dígito. |
| `6441` | `SIF` | Salta si hay fracción. |
| `6442` | `SIA` | Salta si ALM. |
| `6444` | `SID` | Salta si hay dígito. |

#### Display numérico

| Octal | Mnemónico | Operación |
|---|---|---|
| `6500` | `TAS` | Prepara signo negativo. |
| `6511` | `LIB` | Limpia el indicador BOR. |
| `6512` | `LIS` | Limpia el indicador de signo. |
| `6514` | `LIR` | Limpia el indicador REC. |
| `6521` | `TAF` | `AC → registro de fracción`. |
| `6522` | `TAT` | `AC → registro de títulos`. |
| `6524` | `TAR` | `AC → registro de números`. |
| `6541` | `SIB` | Salta si BOR. |
| `6542` | `SIS` | Salta si signo. |
| `6544` | `SIR` | Salta si REC. |
| `6310` | `LIG` | Limpia los indicadores de teclado, display y E/S externas (no los del lector/perforador). |

#### Dispositivos externos

| Octal | Mnemónico | Operación |
|---|---|---|
| `6611` | `LIO` | Limpia el indicador de salida. |
| `6612` | `LIM` | Limpia el indicador de muestreo. |
| `6621` | `TAU` | `AC → registro externo Uno` (*stub*: `AC + 0`). |
| `6622` | `TAD` | `AC → registro externo Dos` (*stub*: `AC + 0`). |
| `6631` | `TUA` | `Registro externo Uno → AC` (**no implementado**). |
| `6632` | `TDA` | `Registro externo Dos → AC` (**no implementado**). |
| `6641` | `SIO` | Salta si hay salida. |
| `6642` | `SIM` | Salta si hay muestreo. |

> **Combinaciones de E/S:** solo se combinan microinstrucciones del mismo periférico. Son válidas `TAP LIP` (perforador), `TLA LIL` (lector) y `TRA LID` (teclado). `TLA LID` mezcla lector y teclado y el ensamblador la rechaza con «no combinar E/S de periféricos distintos».

## 6. Operaciones comunes

```text
LAC          / AC = 0
SUM DATO     / AC = [DATO]
DEL RES      / [RES] = AC, AC = 0
CAC IAC      / AC = -AC

LAC          / AC = 0
SUM A        / AC = [A]
CAC IAC      / AC = -[A]
SUM B        / AC = [B] - [A]
```

**Multiplicar por 2** (desplazamiento a la izquierda):

```text
LEN RI1      / EN = 0; AC = AC << 1
```

**OR con la ley de De Morgan** (solo NOT y AND son hardware):

```text
CAC          / AC = ~A
DEL TEMP     / guarda ~A; AC = 0
SUM B        / AC = B
CAC          / AC = ~B
YLO TEMP     / AC = ~B & ~A = ~(A | B)
CAC          / AC = A | B
```

## 7. Ejemplos

### 7.1 Suma de dos números

```text
*0200
        LAC
        SUM A       / AC = [A]
        SUM B       / AC = [A] + [B]
        DEL RES
        ALT
A,      0005
B,      0007
RES,    0000
$
```

### 7.2 MINIBIN: carga desde cinta perforada

```text
*0
        COL LIL         / inicia el lector y limpia su indicador
NuevaPalabra,   LAC LEN / limpia AC y EN
Nuevos5bits,    RI5     / desplaza 5 bits (cinta de 5 canales)
EsperaLector,   SIL     / salta si el lector tiene dato
        VEA EsperaLector
        TLA LIL         / lee 5 bits del lector
        SED             / salta si EN=1 (palabra completa)
        VEA Nuevos5bits
        DEL I Hacia     / deposita AC en [Hacia]
        ISC Hacia       / incrementa el puntero
        VEA NuevaPalabra
Hacia,  200             / dirección inicial de destino
$
```

### 7.3 Subrutina

```text
*0200
        VAS SUBRUT
        ALT
SUBRUT, 0000            / slot de retorno
        LAC
        SUM A
        DEL RES
        VEA I SUBRUT     / retorno
A,      0005
RES,    0000
$
```

## 8. Listado, ejecución y depuración

### 8.1 Panel LST

| Columna | Significado |
|---|---|
| **Pág** | Página de la palabra (`00`–`37` octal). |
| **Rel** | Dirección relativa dentro de la página (`000`–`177`). |
| **Abs** | Dirección absoluta de 12 bits (`0000`–`7777`). |
| **Cont** | Contenido de la palabra en octal (instrucción o dato). |
| **Fuente** | Línea LEN-1 que la generó y su comentario. Las filas modificadas por el programa se resaltan en ámbar. |

La barra de estado muestra el número de palabras ensambladas; **Save .lst** guarda el listado. Un clic en una fila pone o quita un punto de ruptura en la línea LEN-1 asociada.

### 8.2 Ejecutor

| Control | Acción |
|---|---|
| **▶** | Ejecuta o detiene el programa. |
| **CONT / INSTR** | Ejecución continua o instrucción a instrucción. |
| **•⇢•** (barra) | Resalta en el LST la instrucción en ejecución. |
| **CP** | Contador de programa en octal. Enter salta a la dirección; Esc cancela. |
| **▦** | Muestra u oculta el mapa de memoria. |
| **⟲** | Recarga el programa y reinicia la ejecución desde el punto de entrada. |
| **◀ / ▶** (mapa) | Página anterior / siguiente del mapa. |
| **Ir al CP** / **Ir a la instr.** | Centra el mapa en el CP o en la instrucción en ejecución. |

Los registros EN, AC, RM y CP se muestran sobre el teletipo; el ciclo y el estado del teclado (KBD) y del modo ITA2 (LTRS/FIGS) aparecen a la derecha. Los puntos de ruptura detienen la ejecución continua; al recompilar se borran, y **⊘ BPs** los borra todos.

> **Teclado del teletipo:** con el foco en el papel, escriba normalmente; Enter envía CR. Los caracteres que no existen en ITA2 se avisan y no se envían. Backspace no se usa: el teclado histórico no lo tiene.

## 9. Optimizaciones (solo LEAL → LEN-1)

Estas transformaciones se aplican al texto que genera el compilador LEAL, **no** cuando se abre y ensambla un archivo `.len1` escrito a mano: en ese caso el ensamblador respeta el fuente tal cual.

1. **Eliminación** de `LAC` redundantes después de `DEL` o `LAC` (esas instrucciones ya dejan AC = 0).
2. **Combinación** de dos microinstrucciones consecutivas compatibles, respetando las etiquetas: nunca combina si la segunda línea lleva etiqueta.
3. **Página 0:** elimina punteros muertos, comparte punteros y constantes equivalentes, mueve al final de su página los punteros usados en una sola página y omite variables runtime y constantes que el programa no usa.

El panel de mensajes del editor informa: «N eliminadas, M combinadas». En el LST aparecen los mensajes de página 0: punteros de origen → destino, palabras ahorradas y constantes deduplicadas.

## 10. Errores frecuentes

### Sintaxis, etiquetas y datos

- Escribir `8` o `9` sin prefijo: la base por defecto es octal y da «número octal inválido». Use `0d8`, `0d9` o `0x8`.
- Etiquetas duplicadas o etiqueta que coincide con una palabra clave.
- Olvidar la coma entre etiqueta e instrucción: `A LAC` da «falta coma tras etiqueta». En una línea de datos la coma es opcional (`A 0005` es válido).
- `RI 1` / `RD 2`: no dan error, pero **no son rotaciones**; se interpretan como la etiqueta `RI`/`RD` con un dato. Escriba `RI1`, `RD2`.
- Usar una palabra clave como operando o como etiqueta.
- Creer que `$` es obligatorio: es opcional.
- Usar `;` como comentario (en LEN-1 el comentario empieza por `/`).

### Direcciones y memoria

- Usar un operando directo fuera de página 0 y de la página actual: `VEA 0400` desde `PC=0200` da «dirección fuera de alcance directo».
- Creer que la indirección alcanza cualquier celda: el **puntero** también debe estar en página 0 o en la página actual; lo que puede apuntar lejos es su **contenido**.
- Combinar indirección y salto relativo: `VEA I +2` da «indirección no se combina con relativo».
- Solapar código o datos: el ensamblador da «Solapamiento en…». Para *overlays* intencionales, añada `/ @@overlap-ok@@`.
- Colocar código o datos que deben ser accesibles desde otra página fuera de página 0 sin un puntero intermedio.

### Microinstrucciones y E/S

- Mezclar saltos OR y AND: `SAP SAC` da «no mezclar OR y AND».
- Combinar `SLT` con cualquier otra micro: «SLT no se combina».
- Combinar `ORI` con AND, `ALT` o `SLT`: «ORI solo con OR».
- Combinar `ALT` con `ORI` o `SLT`.
- Combinar dos rotaciones («no combinar rotaciones») o grupos distintos: `LAC RI3` da «sólo LEN con rotación» y `LAC SUM` da «secuencia inválida».
- Combinar E/S de periféricos distintos (por ejemplo `LIP TLA`): «no combinar E/S de periféricos distintos».
- Olvidar que `LAC` con un salto evalúa el **AC anterior**: `LAC SAC` salta si el AC anterior era 0.
- Esperar un registro real en `TAU`/`TAD` (son *stubs* `AC + 0`) o usar `TUA`/`TDA` (no implementados).

**Checklist rápido:** ¿todo operando directo está en página 0 o en la página actual? ¿Los punteros indirectos están en página 0 o en la página actual? ¿Las rotaciones van sin espacio? ¿Las etiquetas de instrucción llevan coma? ¿El fuente tiene un `*` al principio si espera arrancar en otra dirección?

## 11. Del LEN-1 al emulador completo

El listado `.lst` guardado puede cargarse en el emulador completo de la CID 201-A, disponible en [CIDmulator-A](https://github.com/maikelsz/CIDmulatorCID201A) y en [GitHub Pages](https://maikelsz.github.io/CIDcoder-A/CIDcoder-A.html):

1. «Encender» la computadora.
2. Cargar el archivo `.lst`.
3. Poner los 12 interruptores con la dirección de inicio. Los programas compilados desde LEAL tienen el vector de inicio en `0000`; los escritos en LEN-1 pueden empezar en cualquier dirección que el autor decidiera.
4. Pulsar `CPR`: la dirección seleccionada pasa al Contador de Programa.
5. Pulsar `COM` para comenzar.

Si la computadora está en ejecución, primero deténgala con `ALT` y siga los pasos desde el 2.

## 12. Concordancia con el manual histórico y la Ayuda

Este manual usa como referencia de comportamiento la **Ayuda integrada de CIDcoder-A** y el **manual histórico de la CID 201-A** (`CID201A.md`). Puntos que conviene tener presentes:

1. **Nombre del grupo de E/S.** El manual histórico lo llama **Grupo 6** por el opcode `110`; la Ayuda del editor lo llama **Grupo 3** por el campo de grupo del microcódigo. Es el mismo conjunto; aquí se usa «Grupo 6 (opcode 110)» y se menciona el alias.
2. **`7770`.** El manual histórico lo describe como `LAC SAD SEC`; el octal real incluye también `SAP`. La forma correcta es **`LAC SAP SAD SEC`**. `LAC SAD SEC` sin `SAP` es `7670`.
3. **`7200` y `7600`.** La tabla histórica lista `LAC` en el Grupo 0 como `7200` y en el Grupo 2 como `7600`. Ambas codificaciones son correctas: la misma operación (`AC = 0`) aparece en dos grupos. `7200` se combina con las demás micros del Grupo 0; `7600` es la base de las combinaciones `LAC` + salto del Grupo 2. El ensamblador escribe `7200` para `LAC` aislado y las combinaciones verificadas de la sección 5.3 para `LAC` + salto. Si encuentra `7600` en un listado histórico, es `LAC`.
4. **`LIG`.** El manual histórico dice «limpiar los indicadores»; el emulador precisa que limpia teclado, display y E/S externas, **no** los del lector/perforador.
5. **`LIL`.** Deja el indicador de lectura en 0; un carácter ya presente en el buffer no lo reactiva de inmediato.
6. **`TAP LIP`, `TLA LIL`, `TRA LID`.** Son las combinaciones válidas de E/S del mismo periférico; no mezcle periféricos.
7. **`TAU`/`TAD`.** Son *stubs* (`AC + 0`). `TUA`/`TDA` no están implementados.
8. **`SAP`.** Es estricto: salta solo si `AC > 0` (no si `AC >= 0`). `SEC` salta con `EN == 0`; `SED` con `EN == 1`.
9. **Comentarios en ejemplos históricos.** El manual histórico usa `;` en algunos ejemplos; en LEN-1 el comentario es `/`. También usa `R1` en un ejemplo de multiplicación; la rotación correcta es `RI1`.
10. **Optimizaciones.** Las optimizaciones de página 0 y de microinstrucciones descritas en la sección 9 solo se aplican al LEN-1 generado por LEAL; un `.len1` escrito a mano se ensambla tal cual.
11. **Listado, ejecutor y depuración.** Esta materia es una sección compartida de la Ayuda: se implementa una sola vez y se muestra como §12 en LEN-1 y §12d en LEAL. En modo LEN-1 los BP se marcan en el gutter del editor; en modo LEAL, en las líneas del LEN-1 generado o en las filas del LST.
