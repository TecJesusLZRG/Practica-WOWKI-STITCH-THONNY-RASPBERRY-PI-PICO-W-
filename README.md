# Raspberry Pi Pico W · Servidor Web Embebido con MicroPython (ADC4 en Tiempo Real)

## Descripción Ejecutiva
Este proyecto implementa un **servidor web embebido** sobre una **Raspberry Pi Pico W** usando **MicroPython**. El firmware lee el valor del sensor interno de temperatura mediante el canal **ADC4** (`read_u16`) y lo expone en tiempo real a un dashboard web responsivo.

La solución integra:
- **Backend embebido:** MicroPython + sockets TCP/HTTP básicos.
- **Frontend web:** HTML + JavaScript + Tailwind CSS + Chart.js.
- **Telemetría en tiempo real:** consulta periódica del endpoint `GET /adc` cada 1 segundo.

> Alcance actual: monitoreo local en LAN, con una arquitectura ligera y didáctica para IoT/Edge.

---

## 1) Estructura de Repositorio Recomendada
A continuación se muestra una estructura estándar de industria para este tipo de proyecto, incluyendo la ubicación para evidencias visuales (capturas/fotos de pruebas):

```text
pico-w-adc-dashboard/
├─ README.md
├─ LICENSE
├─ .gitignore
├─ firmware/
│  ├─ main.py                 # Script principal MicroPython desplegado en la Pico W
│  ├─ boot.py                 # (Opcional) Inicialización de red/board
│  └─ config.example.py       # Plantilla de configuración (sin credenciales reales)
├─ docs/
│  ├─ architecture/
│  │  └─ system-diagram.md    # Diagrama lógico y flujo de datos
│  ├─ api/
│  │  └─ adc-endpoint.md       # Especificación detallada de /adc
│  ├─ testing/
│  │  ├─ stress-test-report.md # Resultados de pruebas de estrés
│  │  └─ edge-cases.md         # Casos extremos y comportamiento esperado
│  └─ images/
│     ├─ dashboard-desktop.png
│     ├─ dashboard-mobile.png
│     ├─ thonny-console.png
│     └─ hardware-setup.jpg
└─ tools/
   └─ requests_benchmark.py    # (Opcional) Script auxiliar para saturación de peticiones
```

> Si quieres mantener un repositorio mínimo, puedes comenzar solo con `README.md` y `firmware/main.py`, e ir agregando `docs/` conforme evolucione el proyecto.

---

## 2) Requisitos de Hardware y Software

### Hardware
- 1x **Raspberry Pi Pico W**.
- 1x Cable USB de datos (USB-A/USB-C a micro-USB, según host).
- 1x PC/Laptop (Windows, Linux o macOS).
- Red WiFi 2.4 GHz disponible.
- (Opcional) Protoboard para montaje físico/documentación del laboratorio.

### Software
- **MicroPython firmware** para Raspberry Pi Pico W.
- **Thonny IDE** (recomendado para carga y ejecución).
- Navegador web moderno (Chrome, Edge, Firefox, Brave).

### Librerías/Dependencias en MicroPython
El código utiliza únicamente módulos estándar del runtime MicroPython:
- `network`
- `socket`
- `time`
- `machine` (`ADC`)
- `json`

### Dependencias Frontend (CDN)
Cargadas dinámicamente en el navegador:
- Tailwind CSS (CDN)
- Google Fonts (Inter + Material Symbols)
- Chart.js (CDN)

---

## 3) Arquitectura del Sistema (Diagrama Lógico explicado)

### Vista de componentes
```text
[Usuario/Navegador]
        |
        | HTTP GET /
        v
[Pico W - Socket Server :80] -------------------------> Responde HTML/JS/CSS (dashboard)
        |
        | HTTP GET /adc (cada 1s desde JS)
        v
[Lectura ADC4: adc.read_u16()]
        |
        v
[Respuesta JSON]
        |
        v
[Frontend actualiza KPI, barra y Chart.js]
```

### Flujo detallado
1. La Pico W se conecta a la red WiFi en modo estación (`network.STA_IF`).
2. Levanta un socket TCP en el puerto 80 (`listen(5)`).
3. Cuando el cliente solicita `/`, el servidor responde el HTML completo del dashboard.
4. En el navegador, `setInterval(obtenerDatosReales, 1000)` dispara un `fetch('/adc')` cada segundo.
5. El endpoint `/adc` toma lectura `adc.read_u16()` en rango `0..65535`.
6. El backend responde JSON con metadatos del sensor y valor actual.
7. El frontend:
   - Actualiza el texto de lectura actual.
   - Ajusta la barra de progreso proporcional al valor ADC.
   - Inserta el dato al buffer de Chart.js para traza temporal.

### Consideraciones de diseño
- Arquitectura **monolítica embebida** (UI + API servida desde el mismo dispositivo).
- Comunicación **polling HTTP** simple (suficiente para telemetría básica).
- Baja complejidad operativa, ideal para cursos/prototipos.

