# Laboratorio 5 – Comunicaciones Industriales  
Universidad Santo Tomás, noviembre 2025  

## Resumen
Se implementó una arquitectura de red enrutada validando la segmentación de dominios de difusión mediante ARP; se estableció comunicación Maestro-Esclavo con Modbus RTU sobre RS-485 y se virtualizó un entorno de control con Siemens TIA Portal y S7-PLCSIM. Los ensayos demuestran confinamiento de broadcast, robustez en bus de campo y validación completa de la lógica Ladder antes de invertir en hardware físico.

## Objetivos
- Comprobar experimentalmente que ARP no atraviesa routers y que la segmentación reduce la tormenta de broadcast.  
- Verificar la integridad de tramas Modbus RTU en bus RS-485 a 9600 bps, 8N1, con transceptores MAX485.  
- Realizar “virtual commissioning” de un PLC S7-1200: configuración de hardware, mapeo de variables, programación Ladder y simulación en tiempo real con PLCSIM.

## Marco teórico (resumido)
- ARP opera solo dentro de la LAN; los hosts delegan la entrega remota al gateway.  
- RS-485: señal diferencial A/B, impedancia 120 Ω, longitud máx. 1200 m, alta inmunidad EMI.  
- Modbus RTU: trama [Dirección(1) | Función(1) | Datos(n) | CRC(2)], polling maestro-esclavo.  
- PLC: ciclo de barrido OB1, lenguaje Ladder (IEC 61131-3), direccionamiento simbólico vs absoluto.

## Desarrollo experimental

### 1. Red IP segmentada
**Montaje**  
- Nivel acceso: switches Cisco Catalyst 2960 (capa 2).  
- Núcleo: router Cisco ISR 4321 (capa 3).  
- Hosts: PC y Raspberry Pi 4B conectados mediante par trenzado CAT-6.

**Plan de direccionamiento**  
- Red A (VLAN 10): 192.168.10.0/24 – Gateway 192.168.10.1  
- Red B (VLAN 20): 192.168.20.0/24 – Gateway 192.168.20.1

**Prueba y análisis**  
Ping inter-red exitoso; captura con `arp -a` muestra que la IP remota se resuelve a la MAC del gateway local. Se confirma que ARP es un protocolo de enlace local y que la segmentación contiene los broadcasts.

### 2. Bus RS-485 con Modbus RTU
**Hardware**  
- Maestro: Raspberry Pi 4B + módulo MAX3485 (3,3 V).  
- Esclavo: Arduino Nano + MAX485 (5 V) + sensor DS18B20.  
- Topología bus lineal, extremos terminados en 120 Ω, bias-resistors 680 Ω.

**Software**  
- Librerías: pyserial y minimalmodbus.  
- Parámetros: 9600 bps, 8 bits, paridad N, 1 stop-bit, timeout 200 ms.  
- Esquema: polling cíclico (1 s) al registro 0 del esclavo (dirección 1, función 0x03).

**Resultados**  
1000 tramas transmitidas; 0 errores CRC. Voltaje diferencial A-B = 2,1 V, CMRR &gt; 40 dB. La librería calcula y verifica CRC automáticamente, descartando tramas con ruido eléctrico.

### 3. Automatización y virtualización con TIA Portal
**Instalación del entorno**  
- Descarga oficial desde Siemens Support.  
- Ejecutar Start.exe con todos los paquetes en la misma carpeta (STEP 7, WinCC, Automation License Manager).  
- Proceso de instalación 30-60 min; reinicio y activación de licencias con Automation License Manager.

**Configuración de hardware**  
- CPU virtual: S7-1212C DC/DC/DC, firmware V4.6.  
- Alimentación y entradas a 24 V DC; salidas a transistor para conmutación rápida.  
- Interfaz de simulación: PLCSIM Virtual Ethernet.

**Direccionamiento y variables**  
Uso de direccionamiento simbólico para mantenibilidad:  
- Entrada: %I0.0 → etiqueta “Start_Button”.  
- Salida: %Q0.0 → etiqueta “Motor_Lamp”.  
Tabla de tags creada dentro del proyecto TIA.

**Programa Ladder**  
Bloque OB1, ciclo 10 ms:  
- Contacto normalmente abierto (NO) con dirección “Start_Button”.  
- Bobina de salida con dirección “Motor_Lamp”.  
Circuito de mando directo con auto-mantenimiento opcional (no implementado en esta versión básica).

**Validación en PLCSIM**  
1. Descarga sin errores.  
2. Modo “Online”; forzado de %I0.0 = TRUE.  
3. Visualización del flujo de corriente virtual (línea verde) y cambio inmediato de %Q0.0 a TRUE.  
4. Tiempo de respuesta medido: 9,8 ms (coincide con ciclo de barrido).  
5. Grabación de pantalla para evidencia de funcionamiento.

**Ventajas del virtual commissioning**  
- Depuración de lógica sin hardware físico.  
- Reducción de riesgos y costos durante la puesta en marcha.  
- Validación completa de mapeo de memoria, tiempos de escaneo y visualización HMI antes de integrar la planta.

## Resultados globales
- Segmentación de broadcast: dominios ARP aislados entre VLANs.  
- Bus de campo: 0 errores en 1000 tramas; señal diferencial estable.  
- PLC: lógica Ladder validada 100 % en PLCSIM; listo para descarga a CPU física.

## Conclusiones
1. La segmentación a nivel capa 3 contiene las tormentas de broadcast y mejora la seguridad OT.  
2. RS-485 con Modbus-RTU mantiene integridad de datos en entornos ruidosos; su simplicidad y robustez lo mantienen vigente.  
3. El uso de TIA Portal + S7-PLCSIM permite “virtual commissioning” completo: configuración de hardware, mapeo simbólico, programación Ladder y validación en tiempo real, reduciendo tiempos y costos de puesta en marcha.

## Referencias
- Tanenbaum, A. S. *Redes de Computadoras*, 5.ª ed., Pearson, 2012.  
- Siemens AG, *SIMATIC S7-1200 System Manual*, Edition 09/2023.  
- Modbus Organization, *Modbus over Serial Line Specification V1.02*, 2020.

## Imágenes representativas
![Topología red](imagenes/Fig1_topologia_red.png)  
![Captura ARP](imagenes/Fig2_tabla_arp.png)  
![Bus RS-485](imagenes/Fig3_bus_rs485.png)  
![Configuración TIA hardware](imagenes/Fig5_tia_hardware.png)  
![Ladder OB1](imagenes/Fig6_ladder_ob1.png)  

## Video de demostración PLCSIM
[PLCSIM_Demo_2025-11-21.mp4 (1 min 15 s)](videos/PLCSIM_Demo_2025-11-21.mp4) – Inicio del emulador, descarga del programa, forzado de entrada y activación de salida.

## Licencia
CC BY-NC-SA 4.0
