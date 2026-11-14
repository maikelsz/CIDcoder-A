# Manual de LEAL — CID 201-A

> **LEAL** (*LEnguaje ALgorítmico*) es el lenguaje de alto nivel de las minicomputadoras cubanas CID. Este manual describe el dialecto que compila **CIDcoder-A** para la **CID 201-A**. La referencia ejecutable es la pestaña **Ayuda** del editor; el manual histórico de la máquina (`CID201A.md`) se usa para todo lo relativo a LEN-1 y al hardware. Cuando una fuente histórica y el emulador difieren, este manual sigue al emulador y lo indica.

## 1. Historia y alcance

LEAL fue desarrollado en Cuba en el Centro de Investigación Digital (CID) entre 1970 y 1971, bajo la dirección del Ing. **Víctor M. Llopis Zayas-Bazán**, con la colaboración de Victor A. Orsini (diseño original y aritmética), Emilio Lino (E/S), Fedor Sierra (subrutinas básicas), María E. Bragado y Miguel Fonfría (funciones trascendentes). Sigue algunos conceptos del *Autocode* de la Elliott 803B Mark III y se orientó al cálculo científico en las minicomputadoras CID 201/201-A/201-B y CID 300/10.

Durante los años 1970 se desarrolló en tres versiones:

| Versión | Orientación |
|---|---|
| **LEAL-A** | Cálculo científico; cintas de papel; sin comentarios; operaciones solo entre enteros o solo entre flotantes. Es un subconjunto de LEAL-B. |
| **LEAL-B** | Añade comentarios, mezcla de tipos, división entera, asignaciones encadenadas, subrutinas y E/S de caracteres. |
| **LEAL-F** | CID 300/10; cálculo científico y gestión económica. |

**CIDcoder-A** compila el dialecto **LEAL-B** para la CID 201-A. LEAL-A sigue siendo válido porque es un subconjunto; las particularidades de la CID 201-B quedan fuera de este emulador.

> **Nombre del lenguaje:** se llama **LEAL**, no LEA ni EAB. `EAB` es una función del compilador: el valor absoluto entero.

## 2. Estructura de un programa

Todo fuente LEAL tiene un **encabezamiento** y un **cuerpo**:

```leal
LEAL                 ; primera línea obligatoria
TITULO SUMA DEMO     ; título del programa (declaración inerte)
ENT A B C            ; variables enteras (puede quedar vacía)
FLO X Y              ; variables flotantes (puede quedar vacía)
REF (1)              ; rango de números de referencia (opcional)
A + B = C            ; cuerpo
PER C
FIN                  ; fin de la compilación
```

| Elemento | Regla |
|---|---|
| `LEAL` | Primera línea obligatoria. |
| `TITULO ...` | Da nombre al programa. Es **inerte**: no ejecuta nada ni emite palabras. Solo se admite en la zona de declaraciones, antes de la primera instrucción. |
| `ENT ...` | Declara variables enteras de 24 bits. Puede quedar vacía. |
| `FLO ...` | Declara variables flotantes de 24 bits. Puede quedar vacía. |
| `REF (n)` | Declara el rango máximo de números de referencia. Es opcional si las etiquetas están definidas. |
| `FIN` | Termina la compilación. El primer `FIN` detiene la lectura del fuente. |

Los **comentarios** empiezan por `;` y llegan hasta el final de la línea. Se eligió `;` porque `/` es el operador de división.

## 3. Variables y tipos

### 3.1 Nombres

- Una variable es **una sola letra mayúscula de la A a la Z**; se excluye la Ñ.
- `OP`, `T1` o `VAR` no son variables válidas.
- El tipo se fija en la declaración y **no cambia** durante el programa.

### 3.2 Escalares y arreglos

```leal
ENT A B C            ; escalares enteros
FLO X Y              ; escalares flotantes
ENT V(10)            ; arreglo entero V(0)..V(10)
FLO M(4)             ; arreglo flotante M(0)..M(4)
```

- `V(10)` declara automáticamente `V(0)`, `V(1)`, …, `V(10)`: **11 elementos**.
- Se admiten sufijos constantes, variables y anidados: `A(3)`, `A(I)`, `A(I(3))`.
- **No** se admiten expresiones dentro del paréntesis: `A(I+1)` es error. Calcule el índice antes (`I + 1 = K`) y use `A(K)`.

