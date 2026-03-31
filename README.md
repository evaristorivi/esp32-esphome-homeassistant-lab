# esphome-homeassistant-lab
<img src="images/portada.png" alt="Portada del proyecto" style="width:90%;">



# Lista de piezas recomendadas

A continuación se muestra una lista de módulos y componentes compatibles, con enlaces de ejemplo para su compra:

- [Mini Micro USB tipo C USB 5V 1A 18650 TP4056 módulo de cargador de batería de litio con protección](https://es.aliexpress.com/item/1005009887303870.html)
- [Pantalla OLED 0,96" I2C 128x64 SSD1306](https://es.aliexpress.com/item/1005006365881525.html)
- [Placa de pruebas PCB sin soldadura, 400 puntos](https://es.aliexpress.com/item/1005010426826947.html)
- [Placa de desarrollo ESP32-C3 Super Mini, 16 pines tipo C](https://es.aliexpress.com/item/1005009014269977.html)
- [Sensor de Gas TENSTAR SCD40/SCD41 CO₂, temperatura y humedad I2C](https://es.aliexpress.com/item/1005009897956849.html)
- [AHT20 + BMP280 Módulo de temperatura, humedad y presión del aire](https://es.aliexpress.com/item/1005005321276932.html)
- [Módulo de expansión de interfaz IIC Hub I2C](https://es.aliexpress.com/item/1005002811407142.html)

> Puedes usar módulos equivalentes compatibles con ESP32 y ESPHome.


Ejemplos de integración de sensores ambientales (temperatura, humedad, presión, CO₂, etc.) y financieros (precio de ETF VWCE XETRA) con ESPHome y Home Assistant usando ESP32.

## Requisitos

### ESPHome

- **Python 3.8+** instalado en tu sistema.
- **pip** (gestor de paquetes de Python).
- Instala ESPHome con:
  ```sh
  pip install esphome
  ```
- Más información y documentación oficial:  
  [https://esphome.io/guides/installing_esphome.html](https://esphome.io/guides/installing_esphome.html)

### Home Assistant

- Puede instalarse en Raspberry Pi, PC, NAS, etc.
- Documentación oficial:  
  [https://www.home-assistant.io/installation/](https://www.home-assistant.io/installation/)

#### Ejemplo de instalación en Raspberry Pi 3 usando Docker

```sh
docker pull ghcr.io/home-assistant/raspberrypi4-homeassistant:stable

docker run -d \
  --name homeassistant \
  --restart=unless-stopped \
  -v /root/homeassistant:/config \
  -e TZ=Europe/Madrid \
  --network=host \
  ghcr.io/home-assistant/raspberrypi4-homeassistant:stable
```

> Cambia `/root/homeassistant` por la ruta donde quieras guardar la configuración.

---

## Estructura del proyecto

```
esphome/
  vwce.yaml           # Ejemplo completo de ESPHome con HTTP request
  vwce_dummy.yaml     # Ejemplo ESPHome usando integración Home Assistant
  secrets.yaml.example # Plantilla de secretos (sin datos reales)
homeassistant/
  configuration.yaml  # Configuración principal de Home Assistant
  vwce_sensor.yaml    # Sensor REST de VWCE para incluir
```

---

Cada archivo contiene comentarios explicativos sobre su propósito y uso.

---
# Ejemplos de Configuración ESPHome y Home Assistant

<img src="images/Captura%20de%20pantalla%202026-03-31%20224607.png" alt="Ejemplo de uso" style="width:60%;">

Este repositorio contiene ejemplos prácticos para integrar un ESP32 con ESPHome y Home Assistant, mostrando cómo obtener y visualizar el precio de VWCE desde Yahoo Finance.

## Estructura

- `esphome/`
  - `vwce.yaml`: Ejemplo completo de ESPHome usando petición HTTP directa a Yahoo.
  - `vwce_dummy.yaml`: Ejemplo de ESPHome que consume el sensor de Home Assistant.
  - `secrets.yaml.example`: Plantilla para tus credenciales (no subas tus claves reales).

- `homeassistant/`
  - `configuration.yaml`: Ejemplo de configuración principal mínima.
  - `vwce_sensor.yaml`: Sensor REST de VWCE listo para incluir en Home Assistant.


## Compilación y flasheo de ESPHome

1. Copia `secrets.yaml.example` a `secrets.yaml` y pon tus claves.
2. Incluye `vwce_sensor.yaml` en tu `configuration.yaml` de Home Assistant:
  ```yaml
  sensor: !include homeassistant/vwce_sensor.yaml
  ```
3. Compila y flashea tu ESP32 con ESPHome:
  - **Primera vez (por USB):**
    - Pon el ESP32 en modo flasheo (normalmente manteniendo pulsado el botón BOOT al conectar el USB y botón RESET).
    - Ejecuta:
     ```sh
     esphome run esphome/vwce.yaml --device COMx
     ```
     (Reemplaza `COMx` por el puerto serie de tu ESP32, por ejemplo `COM3` en Windows o `/dev/ttyUSB0` en Linux).
  - **Siguientes veces (por OTA/WiFi):**
    - Una vez flasheado y conectado a la red, puedes actualizar el firmware por WiFi:
     ```sh
     esphome run esphome/vwce.yaml --device nombre-o-ip.local
     ```
     (Reemplaza `nombre-o-ip.local` por el nombre mDNS o la IP de tu ESP32, por ejemplo `vwce-ticker.local` o `192.168.1.106`).

> Nota: La primera vez siempre debe ser por USB. Después, si tienes configurado OTA, puedes actualizar por WiFi cómodamente.


---

**¡Contribuciones y mejoras son bienvenidas!**
