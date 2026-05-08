---
name: amplitude-data-seeder
description: >
  Lee el tracking plan de Amplitude via MCP, genera eventos dummy con funnels realistas
  y los carga via Batch API. Ejecuta directo + genera script Python descargable.
  Activar con: "generar datos demo", "seed eventos amplitude", "poblar proyecto amplitude",
  "datos sintéticos amplitude", "amplitude data seeder", "cargar eventos de prueba",
  "generar demo data", o cualquier variante de querer poblar un proyecto Amplitude con
  datos sintéticos para demos, sandbox o desarrollo.
---

# Amplitude Data Seeder

Sos un especialista en Amplitude Analytics y generación de datos sintéticos.
Tu misión es poblar un proyecto de Amplitude con eventos realistas basados en
la taxonomía real del proyecto, usando el MCP de Amplitude para leerla y la
Batch API para cargar los datos.

## Flujo completo

Seguí estos pasos en orden. No saltees ninguno.

---

### PASO 1 — Recolectar información del usuario

Pedí los siguientes datos usando preguntas directas:

1. **App ID del proyecto** — el ID numérico del proyecto en Amplitude
2. **API Key** — la clave pública del proyecto (Browser SDK Key)
3. **API Secret** — la clave secreta (necesaria para Batch API)
4. **Cantidad de usuarios sintéticos** — recomendá entre 50 y 500
5. **Rango de fechas** — desde / hasta (por defecto: últimos 30 días)
6. **Densidad de eventos** — eventos por usuario por día (por defecto: 3–8)
7. **¿Hay algún funnel específico que quieras enfatizar?** — opcional

Si el usuario no provee el API Secret, explicá que es necesario para la Batch API
y que se encuentra en Amplitude → Settings → Projects → [nombre del proyecto] → API Keys.

---

### PASO 2 — Leer el tracking plan via MCP

Usá las herramientas del MCP de Amplitude para:

1. **Listar los eventos del proyecto** — obtené todos los eventos activos con sus propiedades
2. **Leer propiedades de usuario** — user properties disponibles en el proyecto
3. **Identificar funnels** — buscá secuencias lógicas de eventos que formen flujos
   (ej: si hay `view_product`, `add_to_cart`, `checkout_started`, `purchase_completed`
   → eso es un funnel de conversión)

Documentá en el hilo:
- Total de eventos detectados
- Funnels identificados (con la secuencia de eventos)
- User properties disponibles

---

### PASO 3 — Diseñar los datos sintéticos

Antes de generar, diseñá la estructura:

**Usuarios sintéticos:**
- Generá IDs únicos: `synthetic_user_001`, `synthetic_user_002`, etc.
- Asigná user properties coherentes: plan (free/premium), country, signup_date, etc.
- Distribuí los usuarios en cohortes de signup realistas (no todos el mismo día)

**Eventos por usuario:**
- Seguí la lógica del funnel — no puede haber `purchase_completed` sin `checkout_started`
- No todos los usuarios completan el funnel — simulá drop-offs realistas:
  - 100% llegan al primer paso
  - 60-70% al segundo
  - 40-50% al tercero
  - 20-30% al último (conversión)
- Distribuí los eventos en el rango de fechas indicado
- Variá los horarios (no todos a la misma hora)

**Propiedades de eventos:**
- Usá las propiedades reales del proyecto cuando existan
- Generá valores coherentes (ej: `revenue` entre 10 y 500, `item_count` entre 1 y 5)

---

### PASO 4 — Generar y cargar via Batch API

#### 4a — Generar el payload

Construí el payload en formato JSON para la Amplitude Batch API:

```json
{
  "api_key": "API_KEY_DEL_PROYECTO",
  "events": [
    {
      "user_id": "synthetic_user_001",
      "event_type": "nombre_del_evento",
      "time": 1700000000000,
      "event_properties": {
        "propiedad_1": "valor_1"
      },
      "user_properties": {
        "plan": "free",
        "country": "Argentina"
      }
    }
  ]
}
```

La Batch API acepta hasta 2000 eventos por request. Si generás más, dividí en batches.

#### 4b — Cargar via bash

Ejecutá el upload usando curl o Python:

```bash
curl -X POST https://api2.amplitude.com/batch \
  -H "Content-Type: application/json" \
  -d @/tmp/amplitude_seed_batch_1.json
```

Verificá que la respuesta sea `{"code": 200}`. Si hay errores, mostralos al usuario.

#### 4c — Confirmar en Amplitude

Indicale al usuario que puede verificar los datos en:
- Amplitude → User Look-Up → buscar `synthetic_user_001`
- Amplitude → Event Segmentation → el evento principal → últimas 24h

---

### PASO 5 — Generar script Python descargable

Creá un archivo `amplitude_data_seeder.py` con el código completo para repetir
el proceso sin necesitar a Claude:

```python
#!/usr/bin/env python3
"""
Amplitude Data Seeder
Generado por Claude — /amplitude-data-seeder skill
Proyecto: [NOMBRE_PROYECTO] (App ID: [APP_ID])
Generado el: [FECHA]
"""

import json
import random
import time
import requests
from datetime import datetime, timedelta

# ── CONFIGURACIÓN ──────────────────────────────────────────────────
API_KEY = "[API_KEY]"
API_SECRET = "[API_SECRET]"  # No compartir — mantener privado
APP_ID = "[APP_ID]"

NUM_USERS = [CANTIDAD]
DATE_START = "[FECHA_INICIO]"  # YYYY-MM-DD
DATE_END = "[FECHA_FIN]"       # YYYY-MM-DD
EVENTS_PER_USER_PER_DAY = [DENSIDAD]  # min–max

# ── EVENTOS Y FUNNELS ──────────────────────────────────────────────
# [Acá van los eventos y funnels detectados del tracking plan]
FUNNEL = [
    "[evento_1]",
    "[evento_2]",
    "[evento_3]",
]

CONVERSION_RATES = [1.0, 0.65, 0.30]  # % que llega a cada paso

# ── GENERACIÓN ─────────────────────────────────────────────────────
def generate_users(n):
    users = []
    for i in range(1, n + 1):
        users.append({
            "user_id": f"synthetic_user_{i:03d}",
            "plan": random.choice(["free", "free", "premium"]),
            "country": random.choice(["Argentina", "Mexico", "Colombia", "Chile"]),
            "signup_date": (datetime.now() - timedelta(days=random.randint(1, 90))).strftime("%Y-%m-%d"),
        })
    return users

def generate_events(users, date_start, date_end):
    events = []
    start = datetime.strptime(date_start, "%Y-%m-%d")
    end = datetime.strptime(date_end, "%Y-%m-%d")
    days = (end - start).days

    for user in users:
        for day_offset in range(days):
            current_date = start + timedelta(days=day_offset)
            for step_idx, event_name in enumerate(FUNNEL):
                if random.random() > CONVERSION_RATES[min(step_idx, len(CONVERSION_RATES)-1)]:
                    break
                hour = random.randint(8, 22)
                minute = random.randint(0, 59)
                ts = int(current_date.replace(hour=hour, minute=minute).timestamp() * 1000)
                events.append({
                    "user_id": user["user_id"],
                    "event_type": event_name,
                    "time": ts,
                    "user_properties": {
                        "plan": user["plan"],
                        "country": user["country"],
                        "signup_date": user["signup_date"],
                    }
                })
    return events

def upload_batch(events, api_key, batch_size=1000):
    url = "https://api2.amplitude.com/batch"
    total = len(events)
    uploaded = 0
    for i in range(0, total, batch_size):
        batch = events[i:i+batch_size]
        payload = {"api_key": api_key, "events": batch}
        resp = requests.post(url, json=payload, timeout=30)
        if resp.status_code == 200:
            uploaded += len(batch)
            print(f"✓ Batch {i//batch_size + 1}: {len(batch)} eventos — total {uploaded}/{total}")
        else:
            print(f"✗ Error en batch {i//batch_size + 1}: {resp.status_code} — {resp.text}")
    return uploaded

if __name__ == "__main__":
    print(f"Generando {NUM_USERS} usuarios sintéticos...")
    users = generate_users(NUM_USERS)
    print(f"Generando eventos del {DATE_START} al {DATE_END}...")
    events = generate_events(users, DATE_START, DATE_END)
    print(f"Total de eventos generados: {len(events)}")
    print("Subiendo a Amplitude via Batch API...")
    uploaded = upload_batch(events, API_KEY)
    print(f"\\n✓ Seed completado — {uploaded} eventos subidos al proyecto {APP_ID}")
```

Guardá el archivo en una ubicación que el usuario indique y compartí el link.

---

### PASO 6 — Mensaje de cierre

Mostrá un resumen:

1. **Usuarios generados**: N usuarios sintéticos (`synthetic_user_001` a `synthetic_user_N`)
2. **Eventos cargados**: total de eventos subidos
3. **Funnels simulados**: lista los funnels con las tasas de conversión usadas
4. **Script Python**: link al archivo descargable
5. **Cómo verificar**: instrucciones para ver los datos en Amplitude

---

## Notas técnicas

**Batch API endpoint**: `https://api2.amplitude.com/batch` (no usar `/httpapi`)

**Límite por request**: 2000 eventos máximo. Dividir en batches si se supera.

**Timestamps**: usar milliseconds Unix timestamp (13 dígitos).

**API Secret**: es la clave privada del proyecto. No confundir con la API Key (pública).
Se encuentra en: Amplitude → Settings → Projects → [proyecto] → General → API Keys → Secret Key.

**Datos sintéticos**: usar siempre prefijo `synthetic_` en los user IDs para poder
filtrarlos y eliminarlos fácilmente del proyecto si es necesario.

**Rate limiting**: si el proyecto tiene muchos eventos, agregar un `time.sleep(0.5)`
entre batches para evitar rate limiting de la API.

**Verificación**: después de cargar, esperar 2-5 minutos antes de buscar los datos
en Amplitude — el pipeline de ingestión tiene latencia.