### 3.3 Coste de memoria

| Elemento | Palabras |
|---|---|
| Escalar `ENT` o `FLO` | **2** en página 0 |
| Elemento de arreglo numérico (`ENT` o `FLO`) | **2** |
| Arreglo `v(N)` numérico | `2 × (N+1)` |
| Buffer de caracteres `v(N)` sin `STRPK` | `N+1` |
| Buffer de caracteres `v(N)` con `STRPK` | `ceil((N+1)/2)` |

Un `ENT A(100)` reserva **202 palabras**. Los escalares declarados y nunca usados **no se emiten**; los arreglos declarados **siempre se reservan**.

- Máximo recomendado y validado: **32 escalares** (64 palabras de página 0).
- Cada escalar ocupa 2 palabras porque el valor es de 24 bits y la máquina tiene palabras de 12 bits.

## 4. Números de referencia

Los números de referencia son etiquetas numéricas de salto:

```leal
(1) A + B = C
(2) PER C
VEA (1)
```

- Se escriben al principio de una línea del cuerpo, entre paréntesis: `(n)`.
- Deben ser enteros positivos; no se usa el `0`.
- No es necesario que sean consecutivos: `(1)` y `(3)` pueden convivir.
- `REF (n)` declara el rango máximo, pero **no crea las etiquetas**.
- El compilador amplía el rango si encuentra etiquetas en el cuerpo; por eso `REF` es opcional si todas las etiquetas usadas están definidas.
- Todo `(n)` usado en un salto debe estar definido en el cuerpo. Si no, el compilador da «Número de referencia (n) usado en el programa pero nunca definido como (n) en el cuerpo».
- `REF 3` (sin paréntesis) es error de sintaxis.

## 5. Operandos y expresiones

| Forma | Significado |
|---|---|
| `A` | Variable escalar |
| `A(3)` | Elemento con subíndice constante |
| `A(I)` | Elemento con subíndice variable |
| `A(I(3))` | Subíndice compuesto |
| `4` / `1.56` | Constante numérica |

La evaluación es **estrictamente de izquierda a derecha, sin precedencia y sin paréntesis de prioridad**:

```leal
A + B - C : D - E / F = G
; significa G = ((((A + B) - C) : D) - E) / F
```

No existen `AND` ni `OR` lógicos. Los condicionales se construyen con saltos.

## 6. Asignación y aritmética

### 6.1 Forma general

La asignación se escribe **expresión = destino**:

```leal
A + B = C        ; C = A + B
5 = A            ; A = 5
A + B : C = D    ; D = (A + B) * C, no A + (B*C)
-A = A           ; cambia el signo de A
```

- `C = A + B` es error: el destino de una operación va a la derecha.
- En una **copia simple** se aceptan los dos órdenes: `A = B` copia `A` en `B`; `B = A` copia `B` en `A`.
- Si el lado derecho es un literal y el izquierdo una variable de una letra, el compilador los intercambia: `A = 1` equivale a `1 = A`.

### 6.2 Operadores

| Operador | Operación |
|---|---|
| `+` | Suma |
| `-` | Resta |
| `:` | Multiplicación (el teletipo histórico no tenía `*`) |
| `/` | División entera, truncada hacia cero |

La división trunca hacia cero: `-100/3 = -33`, no `-34`.

### 6.3 Asignaciones encadenadas

Varias asignaciones pueden ir en una línea. Cada tramo intermedio recibe el valor anterior y es el punto de partida del siguiente:

```leal
A + B = C : 2 = D      ; C = A+B;  D = C:2
A + B = C / D = F      ; C = A+B;  F = C/D
5 = A = B              ; A = 5;    B = A
```

El tramo intermedio debe empezar por su variable destino; puede llevar subíndice o continuar con una operación: `A+B = C+2 = D`, `A+B = X(0):2 = C`. El destino de cada tramo debe ser una variable o un elemento de arreglo, no una expresión.

### 6.4 Signos y negación

- `-A = A` cambia el signo de `A`.
- `A NEG = B` es la forma funcional equivalente.
- No se admiten dos signos seguidos: `-A:-B` da error. Use variables intermedias.

## 7. Tipos numéricos y límites

### 7.1 Enteros (`ENT`)

