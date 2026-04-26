# ESP32-C3 Super Mini + OLED

<p align="center">
  <img src="../images/esquema.webp" alt="Pinout ESP32-C3 Super Mini" style="width:80%;">
</p>

Guía para los proyectos basados en el **ESP32-C3 Super Mini** con pantalla **OLED SSD1306 128×64** y sensores I²C físicos. Incluye variantes con ticker VWCE y encoder rotatorio.

---

## Lista de piezas

- [Placa ESP32-C3 Super Mini, 16 pines tipo C](https://es.aliexpress.com/item/1005009014269977.html)
- [Pantalla OLED 0,96" I2C 128×64 SSD1306](https://es.aliexpress.com/item/1005006365881525.html)
- [Sensor SCD40/SCD41 — CO₂, temperatura y humedad I2C](https://es.aliexpress.com/item/1005009897956849.html)
- [AHT20 + BMP280 — temperatura, humedad y presión](https://es.aliexpress.com/item/1005005321276932.html)
- [Módulo hub I2C (expansión de bus)](https://es.aliexpress.com/item/1005002811407142.html)
- [Placa de pruebas sin soldadura, 400 puntos](https://es.aliexpress.com/item/1005010426826947.html)
- [Módulo KY-040 — encoder rotatorio 360°](https://es.aliexpress.com/item/1005008796691020.html) *(solo Proyecto 7)*
- [TP4056 — cargador LiPo con protección](https://es.aliexpress.com/item/1005009887303870.html) *(descartado — ver nota)*

> **Nota sobre la protoboard y el hub I²C:** La protoboard es ideal para montar y probar el circuito rápidamente. El hub I²C permite centralizar todas las conexiones en un módulo pequeño y ordenado — útil para un montaje permanente dentro de una carcasa.

> **Nota sobre la batería:** El TP4056 se incluyó para explorar alimentación con LiPo. Se descartó: una LiPo descargándose baja la tensión y el ESP32 se reinicia. La solución requeriría un *boost converter* a 3.3V. El proyecto usa alimentación USB directa.

---

## Esquema de conexión

Todos los sensores y la pantalla OLED se comunican por **I²C** — 4 cables.

| Pin módulo | Pin ESP32-C3 |
|---|---|
| **VCC** | **3V3** |
| **GND** | **GND** |
| **SDA** | **GPIO4** |
| **SCL** | **GPIO5** |

> Si usas el hub I²C, conecta un único par SDA/SCL desde el ESP32 al hub y desde ahí reparte al resto de módulos.

### Direcciones I²C

`scan: true` escanea el bus al arrancar y muestra las direcciones detectadas en el log:

```
[C][i2c.idf:119]: Found device at address 0x38   ← AHT20
[C][i2c.idf:119]: Found device at address 0x3C   ← SSD1306
[C][i2c.idf:119]: Found device at address 0x62   ← SCD40
[C][i2c.idf:119]: Found device at address 0x77   ← BMP280
```

> La frecuencia del bus es 100 kHz (máximo del SCD40). El SSD1306 y BMP280 soportan 400 kHz pero se limitan al más lento.

### Conexión del encoder KY-040 (solo Proyecto 7)

| Pin KY-040 | Pin ESP32-C3 |
|---|---|
| **CLK** | **GPIO6** |
| **DT** | **GPIO7** |
| **SW** | **GPIO10** |
| **+** | **3V3** |
| **GND** | **GND** |

> GPIO4 y GPIO5 ocupados por I²C. GPIO6, GPIO7 y GPIO10 están libres y soportan pull-up interno.

---

## LED integrado GPIO8

El ESP32-C3 Super Mini tiene un LED azul en GPIO8 (invertido: LOW = encendido):

Todos los YAML incluyen el bloque comentado:

```yaml
# light:
#   - platform: status_led
#     name: "LED"
#     pin:
#       number: GPIO8
#       inverted: true
#     restore_mode: ALWAYS_OFF
```

Descomentarlo añade un botón en HA para encender/apagar el LED. Útil para verificar que el pin funciona.

> El ESP32-C3 *Mini* (distinto al *Super Mini*) tiene un LED RGB (WS2812) en GPIO8 — requiere `esp32_rmt_led_strip`.

---

## Proyectos

### Proyecto 1: ticker VWCE autónomo (`c3_vwce.yaml`)

El ESP32 consulta **directamente** Yahoo Finance cada 60 s y muestra el precio en la OLED. **No necesita Home Assistant.**

**Cómo funciona:**
1. Petición GET a `query1.finance.yahoo.com` cada 60 s
2. Parsea `regularMarketPrice` del JSON
3. Publica en `vwce_price` con `publish_state()`
4. La OLED lo muestra en tiempo real
5. Si tienes HA, también expone `sensor.vwce_ticker_vwce_precio`

> **`useragent: "Mozilla/5.0"`** — Yahoo Finance bloquea peticiones que no parecen de un navegador.

<p align="center">
  <img src="../images/1.png" alt="Ticker VWCE en OLED" style="width:50%;">
</p>

> **Primera vez:** mantén pulsado el botón **BOOT** del ESP32 mientras lo conectas por USB, suéltalo, y ejecuta el comando con `--device COMx` (reemplaza `COMx` por el puerto de tu equipo: `COM3` en Windows, `/dev/ttyUSB0` en Linux). A partir de la segunda vez, ESPHome actualiza por WiFi (OTA).

```sh
# Primera vez por USB
esphome run esphome/c3_vwce.yaml --device COMx
# Siguientes veces por OTA
esphome run esphome/c3_vwce.yaml --device vwce-ticker.local
```

---

### Proyecto 2: ticker VWCE via Home Assistant (`c3_vwce_dummy.yaml`)

Variante para cuando ya tienes HA con el sensor REST de VWCE configurado. El ESP32 **recibe el precio desde HA** en vez de consultar Yahoo Finance.

| | `c3_vwce.yaml` | `c3_vwce_dummy.yaml` |
|---|---|---|
| Quién consulta Yahoo Finance | El ESP32 | Home Assistant |
| Requiere HA activo | No | Sí |
| `http_request` en el YAML | Sí | No |
| `vwce_price` se expone a HA | Sí | No (`internal: true`) |

#### Configurar Home Assistant

Añade en `homeassistant/configuration.yaml`:

```yaml
sensor: !include vwce_sensor.yaml
```

El archivo `vwce_sensor.yaml` contiene el sensor REST listo para usar.

```sh
esphome run esphome/c3_vwce_dummy.yaml --device COMx
esphome run esphome/c3_vwce_dummy.yaml --device vwce-dummy.local
```

---

### Proyecto 3: estación de calidad del aire — laboratorio (`c3_sensors_lab.yaml`)

Conecta los tres sensores y muestra **todos** sus datos en dos páginas rotatorias (cada 6 s). Expone todo a HA para poder comparar sensores.

<p align="center">
  <img src="../images/todos_los_sensores.png" alt="Todos los sensores conectados" style="width:80%;">
</p>

#### Sensores conectados

| Sensor | Plataforma ESPHome | Dirección I²C | Datos |
|---|---|---|---|
| **SCD40 / SCD41** | `scd4x` | `0x62` (fija) | CO₂ (ppm), temperatura, humedad |
| **AHT20** | `aht10` con `variant: AHT20` | `0x38` | Temperatura, humedad |
| **BMP280** | `bmp280_i2c` | `0x77`* | Temperatura, presión (hPa) |
| **SSD1306** | `ssd1306_i2c` | `0x3C` | Pantalla 128×64 |

> \* El módulo combo AHT20+BMP280 habitual en AliExpress lleva SDO a VCC → dirección `0x77`. Si tienes un BMP280 suelto (SDO a GND), la dirección será `0x76`. El log de arranque con `scan: true` te lo confirma.

#### Entidades en Home Assistant

| Entidad | Unidad | Fuente |
|---|---|---|
| `sensor.sensors_lab_co2` | ppm | SCD40 |
| `sensor.sensors_lab_scd4x_temperatura` | °C | SCD40 |
| `sensor.sensors_lab_scd4x_humedad` | % | SCD40 |
| `sensor.sensors_lab_aht20_temperatura` | °C | AHT20 |
| `sensor.sensors_lab_aht20_humedad` | % | AHT20 |
| `sensor.sensors_lab_bmp280_temperatura` | °C | BMP280 |
| `sensor.sensors_lab_bmp280_presion` | hPa | BMP280 |

<p align="center">
  <img src="../images/HA.jpeg" alt="Entidades en Home Assistant" style="width:80%;">
</p>

#### Comparativa de sensores — ¿por qué hay datos duplicados?

El SCD40, el AHT20 y el BMP280 miden algunas magnitudes en común. En `c3_sensors_lab.yaml` se exponen todas para poder verlas y comparar. `c3_sensors_best_pages.yaml` (Proyecto 4) aplica la conclusión y expone solo el dato óptimo.

**Temperatura:**

| Sensor | Precisión | Problema |
|---|---|---|
| **AHT20** | **±0.3 °C** | Ninguno — **mejor opción** |
| BMP280 | ±0.5 °C | Aceptable |
| SCD40 | ±0.8 °C | *Self-heating*: la cámara de CO₂ genera calor y lee **1.5–2 °C por encima** aunque aplique un offset interno (`Temperature offset: 4.00°C` en el log) |

**Humedad:**

| Sensor | Precisión | Problema |
|---|---|---|
| **AHT20** | **±2 % RH** | **Mejor opción** |
| SCD40 | ±6 % RH | Error triple + self-heating |

**CO₂ y presión:** sin competencia — SCD40 es el único NDIR, BMP280 el único barómetro.

#### Pantalla OLED

La pantalla alterna cada **6 segundos** entre dos páginas. Los sensores se leen cada **30 s**; la pantalla se redibuja cada segundo con el último valor disponible.

- **Página 1 — CO₂:** Cabecera `CO2` / `SCD40` + barras WiFi. Valor grande centrado (font 24 px). Temperatura y humedad del SCD40 en pie *(se muestran para comparación, aunque AHT20 sea más preciso)*
- **Página 2 — Clima:** Cabecera `CLIMA` / `AHT+BMP` + barras WiFi. Temperatura AHT20 · Humedad AHT20 · Presión BMP280

> Un espacio interior con personas y ventanas cerradas sube fácilmente a 1000–1400 ppm. Abrir una ventana 10 minutos lo baja rápidamente.

#### Auto-calibración del SCD40 (ASC)

El SCD40 asume que el mínimo de 7 días es ~400 ppm y se recalibra. Si nunca se expone a aire exterior puede descalibrarse. Para desactivar:

```yaml
- platform: scd4x
  automatic_self_calibration: false
```

> La ASC funciona en los modos soportados por el sensor. En SCD40 usa `periodic` (por defecto) o `low_power_periodic`. `single_shot` es solo para SCD41.

#### Compensación de presión ambiental

El SCD4x asume por defecto 1013.25 hPa (nivel del mar). Si la presión real difiere, la lectura de CO₂ se desviará ~1.6 ppm por cada hPa. Como ya tenemos el BMP280 en el mismo bus I²C, todos los proyectos con ambos sensores usan `ambient_pressure_compensation_source` para que el SCD4x corrija en tiempo real:

```yaml
- platform: scd4x
  ambient_pressure_compensation_source: bmp280_press
```

La variación meteorológica diaria (±20–30 hPa) introduce un error máximo de ±50 ppm sin compensación — irrelevante para detectar niveles de calidad del aire, pero la compensación está disponible y tiene coste cero.

#### Compatibilidad de modos SCD4x

Importante: según la documentación oficial de ESPHome y la especificación de Sensirion, `measurement_mode: single_shot` solo está soportado por SCD41. En SCD40 puede parecer que funciona en algunos casos, pero está fuera de especificación y no debe considerarse válido.

Por eso, en este repositorio los YAML con SCD40 ya no fuerzan `measurement_mode: single_shot`; se usa el modo por defecto (`periodic`) o, si se prioriza consumo, `low_power_periodic`.

#### Frecuencias

| Componente | Frecuencia |
|---|---|
| Lectura de sensores | 30 s |
| Redibujado de pantalla | 1 s |
| Rotación de página | 6 s |
| SCD40 internamente | modo de medición soportado por especificación (`periodic` por defecto) |

```sh
esphome run esphome/c3_sensors_lab.yaml --device COMx
esphome run esphome/c3_sensors_lab.yaml --device sensors-lab.local
```

---

### Proyecto 4: estación óptima — 4 páginas (`c3_sensors_best_pages.yaml`)

Mismos sensores pero expone **solo el dato más preciso** de cada magnitud. Una magnitud por página en font 40 px, legible a distancia.

| Magnitud | Sensor elegido | Motivo |
|---|---|---|
| CO₂ | SCD40 | Único NDIR |
| Temperatura | AHT20 | ±0.3 °C, sin self-heating |
| Humedad | AHT20 | ±2 % RH |
| Presión | BMP280 | Único barómetro |

#### Entidades en HA (solo 4)

| Entidad | Unidad | Sensor |
|---|---|---|
| `sensor.sensors_best_pages_co2` | ppm | SCD40 |
| `sensor.sensors_best_pages_temperatura` | °C | AHT20 |
| `sensor.sensors_best_pages_humedad` | % | AHT20 |
| `sensor.sensors_best_pages_presion` | hPa | BMP280 |

<p align="center">
  <img src="../images/sensores.jpeg" alt="Pantalla OLED 4 páginas" style="width:50%;">
</p>

#### Layout de cada página (128×64 px)

```
█ CO2                      ▂▄▆█ █

           1295

             ppm
```

| Zona | y px | Contenido |
|---|---|---|
| Cabecera invertida | 0–12 | Etiqueta + barras WiFi |
| Valor | 13–53 | Número en 40 px centrado |
| Pie | 55–63 | Unidad |

En CO₂ el pie muestra la calidad del aire:

| CO₂ (ppm) | Pantalla |
|---|---|
| ≤ 700 | `BUENO` |
| 701–1000 | `ACEPTABLE` |
| 1001–1500 | `MALO` |
| > 1500 | `PELIGROSO` |

> `c3_sensors_best.yaml` es la versión compacta: los 4 datos en una sola página.

```sh
esphome run esphome/c3_sensors_best_pages.yaml --device COMx
esphome run esphome/c3_sensors_best_pages.yaml --device sensors-best-pages.local
```

---

### Proyectos 5 y 6: estación + VWCE

Añaden una **5ª página** con el precio del ETF VWCE:

| YAML | Quién consulta VWCE |
|---|---|
| `c3_sensors_best_pages_vwce.yaml` (Proyecto 5) | El ESP32 directo a Yahoo Finance |
| `c3_sensors_best_pages_vwce_dummy.yaml` (Proyecto 6) | Home Assistant (sensor REST) |

```sh
# Proyecto 5
esphome run esphome/c3_sensors_best_pages_vwce.yaml --device COMx
# Proyecto 6
esphome run esphome/c3_sensors_best_pages_vwce_dummy.yaml --device COMx
```

---

### Proyecto 7: estación + VWCE + encoder KY-040 (`c3_sensors_best_pages_vwce_dummy_encoder.yaml`)

Igual que Proyecto 6 pero la navegación combina **rotación automática inteligente** con **control manual** mediante encoder rotatorio KY-040.


#### Comportamiento del encoder

| Acción | Resultado |
|---|---|
| Girar el eje | Si el OLED está apagado, lo enciende. Si está encendido, cambia de página y pausa la auto-rotación 10 s |
| Click simple | Si el OLED está apagado, lo enciende. Si está encendido, vuelve a página 1 (CO₂) |
| Doble click (< 400 ms) + mantener 2 s | Apaga o enciende el OLED |
| Doble click (< 400 ms) | Toggle auto-rotación ON/OFF permanente — muestra estado 2 s en pantalla |
| Pulsación ≥ 10 s | Factory reset del SCD4x — muestra confirmación 3 s en pantalla |
| Sin tocar 10 s | La auto-rotación se reanuda sola (si está activa) |

#### Páginas

| Índice | ID | Contenido |
|---|---|---|
| 0 | `page_co2` | CO₂ (ppm) |
| 1 | `page_temp` | Temperatura (°C) |
| 2 | `page_hum` | Humedad (%) |
| 3 | `page_press` | Presión (hPa) |
| 4 | `page_vwce` | VWCE (EUR) |
| — | `page_rotate_status` | Estado AUTO-ROTACIÓN ON/OFF (temporal, no entra en rotación) |
| — | `page_co2_reset` | Confirmación factory reset CO₂ (temporal, no entra en rotación) |

> Las páginas de estado nunca aparecen en la rotación automática ni al girar el encoder. Solo se muestran al disparar la acción correspondiente.


#### Apagado/encendido del OLED

Para apagar el OLED: haz doble clic y mantén pulsado el segundo clic durante 2 segundos.

Para encender el OLED: basta con girar el encoder o pulsar el botón (cualquier acción lo reactiva).

#### Factory reset del SCD4x

Mantener pulsado el botón del encoder durante **10 segundos** ejecuta `scd4x.factory_reset`: borra la calibración acumulada y reinicia la Auto Self-Calibration (ASC) desde cero. Útil al cambiar de ubicación o si las lecturas se han desviado.

Secuencia tras soltar el botón:
1. Se ejecuta `factory_reset` en el SCD4x
2. Aparece la pantalla `CO2 RESET` con una cuenta atrás **3 → 2 → 1** en la esquina superior derecha
3. Al llegar a 0 el ESP32 se reinicia solo — el SCD4x arranca limpio en modo medición continua

La ASC tarda ~7 días en estabilizarse de nuevo.

| | Proyecto 6 | Proyecto 7 |
|---|---|---|
| Rotación | Automática cada 6 s | Automática + pausa al girar |
| Hardware extra | Ninguno | KY-040 (3 pines GPIO) |
| Reset calibración CO₂ | — | Pulsación 10 s |

Componentes ESPHome: `rotary_encoder` (giro CW/CCW navega 0–4 con índice manual) + `binary_sensor` GPIO (pulsador activo-bajo, anti-rebote 10 ms, lógica en `on_release`).

```sh
esphome run esphome/c3_sensors_best_pages_vwce_dummy_encoder.yaml --device COMx
esphome run esphome/c3_sensors_best_pages_vwce_dummy_encoder.yaml --device sensors-encoder.local
```
