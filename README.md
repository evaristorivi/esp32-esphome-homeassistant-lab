# esphome-homeassistant-lab

<p align="center">
  <img src="images/portada.png" alt="Portada del proyecto" style="width:90%;">
</p>

Ejemplos prácticos para integrar un **ESP32-C3** con **ESPHome** y **Home Assistant**, mostrando cómo obtener y visualizar datos ambientales (temperatura, humedad, presión, CO₂) y financieros (precio del ETF VWCE en XETRA) en una pantalla OLED.

---

## Lista de piezas

A continuación se muestra una lista de módulos y componentes compatibles, con enlaces de ejemplo para su compra:

- [Placa de desarrollo ESP32-C3 Super Mini, 16 pines tipo C](https://es.aliexpress.com/item/1005009014269977.html)
- [Pantalla OLED 0,96" I2C 128x64 SSD1306](https://es.aliexpress.com/item/1005006365881525.html)
- [Sensor SCD40/SCD41 CO₂, temperatura y humedad I2C](https://es.aliexpress.com/item/1005009897956849.html)
- [AHT20 + BMP280 — temperatura, humedad y presión del aire](https://es.aliexpress.com/item/1005005321276932.html)
- [Módulo hub I2C (expansión de bus)](https://es.aliexpress.com/item/1005002811407142.html)
- [Placa de pruebas sin soldadura, 400 puntos](https://es.aliexpress.com/item/1005010426826947.html)

> **Nota sobre la protoboard y el hub I2C:** La protoboard es ideal para montar y probar el circuito rápidamente al principio. Sin embargo, si la idea es dejarlo instalado de forma permanente —por ejemplo imprimiendo una carcasa en 3D— la protoboard no es la solución más compacta ni limpia. El hub I2C entra en juego aquí: permite centralizar todas las conexiones I2C en un módulo pequeño y ordenado, facilitando el montaje final dentro de una caja. Para un acabado definitivo se recomienda sustituir la protoboard por soldadura directa o una PCB pequeña.
- [TP4056 — módulo cargador de batería LiPo con protección](https://es.aliexpress.com/item/1005009887303870.html)

> **Nota sobre la batería:** El módulo TP4056 aparece en la lista porque inicialmente se planteó alimentar el proyecto con una batería LiPo de 3,7V (tipo dron). Funciona, pero no es la solución óptima: una LiPo recién cargada entrega ~4,2V (aceptable), pero conforme se descarga el voltaje baja progresivamente. Al alimentar el ESP32 directamente por el pin 3V3, a partir de cierto nivel de descarga la tensión es insuficiente para un funcionamiento fiable. La solución correcta sería interponer un *boost converter* que mantenga una salida estable de 3,3V independientemente del nivel de la batería. Finalmente se optó por simplificar y construir el proyecto **sin batería**, alimentado directamente por USB. La lista se mantiene por si alguien quiere explorar esa vía.

> Puedes usar módulos equivalentes compatibles con ESP32 y ESPHome.

---

## Esquema de conexión

Todos los módulos se comunican mediante el bus **I2C**, que solo requiere 4 cables:

<p align="center">
  <img src="images/esquema.webp" alt="Pinout ESP32-C3 Super Mini" style="width:80%;">
</p>

| Pin módulo | Pin ESP32-C3 Super Mini |
|-----------|------------------------|
| **VCC** | **3V3** |
| **GND** | **GND** |
| **SDA** | **GPIO4** |
| **SCL** | **GPIO5** |

> Si usas el hub I2C, conecta **un único par SDA/SCL** desde el ESP32 al hub y desde ahí reparte al resto de módulos. VCC y GND se pueden llevar en paralelo directamente desde el ESP32.

> Los pines GPIO4 (SDA) y GPIO5 (SCL) están definidos en `esphome/vwce.yaml`, `esphome/vwce_dummy.yaml` y `esphome/sensors_lab.yaml`. Si cambias los pines físicos, actualízalos también en el YAML correspondiente.

---

## Estructura del proyecto

```
esphome/
  vwce.yaml             # ESP32 obtiene el precio de VWCE directamente de Yahoo Finance
  vwce_dummy.yaml       # ESP32 recibe el precio desde un sensor REST de Home Assistant
  sensors_lab.yaml      # Laboratorio completo: SCD40 + AHT20 + BMP280 + OLED (todos los datos)
  sensors_best.yaml     # Estación óptima: un sensor por magnitud, pantalla de una sola página
  secrets.yaml.example  # Plantilla de credenciales (no subas tus claves reales)
homeassistant/
  configuration.yaml    # Configuración principal mínima de Home Assistant
  vwce_sensor.yaml      # Sensor REST de VWCE listo para incluir
images/
  portada.png           # Imagen de portada
  esquema.webp          # Pinout del ESP32-C3 Super Mini
  1.png                 # Ejemplo de uso en pantalla
```

### ¿Cuál YAML usar?

| | `vwce.yaml` | `vwce_dummy.yaml` | `sensors_lab.yaml` | `sensors_best.yaml` |
|---|---|---|---|---|
| **Función principal** | Precio VWCE vía Yahoo Finance | Precio VWCE desde Home Assistant | Datos ambientales (todos los sensores) | Calidad del aire (sensor óptimo por magnitud) |
| **Quién obtiene el dato** | El propio ESP32 (HTTP) | Home Assistant (sensor REST) | Los sensores físicos conectados | Los sensores físicos conectados |
| **Requiere Home Assistant activo** | No (autónomo) | Sí | No (también funciona solo) | No (también funciona solo) |
| **Sensores hardware** | Solo OLED | Solo OLED | SCD40 + AHT20 + BMP280 + OLED | SCD40 + AHT20 + BMP280 + OLED |
| **Entidades en HA** | 1 (precio) | 1 (precio) | 7 (todas) | 4 (una por magnitud) |
| **Pantalla OLED** | 1 página (precio) | 1 página (precio) | 2 páginas rotatorias | 1 página (todo junto) |
| **Ideal cuando...** | Quieres un ticker independiente | Ya tienes HA con el dato | Quieres explorar todos los datos | Quieres el dato correcto de cada sensor |

Cada archivo contiene comentarios explicativos sobre su propósito y uso.

---

## Requisitos

### Drivers USB-Serie (primera vez por USB)

Para que el ordenador reconozca el ESP32 al conectarlo por USB necesitas el driver del chip USB-serie que lleva la placa. El ESP32-C3 Super Mini puede venir con chips distintos según el fabricante:

| Chip | Driver | Plataformas |
|------|--------|-------------|
| **CP2102** | [Silicon Labs VCP Drivers](https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers?tab=overview) | Windows, Mac |
| **CH340 / CH341** | [WCH CH341SER](https://www.wch.cn/downloads/CH341SER_ZIP.html) | Windows, Mac |
| **CH342 / CH343 / CH9102** | [WCH CH343SER](https://www.wch.cn/downloads/CH343SER_ZIP.html) | Windows, Mac |

> Si al conectar el ESP32 no aparece ningún puerto COM (Windows) o `/dev/tty.usbserial-*` (Mac/Linux), instala ambos drivers — no hay forma rápida de saber qué chip lleva la placa sin abrirla o probarlo.

### ESPHome

- **Python 3.8+** y **pip** instalados en tu sistema.
- Instala ESPHome con:
  ```sh
  pip install esphome
  ```
- Documentación oficial: [https://esphome.io/guides/installing_esphome.html](https://esphome.io/guides/installing_esphome.html)

### Home Assistant

- Puede instalarse en Raspberry Pi, PC, NAS, máquina virtual, etc.
- Documentación oficial: [https://www.home-assistant.io/installation/](https://www.home-assistant.io/installation/)

#### Ejemplo de instalación en Raspberry Pi 4 con Docker

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

## Puesta en marcha

### 1. Configura las credenciales

Copia `esphome/secrets.yaml.example` a `esphome/secrets.yaml` y rellena tus datos:

```yaml
wifi_ssid: "TuRedWiFi"
wifi_password: "TuContraseña"
ota_password: "una_clave_ota"
api_encryption_key: "clave_base64_de_32_bytes"
```

### 2. (Opcional) Configura Home Assistant

Si usas `vwce_dummy.yaml`, incluye el sensor REST en tu `configuration.yaml`:

```yaml
sensor: !include homeassistant/vwce_sensor.yaml
```

### 3. Compila y flashea el ESP32

**Primera vez — por USB:**

Pon el ESP32 en modo flasheo (mantén pulsado BOOT al conectar el USB, luego suelta RESET) y ejecuta:

```sh
esphome run esphome/vwce.yaml --device COMx
```

Reemplaza `COMx` por el puerto serie de tu ESP32 (por ejemplo `COM3` en Windows o `/dev/ttyUSB0` en Linux).

**Siguientes veces — por OTA/WiFi:**

Una vez flasheado y conectado a la red, las actualizaciones se hacen por WiFi:

```sh
esphome run esphome/vwce.yaml --device vwce-ticker.local
```

Reemplaza `vwce-ticker.local` por el nombre mDNS o la IP de tu ESP32 (por ejemplo `192.168.1.106`).

> La primera vez siempre debe ser por USB. A partir de ahí OTA permite actualizar sin tocar el cable.

**Para `sensors_lab.yaml`:**

```sh
# Primera vez por USB
esphome run esphome/sensors_lab.yaml --device COMx

# Siguientes veces por OTA (el nombre mDNS es el campo `name` del YAML)
esphome run esphome/sensors_lab.yaml --device sensors-lab.local
```

**Para `sensors_best.yaml`:**

```sh
# Primera vez por USB
esphome run esphome/sensors_best.yaml --device COMx

# Siguientes veces por OTA
esphome run esphome/sensors_best.yaml --device sensors-best.local
```

---

## Entendiendo la configuración

Esta sección explica las decisiones técnicas detrás de los archivos YAML de ESPHome.

### Secretos (`!secret`)

Los valores marcados con `!secret` se leen de `esphome/secrets.yaml`, que **no se sube al repositorio** (está en `.gitignore`). Copia `secrets.yaml.example` a `secrets.yaml` y rellena tus datos reales. Así puedes compartir o publicar los YAML sin exponer contraseñas ni claves.

```yaml
# secrets.yaml (no subir al repo)
wifi_ssid: "TuRedWiFi"
wifi_password: "TuContraseña"
ota_password: "una_clave_segura"
api_encryption_key: "clave_base64_de_32_bytes"
```

### Framework: `esp-idf`

```yaml
esp32:
  framework:
    type: esp-idf
```

ESP-IDF (*Espressif IoT Development Framework*) es el framework oficial de **Espressif**, la empresa creadora del ESP32. Durante años ESPHome usó Arduino como base porque era lo más compatible, pero ESP-IDF es la evolución natural: más estable, mejor gestión de WiFi y memoria, y mantenido directamente por el fabricante del chip. Desde que Espressif adquirió mayor control del ecosistema IoT, ESP-IDF se ha convertido en el estándar recomendado para producción.

### OTA — actualización por WiFi

```yaml
ota:
  platform: esphome
  password: !secret ota_password
```

OTA (*Over The Air*) permite flashear el firmware del ESP32 por WiFi sin conectar el cable USB. La primera vez es obligatorio hacerlo por USB; a partir de ahí todas las actualizaciones son en remoto. La contraseña impide que cualquier dispositivo en la misma red pueda sobrescribir el firmware.

### API — integración con Home Assistant

```yaml
api:
  encryption:
    key: !secret api_encryption_key
```

Activa el protocolo nativo de ESPHome para que Home Assistant descubra el dispositivo automáticamente. La `encryption.key` cifra toda la comunicación entre el ESP32 y Home Assistant con AES-128. Las versiones antiguas de ESPHome usaban `password:` en texto plano; `encryption` es la forma actual y recomendada.

Para generar una clave válida:
```sh
openssl rand -base64 32
```

### Logger

```yaml
logger:
  level: DEBUG
```

Envía mensajes de diagnóstico por el puerto serie (visible al conectar por USB) y también desde la UI web de ESPHome. `DEBUG` muestra todo; en un dispositivo ya estable se puede bajar a `INFO` o `WARNING` para reducir ruido.

### WiFi

Con ESP-IDF el único ajuste necesario respecto a los defaults es `power_save_mode: none`:

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  power_save_mode: none  # El default en esp-idf es 'light' (ahorro de energía WiFi activado).
                          # Con 'light' el chip duerme entre paquetes, lo que puede causar
                          # latencia, reconexiones ocasionales o que HA marque el dispositivo
                          # como 'unavailable' momentáneamente. Para un dispositivo enchufado,
                          # 'none' da mayor estabilidad sin ningún coste.
  # Opciones útiles si tienes problemas de conexión:
  # output_power: 8.5dB    # Reduce potencia si el ESP32 está muy cerca del router
  # fast_connect: false    # Si true, salta el escaneo de canales (más rápido pero menos fiable)
  # reboot_timeout: 15min  # Reiniciar si no conecta en X tiempo (default: 15min)
```

Las tres opciones comentadas son workarounds para problemas de conectividad y no son necesarias en condiciones normales. El resto de opciones WiFi (`fast_connect`, `reboot_timeout`) coinciden con los valores por defecto de ESPHome y no es necesario especificarlos.

### HTTP request

```yaml
http_request:
  useragent: "Mozilla/5.0"
  timeout: 10s
```

`useragent: "Mozilla/5.0"` hace que el ESP32 se identifique como un navegador al conectarse a Yahoo Finance. Sin esto, Yahoo detecta la petición como un bot y responde con error o datos vacíos. Es un workaround habitual para APIs públicas sin acceso oficial.

`verify_ssl` no se especifica porque el default es `true` — la conexión HTTPS verifica el certificado de Yahoo correctamente con versiones recientes de ESPHome y esp-idf.

### Bus I2C

```yaml
i2c:
  sda: GPIO4
  scl: GPIO5
  scan: true
  frequency: 100kHz
```

I2C es un protocolo de comunicación serie que usa solo 2 cables de datos (SDA y SCL) y permite conectar varios módulos en paralelo en el mismo bus.

`scan: true` actúa una sola vez al arrancar: escanea el bus e imprime en el logger las direcciones de todos los módulos detectados. No tiene ningún coste durante el funcionamiento normal y es muy útil para verificar el cableado si algo falla.

La frecuencia se fija en **100kHz** porque en un bus I2C compartido la velocidad la marca el módulo más lento. El sensor **SCD40/SCD41** especifica en su datasheet un máximo de 100kHz, por lo que aunque otros módulos como el SSD1306 o el BMP280 soporten 400kHz, todo el bus debe operar al límite del más restrictivo.

### Sensores

**`vwce.yaml` — sensor de precio (template):**

```yaml
sensor:
  - platform: template
    name: "VWCE Precio"
    id: vwce_price
    unit_of_measurement: "EUR"
    update_interval: never
```

- `platform: template` — sensor sin hardware detrás. Es un contenedor vacío cuyo valor se publica manualmente desde el código C++ del `interval` con `id(vwce_price).publish_state(price)`.
- `name` — nombre con el que aparece en Home Assistant como entidad (`sensor.vwce_precio`). Al tener nombre y no tener `internal: true`, HA lo recibe automáticamente, guarda el histórico y permite hacer gráficas.
- `id` — nombre interno para referenciarlo desde el código: `id(vwce_price).state` (leer) y `id(vwce_price).publish_state(price)` (escribir).
- `unit_of_measurement: "EUR"` — unidad que ve HA, permite que trate el sensor como valor monetario.
- `update_interval: never` — el sensor no se actualiza en ningún ciclo automático; solo cambia cuando el código llama explícitamente a `publish_state()`.

**`internal: true`** — propiedad disponible en cualquier sensor. Cuando está activo, el sensor funciona internamente en el ESP32 pero **no se expone a Home Assistant**: HA no crea ninguna entidad, no guarda histórico ni consume espacio en su base de datos. Se usa en `wifi_rssi` (solo necesario para dibujar las barras en la pantalla) y en `vwce_price` de `vwce_dummy.yaml` (HA ya tiene su propio sensor REST con ese dato; reexponérselo desde el ESP32 sería redundante).

### LED integrado

El **ESP32-C3 Super Mini** tiene un LED azul simple en **GPIO8**. No es RGB — solo encendido/apagado.

El pin es **invertido**: `LOW` = encendido, `HIGH` = apagado. De ahí el `inverted: true` en la configuración.

Ambos YAML incluyen el bloque siguiente **comentado**:

```yaml
# light:
#   - platform: status_led
#     name: "LED"
#     id: led_status
#     pin:
#       number: GPIO8
#       inverted: true
#     restore_mode: ALWAYS_OFF
```

Si lo descomentas y flasheas, aparece un botón en Home Assistant para encender y apagar el LED manualmente. Es útil para verificar que el pin funciona y entender cómo ESPHome expone controles a HA — pero para uso normal no hace falta tenerlo activo.

> **Nota:** el ESP32-C3 *Mini* (distinto al *Super Mini*) sí tiene un LED RGB (WS2812) también en GPIO8 pero con lógica diferente. Para ese board habría que usar `esp32_rmt_led_strip` en lugar de `status_led`.

---

## Variante: laboratorio de sensores (`sensors_lab.yaml`)

Esta variante prescinde del precio de VWCE y convierte el ESP32 en una **estación de calidad del aire** con tres sensores físicos y una pantalla OLED con dos páginas rotatorias.

### Sensores conectados

| Sensor | Plataforma ESPHome | Dirección I2C | Datos expuestos |
|---|---|---|---|
| **SCD40 / SCD41** | `scd4x` | `0x62` (fija) | CO₂ (ppm), temperatura, humedad |
| **AHT20** | `aht10` con `variant: AHT20` | `0x38` | Temperatura, humedad |
| **BMP280** | `bmp280_i2c` | `0x77`\* | Temperatura, presión (hPa) |
| **SSD1306** | `ssd1306_i2c` | `0x3C` | Pantalla 128×64 |

> \* El módulo combo AHT20+BMP280 habitual en AliExpress lleva SDO conectado a VCC → dirección `0x77`. Si tienes un BMP280 suelto con SDO libre (a GND), la dirección será `0x76`. Usa `scan: true` en el bloque `i2c:` para confirmar qué dirección responde en tu bus.

### Entidades en Home Assistant

Todas se crean automáticamente al añadir el dispositivo en HA (sin YAML adicional en Home Assistant):

| Entidad | Unidad | Fuente |
|---|---|---|
| `sensor.sensors_lab_co2` | ppm | SCD40 |
| `sensor.sensors_lab_scd4x_temperatura` | °C | SCD40 |
| `sensor.sensors_lab_scd4x_humedad` | % | SCD40 |
| `sensor.sensors_lab_aht20_temperatura` | °C | AHT20 |
| `sensor.sensors_lab_aht20_humedad` | % | AHT20 |
| `sensor.sensors_lab_bmp280_temperatura` | °C | BMP280 |
| `sensor.sensors_lab_bmp280_presion` | hPa | BMP280 |

### Pantalla OLED: dos páginas rotatorias

La pantalla alterna cada **6 segundos** entre dos páginas. Los sensores se leen cada **30 segundos**; la pantalla se redibuja cada segundo con el último valor disponible.

**Página 1 — CO₂:**
- Cabecera: `CO2` / `SCD40` + barras de señal WiFi
- Valor de CO₂ en número grande centrado (font 24 px)
- Unidad `ppm` debajo
- Temperatura y humedad del SCD40 en el pie (font 9 px)

**Página 2 — Clima:**
- Cabecera: `CLIMA` / `AHT+BMP` + barras de señal WiFi
- Temperatura AHT20
- Humedad AHT20
- Presión BMP280

### Niveles de CO₂ y calidad del aire

El SCD40 mide el CO₂ real por absorción infrarroja no dispersiva (NDIR) — no es una estimación como los sensores VOC baratos.

| ppm | Estado |
|---|---|
| < 400 | Aire exterior limpio |
| 400–700 | Excelente |
| 700–1000 | Aceptable |
| 1000–1500 | Malo — somnolencia, dificultad de concentración |
| 1500–2000 | Muy malo — dolores de cabeza, cansancio |
| > 2000 | Peligroso |

> Un espacio interior con personas y ventanas cerradas sube fácilmente a 1000–1400 ppm. Abrir una ventana 10 minutos baja el nivel rápidamente.

### Auto-calibración del SCD40

El SCD40 incluye **ASC** (*Automatic Self-Calibration*): asume que el valor mínimo registrado en los últimos **7 días** corresponde a aire fresco (~400 ppm) y se recalibra en consecuencia. Está activo por defecto (`Automatic self calibration: ON` en el log de arranque).

Si el sensor nunca se expone a aire exterior o en un local con ventilación deficiente, puede acabar descalibrándose hacia abajo. Para desactivarlo:

```yaml
- platform: scd4x
  automatic_self_calibration: false
```

### Actualización de datos

| Componente | Frecuencia |
|---|---|
| Lectura de sensores | cada 30 s |
| Redibujado de pantalla | cada 1 s |
| Rotación de página | cada 6 s |
| SCD40 internamente | cada 5 s (modo periódico) |

El SCD40 mide cada 5 segundos internamente, pero ESPHome solo recoge y publica ese valor cada 30 s. Si se quiere mayor frecuencia de actualización basta con cambiar `update_interval: 5s` en el bloque `scd4x`.

### Nota sobre `scan: true`

El bus I2C arranca con un escaneo que imprime en el log todas las direcciones que responden. Es muy útil para diagnosticar problemas de cableado o confirmar la dirección del BMP280:

```
[C][i2c.idf:119]: Found device at address 0x38   ← AHT20
[C][i2c.idf:119]: Found device at address 0x3C   ← SSD1306
[C][i2c.idf:119]: Found device at address 0x62   ← SCD40
[C][i2c.idf:119]: Found device at address 0x77   ← BMP280
```

---

## Comparativa de sensores y elección óptima

El SCD40, el AHT20 y el BMP280 miden algunas magnitudes en común (temperatura y humedad). No todos tienen la misma precisión para cada dato.

### Temperatura

| Sensor | Precisión datasheet | Observación |
|---|---|---|
| **AHT20** | ±0.3 °C | **Mejor opción** — sin self-heating |
| BMP280 | ±0.5 °C | Aceptable, pero AHT20 gana |
| SCD40 | ±0.8 °C | Se descarta — sufre *self-heating* |

El SCD40 incorpora una cámara de medición de CO₂ que genera calor. Aplica un offset de compensación interno (visible en el log como `Temperature offset: 4.00°C`), pero en un entorno cerrado sin airflow sigue leyendo **1.5–2 °C por encima** de la temperatura real. Es un problema conocido y documentado por Sensirion.

### Humedad

| Sensor | Precisión datasheet | Observación |
|---|---|---|
| **AHT20** | ±2 % RH | **Mejor opción** |
| SCD40 | ±6 % RH | Error triple; además afectado por el self-heating de temperatura |

### Presión y CO₂

Sin competencia: el BMP280 es el único con barómetro y el SCD40 el único con sensor NDIR de CO₂.

### Resumen

| Magnitud | Sensor óptimo | Motivo |
|---|---|---|
| CO₂ | SCD40 | Único sensor NDIR del bus |
| Temperatura | AHT20 | ±0.3°C, sin self-heating |
| Humedad | AHT20 | ±2% RH |
| Presión | BMP280 | Único barómetro del bus |

Esta es exactamente la elección que implementa `sensors_best.yaml`.

---

## Variante óptima: estación de calidad del aire (`sensors_best.yaml`)

Versión producción de `sensors_lab.yaml`: usa los mismos tres sensores físicos pero expone a Home Assistant **solo el dato más preciso para cada magnitud**. Las mediciones redundantes o menos fiables se marcan `internal: true` — siguen calculándose internamente (visibles en el log) pero no crean entidades en HA ni consumen espacio en su base de datos.

### Entidades en Home Assistant (solo 4)

| Entidad | Unidad | Sensor | Motivo |
|---|---|---|---|
| `sensor.sensors_best_co2` | ppm | SCD40 | Único con NDIR |
| `sensor.sensors_best_temperatura` | °C | AHT20 | ±0.3°C, sin self-heating |
| `sensor.sensors_best_humedad` | % | AHT20 | ±2% RH |
| `sensor.sensors_best_presion` | hPa | BMP280 | Único barómetro |

### Pantalla OLED: una sola página

Todo cabe en 128×64 px sin necesidad de rotar páginas:

```
CO2                    SCD40  [wifi]
────────────────────────────────────
             1295
T: 24.0 C
H: 55.9 %
P: 1028.6 hPa
```

Layout exacto (coordenada Y de cada elemento):

| y | Contenido | Fuente |
|---|---|---|
| 0 | Cabecera + barras WiFi | font_tiny (9 px) |
| 12 | Línea separadora | — |
| 13 | Valor CO₂ centrado | font_big (24 px) |
| 38 | Temperatura AHT20 | font_tiny (9 px) |
| 47 | Humedad AHT20 | font_tiny (9 px) |
| 55 | Presión BMP280 | font_tiny (9 px) |

---

<p align="center">
  <img src="images/1.png" alt="Ejemplo de uso en pantalla" style="width:50%;">
</p>

---

**¡Contribuciones y mejoras son bienvenidas!**
