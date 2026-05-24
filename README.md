Laboratorio: Depuración y Ensamblado con DEBUG en DOSBox
Arquitectura de Computadores — Unidad 3: Manejo del DEBUG
Post-Contenido 1 | Ingeniería de Sistemas | UFPS | 2026

Objetivo del laboratorio
Este laboratorio tiene como finalidad configurar el entorno DOSBox, acceder al depurador DEBUG y emplear los comandos R, D, F y U para examinar el estado de los registros del procesador, manipular bloques de memoria y traducir código máquina a instrucciones ensamblador. La práctica permite entender cómo el procesador 8086 gestiona su estado interno y por qué en modo real de x86 no existe separación entre regiones de código y datos.

Estructura del repositorio
Balaguera-post1-u3/
├── README.md
└── capturas/
    ├── CP1_registros.png
    ├── CP2_volcado_memoria.png
    └── CP3_ensamblado_desensamblado.png

Parte A — Preparación del entorno DOSBox
Se inició DOSBox y se enlazó la carpeta de trabajo local como unidad C: virtual:
Z:\> MOUNT C C:\DOSWork
Drive C is mounted as local directory C:\DOSWork\
Z:\> C:
C:\> MD LAB3POST1
C:\> CD LAB3POST1
C:\LAB3POST1> DEBUG
-
Se creó el directorio LAB3POST1 para mantener organizados los archivos de la sesión. El prompt - indica que DEBUG se ejecutó correctamente.

Parte B — Revisión de Registros con R
Comandos utilizados
-R
-R AX
:1234
-R


Análisis
Al invocar R sin parámetros, DEBUG despliega el estado completo del procesador. Los registros de propósito general AX, BX, CX y DX inician en 0000; el puntero de pila SP apunta a FFFE; los registros de segmento DS, ES, SS y CS comparten el valor del segmento PSP asignado por DOS, y el registro IP comienza en 0100, primera dirección ejecutable tras los 256 bytes del PSP.
Al modificar AX con el valor 1234h mediante R AX, se confirmó que el cambio es puntual: únicamente ese registro se actualiza mientras los demás permanecen intactos.

Parte C — Manipulación de Memoria con D y F
Comandos utilizados
-F 200 L40 AB CD EF
-D 200 L40
-D 0 L20

Análisis
La salida de D se organiza en tres columnas: dirección lógica en formato segmento:desplazamiento, 16 bytes en hexadecimal divididos en dos grupos de 8, y su representación ASCII donde los valores no imprimibles se reemplazan por ..
El patrón AB CD EF se repite de forma cíclica cubriendo los 64 bytes desde DS:0200. El volcado del PSP con D 0 L20 reveló los bytes CD 20 al inicio, correspondientes a la instrucción INT 20 que DOS utiliza como mecanismo de terminación de programas COM.

Parte D — Ensamblado y Desensamblado con A y U
Comandos utilizados
-U 100 L10
-A 100
1357:0100 MOV AX, 0005
1357:0103 MOV BX, 0003
1357:0106 ADD AX, BX
1357:0108 INT 20
1357:010A
-U 100 109

Análisis
El comando U 100 L10 sobre el estado inicial mostró únicamente INT 20 seguido de bytes cero, decodificados como ADD [BX+SI],AL. Esto evidencia que en modo real x86 cualquier byte al que apunte CS:IP es interpretado como instrucción, sin distinguir entre código y datos.
Tras ensamblar con A, el comando U 100 109 verificó la correspondencia entre mnemónicos y bytes:
DirecciónBytesInstrucciónDescripción1357:0100B8 05 00MOV AX,0005Opcode B8 + inmediato en little-endian1357:0103BB 03 00MOV BX,0003Opcode BB + inmediato en little-endian1357:010603 C3ADD AX,BXOpcode 03 + ModRM C31357:0108CD 20INT 20Interrupción de terminación DOS
El programa ocupa 10 bytes en total. El formato little-endian se aprecia claramente en MOV AX,0005 donde 0x0005 se almacena como 05 00.

Conclusiones
La práctica permitió comprobar cómo DEBUG da acceso directo al estado del procesador 8086 y a la memoria del proceso. Se verificó que en modo real x86 la memoria es un espacio plano sin separación entre código, datos y estructuras del sistema operativo. El comando F facilitó inicializar zonas de memoria con patrones identificables, y el ciclo A → U dejó clara la relación entre instrucciones ensamblador y su codificación en bytes, reforzando la comprensión del formato little-endian.

Referencias

Liu, Y. C., & Gibson, G. A. (1986). Microcomputer Systems: The 8086/8088 Family (2nd ed.). Prentice-Hall.
Triebel, W. A., & Singh, A. (2003). The 8088 and 8086 Microprocessors (4th ed.). Pearson.
DOSBox Team. (2023). DOSBox 0.74-3 documentation. https://www.dosbox.com/wiki/Main_Page