| Concepto | Valor |
|---|---|
| Representación | 24 bits con signo, complemento a dos |
| Rango | −8 388 608 … +8 388 607 |
| Literal máximo positivo seguro | +8 388 607 |
| Literal máximo negativo seguro | −8 388 607 |
| `8388608` como literal | Error de compilación (regla D2) |
| Overflow en `+`, `-`, `:` | `E:DESB` y parada; **sin envoltura** |
| División por cero | Divisor variable: `E:DIV0` y parada; divisor literal 0: error de compilación |
| Redondeo de división | Truncado hacia cero |
| Constante con decimales | Redondeada a entero, con aviso |

El valor `−8388608` solo se alcanza por aritmética (por ejemplo, `-8388607 - 1`); como literal positivo `+8388608` no cabe.

### 7.2 Flotantes (`FLO`)

| Concepto | Valor |
|---|---|
| Formato | 24 bits en 2 palabras: mantisa 18 (1 signo + 17 fracción) + exponente 6 (1 signo + 5 valor) |
| Mantisa | Normalizada: `0.1xxx × 2^exp` |
| Rango | ≈ ±2,15×10⁹; máximo exacto `(1−2⁻¹⁷)·2³¹ = 2 147 467 264` |
| Exponente | −31 … +31 (base 2) |
| Precisión efectiva | ≈ 5,11 dígitos decimales (17 bits fraccionarios) |
| Mínimo positivo no nulo | `2⁻³² ≈ 2,33×10⁻¹⁰` |
| Redondeo | *Round-half-up* en `PACK_FLT` |
| Overflow / underflow | Exponente fuera de −31…+31: `E:DESB` y parada (FMUL/FDIV); subflujo → `+0.0` silencioso |
| División por cero | Igual que enteros: `E:DIV0` y parada |
| `PEN` / `F2I` fuera de rango | `|v| ≥ 2²³` → `E:DESB` |

Los literales decimales se cuantizan a 17 bits: `0.1` y `1234.5678` no son exactos; este último queda `1234.5625`.

### 7.3 Mezcla de enteros y flotantes

El **tipo del destino** decide la aritmética:

- Si el destino es `FLO`, las variables `ENT` de la expresión se convierten automáticamente a flotante con `NOR`.
- Si el destino es `ENT`, la expresión debe ser entera. Para usar un valor flotante, conviértalo explícitamente con `PEN` (floor).

```leal
A + J = C        ; C FLO: J (ENT) se convierte a J.0 y se suma en flotante
J + A = C        ; ídem, con el entero a la izquierda
J NOR + A = C    ; forma explícita equivalente
A PEN = J        ; conversión explícita de flotante a entero (floor)
```

`NOR` sobre una variable `FLO` sigue siendo error. La conversión automática no cambia el tipo de las variables ni del destino.

## 8. Funciones

Las funciones se aplican sobre la expresión inmediatamente anterior:

```leal
A NEG = B              ; B = −A
A + B NEG = C          ; C = −(A + B)
A - B EAB = C          ; C = |A − B| (enteros)
-5.0 FAB = D           ; D = 5.0 (flotante)
123.456 PEN = D        ; D = 123 (entero, floor)
4.0 INV = D            ; D = 0.25
100.0 LOG = D          ; D = 2.0
1.5707963 SEN = D      ; D ≈ 1.0
0.0 COS = D            ; D ≈ 1.0
1.0 ACT = D            ; D ≈ 0.785398 (radianes)
123.456 FRA = D        ; D ≈ 0.456 (en [0, 1))
9.0 RAC = D            ; D = 3.0
```

