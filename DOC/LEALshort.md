# MANUAL DEL LENGUAJE LEAL
## Especificación, Historia, Uso y Guía de Programación para la CID 201-A

---

### 1. Introducción e Historia

El **LEAL** (*LEnguaje ALgorítmico*) es un lenguaje de programación de alto nivel creado en Cuba entre 1970 y 1971 por el Centro de Investigación Digital (CID), bajo la dirección de Víctor Llopis Zayas-Bazán. Fue diseñado específicamente para las minicomputadoras de la serie CID, comenzando por la **CID 201-A**.

Su diseño tomó conceptos del *Autocode* de la computadora británica Elliot 803B Mark III. A diferencia de lenguajes como FORTRAN o BASIC que usan paréntesis y precedencia de operadores, LEAL utiliza una notación polaca y simbólica estricta, evaluando las expresiones **siempre de izquierda a derecha**.

Con el tiempo, el lenguaje evolucionó en tres versiones principales:
*   **LEAL-A:** Orientado al cálculo científico. Funcionaba con cintas de papel perforado y carecía de comentarios.
*   **LEAL-B:** Más evolucionado. Introdujo comentarios (usando `;`), mejoró el manejo de subrutinas, la gestión de periféricos (teletipos, discos) y la lectura/escritura de caracteres.
*   **LEAL-F:** Desarrollado para la CID 300/10. Combinaba cálculo científico con gestión económica.

En la actualidad, el emulador **CIDcoder-A** implementa un dialecto moderno compatible con el LEAL original, adaptado a las limitaciones de la arquitectura de 12 bits.

---

### 2. Estructura General de un Programa

Todo programa en LEAL se divide en dos partes principales: el **Encabezamiento** y el **Cuerpo**.

#### 2.1. El Encabezamiento
Contiene la configuración inicial del programa y la declaración de variables. Es de cumplimiento obligatorio.

```leal
LEAL                 ; Primera línea obligatoria
TITULO MI PROGRAMA   ; Título descriptivo (opcional)
ENT A B C D          ; Declaración de variables enteras (24 bits)
FLO X Y Z            ; Declaración de variables flotantes (24 bits)
REF (10)             ; Declaración del número de referencia máximo
```

#### 2.2. El Cuerpo
Contiene las instrucciones ejecutables. Puede incluir asignaciones, saltos, bucles, E/S y subrutinas. Debe finalizar con la instrucción `FIN`.

---

### 3. Variables, Tipos y Números de Referencia

#### 3.1. Variables
*   **Nomenclatura:** Se utiliza **exclusivamente una letra mayúscula de la A a la Z** (se excluye la Ñ). Nombres como `OP`, `T1` o `VAR` generarán un error de compilación.
*   **Tipos:** Una variable es Entera (`ENT`) o Flotante (`FLO`). El tipo es inmutable y se define en el encabezamiento. No se permite mezclar tipos en una misma operación.
*   **Sufijos (Arreglos):** Se permiten subíndices entre paréntesis. Ejemplo: `A(0)`, `A(I)`. El tamaño máximo se declara al definir la variable: `ENT V(10)` crea un arreglo de 0 a 10.

#### 3.2. Números de Referencia (Etiquetas)
Los números de referencia sirven como etiquetas para los saltos.
*   Se escriben al inicio de la línea, entre paréntesis y seguidos de la instrucción: `(1) A + B = C`.
*   Deben ser constantes enteras positivas (no se permite el `0`).
*   **Regla del compilador:** Deben numerarse de forma consecutiva `(1)`, `(2)`, `(3)`... y la declaración `REF(n)` debe indicar exactamente el número más alto utilizado en el cuerpo. Si usas hasta la etiqueta `(15)`, debes poner `REF(15)`.

---

### 4. Asignación, Aritmética y Evaluación

La regla de oro de LEAL es: **La asignación se escribe con el destino a la derecha** (`expresión = destino`), y **las operaciones se evalúan estrictamente de izquierda a derecha, sin precedencia**.

#### 4.1. Operadores
*   `+` : Suma
*   `-` : Resta
*   `:` : Multiplicación
*   `/` : División (trunca hacia cero)

#### 4.2. Ejemplos de Asignación
```leal
A + B = C         ; Equivale a C = A + B
5 = A             ; Equivale a A = 5
A + B : C = D     ; Equivale a D = (A + B) * C (¡NO A + (B*C)!)
-A = A            ; Cambia el signo de A
```

