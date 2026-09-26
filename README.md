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

---

# Parte 2 — Ensamblado y Ejecución Paso a Paso

## Comandos empleados en la Parte 2

| Comando | Función | Paso donde se usa |
|---|---|---|
| A | Ensambla los tres programas del laboratorio | 2; 5 y 9 |
| U | Verifica la codificación antes de ejecutar | 3; 6 y 9 |
| R IP | Restablece el puntero de instrucción antes de cada traza | 3; 7; 11 y 12 |
| T | Ejecuta una instrucción y muestra el estado resultante | 4; 7 y 11 |
| G | Ejecuta a velocidad completa hasta un punto de interrupción | 12 |
| D | Vuelca el código máquina del programa de bucle | 8 |

## Checkpoint 1 — Tabla de traza del programa de suma

Capturas: `capturas/CP1_traza_suma.png` y `capturas/CP1_traza_suma_parte2.png`

Programa ensamblado.

| Dirección | Bytes | Instrucción |
|---|---|---|
| 0100 | B8 0A 00 | MOV AX,000A |
| 0103 | BB 05 00 | MOV BX,0005 |
| 0106 | B9 03 00 | MOV CX,0003 |
| 0109 | 01 D8 | ADD AX,BX |
| 010B | 01 C8 | ADD AX,CX |
| 010D | CD 20 | INT 20 |

Tabla de traza. Valores observados tras cada ejecución de T.

| Instrucción | AX | BX | CX | IP siguiente | ZF | CF | SF |
|---|---|---|---|---|---|---|---|
| MOV AX,000A | 000A | 0000 | 0000 | 0103 | NZ | NC | PL |
| MOV BX,0005 | 000A | 0005 | 0000 | 0106 | NZ | NC | PL |
| MOV CX,0003 | 000A | 0005 | 0003 | 0109 | NZ | NC | PL |
| ADD AX,BX | 000F | 0005 | 0003 | 010B | NZ | NC | PL |
| ADD AX,CX | 0012 | 0005 | 0003 | 010D | NZ | NC | PL |
| INT 20 | 0012 | 0005 | 0003 | fin | NZ | NC | PL |

Observaciones. Las tres primeras instrucciones son transferencias de datos y no alteran ninguna bandera; las columnas ZF; CF y SF repiten el estado inicial NZ NC PL. La aritmética empieza en la cuarta instrucción; 000Ah más 0005h da 000Fh y luego 000Fh más 0003h da 0012h; 18 en decimal. Ninguna de las dos sumas activa el acarreo ni el cero ni el signo porque el resultado cabe holgadamente en 16 bits y es positivo. Sí cambian otras banderas que el DEBUG muestra en la misma línea; la paridad pasa de PO a PE al obtener 000Fh y el acarreo auxiliar pasa a AC al obtener 0012h; la suma arrastró desde el nibble bajo. Ese detalle ilustra que una operación aritmética actualiza el registro de banderas completo y no solo las banderas que el programador está mirando.

## Checkpoint 2 — Tabla de traza del bucle con LOOP

Capturas: `capturas/CP2_traza_loop.png`; `capturas/CP2_traza_loop_parte2.png` y `capturas/CP2_traza_loop_parte3.png`

Programa ensamblado.

| Dirección | Bytes | Instrucción |
|---|---|---|
| 0100 | B8 00 00 | MOV AX,0000 |
| 0103 | B9 04 00 | MOV CX,0004 |
| 0106 | 83 C0 02 | ADD AX,0002 |
| 0109 | E2 FB | LOOP 0106 |
| 010B | CD 20 | INT 20 |

Cálculo del desplazamiento de LOOP. La instrucción ocupa las direcciones 0109h y 010Ah; la instrucción siguiente comienza en 010Bh. Los saltos cortos del 8086 son relativos y se calculan desde la dirección de la instrucción siguiente.

```
destino - direccion_siguiente = 0106h - 010Bh = -5 = FBh
```

El byte E2h es el opcode de LOOP y FBh es ese desplazamiento de ocho bits con signo. El rango alcanzable por un salto corto es por tanto de -128 a +127 bytes.

Tabla de traza. Valores observados tras cada ejecución de T.

