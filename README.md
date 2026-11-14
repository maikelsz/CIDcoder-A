# CIDcoder-A

**Editor, compilador para el lenguaje LEAL y ensamblador LEN1 sobre la minicomputadora cubana CID 201-A — todo en una sola página HTML.**

Este proyecto recupera en software la cadena de herramienta de la CID 201-A (ICID, 1971): se escribe un programa en el lenguaje algorítmico **LEAL**, se compila al ensamblador **LEN-1** y se ejecuta en un simulador completo de la máquina — con teletipo RFT T-51, perforador, mapa de memoria y microcódigo. Sin dependencias: se abre directo en el navegador, no hay build.

- **Estado:** hito vigente versión **0.1.51** (13/Nov/2026), «Baseline limpio; fases 0–2 en curso». Hito anterior 0.1.50.2, eliminación de `*`; antes 0.1.50.1, revisión interna y 0.1.50, Optimización.
- **Calidad:** 8 suites de regresión permanentes en verde — **1046 checks / 0 fallos** + `survey` **50/50 modos**.

## Primeros pasos

1. **Abre** `CIDcoder-A.html` en un navegador moderno. No necesita servidor (solo para las suites de `TEST/`).
2. **Escribe** el programa en el editor. El análisis estático avisa antes de generar LEN-1 (÷0 literal, desborde de plegado, variables no declaradas, mezclas char/entero…).
3. **Compila** en modo **UNPK** (sin compactar) o **PKD** (compactado, 2 caracteres por palabra en cadenas). Revisa el listado LEN-1 resultante.
4. **Ejecuta** en el modelo del CID 201-A: la salida aparece en el teletipo simulado.
5. **Ayuda integrada:** la pestaña **Ayuda** de la aplicación es el manual de usuario — secciones LEAL (§1–§15: lenguaje completo con ejemplos y errores frecuentes) y LEN-1 (§1–§10: ensamblador). Ese manual está alineado con cada hito.

### Ejemplo mínimo

```text
LEAL
TITULO SUMA DEMO        ; declaración inerte: no emite ninguna palabra
ENT A B C               ; variables enteras de 24 bits
5 = A
3 = B
A + B = C
PER C
FIN
```

**Salida en el teletipo:** `8`.

### El lenguaje LEAL en un minuto

- Cabecera obligatoria `LEAL`; zona de declaraciones (`TITULO`, `ENT`, `FLO`, `REF`) antes de la primera instrucción; termina con `FIN`.
- Comentarios con `;` hasta fin de línea.
- Variables de una letra A–Z, del tipo declarado con `ENT` (entero 24 bits) o `FLO` (flotante de 2 palabras), con sufijos `A(n)`, `A(I)`, `A(I(n))`.
- Asignación `expresión = destino` y cadenas `expr = tramo = ... = destino` (evaluación izq→der sin paréntesis); condición `A ? SSx (etiq)`; salto `VEA (etiq)`; etiquetas `(n)`; ciclos `CIC/REP`; subrutinas `VAS/CRU/FRU`; cadenas con buffers `LCA/PCA`.

## Simulación más completa.

Los archivos `.lst` guardados pueden cargarse en el emulador completo de la CID 201-A, disponible en el repositorio de GitHub: [CIDmulator-A](https://github.com/maikelsz/CIDmulatorCID201A), y en GitHubPages: [CIDmulator-A](https://maikelsz.github.io/CIDcoder-A/CIDcoder-A.html)
Para ello:
1-"Encender" la computadora.
2-Cargar el archivo `.lst`.
3-Poner los 12 interruptores con la dirección de inicio. Los programas compilados desde LEAL el vector de incio esta en la dicección 0000, los que fueorn escritos en LEN1 pueden empezar en cualquier lugar que el autor decidiera, revisar el archivo .lst o el LEN1 fuente.
4-Apretar CPR. La direccion seleccionada pasara al registro Contador de Programa.
5-Apretar COM para comenzar.
Si la computadora esta en ejecución, primero detenerla con ALT, luego seguir los pasos anteriores desde el 2.

## Qué soporta hoy

| Área | Estado |
|---|---|
| Enteros 24 bits | Completo: `+ − : /` con `E:DESB`/`E:DIV0` y parada (sin envoltura), condiciones, saltos combinados, ciclos |
| Flotantes | Declaraciones, aritmética `+ - : /`, `NEG`/`FAB`/`PEN`/`FRA`/`NOR`/`INV` (1/x), `RAC` (raíz cuadrada, ≤1 ULP), `EXP` (exponencial), `LNE` (logaritmo natural), `LOG` (logaritmo base 10), `SEN` (seno), `COS` (coseno), `FRA` en expresiones complejas, `LEC` y `PER` flotante , **mixtas ENT↔FLO con destino FLO**  |
| E/S | `LEC` int/float/dual, `PER` int/float con formato `DIG m,n` (default 4,2; m ≤ 8, n ≤ 5 saturados con aviso; «DIG n» exponencial → error), `LIN`, `ESP`, `TIT` (inerte), teletipo 100 ch/s en UI/pruebas (histórico 10 ch/s) |
| Cadenas | Buffers `LCA`/`PCA` y `MDI`/`MID` (desplazamientos), en UNPK y PKD |
| Subrutinas | `VAS`/`CRU`/`FRU` con asertos permanentes (Ola AUDIT) |

**Pendiente conocido:** salida exponencial «`DIG n`»  y formato decimal del manual (5 dec, redondeo del 6º ≥ 5).

## Créditos

LEAL fue desarrollado en Cuba por el Ing. Víctor M. Llopis Zayas-Bazán (Centro de Investigación Digital, 1970-1971) con la colaboración de V. A. Orsini, Emilio Lino, Fedor Sierra, María E. Bragado y Miguel Fonfría. La CID 201-A es diseño de la ICID (1971). Proyecto y simulador: MSZ — más en [Tecno-Historia Cubana](https://tecnohistoriacubana.wordpress.com).
