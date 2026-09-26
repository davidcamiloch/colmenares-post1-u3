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

## Checkpoint 3 — Ensamblado con A y verificación con U

Captura: `capturas/CP3_ensamblado_desensamblado.png`

Programa ensamblado y codificación obtenida.

| Dirección | Bytes | Instrucción | Tamaño |
|---|---|---|---|
| 0100 | B8 05 00 | MOV AX,0005 | 3 bytes |
| 0103 | BB 03 00 | MOV BX,0003 | 3 bytes |
| 0106 | 01 D8 | ADD AX,BX | 2 bytes |
| 0108 | CD 20 | INT 20 | 2 bytes |

Observaciones. El programa completo ocupa diez bytes. Se observa la correspondencia directa entre mnemónico y código máquina; el opcode B8 carga un valor inmediato de 16 bits en AX y los dos bytes siguientes son ese valor en orden little-endian; 0005h se almacena como 05 00. El opcode BB cumple la misma función sobre BX. La instrucción ADD AX,BX no lleva operando inmediato; los dos bytes codifican la operación y los registros involucrados.

Sobre la codificación de ADD AX,BX conviene una precisión. La guía del curso anota 03 C3 y este laboratorio obtuvo 01 D8. Las dos son codificaciones válidas de la misma instrucción; el conjunto x86 admite dos formas para sumar dos registros y la diferencia está en cuál operando viaja en el campo reg del byte ModR/M. El opcode 03 corresponde a la forma ADD registro; registro-o-memoria y el opcode 01 a la forma ADD registro-o-memoria; registro. El DEBUG de MS-DOS elige la primera y el de FreeDOS la segunda; el efecto sobre AX es idéntico y el tamaño también.

## Checkpoint 4 — Modificación con E y direccionamiento directo

Captura: `capturas/CP4_memoria_direccionamiento.png`

Secuencia verificada. Se limpia un rango de dieciséis bytes en 0300h con F; se vuelca con D para fijar el punto de partida; se escriben dos bytes puntuales con E 300 78 56 y se vuelca de nuevo.

```
0725:0300  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................
0725:0300  78 56 00 00 00 00 00 00-00 00 00 00 00 00 00 00  xV..............
```

El segundo volcado confirma que E modificó únicamente los dos primeros bytes; los catorce restantes siguen en cero. La columna ASCII pasa a mostrar xV porque 78h y 56h sí son caracteres imprimibles. Leídos como palabra en little-endian esos dos bytes representan el valor 5678h.

| Dirección | Bytes | Instrucción | Modo de direccionamiento |
|---|---|---|---|
| 0320 | A1 00 03 | MOV AX,[0300] | Directo a memoria |
| 0323 | CD 20 | INT 20 | Sin operando |

Resultado tras ejecutar la instrucción con T.

```
AX=5678  BX=0000  CX=0000  DX=0000  SP=FFFE  BP=0000  SI=0000  DI=0000
DS=0725  ES=0725  SS=0725  CS=0725  IP=0323   NV UP EI PL NZ NA PO NC
```

AX toma el valor 5678h; esto confirma que el dato escrito con E fue leído correctamente desde la dirección 0300h y que el orden little-endian se aplica también en la lectura.

### Decisión técnica 1 — Verificación no destructiva de una escritura en memoria

El estudiante selecciona D como comando de verificación. Invocar E 300 sin la lista de bytes abre el modo interactivo; el depurador muestra cada byte y espera una pulsación; una tecla equivocada sobrescribe el dato que se quería comprobar. Con la lista completa de bytes E es determinista; sin ella es una edición a ciegas. F tampoco sirve; su propósito es escribir un patrón sobre un rango completo y destruiría el contenido que se quiere leer. R alcanza registros y no memoria. D es el único que solo lee; recibe un rango y devuelve su contenido sin modificar un byte. Esa propiedad de solo lectura es la que exige esta verificación.

### Decisión técnica 2 — Direccionamiento inmediato frente a directo a memoria

Ambas instrucciones ocupan tres bytes; su costo de ejecución difiere. En MOV AX,0005 el opcode B8 trae el valor incrustado en el flujo de instrucción; el procesador ya dispone del dato al terminar la extracción. En MOV AX,[0300] el opcode A1 trae una dirección; el procesador debe resolverla contra DS y ejecutar un acceso adicional al bus para leerlo. El modo directo paga ese ciclo extra. El estudiante lo prefiere cuando el valor puede cambiar en ejecución; por ejemplo el 5678h escrito con E. Una constante fija conocida al ensamblar se resuelve mejor con inmediato. El comando U confirma cada codificación sin ejecutar nada; B8 05 00 frente a A1 00 03.