| Iteración | Instrucción | AX después | CX después | IP siguiente | ¿LOOP salta? |
|---|---|---|---|---|---|
| — | MOV AX,0000 | 0000 | 0000 | 0103 | no aplica |
| — | MOV CX,0004 | 0000 | 0004 | 0106 | no aplica |
| 1 | ADD AX,0002 | 0002 | 0004 | 0109 | no aplica |
| 1 | LOOP 0106 | 0002 | 0003 | 0106 | sí |
| 2 | ADD AX,0002 | 0004 | 0003 | 0109 | no aplica |
| 2 | LOOP 0106 | 0004 | 0002 | 0106 | sí |
| 3 | ADD AX,0002 | 0006 | 0002 | 0109 | no aplica |
| 3 | LOOP 0106 | 0006 | 0001 | 0106 | sí |
| 4 | ADD AX,0002 | 0008 | 0001 | 0109 | no aplica |
| 4 | LOOP 0106 | 0008 | 0000 | 010B | no |
| — | INT 20 | 0008 | 0000 | fin | no aplica |

Observaciones. La tabla muestra con claridad el mecanismo de LOOP; la instrucción decrementa CX y solo después decide si salta. Se observa que en la cuarta vuelta CX pasa de 0001h a 0000h y el IP avanza a 010Bh en lugar de regresar a 0106h; esa es la salida del bucle. El acumulador crece de dos en dos hasta 0008h; cuatro veces dos; tal como se esperaba. Resulta notable que LOOP no altera las banderas; la línea de estado permanece en NV UP EI PL NZ NA PO NC durante todo el bucle; los únicos cambios de paridad los introduce la suma. Esa es una diferencia de fondo con el mecanismo alterno de la sección siguiente.

## Análisis del código máquina con D

El volcado del programa de bucle se obtiene con `D CS:100 L0D`; el sufijo L0D solicita trece bytes; esa es la extensión exacta del programa.

```
0725:0100  B8 00 00 B9 04 00 83 C0-02 E2 FB CD 20
           --------  --------  --------  -----  -----
           MOV AX,0  MOV CX,4  ADD AX,2  LOOP   INT 20
            3 bytes   3 bytes   3 bytes  2 by.  2 by.
```

Se observa que la longitud de cada instrucción depende de su codificación y no de su complejidad aparente. Las dos cargas inmediatas ocupan tres bytes cada una porque arrastran un operando de 16 bits. La suma inmediata también ocupa tres bytes; por otra razón; el opcode 83h corresponde a la forma que acepta un operando inmediato de ocho bits con extensión de signo; 02h viaja en un solo byte y el byte C0h identifica el registro destino. LOOP ocupa dos bytes porque su operando es un desplazamiento de un solo byte. INT 20 ocupa dos bytes porque su operando es el número de interrupción. El programa completo suma trece bytes y realiza cuatro sumas de forma iterativa; una versión sin bucle que repitiera la suma cuatro veces ocuparía doce bytes solo en las sumas más los seis de la inicialización y la terminación; el bucle no ahorra espacio en casos tan pequeños pero escala mucho mejor.

## Checkpoint 3 — Bucle equivalente con DEC y JNZ

Capturas: `capturas/CP3_traza_dec_jnz.png`; `capturas/CP3_traza_dec_jnz_parte2.png`; `capturas/CP3_traza_dec_jnz_parte3.png`; `capturas/CP3_traza_dec_jnz_parte4.png` y `capturas/CP3_traza_dec_jnz_parte5.png`

Programa ensamblado en 0200h para no sobrescribir el anterior.

| Dirección | Bytes | Instrucción |
|---|---|---|
| 0200 | B8 00 00 | MOV AX,0000 |
| 0203 | B9 04 00 | MOV CX,0004 |
| 0206 | 83 C0 02 | ADD AX,0002 |
| 0209 | 49 | DEC CX |
| 020A | 75 FA | JNZ 0206 |
| 020C | CD 20 | INT 20 |

Cálculo del desplazamiento de JNZ.

```
destino - direccion_siguiente = 0206h - 020Ch = -6 = FAh
```

Comparación de tamaño. LOOP resuelve el control del bucle con dos bytes en una sola instrucción; DEC CX más JNZ requiere tres bytes repartidos en dos instrucciones para producir el mismo efecto observable. La diferencia es de un byte de código y de una instrucción adicional por iteración.

Tabla de traza. Valores observados tras cada ejecución de T.