---

## 4) Guía de Instalación y Despliegue

## Paso 0 — Recomendación de seguridad
Antes de subir el proyecto a GitHub:
- **No publiques SSID/contraseñas reales** dentro de `main.py`.
- Usa placeholders o archivo de configuración local no versionado.

Ejemplo recomendado:
```python
ssid = 'TU_SSID'
password = 'TU_PASSWORD'
```

## Paso 1 — Flashear MicroPython en la Pico W
1. Descarga el firmware UF2 oficial para **Raspberry Pi Pico W**.
2. Conecta la Pico W presionando **BOOTSEL** para entrar en modo almacenamiento USB.
3. Copia el archivo `.uf2` a la unidad montada.
4. Espera el reinicio automático de la placa.

## Paso 2 — Configurar Thonny
1. Abrir Thonny.
2. Ir a **Run > Select interpreter**.
3. Seleccionar **MicroPython (Raspberry Pi Pico)**.
4. Verificar que el puerto serie detectado sea correcto.

## Paso 3 — Cargar el script
1. Crear/abrir `main.py` en Thonny.
2. Pegar el código fuente del servidor.
3. Ajustar `ssid` y `password` a la red objetivo.
4. Guardar como `main.py` **en la Raspberry Pi Pico W** (no solo en PC).

## Paso 4 — Ejecutar y validar en consola
1. Ejecutar el script (botón Run en Thonny).
2. Confirmar mensajes similares a:
   - `Conectando a WiFi...`
   - `Conectado! IP: <IP_LOCAL>`
   - `Servidor web encendido. Esperando peticiones...`

## Paso 5 — Acceder al dashboard
1. Desde un dispositivo en la misma LAN, abrir:
   - `http://<IP_LOCAL_DE_LA_PICO>/`
2. Validar:
   - Estado “En Línea”.
   - Lectura ADC actualizada.
   - Gráfica en tiempo real.

---

## 5) Documentación de API

### Endpoint: `/adc`
- **Método:** `GET`
- **Content-Type de respuesta:** `application/json`
- **Autenticación:** No requerida (entorno LAN de laboratorio)

### Propósito
Entregar la lectura instantánea del canal `ADC4` (sensor de temperatura interno) para actualizar el dashboard en tiempo real.

### Ejemplo de solicitud
```http
GET /adc HTTP/1.1
Host: <ip-pico>
```

### Ejemplo de respuesta exitosa (200)
```json
{
  "sensor": "Temp Interna",
  "value": 31245,
  "pin": "ADC4"
}
```

### Campos
- `sensor` (`string`): etiqueta descriptiva del origen de medición.
- `value` (`integer`): lectura cruda ADC de 16 bits (`0` a `65535`).
- `pin` (`string`): canal lógico de captura (`ADC4`).

### Semántica operacional
Cada invocación de `/adc` ejecuta una lectura nueva `adc.read_u16()`, por lo que la respuesta refleja una muestra instantánea.

### Nota sobre rutas no válidas
Según la implementación actual, cualquier ruta distinta de `/adc` responde con el dashboard HTML y estado HTTP 200.

---

## 6) Pruebas de Estrés y Casos Extremo (Edge Cases)
El sistema se sometió a pruebas funcionales y de resiliencia básica:

1. **Saturación de peticiones (stress/polling intensivo):**
   - Se evaluó comportamiento bajo múltiples solicitudes sucesivas al endpoint `/adc`.
   - Se observó continuidad de servicio dentro de los límites de hardware de la Pico W.

2. **Desconexión abrupta de cliente:**
   - Se validó el cierre de conexión ante errores de socket para evitar bloqueo del bucle principal.

3. **Rutas no válidas:**
   - Se verificó respuesta controlada devolviendo la interfaz HTML en vez de romper la sesión.

> Recomendación futura: formalizar pruebas con script automatizado (latencia, throughput y tasa de error), además de incluir códigos HTTP diferenciados (`404`, `500`) para una API más robusta.

---

## Evidencias sugeridas para GitHub
Ubica las capturas en `docs/images/` y enlázalas en esta sección:

- `docs/images/dashboard-mobile.png`
- `docs/images/dashboard-desktop.png`
- `docs/images/thonny-console.png`
- `docs/images/hardware-setup.jpg`

Ejemplo de inserción en Markdown:
```md
![Dashboard móvil](docs/images/dashboard-mobile.png)
```

---

## Roadmap técnico recomendado
- Externalizar configuración sensible (`config.py` + `.gitignore`).
- Implementar endpoint de salud (`/health`).
- Devolver `404` en rutas no válidas.
- Añadir conversión opcional ADC→°C calibrada.
- Migrar de polling a WebSocket/SSE para mayor eficiencia.
- Agregar pruebas automatizadas y reporte de rendimiento.

---

## Licencia
Define una licencia (MIT, Apache-2.0, etc.) según la política de tu institución/proyecto.
