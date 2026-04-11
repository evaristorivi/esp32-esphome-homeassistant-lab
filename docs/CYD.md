# CYD — Cheap Yellow Display (ESP32-2432S028R)

<p align="center">
  <img src="../images/cydpino.png" alt="Pinout ESP32-2432S028R (CYD)" style="width:80%;">
</p>

<p align="center">
  <img src="../images/cyd.jpeg" alt="Parte trasera del CYD con cables CN1" style="width:60%;">
</p>

Guía para los proyectos basados en el **ESP32-2432S028R** (CYD) — una placa todo-en-uno con pantalla **TFT 2.8" a color** (ST7789, 320×240 px), táctil resistivo (XPT2046) e interfaz gráfica **LVGL**.

> **Con la CYD no necesitas el ESP32-C3 Super Mini ni la pantalla OLED** — la CYD ya incluye su propio ESP32 y display integrado. Solo necesitas los sensores I²C y opcionalmente el hub.

---

## Lista de piezas

- [Pantalla Inteligente ESP32 2.8" — TFT 240×320 táctil (ESP32-2432S028R)](https://es.aliexpress.com/item/1005010399042230.html)
- [Sensor SCD40/SCD41 — CO₂, temperatura y humedad I2C](https://es.aliexpress.com/item/1005009897956849.html)
- [AHT20 + BMP280 — temperatura, humedad y presión](https://es.aliexpress.com/item/1005005321276932.html)
- [Módulo hub I2C (expansión de bus)](https://es.aliexpress.com/item/1005002811407142.html) *(opcional pero recomendado)*

---

## Hardware integrado del CYD

La CYD es una placa completa con ESP32-WROOM-32 y estos periféricos:

| Componente | Pines | Descripción |
|---|---|---|
| Display ST7789 | HSPI: CLK=14, MOSI=13, MISO=12, CS=15, DC=2 | Pantalla TFT 320×240 a color |
| Táctil XPT2046 | VSPI: CLK=25, MOSI=32, MISO=39, CS=33, IRQ=36 | Táctil resistivo |
| Backlight | GPIO21 (LEDC PWM) | Brillo regulable por software |
| LED RGB | R=GPIO4, G=GPIO16, B=GPIO17 (activo-bajo) | Indicador de estado |
| Conector CN1 | GND, GPIO22, GPIO27, 3V3 | **Bus I²C para sensores externos** |

> [Pinout completo del CYD — RandomNerdTutorials](https://randomnerdtutorials.com/esp32-cheap-yellow-display-cyd-pinout-esp32-2432s028r/)

---

## Conexión de sensores (conector CN1)

Los sensores se conectan al conector **CN1** del CYD por **I²C** — el mismo protocolo que en el ESP32-C3, solo cambian los pines:

| | ESP32-C3 | CYD (CN1) |
|---|---|---|
| SDA | GPIO4 | **GPIO22** |
| SCL | GPIO5 | **GPIO27** |
| VCC | 3V3 | 3V3 |
| GND | GND | GND |

Los tres sensores van en paralelo al mismo bus (direcciones distintas: SCD40=`0x62`, AHT20=`0x38`, BMP280=`0x77`).

### Con hub I²C (recomendado)

El hub I²C centraliza las conexiones y queda más limpio:

```
CYD CN1                    Hub I²C
  3.3V ────────────────── VCC
  GPIO22 (SDA) ───────── SDA
  GPIO27 (SCL) ───────── SCL
  GND ─────────────────── GND

Hub I²C puertos:
  Puerto 1 ──── SCD40 (CO₂)
  Puerto 2 ──── AHT20+BMP280 (T, H, P)
```

### Sin hub (daisy-chain)

También se pueden cablear en paralelo directamente:

| Pin sensor | Pin CYD (CN1) |
|---|---|
| **VCC** | **3V3** |
| **GND** | **GND** |
| **SDA** | **GPIO22** |
| **SCL** | **GPIO27** |

> ⚠ GPIO21 está ocupado por el backlight. GPIO35 es input-only. Solo GPIO22 y GPIO27 están disponibles para I²C en el CYD.

---

## Interfaz LVGL — Proyectos 8–10 (datos ambientales)

Los proyectos 8, 9 y 10 (`cyd_dummy`, `cyd_sensors_vwce_dummy`, `cyd_sensors_vwce`) usan una interfaz de **5 páginas** con barra de acento de color único:

- **Tema oscuro** (fondo `#0D1117`) con tarjetas centrales sombreadas
- **5 páginas:**
  - 🟢 Verde — CO₂
  - 🟠 Naranja — Temperatura
  - 🔵 Azul — Humedad
  - 🟣 Púrpura — Presión
  - 🟡 Dorado — VWCE
- **Valor grande** centrado en Montserrat 48 px — legible a distancia
- **Indicador de calidad del aire** en la página CO₂ (verde/amarillo/naranja/rojo)
- **Navegación táctil:**
  - Deslizar (swipe) izquierda/derecha → página siguiente/anterior con animación
  - Botones ‹ y › en los laterales (zona amplia de toque)
  - 5 puntos indicadores de página activa
  - Navegación circular (`page_wrap: true`)
- **Icono WiFi** — verde = conectado, rojo = desconectado
- **Auto-dim** del backlight tras 60 s sin tocar — se restaura al tocar

<p align="center">
  <img src="../images/cyd2.jpeg" alt="Página CO₂ — 686 ppm" style="width:40%;">
  <img src="../images/cyd5.jpeg" alt="Página Humedad — 59%" style="width:40%;">
</p>
<p align="center">
  <img src="../images/cyd4.jpeg" alt="Página Presión — 1025 hPa" style="width:40%;">
  <img src="../images/cyd3.jpeg" alt="Página VWCE — 148.16 EUR" style="width:40%;">
</p>

---

## Interfaz LVGL — Proyectos 11–12 (estación meteorológica)

Los proyectos 11 y 12 (`cyd_weather_dummy`, `cyd_weather`) tienen una interfaz diferente, diseñada para mostrar datos meteorológicos exteriores junto con los sensores interiores.

<p align="center">
  <img src="../images/eltiempo5.jpeg" alt="Panel principal — meteo + sensores" style="width:40%;">
  <img src="../images/eltiempo2.jpeg" alt="Página Previsión" style="width:40%;">
</p>
<p align="center">
  <img src="../images/eltiempo1.jpeg" alt="Página VWCE" style="width:80%;">
</p>

### Estructura de páginas

**3 páginas** con 3 puntos indicadores en la parte inferior:

| Página | Contenido |
|---|---|
| 1 — Panel | Icono meteo + temperatura exterior + viento + PM2.5/PM10 + CO₂ + T.INT + HUM + Presión |
| 2 — Previsión | 4 filas: +3h, +6h, día siguiente (D+1), pasado mañana (D+2) |
| 3 — VWCE | Precio del fondo VWCE (XETRA) en grande |

### Barra superior

La barra superior ocupa los 24 px superiores de la pantalla y tiene dos zonas:

- **Izquierda**: localización (p.ej. "Luanco") + hora (HH:MM) en el centro
- **Derecha** (4 iconos): ⏻ · ⏸ · 💡 · 📶

| Icono | Función |
|---|---|
| ⏻ (power) | Desactiva/activa el ahorro de energía. Rojo = energía siempre encendida; gris azulado = modo normal |
| ⏸/▶ (play/pausa) | Activa/pausa la auto-rotación de páginas. **Arranca en ON por defecto.** |
| 💡 (bombilla) | Cicla el brillo: 100% → 50% → 25% → 100% |
| 📶 (wifi) | Estado de conexión: verde = conectado, rojo = desconectado |

### Página 1 — Panel

- **Zona izquierda**: icono meteorológico grande (FontAwesome) + temperatura exterior (Montserrat 40 px) + símbolo °C
- **Zona derecha**: VIENTO en km/h y PM2.5 / PM10 en µg/m³, apilados verticalmente
- **Grid de sensores interiores** (4 celdas):
  - CO₂ (ppm) con color dinámico: verde < 700 / amarillo 700–1000 / naranja 1000–1500 / rojo > 1500; "ppm" se posiciona justo al lado del número
  - T.INT (°C)
  - HUM (%)
  - Presión (hPa) — número grande, "hPa" en fuente pequeña
- **Reset CO₂** (solo `cyd_weather.yaml`): mantener pulsado el área CO₂ ~1 s → diálogo de confirmación → **Sí** ejecuta `scd4x.factory_reset`

### Página 2 — Previsión

4 filas verticalmente centradas entre el encabezado y los puntos de página:

| Columna | Contenido |
|---|---|
| Etiqueta | +3h / +6h / día de la semana (D+1) / día de la semana (D+2) |
| Icono | Condición meteorológica (FontAwesome) |
| Temperatura | Valor en °C |
| Precipitación | Milímetros previstos (mm) |
| Viento | km/h |

### Página 3 — VWCE

- Precio del fondo VWCE en Montserrat 48 px
- Badge "XETRA" centrado en la parte inferior

### Comportamiento del backlight

- **Auto-dim**: tras 60 s sin interacción táctil, el brillo baja al 10%
- **Restaurar**: cualquier toque reactiva el brillo completo
- **Desactivar**: pulsar ⏻ mantiene el brillo al 100% permanentemente (icono en rojo)
- **Ajuste manual**: pulsar 💡 cicla entre tres niveles de brillo

---

## Calibración del táctil

Los valores por defecto (`x_min: 280`, `x_max: 3860`, `y_min: 340`, `y_max: 3860`) funcionan para la mayoría de unidades CYD. Si el toque no coincide con la posición:

1. Activa el log a `DEBUG`
2. Toca las cuatro esquinas y anota los valores raw
3. Actualiza `calibration` en el YAML

Si la pantalla aparece invertida, cambia `mirror_x` / `mirror_y` en las secciones `transform` del display y del touchscreen.

---

## Configuración de Home Assistant

Los Proyectos 11 y 12 (`cyd_weather_dummy` y `cyd_weather`) requieren archivos adicionales en Home Assistant. Todos están en la carpeta `homeassistant/` del repositorio — cópialos al directorio de configuración de HA.

### Archivos necesarios

| Archivo | Proyectos que lo usan | Qué hace |
|---|---|---|
| `vwce_sensor.yaml` | P8, P9, P11, P12 | Sensor REST — precio VWCE desde Yahoo Finance |
| `air_quality_sensor.yaml` | P11, P12 | Sensor REST — PM2.5 y PM10 desde Open-Meteo (1 req/hora) |
| `weather_sensors.yaml` | P11, P12 | Templates — condición actual + previsión +3h, +6h, día+1, día+2 |

### Incluir en `configuration.yaml`

```yaml
sensor:   !include vwce_sensor.yaml
rest:     !include air_quality_sensor.yaml
template: !include weather_sensors.yaml
```

El `configuration.yaml` del repositorio ya tiene las tres líneas como referencia.

### `secrets.yaml` de Home Assistant

`air_quality_sensor.yaml` lee la URL de la API desde `secrets.yaml` de HA (no del ESPHome). Copia `homeassistant/secrets.yaml.example` a `secrets.yaml` en tu config de HA y sustituye `LAT` y `LON` por tus coordenadas:

```yaml
air_quality_url: "https://air-quality-api.open-meteo.com/v1/air-quality?latitude=43.3603&longitude=-5.8448&current=pm2_5,pm10"
```

> La API de Open-Meteo es gratuita y no requiere clave. Las coordenadas las puedes obtener en [latlong.net](https://www.latlong.net/) o en Google Maps (clic derecho sobre tu ubicación).

### Entidad weather

`weather_sensors.yaml` usa `weather.casa` como entidad base. Si en tu instalación la entidad tiene otro nombre (p.ej. `weather.home` o `weather.mi_ciudad`), actualiza todas las referencias en el archivo.

### Diferencias respecto al ESP32-C3

| | ESP32-C3 (Proyectos 1–7) | CYD (Proyectos 8–10) |
|---|---|---|
| Placa | ESP32-C3 Super Mini | ESP32-2432S028R (ESP32-WROOM-32) |
| `esp32.board` | `esp32-c3-devkitm-1` | `esp32dev` |
| Framework | `esp-idf` | `esp-idf` |
| Display | SSD1306 128×64 mono I²C | ST7789 320×240 color SPI (HSPI) |
| Táctil | — | XPT2046 resistivo SPI (VSPI) |
| Gráficos | Lambdas C++ (canvas) | **LVGL** (widgets nativos) |
| I²C sensores | GPIO4 (SDA) / GPIO5 (SCL) | GPIO22 (SDA) / GPIO27 (SCL) — CN1 |
| Navegación | Rotación automática o encoder | Swipe + botones táctiles |
| Backlight | — | PWM en GPIO21 con auto-dim |
| LED RGB integrado | — | GPIO4/R, GPIO16/G, GPIO17/B |

---

## Proyectos

### Proyecto 8: CYD con datos desde Home Assistant (`cyd_dummy.yaml`)

Los sensores físicos están conectados al **ESP32-C3** y éste publica los datos a Home Assistant. La CYD los recibe desde HA y los muestra en pantalla. **Requiere que el ESP32-C3 esté activo y conectado a HA.**

| Dato | Origen |
|---|---|
| CO₂, Temperatura, Humedad, Presión | Desde HA (publicados por ESP32-C3) |
| VWCE | Desde HA (sensor REST) |

```sh
esphome run esphome/cyd_dummy.yaml --device COMx
esphome run esphome/cyd_dummy.yaml --device sensors-cyd-dummy.local
```

---

### Proyecto 9: CYD con sensores I²C directos + VWCE desde HA (`cyd_sensors_vwce_dummy.yaml`)

Los sensores van conectados **físicamente al CYD** por el conector CN1. No necesita el ESP32-C3 para los datos ambientales. VWCE sigue viniendo de HA.

| Dato | Origen |
|---|---|
| CO₂, Temperatura, Humedad, Presión | **Sensores I²C en CN1** (directos) |
| VWCE | Desde HA (sensor REST) |

```sh
esphome run esphome/cyd_sensors_vwce_dummy.yaml --device COMx
esphome run esphome/cyd_sensors_vwce_dummy.yaml --device sensors-cyd-vwce-dummy.local
```

---

### Proyecto 10: CYD con sensores I²C directos + VWCE desde Yahoo Finance (`cyd_sensors_vwce.yaml`)

Igual que el Proyecto 9 pero **el ESP32 consulta Yahoo Finance directamente** — no necesita HA para el precio de VWCE.

| Dato | Origen |
|---|---|
| CO₂, Temperatura, Humedad, Presión | **Sensores I²C en CN1** (directos) |
| VWCE | HTTP directo a Yahoo Finance (cada 60 s) |

```sh
esphome run esphome/cyd_sensors_vwce.yaml --device COMx
esphome run esphome/cyd_sensors_vwce.yaml --device sensors-cyd-vwce.local
```

---

### Proyecto 11: CYD estación meteorológica sin sensores I²C (`cyd_weather_dummy.yaml`)

Pantalla meteorológica con **3 páginas** (Panel, Previsión, VWCE). La información meteorológica viene de Home Assistant a través del template `weather_sensors.yaml`. Los sensores interiores (CO₂, temperatura, humedad, presión) se leen desde HA — publicados por otro ESP32 (ej. ESP32-C3). No necesita sensores físicos conectados al CYD.

| Dato | Origen |
|---|---|
| Temperatura ext., viento, condición | `weather_sensors.yaml` en HA (`weather.casa`) |
| Previsión +3h, +6h, día+1, día+2 | `weather_sensors.yaml` en HA (`weather.get_forecasts`) |
| CO₂, T interior, H interior, Presión | Desde HA (publicados por ESP32-C3) |
| PM2.5 / PM10 | `air_quality_sensor.yaml` en HA (Open-Meteo) |
| VWCE | Desde HA (`sensor.vwce_precio_yahoo`) |

**Requiere** incluir `weather_sensors.yaml` y `air_quality_sensor.yaml` en Home Assistant. También requiere que el ESP32-C3 con sensores esté activo y publicando a HA.

```sh
esphome run esphome/cyd_weather_dummy.yaml --device COMx
esphome run esphome/cyd_weather_dummy.yaml --device cyd-weather-dummy.local
```

---

### Proyecto 12: CYD estación meteorológica con sensores I²C (`cyd_weather.yaml`)

Igual que el Proyecto 11 pero con **sensores I²C físicos conectados al CN1** para los datos interiores (CO₂, temperatura, humedad, presión). Los datos meteorológicos exteriores siguen viniendo de HA. Permite tener dos estaciones independientes en habitaciones distintas sin que ninguna dependa de la otra.

| Dato | Origen |
|---|---|
| Temperatura ext., viento, condición | `weather_sensors.yaml` en HA (`weather.casa`) |
| Previsión +3h, +6h, día+1, día+2 | `weather_sensors.yaml` en HA (`weather.get_forecasts`) |
| CO₂, T interior, H interior, Presión | **Sensores I²C en CN1** (directos) |
| PM2.5 / PM10 | `air_quality_sensor.yaml` en HA (Open-Meteo) |
| VWCE | Desde HA (`sensor.vwce_precio_yahoo`) |

#### Reset de calibración CO₂

Mantener pulsado el área del valor CO₂ en pantalla (~1 s) muestra un diálogo de confirmación: **"¿Resetear CO₂?"**. Pulsando **Sí** se ejecuta `scd4x.factory_reset` — borra la calibración acumulada y reinicia el ASC desde cero. Útil al cambiar el sensor de habitación o si las lecturas se han desviado. Pulsando **No** el diálogo se cierra sin cambios.

**Requiere** tanto `weather_sensors.yaml` como `air_quality_sensor.yaml` configurados en Home Assistant.

```sh
esphome run esphome/cyd_weather.yaml --device COMx
esphome run esphome/cyd_weather.yaml --device cyd-weather.local
```

---

### Diferencias entre Proyectos 8–12

| | P8 (`cyd_dummy`) | P9 (`cyd_sensors_vwce_dummy`) | P10 (`cyd_sensors_vwce`) | P11 (`cyd_weather_dummy`) | P12 (`cyd_weather`) |
|---|---|---|---|---|---|
| Sensores ambientales | Desde Home Assistant | Físicos I²C en CN1 | Físicos I²C en CN1 | Desde HA (`weather.casa`) | Desde HA (`weather.casa`) |
| Bus I²C | No configurado | GPIO22 / GPIO27 | GPIO22 / GPIO27 | No configurado | GPIO22 / GPIO27 |
| VWCE | Desde HA | Desde HA | HTTP directo Yahoo Finance | Desde HA | Desde HA |
| PM2.5 / PM10 | No | No | No | Desde HA | Desde HA |
| Previsión meteorológica | No | No | No | Desde HA (`weather_sensors.yaml`) | Desde HA (`weather_sensors.yaml`) |
| Sensores interiores | Desde HA (ESP32-C3) | Físicos I²C en CN1 | Físicos I²C en CN1 | Desde HA (ESP32-C3) | Físicos I²C en CN1 |
| Necesita ESP32-C3 activo | Sí | No | No | Sí (para sensores interiores) | No |
| Necesita Home Assistant | Sí | Solo para VWCE | No | Sí | Sí |
| Páginas | 5 (CO₂,T,H,P,VWCE) | 5 (CO₂,T,H,P,VWCE) | 5 (CO₂,T,H,P,VWCE) | 3 (Panel, Previsión, VWCE) | 3 (Panel, Previsión, VWCE) |

> **Agradecimiento:** Este proyecto se inspira en parte en el excelente trabajo de [ESP32-HAM-CLOCK de SP3KON](https://github.com/SP3KON/ESP32-HAM-CLOCK), que sirvió de referencia para el diseño de la CYD estación meteorológica.
