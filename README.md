# Laboratorio 5 – Comunicaciones Industriales  
Universidad Santo Tomás, noviembre 2025  

## Resumen
Se implementó una arquitectura de red enrutada validando la segmentación de dominios de difusión mediante ARP; se estableció comunicación Maestro-Esclavo con Modbus RTU sobre RS-485 y se virtu[...] 

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
Ping inter-red exitoso; captura con `arp -a` muestra que la IP remota se resuelve a la MAC del gateway local. Se confirma que ARP es un protocolo de enlace local y que la segmentación contiene lo[...] 

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
1000 tramas transmitidas; 0 errores CRC. Voltaje diferencial A-B = 2,1 V, CMRR > 40 dB. La librería calcula y verifica CRC automáticamente, descartando tramas con ruido eléctrico.

### 3. Automatización y virtualización con TIA Portal
**Instalación del entorno**  
- Descarga oficial desde Siemens Support.
  <img width="921" height="858" alt="image" src="https://github.com/user-attachments/assets/4045ff24-713a-4494-bc2c-6b970c033014" />
- Ejecutar Start.exe con todos los paquetes en la misma carpeta (STEP 7, WinCC, Automation License Manager).
  <img width="921" height="519" alt="image" src="https://github.com/user-attachments/assets/70bc265a-16ca-467b-8ba9-103b72bcecdf" />
- Proceso de instalación 30-60 min; reinicio y activación de licencias con Automation License Manager.
  <img width="921" height="472" alt="image" src="https://github.com/user-attachments/assets/7521bf26-c26d-4071-bc00-8383fc54ec7e" />

**Configuración de hardware**  
- CPU virtual: S7-1212C DC/DC/DC, firmware V4.6.
  <img width="975" height="526" alt="image" src="https://github.com/user-attachments/assets/cffa270b-5c90-4f8a-9dc9-c6f713a34858" />
- Alimentación y entradas a 24 V DC; salidas a transistor para conmutación rápida.  
- Interfaz de simulación: PLCSIM Virtual Ethernet.

**Direccionamiento y variables**  
Uso de direccionamiento simbólico para mantenibilidad:  
- Entrada: %M0.0 → etiqueta “Start_Button”.  
- Salida: %Q0.0 → etiqueta “Motor_Lamp”.
Tabla de tags creada dentro del proyecto TIA.
<img width="975" height="528" alt="image" src="https://github.com/user-attachments/assets/66fe90b1-b343-45f8-95f9-f5617119ad25" />

**Programa Ladder**  
Bloque OB1, ciclo 10 ms:  
- Contacto normalmente abierto (NO) con dirección “Start_Button”.  
- Bobina de salida con dirección “Motor_Lamp”.
  <img width="975" height="526" alt="image" src="https://github.com/user-attachments/assets/b543e8c6-598a-45f1-aca4-0a7ea7665dc0" />

**Validación en PLCSIM**  
1. Descarga sin errores.  
2. Modo “Online”; forzado de %I0.0 = TRUE.  
3. Visualización del flujo de corriente virtual (línea verde) y cambio inmediato de %Q0.0 a TRUE.  
4. Grabación de pantalla para evidencia de funcionamiento.
<video src="https://github.com/user-attachments/assets/2410d8ee-6f49-4f12-929e-1a1806e414b3" loop autoplay muted playsinline>
    Tu navegador no soporta la etiqueta de video HTML5.
</video>

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
3. El uso de TIA Portal + S7-PLCSIM permite “virtual commissioning” completo: configuración de hardware, mapeo simbólico, programación Ladder y validación en tiempo real, reduciendo tiempo[...] 

## Referencias
- Tanenbaum, A. S. *Redes de Computadoras*, 5.ª ed., Pearson, 2012.  
- Siemens AG, *SIMATIC S7-1200 System Manual*, Edition 09/2023.  
- Modbus Organization, *Modbus over Serial Line Specification V1.02*, 2020.