| Mnemónico | Función |
|---|---|
| `NEG` | Negación (`−v`) |
| `EAB` | Valor absoluto entero (`|v|`) |
| `FAB` | Valor absoluto flotante |
| `PEN` | Parte entera: **floor** `⌊A⌋`; no trunca hacia cero. `PEN(−3.4) = −4` |
| `FRA` | Parte fraccionaria: `A − ⌊A⌋`, siempre en `[0, 1)` |
| `NOR` | Normalizar entero → flotante; también es la conversión automática en expresiones mixtas |
| `INV` | Inverso multiplicativo `1/A`, solo flotante; `INV 0` → `E:DIV0` |
| `SEN` | Seno, argumento en radianes. Precisión absoluta ≲ 5·10⁻⁵ para `|x| ≲ 1000`; `|x| ≥ ~2,6·10⁷` → `E:DESB` |
| `COS` | Coseno, argumento en radianes. Misma precisión y límite que `SEN` |
| `ACT` | Arcotangente, resultado en radianes; argumento flotante cualquiera; precisión absoluta ≲ 5·10⁻⁵ |
| `LOG` | Logaritmo base 10; precisión relativa ≲ 2·10⁻⁵; `x ≤ 0` → `E:DESB` |
| `LNE` | Logaritmo natural; precisión relativa ≲ 2·10⁻⁵; `x ≤ 0` → `E:DESB` |
| `EXP` | Exponencial; precisión relativa ≲ 2·10⁻⁵; desbordamiento → `E:DESB` |
| `RAC` | Raíz cuadrada; precisión ≤ 1 ULP; `RAC 0 = 0`; argumento negativo → `E:DESB` |

`PEN` y `FRA` cumplen `PEN(A) + FRA(A) = A`, con `FRA` siempre en `[0, 1)`:

| `A` | `2.1` | `3.0` | `−3.4` | `−0.1` | `−0.5` | `0.5` | `0.0` |
|---|---|---|---|---|---|---|---|
| `PEN(A)` | 2 | 3 | −4 | −1 | −1 | 0 | 0 |

No se admiten funciones como operando delante de otro operador: `A + NEG = B` da error.

## 9. Saltos condicionales

Forma general: `EXPRESIÓN ? CONDICIÓN (n)`.

| Condición | Salta si el resultado es… |
|---|---|
| `SSP` | positivo (`> 0`) |
| `SSN` | negativo (`< 0`) |
| `SSC` | cero (`= 0`) |
| `SNP` | no positivo (`≤ 0`) |
| `SNN` | no negativo (`≥ 0`) |
| `SNC` | no cero (`≠ 0`) |

```leal
I - 10 ? SSP (20)   ; si I > 10, salta a (20)
I - 5 ? SSC (30)    ; si I == 5, salta a (30)
```

No hay `AND` ni `OR`. Un `IF X<0 OR X>7` se escribe en dos líneas:

```leal
X - 1 ? SSN (error)
X - 8 ? SSP (error)
```

`VEA (n)` es el salto incondicional.

## 10. Ciclos (`CIC` / `REP`)

El ciclo se abre con una etiqueta y `CIC`, y se cierra con `REP`:

```leal
(1) CIC
        ... cuerpo ...
REP (1) ...
```

- El espacio antes del paréntesis es obligatorio: `REP (1)`, nunca `REP(1)`. Igual para `VAS (n)`, `VEA (n)` y `FRU (n)`.
- Antes del `CIC` debe asignarse un valor a la variable de control.
- El tipo del ciclo lo decide la variable de control: entero o flotante, pero no ambos.

### 10.1 Sin argumentos: `REP (n)`

Bucle infinito. Se sale con un salto interno.

### 10.2 Una variable: `REP (n) I`

Igual que el anterior, con `I` como variable de control.

### 10.3 Tipo 1 — lista: `REP (n) I'v1'v2'...`

La primera vuelta usa el valor previo de `I`; después se recorren los valores de la lista. Con `I = 0` y `REP (1) I'10'20'30`, el cuerpo se ejecuta 4 veces con `I = 0, 10, 20, 30`.

### 10.4 Tipo 2 — incremento y límite: `REP (n) I, J, K`

`I = I + J` mientras `I <= K`. Si el valor inicial `I0 > K`, el cuerpo se ejecuta una sola vez.

```leal
(1) CIC
        LEC X(I)
        X(I) + X = X
REP (1) I, 1, N
```

### 10.5 Tipo 3 — incremento y conteo: `REP (n) I, J'K`

Repite `K` veces, incrementando `I` en `J` en cada vuelta. Si `K` es constante, debe ser un entero entre 0 y 4095.

```leal
(3) CIC
        I + J = L
REP (3) I, M'K
```

Los tres tipos funcionan con constantes y con variables de control, y admiten control entero o flotante.

## 11. Subrutinas (`VAS`, `CRU`, `FRU`)

```leal
VAS (1) A, N, 7.1, D(0), D(1), D(2)
(1) CRU F(0) J F(1)
FRU (1) H(6) H(9)
```