#### 4.3. Asignación Encadenada
Permite realizar varias asignaciones en una línea. El tramo intermedio debe empezar por la variable que recibirá el resultado anterior.
```leal
A + B = C : 2 = D  ; C = A+B, luego D = C * 2
```

#### 4.4. Funciones Matemáticas
Las funciones se aplican sobre el resultado de la expresión anterior.
*   `SEN`, `COS`, `ACT`, `LOG`, `LNE`, `EXP`, `RAC` (Raíz cuadrada). 
    *   *Nota de compatibilidad:* En CIDcoder-A, estas funciones trascendentes aún no están implementadas en hardware. Deben emularse con series de Taylor o aproximaciones (como Newton-Raphson para `RAC`).
*   `PEN` (Parte entera), `FRA` (Parte fraccionaria).
*   `NOR` (Normalizar de entero a flotante), `EAB` (Valor absoluto entero), `FAB` (Valor absoluto flotante).
*   `NEG` (Negación).

---

### 5. Control de Flujo (Saltos y Condicionales)

LEAL no tiene bloques `IF/ELSE`. Utiliza saltos condicionales basados en el valor de una expresión.

#### 5.1. Salto Incondicional
*   `VEA (n)`: Salta a la etiqueta `(n)`.

#### 5.2. Saltos Condicionales
Se evalúa una expresión y se salta según el resultado:
*   `SSP (n)`: Salta si el resultado es **Positivo** (> 0).
*   `SSN (n)`: Salta si el resultado es **Negativo** (< 0).
*   `SSC (n)`: Salta si el resultado es **Cero** (= 0).
*   `SNP (n)`: Salta si el resultado es **No Positivo** (≤ 0).
*   `SNN (n)`: Salta si el resultado es **No Negativo** (≥ 0).
*   `SNC (n)`: Salta si el resultado es **No Cero** (≠ 0).

*Ejemplo:*
```leal
I - 10 ? SSP (20)  ; Si I > 10, salta a (20)
I - 5 ? SSC (30)   ; Si I == 5, salta a (30)
```

#### 5.3. Bucles (CIC / REP)
El bucle se inicia con una etiqueta y la instrucción `CIC`, y se cierra con `REP`.
*   `REP (n)`: Bucle infinito (se sale usando `VEA`).
*   `REP (n) I'v1'v2'v3`: Recorre la lista de valores (CIC tipo 1). 
    *   *Advertencia:* Ejecuta N+1 iteraciones (la primera usa el valor previo de I).
*   `REP (n) I, paso, límite`: Bucle numérico clásico (CIC tipo 2). `I = I + paso` mientras `I <= límite`.
*   `REP (n) I, paso' K`: Bucle de conteo exacto (CIC tipo 3). Repite K veces incrementando I en paso.

---

### 6. Subrutinas (VAS, CRU, FRU)

El uso de subrutinas permite modularidad. Se definen en el cuerpo del programa. 

*   **Llamada:** `VAS (n) arg1, arg2, arg3`
*   **Declaración de entrada:** `(n) CRU in1, in2`
*   **Declaración de salida:** `(n) FRU (n) out1`

*Regla de oro:* La cantidad de argumentos en `VAS` debe coincidir exactamente con la suma de receptores en `CRU` y emisores en `FRU`.

---

### 7. Entrada y Salida (E/S)

*   **LEC X:** Lee un número entero desde el teletipo y lo guarda en `X`.
*   **PER X:** Imprime el valor de `X`.
*   **TIT HOLA:** Imprime texto literal.
*   **LIN:** Salto de línea (CR+LF).
*   **ESP n:** Imprime `n` espacios.
*   **DIG m,n:** Define el formato de impresión flotante (m dígitos enteros, n decimales).
*   **LCA A:** Lee una cadena de caracteres en el arreglo `A`.
*   **PCA A:** Imprime la cadena almacenada en `A`.
*   **ALT:** Detiene la ejecución.
*   **FIN:** Fin de la compilación.

---

### 8. Guía de Programación para el Hardware y el Compilador

Para que un programa compile correctamente y no agote los recursos de la CID 201-A (4096 palabras de memoria de 12 bits), el programador debe tener en cuenta las siguientes reglas estrictas:

