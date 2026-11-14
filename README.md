# CIDcoder-A

**Editor, compilador LEAL, ensamblador LEN-1 y simulador de la minicomputadora cubana CID 201-A, todo en una sola página HTML.**

CIDcoder-A recupera en software la cadena de herramientas de la **CID 201-A** (ICID, Cuba, 1971): se escribe un programa en el lenguaje algorítmico **LEAL**, se compila al ensamblador **LEN-1**, se ensambla y se ejecuta en un modelo de la máquina con teletipo RFT T-51, lector y perforador de cinta, display, mapa de memoria y microcódigo. No hay instalación, build ni dependencias: se abre en el navegador.

<p align="center"><img width="1280" src="https://raw.githubusercontent.com/maikelsz/CIDcoder-A/main/CIDcoder-A.png" alt="Editor, listado LEN-1 y teletipo de la CID 201-A en CIDcoder-A"></p>

**[Abrir la demo en GitHub Pages](https://maikelsz.github.io/CIDcoder-A/CIDcoder-A.html)** · **[Repositorio en GitHub](https://github.com/maikelsz/CIDcoder-A)**

## Qué es

- **Editor** con resaltado, análisis estático y errores antes de generar LEN-1.
- **Compilador LEAL** para la CID 201-A: enteros de 24 bits, flotantes, arreglos, cadenas, subrutinas, ciclos y E/S.
- **Ensamblador LEN-1** con microinstrucciones combinables, direccionamiento directo e indirecto, listado `.lst` y detección de solapamientos.
- **Simulador** con teletipo, cinta de papel, display, mapa de memoria, microcódigo, ejecución paso a paso y breakpoints.
- **Herramientas de cinta**: generación y lectura de cintas Murray/binarias.
- **Ayuda integrada** con los manuales de LEAL y LEN-1, adaptada al modo del editor.

## La CID 201-A

| | |
|---|---|
| Palabra | 12 bits |
| Memoria | 4096 palabras (32 páginas × 128) |
| Direcciones | Octal, `0000`–`7777` |
| Reloj | 1 MHz |
| Ciclo de memoria | 1,5 µs |
| Registros | AC (12), EN (1), RM, CP, RI, RD, RP, RIN |
| Instrucciones | 59 |
| Interrupciones | No tiene; E/S por sondeo (*polling*) |
| Periféricos | Teletipo RFT T-51, lector RFT T-63, perforador, display, teclado, grabadora |

La máquina usa **complemento a dos** y representación **octal**. A nivel de 12 bits el rango con signo es −2048 a +2047; los programas construyen aritmética de 24 bits o punto flotante por software.

## LEAL

**LEAL** (*LEnguaje ALgorítmico*) fue creado en Cuba entre 1970 y 1971 en el Centro de Investigación Digital, bajo la dirección de Víctor M. Llopis Zayas-Bazán. Deriva de conceptos del *Autocode* de la Elliott 803B Mark III y se orientó al cálculo científico en las minicomputadoras CID. Se desarrolló en versiones **A**, **B** y **F**; LEAL-A es un subconjunto de LEAL-B. CIDcoder-A compila el dialecto **LEAL-B** para la CID 201-A.

La asignación se escribe **con el destino a la derecha** y las expresiones se evalúan **estrictamente de izquierda a derecha, sin precedencia**:

```leal
LEAL
TITULO SUMA DEMO
ENT A B C
FLO
5 = A
3 = B
A + B = C
PER C
FIN
```

Salida en el teletipo: `8`.

Reglas que evitan la mayoría de los errores:

- Variables de **una sola letra** (`A`–`Z`, sin Ñ); los arreglos se declaran `ENT V(10)` o `FLO M(4)`.
- `TITULO` es una declaración **inerte** del encabezamiento; para imprimir texto se usa `TIT`.
- Las etiquetas de salto son `(n)`; `REF (n)` declara el rango, pero no crea las etiquetas. Es opcional si las etiquetas están definidas.
- `PER` **no** emite CR+LF; use `LIN` para saltar de línea. `DIG m,n` fija el formato del `PER` flotante.
- La página 0 es el recurso más escaso: 128 palabras para bootstrap, escalares, datos del runtime y punteros.
- Cada escalar `ENT` o `FLO` ocupa **2 palabras**; cada elemento de arreglo también. Un `ENT A(100)` reserva 202 palabras.
- No hay acceso a un carácter suelto de una cadena: `LCA`/`PCA` operan sobre el buffer completo.

El manual completo, con tipos, funciones, ciclos, subrutinas, E/S, memoria y errores, está en **[`DOC/MANUAL_LEAL.md`](DOC/MANUAL_LEAL.md)** y en la pestaña **Ayuda** del editor.

## LEN-1

**LEN-1** (*Lenguaje Ensamblador 1*) es el ensamblador nativo de la CID 201-A. El editor lo reconoce por la extensión `.len1` y lo ensambla directamente, sin pasar por LEAL.

- **Memoria:** 4096 palabras de 12 bits, 32 páginas de 128 palabras; direcciones en octal.
- **Entrada:** la primera directiva `*` fija el punto de entrada (`*0200`); sin ella, el programa arranca en `0000`.
- **Sintaxis:** `ETIQUETA, INSTRUCCIÓN` (coma obligatoria tras la etiqueta); comentario con `/`; fin opcional con `$`.
- **Números:** octal por defecto (`47`), octal explícito (`0o47`), decimal (`0d39`), hexadecimal (`0x27`) y negativos en complemento a dos.
- **IRM:** `YLO` (AND), `SUM` (suma), `ISC` (incremento y salto), `DEL` (deposita y limpia AC), `VAS` (llamada a subrutina), `VEA` (salto).
- **Direccionamiento directo:** solo a página 0 o a la página actual. Para otra página se usa indirección (`I` o `@`), con el puntero también en página 0 o en la página actual.

```len1
*0200
        LAC
        SUM A       / AC = [A]
        SUM B       / AC = [A] + [B]
        DEL RES
        ALT
A,      0005
B,      0003
RES,    0000
$
```

`LAC` limpia el acumulador; `SUM A` y `SUM B` suman los operandos; `DEL RES` guarda el resultado y limpia AC; `ALT` detiene la máquina. El resultado queda en `RES` como `0010` octal (8 decimal).

### Microinstrucciones

- **Grupo 0 — limpieza:** `LAC` (`7200`), `LEN` (`7100`), `CAC` (`7040`), `CEN` (`7020`), `IAC` (`7010`), `NOP` (`7000`). Se combinan entre sí; por ejemplo `LAC LEN` (`7300`) y `CAC IAC` (`7050`, complemento a dos).
- **Grupo 1 — rotaciones:** `RD1`…`RD12` / `RI1`…`RI12`, sobre el par (AC, EN) de 13 bits. Se escriben sin espacio (`RI5`, no `RI 5`).
- **Grupo 2 — saltos por *skip*:** `SAN` (AC < 0), `SAC` (AC = 0), `SED` (EN = 1), `SAP` (AC > 0), `SAD` (AC ≠ 0), `SEC` (EN = 0), `SLT` (incondicional), `ALT` (alto), `ORI` (OR con interruptores). Los saltos OR (`SAN`, `SAC`, `SED`) y AND (`SAP`, `SAD`, `SEC`) no se mezclan.
- **Grupo 6 — E/S:** teletipo, lector, perforador, teclado y display. La Ayuda del editor lo llama «Grupo 3» por el campo de grupo del microcódigo; es el mismo conjunto.

> **`LAC` aparece en dos grupos.** En el Grupo 0 es `7200`; en el Grupo 2 es `7600`, la base de las combinaciones `LAC` + salto (`7700 LAC SAN`, `7710 LAC SAP`, `7770 LAC SAP SAD SEC`). Ambas codificaciones ejecutan `AC = 0`; el ensamblador escribe `7200` para `LAC` aislado y compone las combinaciones con los códigos de salto.

El ensamblador valida etiquetas, alcance directo, solapamientos y combinaciones de microinstrucciones. El listado muestra página, dirección relativa y absoluta, contenido y fuente; el ejecutor permite paso a paso, breakpoints y mapa de memoria.

El manual completo, con IRM, microinstrucciones, direccionamiento, E/S, depuración y errores, está en **[`DOC/MANUAL_LEN1.md`](DOC/MANUAL_LEN1.md)** y en la pestaña **Ayuda** del editor.

## Del fuente al teletipo

1. Escriba LEAL o LEN-1 en el editor, o cargue un ejemplo de `CODE/`.
2. Pulse **⚙ Compile** (`F9`): se compila LEAL a LEN-1 y se ensambla el listado.
3. Revise el panel **LST**: página, dirección, contenido y fuente de cada palabra.
4. Pulse **▶ Execute**: el programa se carga en memoria y aparece el teletipo.
5. Guarde el fuente (`.leal`/`.len1`) o el listado (`.lst`) con los botones del editor.

Los archivos `.lst` guardados pueden cargarse en el emulador completo de la CID 201-A, disponible en **[CIDmulator-A](https://github.com/maikelsz/CIDmulatorCID201A)** y en **[GitHub Pages](https://maikelsz.github.io/CIDcoder-A/CIDcoder-A.html)**:

1. «Encender» la computadora.
2. Cargar el archivo `.lst`.
3. Poner los 12 interruptores con la dirección de inicio. Los programas compilados desde LEAL tienen el vector de inicio en `0000`; los escritos en LEN-1 pueden empezar en cualquier dirección que el autor decidiera.
4. Pulsar **CPR**: la dirección seleccionada pasa al Contador de Programa.
5. Pulsar **COM** para comenzar.

Si la computadora está en ejecución, deténgala primero con **ALT**.


## Créditos

LEAL fue desarrollado en Cuba por el Ing. **Víctor M. Llopis Zayas-Bazán** (Centro de Investigación Digital, 1970–1971) con la colaboración de V. A. Orsini, Emilio Lino, Fedor Sierra, María E. Bragado y Miguel Fonfría. La CID 201-A es un diseño de la ICID (1971). Proyecto y simulador: **MSZ** — más en [Tecno-Historia Cubana](https://tecnohistoriacubana.wordpress.com).