- `VAS (n)` llama a la subrutina `(n)` y pasa los argumentos.
- `(n) CRU ...` declara los receptores de entrada.
- `FRU (n) ...` declara los emisores de salida.
- Tras el `FRU`, la máquina regresa al `VAS` que causó la entrada.
- Cada subrutina tiene un único `CRU` y un único `FRU` con el mismo `(n)`.
- La suma de receptores + emisores debe coincidir con el número de argumentos del `VAS`.
- No se admiten constantes en `CRU` ni en `FRU`.
- Las constantes de la llamada se codifican según el tipo del receptor: un literal flotante hacia receptor `FLO` se codifica como flotante; hacia receptor `ENT` avisa y redondea.
- Las subrutinas pueden llamar a otras subrutinas (anidamiento).
- Máximo **16 argumentos** por subrutina: `PARM` son 32 palabras, 2 por argumento. Si necesita más, empaquete valores en un arreglo.
- En `VAS`, `CRU` y `FRU` se admiten comas y/o espacios como separadores.
- El `FRU` puede llevar una etiqueta de salto al frente: `(40) FRU (2)`.
- El primer `FIN` detiene la compilación: declare las subrutinas antes del `FIN` final y termine el cuerpo principal con `ALT`.

### 11.1 Subíndice acotado con constante

En `VAS`, `CRU` y `FRU`, la forma `X(m)` con `m` literal expande a `X(0), X(1), …, X(m)`:

| Forma | Expansión |
|---|---|
| `X(0)` | `X(0)` (1 elemento) |
| `X(1)` | `X(0), X(1)` (2 elementos) |
| `X(2)` | `X(0), X(1), X(2)` (3 elementos) |

> **Atención:** `CRU X(0) X(1) X(2)` expande a **6** elementos (con duplicados), no a 3. Para pasar 3 elementos escriba `CRU X(2)`.

El subíndice acotado con variable (`A(I)`, tamaño variable en runtime) **no está implementado**; solo el tamaño constante `A(m)`.

## 12. Entrada y salida

| Instrucción | Uso |
|---|---|
| `LEC v` | Lee un entero de 24 bits desde el teletipo hasta CR. Si `v` es `FLO`, lee un flotante. |
| `PER v` | Imprime un entero de 24 bits **sin CR+LF**; usa `LIN` para saltar de línea. |
| `PER v` (`v` `FLO`) | Imprime el flotante con el formato `DIG m,n`: signo + parte entera + punto + `n` decimales + **3 espacios, sin CR+LF**. |
| `DIG m,n` | Fija el formato del `PER` flotante: `m` dígitos de parte entera y `n` decimales (por defecto `4,2`). |
| `LIN` | Emite CR+LF. |
| `TIT ...` | Imprime un título, intercalando LTRS/FIGS automáticamente. **No** añade CR+LF. |
| `ESP n` | Imprime `n` espacios. |
| `LCA v` | Lee una cadena hasta CR en el buffer `v`. |
| `PCA v` | Imprime la cadena del buffer `v` hasta el terminador `0`. |
| `MDI v` | Desplaza la cadena una posición a la izquierda; deja un espacio antes del terminador. |
| `MID v` | Desplaza la cadena una posición a la derecha; `v(0)` queda en blanco y se pierde el último carácter. |
| `ALT` | Detiene la ejecución. |
| `FIN` | Termina la compilación. |

### 12.1 `PER` y `DIG`

- `PER` **no** emite CR+LF: el papel queda en la misma línea y los `PER` consecutivos se concatenan. El único salto automático es el de `E:DESB`, que entra en línea nueva.
- `TIT` tampoco salta: use `LIN` para líneas separadas.
- `DIG m,n`: `m` se satura a 0..8 y `n` a 1..5, con aviso en el listado (`n=0` → 1). La parte entera siempre sale completa; `m` queda como ancho declarado, sin chequeo de exceso.
- La impresión flotante **trunca** (no redondea) sobre el valor almacenado.
- La forma `DIG n` (un solo argumento, exponencial) es **error de compilación**: el modo exponencial está pendiente. Use `DIG m,n`.

### 12.2 `LEC`

