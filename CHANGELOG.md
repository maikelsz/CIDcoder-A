# Registro de cambios - CID 201-B

Un archivo por hito. Nunca se sobrescribe el hito anterior.

## v0.13.0 - `CID201B_13.html` (2026-11-15)

**Cargador de programas compilados `.lstb`**

- Nuevo `CID201B_13.html`; `CID201B_12.html` queda intacto.
- El botón de carpeta de la barra inferior abre el selector de archivos y carga
  el programa en memoria. Solo se acepta la extensión `.lstb`; cualquier otra
  se rechaza con un aviso.
- Formato por líneas: dirección y palabra en octal, separadas por coma,
  tabulador o espacios (`7600, 7140`). Se ignoran comentarios que empiezan por
  `;`, `#`, `//` o `/*`.
- Validación completa antes de escribir: dirección octal de hasta 15 bits
  (`77777`), palabra octal de hasta 12 bits (`7777`), duplicados (gana la
  última), página 0 y líneas sin datos. Un archivo inválido no deja la memoria a
  medio cargar.
- Si la máquina está ejecutando, el cargador la detiene primero con alto `AD`;
  después deposita la memoria y la deja en `ALTO`, sin arrancarla.
- El `.lstb` contiene **solo memoria**: no define la dirección de inicio. El
  cargador no modifica `CP`, `RCI`, `RCD`, `RNI` ni `RVI`. Tras cargar, el
  usuario fija el vector con los interruptores, pulsa `CONTINUAR` y luego
  `COMENZAR`.
- Aviso flotante con el número de palabras, el alto automático y las
  instrucciones para fijar el vector.
- Se expone `window.cargarLstbEnMemoria(texto)` para automatización y pruebas.

**Ejemplo y herramienta**

- `EJEMPLOS/MONITOR_Ultima_Pagina_201B.lstb`: monitor «Última Página» con sus
  128 palabras, listo para cargar.
- `CIDcoder/CODE/LEN CID201B/MONITOR_Ultima_Pagina_201B.lstb`: el mismo
  programa junto al fuente y al listado.
- `DOCwork/tools/asm201b.py`: nueva opción `--lstb-out` para generar el
  programa compilado.

**Pruebas y documentación**

- Nuevo grupo `08-lstb.js`: 11 pruebas y 51 comprobaciones.
- Resultado en `_13`: **88/88 pruebas, 311 comprobaciones**, sin errores de
  consola. En `_12` pasan los grupos 00–07 y fallan las 11 pruebas del
  cargador (77/88).
- `TEST/README.md`, `DOCwork/TODO.md` y `DOCwork/HITO-13.md` actualizados.
- Título: v0.13.0.

**PENDIENTE-DOC**

- Los pendientes del CODEC siguen sin confirmar: desplazamiento de `RFD` con
  `TAR` y tipo de alto de `ALT` (`AD`, `AP` o ambos).

## Entregables de código (2026-11-14)

Sin cambios en `CID201B_12.html`.

- `CIDcoder/CODE/LEN CID201B/MONITOR_Ultima_Pagina_201B.len1`: fuente
  ensamblador del monitor, con la codificación de la 201-B.
- `CIDcoder/CODE/LEN CID201B/MONITOR_Ultima_Pagina_201B.lst`: listado con
  dirección, palabra, nemotécnico/operando y comentario, al estilo de la
  201-A.
- `CIDcoder/CODE/LEN CID201B/README.md`: alcance, codificación, formato y
  comprobación.
- `DOCwork/tools/asm201b.py`: ensamblador mínimo para regenerar el `.lst` y
  compararlo con el `.len1` extraído.
- Verificación: **128/128 palabras** coinciden con `CID201B_12.html` y con
  `EJEMPLOS/MONITOR_Ultima_Pagina_201B.len1`. El listado de tres columnas
  generado con `--len1-out` es idéntico byte por byte al `.len1` extraído.

## v0.12.0 - `CID201B_12.html` (2026-11-14)

**Consola CODEC**

- `RE` de 4 bits: el teclado guarda el dígito completo (`num & 0xF`), incluidos
  8 y 9; ya no los convierte en 0 y 1.
- `RDP` de 8 posiciones de 4 bits y `RFD` de 8 bits (un punto por posición).
- `TAR`: desplaza `RDP` una posición a la izquierda, inserta `AC & 0xF` por la
  derecha y limpia `AC`. No desplaza `RFD`.
