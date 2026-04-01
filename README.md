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

> Los pines GPIO4 (SDA) y GPIO5 (SCL) están definidos en `esphome/vwce.yaml` y `esphome/vwce_dummy.yaml`. Si cambias los pines físicos, actualízalos también en el YAML.

---

## Estructura del proyecto

```
esphome/
  vwce.yaml             # ESP32 obtiene el precio de VWCE directamente de Yahoo Finance
  vwce_dummy.yaml       # ESP32 recibe el precio desde un sensor REST de Home Assistant
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

| | `vwce.yaml` | `vwce_dummy.yaml` |
|---|---|---|
| **Quién obtiene el precio** | El propio ESP32 (petición HTTP a Yahoo) | Home Assistant (sensor REST) |
| **Requiere Home Assistant activo** | No (funciona autónomo) | Sí |
| **Ideal cuando...** | Quieres un dispositivo independiente | Ya tienes HA y quieres compartir el dato con varios ESP32 |

Cada archivo contiene comentarios explicativos sobre su propósito y uso.

---

## Requisitos

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

### HTTP request (`verify_ssl`)

```yaml
http_request:
  verify_ssl: false
```

`verify_ssl: false` desactiva la verificación del certificado SSL al conectarse a Yahoo Finance. Esto significa que la conexión HTTPS se cifra pero no se autentica — el ESP32 no comprueba que realmente está hablando con Yahoo. El riesgo real es bajo en una red doméstica para este uso concreto (leer un precio público de bolsa), pero la opción correcta sería `true`. Con versiones recientes de ESPHome y esp-idf funciona bien si quieres activarlo.

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

---

<p align="center">
  <img src="images/1.png" alt="Ejemplo de uso en pantalla" style="width:50%;">
</p>

---

**¡Contribuciones y mejoras son bienvenidas!**
