# Smart-Pet-Feeder-Comedero-Inteligente-
Proyecto de un comedero inteligente para mascotas basado en dos placas STM32 que se comunican en arquitectura cliente–servidor. El cliente controla los sensores y el mecanismo de dispensado, mientras que el servidor expone un servidor web embebido para monitorizar el estado del comedero en tiempo real y consultar un histórico de eventos.
Sistema embebido **cliente–servidor** sobre dos microcontroladores STM32 que automatiza y monitoriza la alimentación de una mascota: el cliente controla sensores y el mecanismo de dispensado, y el servidor expone un **servidor web embebido** (HTTP, sin frameworks de terceros) para ver el estado en tiempo real y consultar un histórico de eventos.
 
Proyecto desarrollado para la asignatura de **Ingeniería de Sistemas Embebidos**, con foco en sistemas de tiempo real, comunicación entre dispositivos y desarrollo de servicios web sobre hardware con recursos limitados.
 
## 💡 Aspectos técnicos destacados
 
- Diseño de una **arquitectura distribuida cliente-servidor** entre dos microcontroladores, con protocolo de comunicación propio sobre UART.
- Programación concurrente con **RTOS (CMSIS-RTOS2 / Keil RTX5)**: hilos dedicados a lectura de sensores, comunicación y gestión del servidor web.
- Implementación de un **servidor HTTP embebido** con generación dinámica de contenido (CGI/CGX) y actualización de datos en el navegador vía AJAX, sin frameworks externos.
- Integración de sensores reales (ToF **VL53L0X** por I2C/SPI, ADC para humedad, sensor de peso) y un actuador controlado por **PWM**.
- Persistencia de configuración y **log de eventos en EEPROM**, con gestión de slots para evitar corrupción entre páginas de memoria.
- Gestión de energía con **modo de bajo consumo** ante inactividad del sistema.
## 📋 Descripción
 
El sistema está pensado para automatizar y supervisar la alimentación de una mascota:
 
- Mide el **nivel de comida** en el depósito mediante un sensor de distancia por tiempo de vuelo (VL53L0X).
- Mide el **peso de comida en el cuenco**.
- Mide la **humedad** del depósito para detectar condiciones que puedan estropear el alimento.
- Acciona un **motor/servo (PWM)** para dispensar comida.
- Cuenta con un **modo de bajo consumo** cuando el sistema está inactivo.
- Envía todos estos datos al servidor, que los muestra en una **página web** actualizada periódicamente (vía AJAX) y guarda un **log de eventos** (hasta 15 entradas) en memoria no volátil (EEPROM), con la fecha, hora, evento y origen de cada uno.
## 🏗️ Arquitectura
 
```
┌─────────────────────┐        UART / COM        ┌─────────────────────┐         HTTP / Ethernet
│   Placa CLIENTE     │ ───────────────────────▶ │   Placa SERVIDOR   │ ───────────────────────▶  Navegador web
│  (STM32F429ZIT)     │ ◀─────────────────────── │ (STM32F407 / F429) │
│                     │                          │                     │
│ - Sensor distancia  │                          │ - RTC (fecha/hora)  │
│   VL53L0X           │                          │ - HTTP Server       │
│ - Sensor humedad    │                          │ - Log en EEPROM     │
│   (ADC)             │                          │ - LCD               │
│ - Sensor de peso    │                          │                     │
│ - Motor/servo (PWM) │                          │                     │
│ - Modo bajo consumo │                          │                     │
└─────────────────────┘                          └─────────────────────┘
```
 
Ambas placas ejecutan **CMSIS-RTOS2 (Keil RTX5)** y se desarrollan sobre **Keil MDK-ARM**. El servidor usa además el **Keil MDK Network Middleware** (pila IPv4/IPv6) para implementar el servidor HTTP.
 
## 🌐 Interfaz web
 
La web del servidor (carpeta `Servidor/Web`) incluye, entre otras, estas páginas:
 
