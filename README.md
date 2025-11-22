# 📘 Laboratorio 5 – Comunicaciones Industriales  
**Universidad Santo Tomás – Ingeniería Electrónica**  
**Noviembre 2025**

**Estudiantes:**  
- Ferney Arturo Amaya Gómez  
- David Esteban Diaz Castro  
- Jhonny Alejandro Mejia León  

**Docente:**  
ING. Diego Alejandro Barragán Vargas  

---

# 1. Resumen

Este informe presenta el desarrollo experimental de tres módulos clave en la automatización industrial:

1. **Infraestructura de red IP segmentada**, validando ARP y enrutamiento entre VLANs.  
2. **Comunicación industrial RS-485 con Modbus RTU**, implementada entre Raspberry Pi (maestro) y Arduino Nano (esclavo).  
3. **Automatización y virtual commissioning en TIA Portal**, configurando un PLC S7-1200 y verificando la lógica Ladder mediante PLCSIM.

Los resultados demuestran comunicación estable, enrutamiento correcto y validación funcional completa en simulación.

---

# 2. Introducción

El laboratorio integra tecnologías **IT** (switching, routing, ARP) con **OT** (buses industriales, PLC, sensores).  
Los procedimientos abarcan desde capas físicas como **RS-485**, pasando por protocolos de enlace y red, hasta la capa de aplicación mediante **Modbus RTU** y lógica Ladder en PLC.

---

# 3. Marco Teórico

## 3.1 ARP y segmentación de red
ARP traduce direcciones **IP → MAC**, pero funciona solo dentro del **dominio de broadcast**, lo que significa que **no atraviesa routers**.  
La segmentación mediante VLANs o routing mejora la seguridad y reduce ruido de red.

## 3.2 RS-485
- Señal diferencial A/B  
- Hasta 1200 m de distancia  
- Impedancia 120 Ω  
- Topología bus lineal  
- Alta inmunidad al ruido  

## 3.3 Modbus RTU
Estructura general:
Funciona bajo un esquema **Maestro–Esclavo**, con consultas cíclicas del maestro.

## 3.4 PLC y Ladder
El PLC ejecuta cíclicamente OB1: lee entradas → ejecuta lógica → actualiza salidas.  
Ladder es parte del estándar IEC 61131-3.

---

# 4. Desarrollo Experimental

---

# 4.1 Punto 1 — Red IP segmentada

## 4.1.1 Montaje del sistema
Se utilizó:

- Switches Cisco Catalyst 2960  
- Router Cisco ISR 4321  
- PC y Raspberry Pi 4B  
- Cableado CAT-6  

<img width="914" height="521" alt="image" src="https://github.com/user-attachments/assets/71d1bedd-ebb1-453a-bc47-a76e681e65bc" />

---

## 4.1.2 Plan de direccionamiento

| Red | VLAN | Segmento | Gateway |
|-----|------|----------|---------|
| Red A | VLAN 10 | 192.168.10.0/24 | 192.168.10.1 |
| Red B | VLAN 20 | 192.168.20.0/24 | 192.168.20.1 |

---

## 4.1.3 Análisis ARP

- El *ping* entre redes funciona correctamente.  
- La tabla ARP muestra la **MAC del gateway**, no la del host final.  

✔ Esto confirma que **ARP no atraviesa routers**, lo cual valida la segmentación.

---

# 4.2 Punto 2 — Bus RS-485 y Modbus RTU

## 4.2.1 Maestro (Raspberry Pi)
Configuración:

- UART libre mediante `raspi-config`  
- Librerías: `pyserial`, `minimalmodbus`  
- Parámetros: **9600 bps, 8N1, timeout 200 ms**


<img width="921" height="858" alt="image" src="https://github.com/user-attachments/assets/4045ff24-713a-4494-bc2c-6b970c033014" />

---

## 4.2.2 Implementación física

- Esclavo: Arduino Nano + MAX485  
- Terminación 120 Ω en los extremos  
- Bias resistors: 680 Ω  
- Topología bus lineal  



---

## 4.2.3 Pruebas y resultados

- 1000 tramas  
- **0 errores CRC**  
- Diferencial A-B ≈ **2.1 V**  
- La librería descarta automáticamente tramas corruptas  

✔ Comunicación estable incluso en presencia de ruido eléctrico.

---

# 4.3 Punto 3 — Automatización con TIA Portal

## 4.3.1 Instalación del entorno TIA Portal

A continuación se muestran los pasos principales para realizar la instalación del TIA Portal, acompañados de las imágenes que documentan el proceso:

### 1. Descarga desde Siemens Support
En esta pantalla se accede al portal oficial donde se descargan los paquetes de instalación de TIA Portal.  
Se debe descargar la versión completa junto con los módulos asociados (STEP 7, WinCC, etc.).

<img width="921" height="858" alt="image" src="https://github.com/user-attachments/assets/4045ff24-713a-4494-bc2c-6b970c033014" />

### 2. Ejecución del instalador Start.exe
Una vez descargados los archivos, todos deben estar en la misma carpeta.  
La ejecución del archivo **Start.exe** permite iniciar la instalación global de TIA Portal.

