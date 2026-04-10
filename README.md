# esphome-homeassistant-lab

<p align="center">
  <img src="images/portada.png" alt="Portada del proyecto" style="width:90%;">
</p>

Ejemplos prácticos para integrar **ESP32** con **ESPHome** y **Home Assistant**, mostrando cómo obtener y visualizar datos ambientales (temperatura, humedad, presión, CO₂) y financieros (precio del ETF VWCE en XETRA).

> 📝 **Artículo en el blog:** [Monitor de CO₂ DIY con ESP32 y Home Assistant — controla la calidad del aire en casa](https://www.evaristorivieccio.es/2026/04/monitor-de-co%e2%82%82-diy-con-esp32-y-home-assistant-controla-la-calidad-del-aire-en-casa.html)

El proyecto soporta dos plataformas hardware distintas. Elige la que prefieras:

| Plataforma | Descripción | Guía |
|---|---|---|
| **ESP32-C3 Super Mini + OLED** | Placa compacta con pantalla monocromática 128×64. Sensores I²C + encoder rotatorio. | [📖 docs/ESP32_C3.md](docs/ESP32_C3.md) |
| **CYD (Cheap Yellow Display)** | Placa todo-en-uno con TFT 2.8" a color y táctil. No necesita ESP32-C3 ni OLED. | [📖 docs/CYD.md](docs/CYD.md) |

---

## Proyectos disponibles

| YAML | Placa | Qué hace |
|---|---|---|
| `c3_vwce.yaml` | C3 | Ticker VWCE autónomo — consulta Yahoo Finance directo |
| `c3_vwce_dummy.yaml` | C3 | Ticker VWCE — recibe precio desde Home Assistant |
| `c3_sensors_lab.yaml` | C3 | Estación de calidad del aire — todos los datos, 2 páginas rotatorias |
| `c3_sensors_best.yaml` | C3 | Estación óptima — solo el dato más preciso de cada magnitud, 1 página |
| `c3_sensors_best_pages.yaml` | C3 | Estación óptima — 4 páginas a pantalla completa rotatorias |
| `c3_sensors_best_pages_vwce.yaml` | C3 | Estación óptima + VWCE — 5 páginas, Yahoo Finance directo |
| `c3_sensors_best_pages_vwce_dummy.yaml` | C3 | Estación óptima + VWCE — 5 páginas, precio desde HA |
| `c3_sensors_best_pages_vwce_dummy_encoder.yaml` | C3 | Estación óptima + VWCE + encoder KY-040 — rotación automática inteligente, navegación manual, doble click para toggle auto-rotación, pulsación 5s para factory reset CO₂ |
| `cyd_dummy.yaml` | CYD | TFT 2.8" LVGL táctil — todos los datos desde HA (sin sensores físicos en el CYD) |
| `cyd_sensors_vwce_dummy.yaml` | CYD | TFT 2.8" LVGL táctil — sensores I²C directos + VWCE desde HA |
| `cyd_sensors_vwce.yaml` | CYD | TFT 2.8" LVGL táctil — sensores I²C directos + VWCE Yahoo Finance directo |
| `cyd_weather_dummy.yaml` | CYD | TFT 2.8" LVGL táctil — panel meteo + sensores interiores (via HA) + previsión + VWCE |
| `cyd_weather.yaml` | CYD | TFT 2.8" LVGL táctil — panel meteo + sensores I²C directos + previsión + VWCE desde HA |

---

## Sensores I²C

Ambas plataformas usan los mismos sensores, conectados por el bus **I²C** (4 cables: VCC, GND, SDA, SCL):

| Sensor | Dirección I²C | Datos | Precisión |
|---|---|---|---|
| **SCD40 / SCD41** | `0x62` | CO₂ (ppm), temperatura*, humedad* | CO₂ ±50 ppm |
| **AHT20** | `0x38` | **Temperatura**, **humedad** | ±0.3 °C, ±2 % RH |
| **BMP280** | `0x77` | **Presión** (hPa), temperatura* | ±1 hPa |

> \* El SCD40 y el BMP280 también miden temperatura, pero el AHT20 es más preciso y no sufre self-heating. Por eso los proyectos optimizados usan solo la temperatura y humedad del AHT20.

> El SCD4x usa la presión del BMP280 en tiempo real (`ambient_pressure_compensation_source`) para corregir la lectura de CO₂ según la altitud y condiciones atmosféricas locales.

### Niveles de CO₂ y calidad del aire

El SCD40 mide CO₂ real por absorción infrarroja no dispersiva (NDIR) — no una estimación como los sensores VOC baratos.

| ppm | Estado |
|---|---|
| < 400 | Aire exterior limpio |
| 400–700 | Excelente |
| 700–1000 | Aceptable |
| 1000–1500 | Malo — somnolencia, dificultad de concentración |
| 1500–2000 | Muy malo — dolores de cabeza, cansancio |
| > 2000 | Peligroso |

---

## Home Assistant

Home Assistant centraliza los datos de los sensores, muestra gráficas y permite crear automatizaciones. Puede instalarse en Raspberry Pi, PC, NAS, máquina virtual, etc. Es necesario para los proyectos que reciben datos "via HA" y para el sensor REST de VWCE.

Documentación oficial: [https://www.home-assistant.io/installation/](https://www.home-assistant.io/installation/)

### Instalación rápida (Raspberry Pi + Docker)

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

> Cambia `/root/homeassistant` por la ruta donde quieras guardar la configuración y `Europe/Madrid` por tu zona horaria si es diferente.

Accede a `http://<IP_DE_TU_RASPBERRY>:8123` desde el navegador para completar la configuración inicial.

### Archivos de configuración de Home Assistant

El repositorio incluye en `homeassistant/` los archivos listos para copiar al directorio de configuración de HA:

| Archivo | Qué hace | Cómo incluirlo en `configuration.yaml` |
|---|---|---|
| `vwce_sensor.yaml` | Sensor REST — consulta el precio de VWCE a Yahoo Finance | `sensor: !include vwce_sensor.yaml` |
| `air_quality_sensor.yaml` | Sensor REST \u2014 PM2.5 y PM10 desde Open-Meteo Air Quality API | `rest: !include air_quality_sensor.yaml` |
| `weather_sensors.yaml` | Templates de condición actual y previsión horaria/diaria (`weather.get_forecasts`) | `template: !include weather_sensors.yaml` |

El `configuration.yaml` incluido en el repositorio ya tiene las tres líneas configuradas como referencia.

> `air_quality_sensor.yaml` requiere añadir `air_quality_url` en el `secrets.yaml` de HA. Ver `homeassistant/secrets.yaml.example` para la plantilla con instrucciones.

> `weather_sensors.yaml` usa la entidad `weather.casa`. Si tu entidad weather tiene otro nombre, actualiza las referencias en el archivo.

---

## Primeros pasos

### 1. Instala ESPHome

```sh
pip install esphome
```

Necesitas **Python 3.8+** y **pip**. También puedes usar el add-on oficial de ESPHome en Home Assistant.

Documentación oficial: [https://esphome.io/guides/installing_esphome.html](https://esphome.io/guides/installing_esphome.html)

### 2. Configura tus credenciales

Copia `esphome/secrets.yaml.example` a `esphome/secrets.yaml` y rellena tus datos:

```yaml
wifi_ssid: "TuRedWiFi"
wifi_password: "TuContraseña"
ota_password: "una_clave_segura"
api_encryption_key: "clave_base64_de_32_bytes"
```

Para generar la clave de cifrado:

```sh
openssl rand -base64 32
```

> `secrets.yaml` está en `.gitignore` — nunca se sube al repositorio. Los valores `!secret` se leen de este archivo, así puedes publicar los YAML sin exponer contraseñas.

### 3. Instala los drivers USB (primera vez)

| Chip | Driver |
|------|--------|
| **CP2102** | [Silicon Labs VCP Drivers](https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers?tab=overview) |
| **CH340 / CH341** | [WCH CH341SER](https://www.wch.cn/downloads/CH341SER_ZIP.html) |
| **CH342 / CH343 / CH9102** | [WCH CH343SER](https://www.wch.cn/downloads/CH343SER_ZIP.html) |

> Si no aparece ningún puerto COM (Windows) o `/dev/ttyUSB0` (Linux) al conectar el ESP32, instala los dos primeros — no hay forma rápida de saber qué chip lleva sin abrirla.

### 4. Elige tu plataforma y flashea

- **ESP32-C3 Super Mini** → sigue la guía en [docs/ESP32_C3.md](docs/ESP32_C3.md)
- **CYD (Cheap Yellow Display)** → sigue la guía en [docs/CYD.md](docs/CYD.md)

> **Primera flash:** siempre por USB (botón BOOT pulsado al conectar). Las siguientes actualizaciones van por WiFi (OTA) automáticamente.

---

## Referencia técnica

### `framework: esp-idf`

ESP-IDF es el framework oficial de Espressif. Más estable que Arduino, mejor gestión de WiFi y memoria. Todos los proyectos lo usan.

### `ota` — actualización por WiFi

Permite flashear sin cable USB a partir de la segunda vez. La contraseña protege contra actualizaciones no autorizadas.

### `api` — integración con Home Assistant

Protocolo nativo de ESPHome. HA descubre el dispositivo automáticamente. `encryption.key` cifra la comunicación con AES-128.

### `wifi: power_save_mode: none`

El ESP32 siempre conectado — evita latencia y que HA lo marque como *unavailable*.

### `logger: level: DEBUG`

Envía mensajes de diagnóstico por el puerto serie y desde la UI web de ESPHome. En un dispositivo estable se puede bajar a `INFO` o `WARNING` para reducir ruido.

---

## Estructura del repositorio

```
esphome/
  c3_vwce.yaml                          # Ticker VWCE autónomo (HTTP directo)
  c3_vwce_dummy.yaml                    # Ticker VWCE via HA
  c3_sensors_lab.yaml                   # Estación laboratorio
  c3_sensors_best.yaml                  # Estación óptima — 1 página
  c3_sensors_best_pages.yaml            # Estación óptima — 4 páginas
  c3_sensors_best_pages_vwce.yaml       # Estación + VWCE (HTTP directo)
  c3_sensors_best_pages_vwce_dummy.yaml # Estación + VWCE (via HA)
  c3_sensors_best_pages_vwce_dummy_encoder.yaml # Estación + VWCE (via HA) + encoder KY-040
  cyd_dummy.yaml                        # CYD — datos desde HA, sin sensores físicos
  cyd_sensors_vwce_dummy.yaml           # CYD — sensores I²C directos + VWCE via HA
  cyd_sensors_vwce.yaml                 # CYD — sensores I²C directos + VWCE HTTP directo
  cyd_weather_dummy.yaml                # CYD — panel meteo + sensores interiores (HA) + prevision + VWCE
  cyd_weather.yaml                      # CYD — panel meteo + sensores I2C directos + prevision + VWCE
  secrets.yaml.example                  # Plantilla de credenciales ESPHome
homeassistant/
  configuration.yaml                    # Configuración principal de HA
  vwce_sensor.yaml                      # Sensor REST de VWCE (Yahoo Finance)
  air_quality_sensor.yaml               # Sensor REST de calidad del aire (Open-Meteo)
  weather_sensors.yaml                  # Templates: condición actual + previsión horaria/diaria
  secrets.yaml.example                  # Plantilla de credenciales HA (air_quality_url)
docs/
  ESP32_C3.md                           # Guía completa ESP32-C3 Super Mini
  CYD.md                                # Guía completa CYD
images/
```

---

## Mejoras futuras

- Montaje permanente: hub I²C + carcasa 3D impresa
- Diseñar PCB a medida para sustituir la protoboard

---

**¡Contribuciones y mejoras son bienvenidas!**