- `LEC v` con destino `FLO` acumula los dígitos (hasta 8 significativos).
- Si la parte entera no cabe (`|v| ≥ 2²³`), imprime `E:DESB` y para.
- Si son los decimales los que no caben (≥ 9 dígitos significativos), trunca en silencio y continúa.
- La mezcla de `LEC` enteras y flotantes en un programa (modo dual) funciona.

### 12.3 Buffers de caracteres (`LCA`, `PCA`)

En LEAL no existe un tipo «carácter» ni «cadena»: los arreglos se declaran `ENT` o `FLO` y el programador decide cómo usarlos.

- `LCA v` lee una cadena hasta CR en el buffer `v`. **No** sobrescribe `v(N)` (terminador). Si el buffer se llena, ignora los caracteres extra.
- `PCA v` imprime hasta encontrar el terminador `0`.
- `MDI v` desplaza una posición a la izquierda: «ABC» → «BC␠».
- `MID v` desplaza una posición a la derecha: «ABC» → «␠AB».
- Con subíndice (`MDI T(1)`) la operación arranca en esa posición; en modo packed solo se admite la forma entera (`m = 0`).
- El buffer siempre termina en `0`. Los arreglos se inicializan a cero, así que un buffer vacío tiene terminador en la posición 0.
- **No mezcle usos:** si `X(10)` se usa con `LCA X`, no la use también como arreglo de enteros (`X(2) = 5`). El compilador avisa, pero la mezcla corrompe la memoria. `FLO X(4)` + `LCA X` es error; `ENT X` escalar + `LCA X` también es error: `LCA` requiere un arreglo con `N` declarado.

El modo **STRPK** (botón del editor) empaqueta 2 caracteres por palabra para `TIT` y los buffers `LCA`/`PCA`. Apagado, usa 1 carácter por palabra.

## 13. Memoria y página 0

La CID 201-A tiene **4096 palabras** de 12 bits, en 32 páginas de 128 palabras. El layout compacto de CIDcoder-A coloca los bloques en este orden, cada uno alineado al inicio de una página:

```text
página 0:  bootstrap + escalares + datos runtime + tabla de punteros
subrutinas de usuario (CRU/FRU)
rutinas runtime y stubs (solo las usadas)
tablas de conversión de dígitos
cuerpo del programa          [bodyStart, arrayBase)
arreglos                     [arrayBase, arrayEnd)
```

- El cuerpo se emite en `bodyStart` (múltiplo de 128) y los arreglos en `arrayBase = align128(bodyStart + bodyWords)`.
- La ventana real del cuerpo es `[bodyStart, arrayBase)`.
- La comprobación de capacidad es del programa completo: `arrayEnd ≤ 0o10000`.
- El vector de arranque está en `0o000` (`VEA I 0001`) y `0o001` apunta a `bodyStart`.

### 13.1 La página 0

Es el recurso más escaso: 128 palabras para bootstrap, escalares, datos del runtime y tabla de punteros.

- Cada escalar ocupa 2 palabras; más de 32 escalares desbordan la página 0.
- Cada etiqueta de salto, `VAS`/`CRU`/`FRU` o variable runtime puede añadir punteros.
- Si la tabla de punteros invade el cuerpo, el compilador no puede colocar el programa.
- Los arreglos viven fuera de página 0, donde hay más espacio; para muchos valores poco usados, un arreglo es más eficiente que decenas de escalares.

### 13.2 Qué pesa más

1. **Rutinas de flotante y E/S.** Un programa mínimo con una operación mixta y un `PER` flotante ya ocupa ~1000 palabras aunque su cuerpo sea de 43.
2. **Arreglos.** Cuestan 2 palabras por elemento, sea `ENT` o `FLO`. Declare solo el tamaño necesario y elimine los que no use.
3. **Cuerpo.** Use ciclos en lugar de repetir código; elimine ramas y subrutinas muertas.
4. **Página 0.** Cada salto, subrutina o variable runtime puede añadir punteros.
5. **Alineación a página.** Un cuerpo que termina justo después de una frontera puede dejar hasta 127 palabras de relleno.
6. **Cadenas.** `STRPK` ahorra en buffers largos, pero el runtime packed cuesta unas 6 palabras y la alineación puede comerse la ganancia.

### 13.3 Mediciones reales