| Página | Función |
|---|---|
| `comedero.cgi` | Panel principal: hora/fecha RTC, peso en el cuenco, humedad del depósito, nivel/distancia, consumo, estado y alertas. Se actualiza automáticamente cada segundo mediante JavaScript/AJAX. |
| `log.cgi` | Histórico de eventos del sistema (hasta 15), con opción de borrarlo. |
| `system.cgi`, `network.cgi`, `tcp.cgi`, `rtc.cgi`, `lcd.cgi`, `leds.cgi` | Páginas de configuración base heredadas del ejemplo de servidor HTTP de Keil (autenticación, red, TCP, RTC, LCD y LEDs). |
 
## 🛠️ Tecnologías y herramientas
 
- **Microcontroladores:** STM32F407IGHx / STM32F429ZITx (evaluación MCBSTM32F400 / Nucleo F429ZI)
- **IDE / Toolchain:** Keil MDK-ARM (µVision), ARM Compiler
- **RTOS:** CMSIS-RTOS2 – Keil RTX5
- **Red:** Keil MDK Network Middleware (HTTP Server, DHCP, DNS, SNTP)
- **Sensores:** VL53L0X (distancia/nivel), sensor de humedad (ADC), sensor de peso, potenciómetro
- **Actuadores:** Motor/servo controlado por PWM
- **Web:** HTML, JavaScript, CGI/CGX (generación dinámica de contenido embebido)
- **Almacenamiento:** EEPROM para configuración y log de eventos
## 📁 Estructura del repositorio
 
```
├── Cliente/          # Firmware de la placa cliente (sensores + actuador)
│   ├── Principal.c        # Hilo principal, lógica de estados y sensores
│   ├── sensordistancia.c  # Driver del sensor de distancia (VL53L0X)
│   ├── VL53L0X.c
│   ├── adc.c               # Lectura de humedad / analógicos
│   ├── PWM.c                # Control del motor/servo dispensador
│   ├── BAJOCONSUMO.c        # Modo de bajo consumo
│   └── com.c                # Comunicación con el servidor
│
└── Servidor/         # Firmware de la placa servidor (HTTP + monitorización)
    ├── PrincipalServidor.c  # Hilo principal, estados, log en EEPROM
    ├── HTTP_Server.c / HTTP_Server_CGI.c
    ├── rtc.c, lcd.c, mem.c, sntp.c
    ├── com.c                 # Comunicación con el cliente
    └── Web/                  # Páginas web (HTML/CGI/CGX/JS)
```
 
## 🚀 Cómo compilar y ejecutar
 
1. Abrir `Cliente/P6E3.uvprojx` y `Servidor/HTTP_Server.uvprojx` en **Keil µVision (MDK-ARM)**.
2. Compilar y flashear cada proyecto en su placa correspondiente (Cliente y Servidor).
3. Conectar la placa servidor a una red local (router o cable directo/crosslink al PC).
4. Comprobar la IP asignada por DHCP (o la IPv6 auto-asignada) en la pantalla LCD de la placa.
5. Abrir en el navegador `http://<IP_de_la_placa>/` para acceder al panel del comedero (`comedero.cgi`).
> Usuario por defecto: `admin` — Contraseña por defecto: *(ninguna)*
 
## 🎓 Contexto y aprendizajes
 
Proyecto académico (asignatura de Ingeniería de Sistemas Embebidos, curso 2026) desarrollado en equipo. Permitió trabajar de forma práctica con:
 
- Diseño de sistemas de tiempo real sobre RTOS en un microcontrolador con recursos limitados.
- Integración hardware-software: sensores, actuadores, comunicación serie y memoria no volátil.
- Desarrollo de una interfaz web funcional directamente desde firmware embebido (sin sistema operativo de propósito general ni servidor externo).
- Depuración y trabajo con herramientas profesionales del sector (Keil MDK-ARM / µVision).