- `TAF`: carga los 8 bits bajos del `AC` en `RFD` (bit 7 → posición 0) y limpia
  el `AC`.
- `POS` pone el signo a 0 (más, `+`) y `PUS` lo pone a 1 (menos, `−`); ninguno
  limpia el `AC`. Se añadió `codecDisplaySign()` para separar el bit del
  carácter que se muestra.
- Ayuda interna y tabla de instrucciones corregidas para la nueva polaridad.

**Demo «Última Página»**

- El monitor octal vuelve a ser funcional: `5`, `7` → `00000057`; el 8 se
  descarta a nivel de programa; `BOR` → `00000005`; `ALM` escribe y avanza;
  `REC` lee `00000006` y luego `00007500`.
- El display usa máscaras de 3 bits más `RD3` en lugar de `RI3`. `RI3`/`RD3`
  rotan a través de `EN`, y la extracción por `RI3` arrastraba bits de acarreo.
- La validación octal comprueba el bit 3 de `RE` (dígito nuevo), no el
  candidato acumulado.
- El display vuelve fijo a `7602`: el puente 3+2+7 no puede direccionar la
  página 31, donde vive la demo. No se cambió la semántica del núcleo.

**Extracción**

- `EJEMPLOS/MONITOR_Ultima_Pagina_201B.len1`: 128 líneas de la página 31
  completa, con tres columnas separadas por tabulador (dirección octal,
  contenido octal y nemotécnico o valor octal).
- La demo sigue precargada en el emulador; el `.len1` es una extracción.

**Pruebas y documentación**

- Nuevo grupo `07-codec.js`: 12 pruebas y 22 comprobaciones.
- Resultado en `_12`: **77/77 pruebas, 260 comprobaciones**, sin errores de
  consola. En `_11` pasan los grupos 00–06 y fallan las 7 pruebas nuevas del
  CODEC (70/77).
- `TEST/README.md`, `DOCwork/TODO.md` y `DOCwork/HITO-12.md` actualizados.
- Título: v0.12.0.

**PENDIENTE-DOC**

- Si `RFD` debe desplazarse junto con `RDP` al ejecutar `TAR`.
- Si la tecla `ALT` del CODEC debe dar alto `AD`, `AP` o ambos; el código
  actual usa `AD`.

## v0.11.0 — `CID201B_11.html` (2026-11-14)

**Núcleo**

- Puente indirecto con la partición documentada **campo(3) + página(2) + offset(7)**.
  El campo del puente manda en la dirección efectiva.
- `RNI`/`CCIN`: `RCI ← RNI` en cada frontera de instrucción; `RNI` por defecto
  sigue el campo actual. `762N` cambia de módulo de verdad.
- `VAS(I)`/`VEA(I)`: `RVI ← RCI` (origen) y `RCI = RNI = campo del puente`.
- `INT1`: `RVI` guarda el campo de origen.
- Protección: zona = 3 bits altos del puente; `ISC`/`DEL` contra `RCD`,
  `VAS(I)` contra `RCI`; zona directa por página destino.
- `doCD()` usa el código de operación de la instrucción efectiva (`VPP`).
- `doCI`/`doCM`: las microinstrucciones y la E/S (`op >= 6`) se ejecutan en `CM`
  sobre la misma instrucción; se retrocede el `CP` y no se decodifican como
  indirectas.
- Eliminada `cidSelfTest` y sus accesos (`?selftest=1`, `Ctrl+Alt+T`, botón y ayuda).
- Banco de pruebas externo en `TEST/`: 65/65 pruebas, 181 aserciones en `_11`;
  `_10` falla 22, lo que detecta las regresiones.
- Terminología: se distingue el **Registro de Enlace (EN)** de la **Unidad de
  Enlace (ENLACE)** en la ayuda, los comentarios y las pruebas.
- Documentación: `DOChistorico/CID201B.md` validado (987 líneas, 44 tablas,
  0 errores) con seis rejillas gráficas eliminadas a mano; cobertura sin
  diferencias inesperadas.
- Ayuda: apartado de direccionamiento indirecto.
- Título: v0.11.0.

**No incluido** (hitos siguientes): CODEC decimal, cinta del Ricoh, Unidad de
Enlace y editor de memoria, UAB 65xx, periféricos de alta velocidad.