| Programa | Cuerpo | Fin de memoria | Página 0 usada | Palabras en LST |
|---|---|---|---|---|
| Suma `A+B` mínima con `PER` | 22 w (0o1000..0o1026) | 0o1200 | 34 | 285 |
| `ENT A(100)` + `PER A(1)` | 28 w (0o600..0o634) | 0o1312 | 31 | 413 |
| Flotante `1.5` + mixta + `PER` | 43 w (0o2400..0o2453) | 0o2600 | 66 | 1043 |
| `GO_compacto` | 900 w | 0o4162 | 83 | — |
| `CHECKERS` (UNPK; medición de la Ayuda) | 1838 w | 0o6002 | 114 | — |

«Fin de memoria» es `arrayEnd`, la última dirección ocupada. El LST cuenta solo las palabras emitidas, no los huecos de alineación.

### 13.4 Errores de capacidad

| Mensaje | Causa | Solución |
|---|---|---|
| `Demasiadas escalares: página 0 desborda en 0o…` | Más de 32 escalares (64 palabras). | Reutilice variables o agrupe las poco usadas en un arreglo. |
| `Página 0: la tabla de punteros (…) invade el cuerpo en 0o…` | Demasiados punteros de salto/runtime. | Reduzca etiquetas, saltos, subrutinas o variables runtime. |
| `Memoria insuficiente: rutinas X w + tablas Y w + cuerpo Z w + arreglos W w = fin 0o… (límite 0o10000)` | El programa completo no cabe en 4096 palabras. | Reduzca arreglos y cuerpo; quite rutinas o funciones. |
| `Subrutina con N argumentos excede el máximo de 16 slots (32 palabras PARM)` | Más de 16 argumentos en un `VAS`. | Empaquete valores en un arreglo. |

## 14. Compilación y flujo de trabajo

La cadena tiene dos pasos:

1. **Compilación LEAL → LEN-1.** El compilador genera texto LEN-1 y aplica *peephole*: elimina `LAC` redundantes, combina microinstrucciones compatibles y optimiza la página 0 (punteros muertos, punteros y constantes equivalentes, variables runtime no usadas).
2. **Ensamblado LEN-1 → memoria y listado.** El ensamblador valida etiquetas, direcciones, solapamientos y combinaciones de microinstrucciones.

En el editor:

- **⚙ Compile** / `F9` compila y ensambla. Los mensajes aparecen en el editor, en el panel **LEN generado** y en el **LST**.
- **▶ Execute** compila, carga la memoria y abre el ejecutor.
- **☰ LST**, **⚡ Exec** y **🎞 Tape** muestran u ocultan los paneles.
- **Save .len1** guarda el LEN-1 generado; **Save .lst** guarda el listado.
- El panel de mensajes informa «N eliminadas, M combinadas».
- Un clic en el número de línea del editor pone o quita un punto de ruptura **en modo LEN-1**. En modo LEAL, los BP se marcan sobre las líneas del LEN-1 generado (panel **LEN generado**) o en las filas del **LST**; **⊘ BPs** los borra todos.

Atajos: `Ctrl+S` guardar, `Ctrl+Shift+S` guardar como, `Ctrl+O` abrir, `F9` compilar.

## 15. Ejemplos verificados

### 15.1 Suma mínima

```leal
LEAL
TITULO SUMA DEMO
ENT A B C
FLO
REF (1)
5 = A
3 = B
A + B = C
PER C
FIN
```

Salida: `8`.

### 15.2 Suma 1..5 con ciclo

```leal
LEAL
SUMA 1 A 5
ENT I S
FLO
REF (1)
0 = S
1 = I
(1) CIC
        S + I = S
REP (1) I, 1, 5
PER S
FIN
```

Salida: `15`.

### 15.3 Subrutina

```leal
LEAL
SUBRUTINA SUMA
ENT A B C X Y Z
FLO
REF (1)
5 = A
3 = B
VAS (1) A, B, C
PER C
ALT
(1) CRU X Y
X + Y = Z
FRU (1) Z
FIN
```

Salida: `8`.

### 15.4 Flotante con formato

```leal
LEAL
FLOTANTE
ENT
FLO X
REF (1)
1.5 = X
DIG 4,2
PER X
FIN
```

Salida: `1.50   ` (tres espacios finales, sin CR+LF; la parte entera no lleva relleno).