<img width="921" height="519" alt="image" src="https://github.com/user-attachments/assets/70bc265a-16ca-467b-8ba9-103b72bcecdf" />

### 3. Instalación de componentes
En este paso el instalador despliega los módulos que serán instalados: STEP 7, WinCC, soporte de librerías, Automation License Manager, entre otros.  
Dependiendo del equipo, esta fase puede tardar entre 30–60 minutos.

<img width="921" height="472" alt="image" src="https://github.com/user-attachments/assets/7521bf26-c26d-4071-bc00-8383fc54ec7e" />

---
# 4.3 Punto 3 — Automatización con TIA Portal

## 4.3.1 Instalación del entorno TIA Portal

A continuación se muestran los pasos principales para realizar la instalación del TIA Portal, acompañados de las imágenes que documentan el proceso:

### 1. Descarga desde Siemens Support
En esta pantalla se accede al portal oficial donde se descargan los paquetes de instalación de TIA Portal.  
Se debe descargar la versión completa junto con los módulos asociados (STEP 7, WinCC, etc.).

<img width="921" height="858" alt="image" src="https://github.com/user-attachments/assets/4045ff24-713a-4494-bc2c-6b970c033014" />

### 2. Ejecución del instalador Start.exe
Una vez descargados los archivos, todos deben estar en la misma carpeta.  
La ejecución del archivo **Start.exe** permite iniciar la instalación global de TIA Portal.

<img width="921" height="519" alt="image" src="https://github.com/user-attachments/assets/70bc265a-16ca-467b-8ba9-103b72bcecdf" />

### 3. Instalación de componentes
En este paso el instalador despliega los módulos que serán instalados: STEP 7, WinCC, soporte de librerías, Automation License Manager, entre otros.  
Dependiendo del equipo, esta fase puede tardar entre 30–60 minutos.

<img width="921" height="472" alt="image" src="https://github.com/user-attachments/assets/7521bf26-c26d-4071-bc00-8383fc54ec7e" />

---

## 4.3.2 Configuración de hardware (S7-1200)

En este panel se selecciona el dispositivo principal del proyecto.  
En nuestro caso, se utilizó una **CPU S7-1212C DC/DC/DC**, que cuenta con:

- Entradas digitales a 24 V DC  
- Salidas digitales a transistor  
- Puerto PROFINET  
- Capacidad para ampliaciones

Esta selección determina la estructura del proyecto y los bloques disponibles.

<img width="975" height="526" alt="image" src="https://github.com/user-attachments/assets/cffa270b-5c90-4f8a-9dc9-c6f713a34858" />

---

## 4.3.3 Variables y direccionamiento simbólico

En esta sección se crean los **Tags** (etiquetas) asociados a las entradas, salidas y memorias del PLC.  
El direccionamiento simbólico permite usar nombres significativos en lugar de direcciones absolutas, facilitando la mantenibilidad del código.

Ejemplos:

- `%I0.0` → **Start_Button**  
- `%Q0.0` → **Motor_Lamp**

La tabla de variables permite organizar, clasificar y documentar todas las señales del proyecto.

<img width="975" height="528" alt="image" src="https://github.com/user-attachments/assets/66fe90b1-b343-45f8-95f9-f5617119ad25" />

---

## 4.3.4 Lógica Ladder en OB1

Este bloque corresponde al **OB1**, que es el ciclo principal del PLC.  
La lógica implementada consiste en:

- Un contacto normalmente abierto (NO) asociado a `Start_Button`  
- Una bobina de salida que energiza `Motor_Lamp`  

Cuando la entrada es activada (ya sea de forma real o forzada en PLCSIM), la salida se activa inmediatamente, mostrando el flujo lógico en color verde.

<img width="975" height="526" alt="image" src="https://github.com/user-attachments/assets/b543e8c6-598a-45f1-aca4-0a7ea7665dc0" />

---

## 4.3.5 Validación en PLCSIM

En esta fase se valida la lógica de control mediante simulación:

- Se fuerza la entrada `Start_Button`  
- PLCSIM muestra el flujo activo en verde  
- La salida `Motor_Lamp` cambia a **TRUE**  
- Esto confirma el funcionamiento correcto de la lógica creada

Video de validación:
<video src="https://github.com/user-attachments/assets/2410d8ee-6f49-4f12-929e-1a1806e414b3" loop autoplay muted playsinline>
    Tu navegador no soporta la etiqueta de video HTML5.
</video>


# 5. Resultados globales

- **ARP** confirmó aislamiento entre dominios de broadcast.  
- **Modbus RTU** funcionó sin errores (0 CRC).  
- **PLC + PLCSIM** validó el ciclo completo de control sin hardware físico.  

---

# 6. Conclusiones

1. La segmentación mejora seguridad, desempeño y ordenamiento de tráfico.  
2. RS-485 demuestra su robustez y vigencia industrial.  
3. Virtual commissioning reduce costos y permite depurar lógicas antes de energizar equipos reales.  

---

# 7. Referencias

- Tanenbaum, A. S. *Redes de Computadoras*, Pearson, 2012.  
- Siemens AG. *SIMATIC S7-1200 System Manual*, 2023.  
- Modbus Organization. *Modbus over Serial Line V1.02*, 2020.  