| Iteración | Instrucción | AX después | CX después | IP siguiente | ¿JNZ salta? |
|---|---|---|---|---|---|
| — | MOV AX,0000 | 0000 | 0000 | 0203 | no aplica |
| — | MOV CX,0004 | 0000 | 0004 | 0206 | no aplica |
| 1 | ADD AX,0002 | 0002 | 0004 | 0209 | no aplica |
| 1 | DEC CX | 0002 | 0003 | 020A | no aplica |
| 1 | JNZ 0206 | 0002 | 0003 | 0206 | sí |
| 2 | ADD AX,0002 | 0004 | 0003 | 0209 | no aplica |
| 2 | DEC CX | 0004 | 0002 | 020A | no aplica |
| 2 | JNZ 0206 | 0004 | 0002 | 0206 | sí |
| 3 | ADD AX,0002 | 0006 | 0002 | 0209 | no aplica |
| 3 | DEC CX | 0006 | 0001 | 020A | no aplica |
| 3 | JNZ 0206 | 0006 | 0001 | 0206 | sí |
| 4 | ADD AX,0002 | 0008 | 0001 | 0209 | no aplica |
| 4 | DEC CX | 0008 | 0000 | 020A | no aplica |
| 4 | JNZ 0206 | 0008 | 0000 | 020C | no |
| — | INT 20 | 0008 | 0000 | fin | no aplica |

Observaciones sobre las banderas. Aquí sí hay evidencia directa del papel del registro de banderas como puente entre la aritmética y el salto. Mientras CX baja de 0004h a 0001h la línea de estado muestra NZ; la bandera de cero apagada. JNZ salta. En la cuarta vuelta DEC CX deja CX en 0000h y la línea cambia a ZR; en la ejecución siguiente JNZ lee esa bandera y no salta; el IP avanza a 020Ch. La bandera es el único dato que conecta las dos instrucciones; DEC no le dice nada a JNZ por otra vía. Esa dependencia explícita es justo lo que LOOP oculta dentro de una sola instrucción. Al ejecutar la última instrucción el depurador respondió `Program terminated normally (0000)`.

### Conteo comparado de instrucciones ejecutadas

| Mecanismo | Inicialización | Cuerpo por iteración | Total de iteraciones | Terminación | Instrucciones ejecutadas |
|---|---|---|---|---|---|
| LOOP | 2 | 2 instrucciones | 4 | 1 | 2 + 8 + 1 = 11 |
| DEC y JNZ | 2 | 3 instrucciones | 4 | 1 | 2 + 12 + 1 = 15 |

La diferencia de cuatro instrucciones corresponde exactamente a una instrucción adicional por cada una de las cuatro iteraciones; es la instrucción de control extra que el mecanismo DEC y JNZ necesita frente a LOOP. El conteo se verificó en la práctica; la traza del bucle LOOP exigió diez pulsaciones de T para llegar a INT 20 y la del bucle DEC y JNZ exigió catorce. Ambos programas terminan con AX en 0008h; son semánticamente equivalentes pese a su distinto costo.

### Decisión técnica 3 — Selección del mecanismo de control de bucle

El estudiante recomienda LOOP para este bucle contador. La codificación lo sostiene; LOOP ocupa dos bytes en una sola instrucción mientras DEC CX seguido de JNZ ocupa tres bytes en dos instrucciones. El procesador extrae una instrucción menos por iteración; sobre cuatro iteraciones son cuatro extracciones menos y un byte menos de código. CX no se necesita para otro propósito dentro del cuerpo; no hay motivo para pagar ese costo. El estudiante preferiría DEC y JNZ si el cuerpo tuviera que reutilizar CX en otra operación aritmética; también si la salida dependiera de una comparación evaluada con CMP. El comando U compara los tamaños sin ejecutar nada; E2 FB frente a 49 75 FA.

### Decisión técnica 4 — Comando de verificación para bucles de muchas iteraciones

Con un contador de 0064h el comando T exigiría más de trescientas pulsaciones; G 20C alcanza el mismo punto con una sola orden. El depurador sustituye el byte destino por el opcode CC y deja correr el programa a velocidad completa. Lo que se pierde es la granularidad; G no revela ningún estado intermedio de AX; de CX ni de las banderas. T fue necesario en los Pasos 4; 7 y 11; las tres tablas de traza piden el estado después de cada instrucción y esa información solo existe si el procesador se detiene en cada una. Tras G 20C el estudiante confirma el valor final de AX con el comando R.

Salida real de la demostración.

```
-G 20C
AX=0008  BX=0000  CX=0000  DX=0000  SP=FFFE  BP=0000  SI=0000  DI=0000
DS=0725  ES=0725  SS=0725  CS=0725  IP=020C   NV UP EI PL ZR NA PE NC
0725:020C CD20          INT     20
```

El comando entregó AX=0008h en una sola orden; idéntico al valor que la traza alcanzó en catorce pasos. Se observa además que la bandera ZR aparece activa; es el residuo del último DEC CX; G la conserva aunque no haya mostrado la instrucción que la produjo.