#### 8.1. Gestión de la Memoria (Página 0 y Arreglos)
*   **Página 0:** Las primeras 128 palabras de memoria son el recurso más escaso. Aquí se guardan las variables escalares, los punteros de salto y el paso de parámetros. **No declares decenas de variables sueltas.** Si necesitas más de 15 variables, usa un arreglo.
*   **Arreglos:** Los arreglos se ubican automáticamente al final de la memoria, donde hay amplio espacio. Declarar un arreglo `FLO A(50)` es mucho más eficiente en espacio que declarar 50 variables flotantes.
*   **Flotantes:** Cada variable flotante (`FLO`) ocupa 2 palabras de memoria. Los enteros (`ENT`) ocupan 1 palabra.
*   **Subrutinas:** Pasar muchos argumentos a una subrutina (`VAS`) llena rápidamente la página 0. Si una subrutina necesita muchos datos, es mejor que lea y escriba directamente en variables globales.

#### 8.2. Asignación y Aritmética
*   **Subíndices:** LEAL **no permite** expresiones dentro de los paréntesis. `A(I+1)` es un error de compilación. Debes calcular el índice antes: `I + 1 = K` y luego usar `A(K)`.
*   **Precedencia:** Al no haber precedencia, fórmulas complejas como `(A+B)*C` deben descomponerse en pasos o usar la asignación encadenada: `A + B = C : D = X`.
*   **Precisión Flotante:** La precisión es de 24 bits (aproximadamente 5 dígitos decimales). Es propensa a la acumulación de errores en bucles largos. No es apta para contabilidad exacta.
*   **Desbordamiento:** Si una suma o multiplicación entera excede los 24 bits (rango de -8,388,608 a +8,388,607), el programa se detendrá con un error `E:DESB`.

#### 8.3. Control de Flujo y Etiquetas
*   **Condicionales:** No existen `AND` ni `OR`. Un `IF X<0 OR X>7` se traduce en dos líneas consecutivas:
    ```leal
    X - 1 ? SSN (error)
    X - 8 ? SSP (error)
    ```
*   **Finalización de subrutinas:** La instrucción `FIN` le dice al compilador que deje de leer código. Si tienes subrutinas, el flujo principal debe terminar con `ALT` antes de definirlas, y el `FIN` debe estar en la última línea del archivo.
*   **REF(n):** Si usas una etiqueta `(20)`, el `REF` debe ser `(20)` o mayor. Si el `REF` es menor, la compilación fallará.

#### 8.4. Limitaciones con Cadenas (Strings)
*   Debido a un bug de diseño del compilador moderno, intentar leer un carácter individual de un arreglo de caracteres (`A(I)`) devuelve 2 palabras de memoria a la vez. 
*   **Regla:** Usa `LCA` y `PCA` solo para leer e imprimir la cadena completa (cajas opacas). No intentes procesar la cadena carácter por carácter (como en un cifrado César).

---

### 9. Ejemplos Prácticos

#### Ejemplo 1: MCD por Algoritmo de Euclides (Subrutinas y Control)
```leal
LEAL
TITULO EUCLIDES
ENT X Y R A B C
FLO
REF (3)
TIT A:
LEC X
TIT B:
LEC Y
VAS (1) X, Y, R
TIT GCD:
PER R
LIN
ALT
(1) CRU A B
(2) B ? SSC (3)
    A / B = C
    C : B = C
    A - C = C
    B = A
    C = B
    VEA (2)
(3) FRU (1) A
FIN
```

#### Ejemplo 2: Suma de Cuadrados (Bucles y Arreglos)
```leal
LEAL
TITULO CUADRADOS
ENT I N S V(10)
FLO
REF (4)
1 = I
(1) CIC
    TIT VALOR:
    LEC V(I)
    REP (1) I, 1, 10
0 = S
1 = I
(2) CIC
    V(I) : V(I) + S = S
    REP (2) I, 1, 10
TIT SUMA:
PER S
LIN
FIN
```

#### Ejemplo 3: Emulación de Módulo y Condicional Compuesto
```leal
LEAL
TITULO MODULO
ENT N D Q T
FLO
REF (2)
TIT N:
LEC N
TIT D:
LEC D
; Módulo: R = N - (N/D)*D
N / D = Q
Q : D = T
N - T = N
; Si N == 0, es divisible
N ? SSC (1)
TIT NO DIVISIBLE
LIN
VEA (2)
(1) TIT DIVISIBLE
    LIN
(2) FIN
```

#### Ejemplo 4: Interés Compuesto (Flotantes y Formato)
```leal
LEAL
TITULO INTERES
FLO P R T
ENT K
REF (2)
1000.0 = P
0.05 = R
DIG 5, 2
TIT YEAR 0:
ESP 1
PER P
LIN
1 = K
(1) CIC
    1.0 + R = T
    T : P = P
    TIT YEAR:
    ESP 1
    PER K
    TIT :
    ESP 1
    PER P
    LIN
    REP (1) K, 1, 10
(2) FIN
```