### 15.5 Buffer de caracteres

```leal
LEAL
BUFFER
ENT T(20)
FLO
REF (1)
LCA T
PCA T
FIN
```

Lee una línea del teletipo y la imprime hasta el terminador. No procesa caracteres sueltos: use un buffer de 1 (`ENT T(0)`) y copie con `J(m) = T(0)` si necesita un carácter.

### 15.6 Condicional

```leal
LEAL
CONDICIONAL
ENT A
FLO
REF (1)
LEC A
A ? SSP (1)
0 = A
(1) PER A
FIN
```

Si `A > 0`, imprime `A`; si no, imprime `0`.

### 15.7 Asignación encadenada

```leal
LEAL
ENCADENADA
ENT A B C D
FLO
REF (1)
5 = A
3 = B
A + B = C : 2 = D
PER D
FIN
```

Salida: `16`.

## 16. Errores frecuentes

- Poner el destino de una operación a la izquierda: `C = A + B` es error; escriba `A + B = C`. En una copia simple (`A = B`, `A = 1`) se aceptan ambos órdenes.
- Usar `/` como comentario: en LEAL el comentario es `;`.
- Usar paréntesis para prioridad: no existen.
- Dos signos seguidos: `-A:-B`, `-A/-B` → «operando inválido».
- Usar `(n)` en un salto sin definirlo en el cuerpo. `REF (n)` no crea etiquetas.
- `REP` sin `CIC` o con un número distinto.
- Pegar la palabra clave al paréntesis: `REP(1)`, `VAS(1)`, `VEA(3)`, `FRU(1)`.
- Pegar palabras clave: `DIG5`, `CRUZ`, `TITULO` en el cuerpo.
- `VAS` con más argumentos que receptores + emisores.
- Constantes en `CRU` o `FRU`.
- Demasiadas variables o ciclos anidados agotan la página 0.
- Literal fuera de rango (`8388608` o mayor): error de compilación.
- Mezclar una variable como buffer (`LCA X`) y como arreglo numérico (`X(2) = 5`): aviso, pero corrupción silenciosa.
- `FLO X(4)` + `LCA X`: error; los buffers deben ser `ENT`.
- `ENT X` escalar + `LCA X`: error; `LCA` requiere un arreglo con `N`.
- `CRU X(0) X(1) X(2)`: expande a 6 elementos; use `CRU X(2)` para 3.

## 17. Concordancia con la Ayuda del editor

Este manual sigue la Ayuda integrada de `_51h`/`_51.2` y corrige las afirmaciones que quedaron obsoletas en el README anterior:

| Punto | Regla vigente |
|---|---|
| Tamaño de un escalar `ENT` | **2 palabras**, igual que `FLO`; no 1. |
| `TITULO` | Declaración inerte en el encabezamiento; no imprime nada. Para imprimir use `TIT`. |
| `PER` | No emite CR+LF; los `PER` consecutivos se concatenan. Use `LIN`. |
| `DIG m,n` | Formato aritmético; `DIG n` exponencial es error de compilación. |
| `REF` | Opcional si las etiquetas están definidas; no crea etiquetas; no exige numeración consecutiva. |
| `EAB` | Función real de valor absoluto entero; no es el nombre del lenguaje. |
| Cadenas | `LCA`/`PCA` operan sobre el buffer completo; no hay acceso a un carácter suelto. |
| Funciones trascendentes | `RAC`, `EXP`, `LNE`, `LOG`, `SEN`, `COS`, `ACT` están implementadas en el runtime; no son hardware de la CID 201-A. |
| Subrutinas | Máximo 16 argumentos (32 palabras `PARM`); `CRU X(2)` expande a 3 elementos. |
| Página 0 | 32 escalares como máximo; los arreglos viven fuera de página 0. |
| Puntos de ruptura | En LEAL se marcan en las líneas del LEN-1 generado o en las filas del LST; en LEN-1, en el gutter del editor. |

La sección «Listado, ejecutor y depuración» de la Ayuda es compartida con LEN-1: se implementa una sola vez y se muestra como §12d en LEAL y §12 en LEN-1. La fuente de verdad es el emulador. Si una tabla histórica discrepa, este manual documenta la diferencia en la sección correspondiente o en `DOC/MANUAL_LEN1.md` para LEN-1.