---

## Observaciones sobre la guía del laboratorio

Durante la ejecución el estudiante detectó diez discrepancias entre el enunciado de la guía y el comportamiento real del DEBUG dentro de DOSBox. Las codificaciones fueron contrastadas además con un ensamblador NASM. En este repositorio se documentan los valores realmente observados.

| # | Ubicación | Lo que dice la guía | Lo que ocurre en realidad |
|---|---|---|---|
| 1 | Parte 2, Paso 5 | El listado de A ubica LOOP en 010A e INT 20 en 010C | La suma inmediata ocupa tres bytes; LOOP queda en 0109 e INT 20 en 010B. Confirmado en la sesión |
| 2 | Parte 2, Paso 8 | El comando indicado es D CS:100 L0C | L0C vuelca doce bytes; el programa ocupa trece, como afirma el propio texto del paso. El comando exacto es D CS:100 L0D |
| 3 | Parte 2, Paso 8 | El volcado muestra los bytes 05 02 00 para ADD AX,0002 | El DEBUG emitió 83 C0 02, que es la forma con inmediato de ocho bits y extensión de signo. La guía se contradice a sí misma; su listado de U dice ADD AX,+02, que corresponde a 83 C0 02 y no a 05 02 00 |
| 4 | Parte 2, Paso 6 | El rango indicado es U 100 10D | El programa termina en 010C; el rango exacto es U 100 10C |
| 5 | Parte 1, Paso 9 | La columna de bytes aparece como 00000 | Dos bytes en cero se transcriben con cuatro dígitos; 0000 |
| 6 | Parte 1, Paso 5 | La segunda invocación de R AX devuelve la línea completa de registros | R AX responde solo con el valor de AX y el prompt de edición. La línea completa se obtiene con R sin argumento |
| 7 | Entregables | Se habla de ocho checkpoints con captura | El Checkpoint 4 de la Parte 2 verifica el repositorio y no produce captura; las imágenes son siete y así las enumera la rúbrica |
| 8 | Partes 1 y 2, prerrequisitos | Se ordena crear las carpetas LAB3POST1 y LAB3POST2 | Ambos nombres tienen nueve caracteres y DOS admite ocho; el sistema los recorta en silencio al mismo nombre LAB3POST, por lo que el segundo MD falla con Unable to make. Se usaron LAB3POST y LAB3P2 |
| 9 | Rúbrica, criterio de Funcionalidad | Se exige que ADD AX,BX aparezca codificado como 03 C3 | El DEBUG de FreeDOS emitió 01 D8 y ADD AX,CX como 01 C8. Ambas son codificaciones válidas de la misma operación; cambia cuál operando ocupa el campo reg del byte ModR/M |
| 10 | Parte 2, Paso 6 | El desensamblado muestra el mnemónico LOOP | El DEBUG de FreeDOS lo muestra como LOOPW, indicando de forma explícita que el contador es CX de 16 bits |

Las siete primeras son inconsistencias internas del enunciado; se detectaron leyéndolo con cuidado. Las tres últimas surgieron al ejecutar el laboratorio; provienen de que DOSBox no incluye el DEBUG de MS-DOS. El laboratorio se realizó con la implementación libre de FreeDOS; es funcionalmente equivalente pero toma decisiones distintas al ensamblar y al desensamblar.

## Conclusiones

El laboratorio muestra que el DEBUG expone el procesador sin ninguna capa de abstracción; no hay símbolos ni tipos; solo bytes; registros y direcciones. Esa transparencia es lo que permite observar directamente principios que un lenguaje de alto nivel oculta. El estudiante comprueba que en modo real no existe frontera entre código y datos; los mismos bytes se desensamblan como instrucción o se leen como dato según hacia dónde apunte CS:IP. Comprueba también que el tamaño de una instrucción depende de su codificación y no de lo que expresa; que una misma operación admite varias codificaciones y que el ensamblador elige una sin consultar al programador; que los saltos cortos se calculan como desplazamientos relativos a la instrucción siguiente y que dos programas semánticamente idénticos pueden diferir en bytes de código y en instrucciones ejecutadas. La traza del bucle DEC y JNZ aportó además la evidencia más directa del papel del registro de banderas; la bandera de cero es el único canal por el que DEC comunica a JNZ que debe dejar de saltar. La comparación entre T y G reproduce a escala de laboratorio la misma disyuntiva que enfrenta cualquier depurador moderno; el paso a paso entrega información completa a un costo alto y el punto de interrupción entrega velocidad a cambio de visibilidad.
