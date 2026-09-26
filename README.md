# Post-contenido Unidad 3 — Manejo del DEBUG: Exploración, Ensamblado y Ejecución Paso a Paso en DOSBox

Autor: David Camilo Colmenares Hernández
Programa: Ingeniería de Sistemas
Curso: Arquitectura de Computadores — 1154507-B
Docente: Ing. Jonathan Rolando Rey Castillo
Facultad de Ingeniería, Universidad Francisco de Paula Santander

---

## Descripción del laboratorio

El presente repositorio documenta un laboratorio guiado de dos partes ejecutado sobre el emulador DOSBox. La Parte 1 explora el estado inicial del procesador 8086 y los comandos de inspección del depurador DEBUG; el estudiante examina registros con R; rellena y vuelca memoria con F y D; desensambla código con U; modifica bytes puntuales con E y verifica el resultado mediante una instrucción con direccionamiento directo a memoria. La Parte 2 ensambla programas propios con el comando A; los ejecuta instrucción por instrucción con T registrando la evolución del estado en tablas de traza; analiza el mecanismo de bucle basado en LOOP y CX; lo compara con un bucle equivalente construido con DEC y JNZ y contrasta el comando G como alternativa de verificación rápida frente a T.

El laboratorio no produce código fuente compilable. La evidencia son las capturas de pantalla de cada checkpoint y este documento; ambos elementos se evalúan en conjunto bajo la rúbrica R2-Lab.

## Entorno utilizado

| Elemento | Valor |
|---|---|
| Emulador | DOSBox 0.74-3 |
| Sistema anfitrión | Windows |
| Carpeta montada como unidad C | C:\DOSWork |
| Depurador | DEBUG.COM de FreeDOS; implementación libre de Paul Vojta bajo licencia MIT |
| Segmento asignado por el DOS en esta sesión | 0725 |
| Subcarpeta Parte 1 | C:\LAB3POST |
| Subcarpeta Parte 2 | C:\LAB3P2 |

El DEBUG original de MS-DOS es un programa de 16 bits y no se ejecuta en las ediciones de 64 bits de Windows; por esa razón el laboratorio corre dentro de DOSBox y emplea la implementación libre equivalente de FreeDOS.

Dos observaciones sobre el entorno que condicionan lo que muestran las capturas. La primera es el segmento; el DOS asignó 0725 a esta sesión en lugar del 1357 que aparece en los ejemplos de la guía. El valor cambia en cada máquina y en cada ejecución; no altera ningún resultado porque todas las direcciones del laboratorio son desplazamientos dentro de ese segmento. La segunda es el nombre de las subcarpetas; se explica en la sección de observaciones al final.

---

# Parte 1 — Exploración con DEBUG en DOSBox

## Comandos empleados en la Parte 1

| Comando | Función | Paso donde se usa |
|---|---|---|
| R | Muestra o modifica registros y banderas | 4 y 5 |
| F | Rellena un rango de memoria con un patrón cíclico | 6 y 11 |
| D | Vuelca un rango de memoria en hexadecimal y ASCII | 7; 8 y 11 |
| U | Desensambla bytes de memoria a mnemónicos | 9; 10 y 13 |
| A | Ensambla mnemónicos y los escribe como bytes en memoria | 10 y 13 |
| E | Escribe bytes puntuales a partir de una dirección | 11 |
| T | Ejecuta una sola instrucción y muestra el estado resultante | 14 |

## Checkpoint 1 — Estado inicial de los registros

Captura: `capturas/CP1_registros.png`

Estado inicial observado.

```
AX=0000  BX=0000  CX=0000  DX=0000  SP=FFFE  BP=0000  SI=0000  DI=0000
DS=0725  ES=0725  SS=0725  CS=0725  IP=0100   NV UP EI PL NZ NA PO NC
```

Observaciones. El comando R sin argumento entrega el estado completo del procesador en tres líneas; la primera reúne los registros de propósito general y los de puntero e índice; la segunda reúne los registros de segmento con el IP y las ocho banderas en notación de pares de letras; la tercera muestra la instrucción que apunta CS:IP ya desensamblada. Se observa que AX; BX; CX y DX arrancan en cero; que SP arranca en FFFEh como tope inicial de la pila y que los cuatro registros de segmento comparten el valor 0725h porque el DOS asigna un único bloque de memoria al programa. El IP arranca en 0100h; los 256 bytes anteriores corresponden al PSP. Tras cargar 1234h con R AX el segundo volcado muestra AX=1234 y todos los demás registros intactos; la modificación es selectiva y no altera el resto del estado.

## Checkpoint 2 — Volcado hexadecimal del patrón AB CD EF

Captura: `capturas/CP2_volcado_memoria.png`

Salida obtenida con `F 200 L40 AB CD EF` seguido de `D 200 L40`.

```
0725:0200  AB CD EF AB CD EF AB CD-EF AB CD EF AB CD EF AB  ................
0725:0210  CD EF AB CD EF AB CD EF-AB CD EF AB CD EF AB CD  ................
0725:0220  EF AB CD EF AB CD EF AB-CD EF AB CD EF AB CD EF  ................
0725:0230  AB CD EF AB CD EF AB CD-EF AB CD EF AB CD EF AB  ................
```

Interpretación de las columnas del comando D. La salida se organiza en tres bloques. El primero es la dirección de inicio de la fila; se escribe como segmento y desplazamiento separados por dos puntos e indica dónde comienza el contenido que sigue. El segundo bloque son dieciséis valores hexadecimales; cada valor es un byte de memoria y el guion central separa los ocho primeros de los ocho últimos para facilitar el conteo visual. El tercer bloque es la interpretación ASCII de esos mismos dieciséis bytes; los valores que caen fuera del rango imprimible 20h a 7Eh se representan con un punto. Por esa razón el patrón AB CD EF aparece como una hilera de puntos en la columna derecha; ninguno de esos tres valores corresponde a un carácter visible.

Observaciones. El comando F no confirma la operación con mensaje alguno; el retorno del prompt es la única señal de éxito. El patrón de tres bytes se repite de forma cíclica hasta completar los 40h bytes solicitados; por eso la alineación se desplaza de fila en fila; 16 no es múltiplo de 3. La primera fila empieza en AB; la segunda en CD y la tercera en EF.
