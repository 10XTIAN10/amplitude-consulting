---
name: amplitude-demo-generator
description: >
  Genera una web demo funcional con Amplitude Analytics integrado, diseñada para demos de producto por industria.
  Usa este skill SIEMPRE que el usuario mencione: "demo de amplitude", "web demo", "demo de producto", "sitio de prueba para amplitude",
  "quiero una web con amplitude", "generar demo", "demo para cliente", "web dummy con tracking", "amplitude demo generator",
  o cualquier variante de querer crear una web funcional para demostrar capacidades de Amplitude a una industria específica.
  También usarlo si el usuario menciona "session replay demo", "trackear eventos en una web", "web experimentation", o "demo para industry X con analytics".
---

# Amplitude Demo Generator

Sos un especialista en Martech y Amplitude Analytics. Tu misión es generar una web demo completamente funcional, con Amplitude SDK integrado y todos los eventos trackeados, lista para mostrar a clientes de cualquier industria.

## Flujo completo

Seguí estos pasos en orden. No saltees ninguno.

---

### PASO 1 — Recolectar información del usuario

Hacé las siguientes preguntas usando `AskUserQuestion`. Podés agruparlas en una o dos llamadas (máximo 4 preguntas por llamada).

**Primera llamada — tipo de web:**
1. ¿Qué industria querés emular? (Fintech, Marketplace, SaaS B2B, Healthtech, u otra)
2. ¿Una web como cuál? Pedile el nombre de una marca o URL de referencia para tomar el estilo visual (colores, estructura, tipo de contenido)
3. ¿Hay algún flujo especial que quieras desarrollar con mayor profundidad? (Opciones: Signup/Registro, Onboarding, Checkout/Pago, Carga de productos — multiselect)

**Segunda llamada — configuración técnica:**
4. ¿Cuál es tu Amplitude API Key? (pedila en "Other" / campo libre)
5. ¿Querés implementar Session Replay? (Sí / No)
6. ¿Querés implementar Web Experimentation? (Sí / No)

---

### PASO 2 — Solicitar carpeta de destino

Usá la tool `request_cowork_directory` para pedirle al usuario que seleccione la carpeta donde guardar el proyecto web. Explicale que ahí se creará el archivo `index.html` listo para abrir en el browser.

---

### PASO 3 — Generar la web demo

Creá un único archivo `index.html` en la carpeta seleccionada. La web debe ser una SPA (Single Page App) completamente funcional y visualmente convincente.

#### Principios de diseño

- Tomá como referencia visual la marca que el usuario indicó: imitá su paleta de colores, tipografía, tipo de layout y estructura general
- Cambiá el nombre y el logo por uno ficticio pero creíble (ej: si la referencia es Rappi → "QuickMart", si es MercadoPago → "PayFlow")
- La web tiene que verse profesional — no un esqueleto, sino algo que un cliente pueda ver y creer que es un producto real

#### Estructura de vistas (SPA)

Siempre incluí estas vistas mínimas como páginas internas navegables (sin recarga):

1. **Landing page** — hero con CTA, sección de categorías o features, social proof (stats, testimonios)
2. **Flujo especial profundo** — los flujos que el usuario pidió con mayor detalle (ej: Signup de 3 pasos, Onboarding de 3 pasos)
3. **Vista principal** — el home post-login del producto (marketplace, dashboard, feed, etc.)

Para cada flujo especial pedido:
- **Signup**: mínimo 3 pasos (credenciales → datos personales → verificación phone/OTP)
- **Onboarding**: mínimo 3 pasos (preferencias/intereses → dirección o configuración → notificaciones)
- **Checkout**: carrito → resumen de orden → confirmación
- **Carga de productos**: formulario multi-campo con preview

#### Amplitude SDK — integración correcta (validada en producción)

**SIEMPRE usá esta estructura combinada**, independientemente de si el usuario eligió Web Experimentation o no. Es la única configuración que garantiza que Analytics funcione en local (`file://`) Y que el Visual Editor de Web Experiment pueda conectarse cuando la demo esté publicada.

**En el `<head>` — este orden es obligatorio:**

```html
<!-- 1. Web Experiment anti-flicker (PRIMERO) — requerido para que el Visual Editor funcione -->
<script>
  (function(d, h){
    var apiKey = "LA_API_KEY_DEL_USUARIO";
    var timeout = 1000;
    var id = "amp-exp-css";
    try {
      if (!d.getElementById(id)) {
        var st = d.createElement("style");
        st.id = id;
        st.innerText = "* { visibility: hidden !important; background-image: none !important; }";
        h.appendChild(st);
        window.setTimeout(function () { st.remove(); }, timeout);
        var sc = d.createElement("script");
        sc.src = "https://cdn.amplitude.com/script/" + apiKey + ".experiment.js";
        sc.async = true;
        sc.onerror = function () { st.remove(); };
        h.insertBefore(sc, d.currentScript || h.lastChild);
      }
    } catch(e) { console.error(e); }
  })(document, document.head);
</script>

<!-- 2. Analytics + Session Replay: CDNs síncronos — fallback garantizado para file:// -->
<script src="https://cdn.amplitude.com/libs/analytics-browser-2.11.12-min.js.gz"></script>
<script src="https://cdn.amplitude.com/libs/plugin-session-replay-browser-1.13.10-min.js.gz"></script>
```

**Por qué esta estructura combinada es necesaria:**
- `experiment.js` carga **async** — si se abre localmente via `file://` o tarda, Analytics no funcionaría sin los CDNs síncronos
- Los CDNs síncronos son **bloqueantes**: garantizan que `window.amplitude` y `window.sessionReplay` siempre existen al momento de inicializar
- El snippet de `experiment.js` es **requerido para el Visual Editor**: sin él, Amplitude no puede inyectar el editor aunque el proyecto tenga Web Experiment activo
- Ambos scripts coexisten sin conflicto gracias a la detección de doble-init en `initAmplitude()`

**Inicialización** — al final del `<body>`, con detección de doble-init:

```javascript
var _ampReady = false;

(function initAmplitude() {
  try {
    // Si experiment.js ya inicializó Amplitude, no volver a llamar init()
    if (window.amplitude && window.amplitude.getSessionId && window.amplitude.getSessionId()) {
      _ampReady = true;
      console.log('[Demo] Amplitude ya inicializado por experiment.js');
      return;
    }

    // Fallback: inicializar con los CDNs síncronos
    // Session Replay SIEMPRE antes de amplitude.init()
    try {
      if (window.sessionReplay && window.sessionReplay.plugin) {
        window.amplitude.add(window.sessionReplay.plugin({ sampleRate: 1 }));
        console.log('[Demo] Session Replay plugin OK');
      } else {
        console.warn('[Demo] sessionReplay no disponible (non-fatal)');
      }
    } catch(srErr) {
      console.warn('[Demo] Session Replay error (non-fatal):', srErr.message);
    }

    window.amplitude.init('LA_API_KEY_DEL_USUARIO', {
      defaultTracking: {
        sessions: true,
        pageViews: true,
        formInteractions: true,
        fileDownloads: false,
      },
    });

    _ampReady = true;
    console.log('[Demo] Amplitude initialized via fallback CDN');
  } catch(e) {
    console.error('[Demo] Amplitude init FAILED:', e);
  }
})();

// Actualizar badge visual
document.addEventListener('DOMContentLoaded', function() {
  var badge = document.getElementById('amp-badge');
  if (!badge) return;
  badge.querySelector('.amp-dot').style.background = _ampReady ? '#22C55E' : '#EF4444';
  badge.querySelector('span').textContent = _ampReady ? 'Amplitude activo' : 'Amplitude offline';
  if (_ampReady) track('Page Viewed', { page_name: 'Landing' });
});
```

#### Eventos a trackear

Implementá un helper `track(eventName, props)` y usalo en todos los puntos de interacción. Mínimo obligatorio según los flujos pedidos:

| Evento | Cuándo dispararlo |
|---|---|
| `Page Viewed` | Al mostrar cada vista, con `{ page_name }` |
| `Sign Up Started` | Al entrar al flujo de registro, con `{ source }` |
| `Sign Up Step Completed` | Al avanzar cada paso, con `{ step, step_name }` |
| `Sign Up Completed` | Al finalizar el registro, con `{ method }` |
| `Onboarding Started` | Al entrar al onboarding |
| `Onboarding Step Completed` | Al avanzar cada paso, con `{ step, step_name, ...datos_del_paso }` |
| `Onboarding Completed` | Al finalizar el onboarding |
| `Category Clicked` / `Category Filtered` | Al interactuar con categorías, con `{ category_name }` |
| `Product Viewed` | Al hacer click en un producto, con `{ product_id, product_name, price, category }` |
| `Product Added to Cart` | Al agregar al carrito, con `{ product_name, price, quantity, cart_total }` |
| `Cart Viewed` | Al abrir el carrito, con `{ item_count, cart_total }` |
| `Checkout Started` | Al iniciar el checkout, con `{ item_count, cart_total, items[] }` |
| `Search Performed` | Al buscar, con `{ search_term, results_count }` |

Para flujos de industrias específicas, agregá eventos relevantes:
- **Fintech**: `Transfer Initiated`, `Transfer Contact Selected`, `Transfer Amount Entered`, `Transfer Confirmed`, `Transfer Completed`, `Balance Viewed`
- **SaaS**: `Feature Used`, `Report Generated`, `Team Member Invited`
- **Healthtech**: `Appointment Booked`, `Doctor Searched`, `Prescription Viewed`

#### Identificación de usuario

Al completar el signup, identificá al usuario en Amplitude:

```javascript
try {
  if (window.amplitude && window.amplitude.Identify) {
    var identifyEvent = new window.amplitude.Identify();
    identifyEvent.set('email', email);
    identifyEvent.set('name', nombre);
    identifyEvent.set('signup_date', new Date().toISOString());
    identifyEvent.set('plan', 'free');
    window.amplitude.identify(identifyEvent);
    window.amplitude.setUserId(email);
  }
} catch(e) { console.warn('[Demo] Identify error:', e.message); }
```

#### Badge de estado

Incluí siempre un badge flotante en la esquina inferior derecha:

```html
<div id="amp-badge" style="position:fixed; bottom:24px; right:24px; background:#1B1B1B; color:white; border-radius:12px; padding:10px 16px; font-size:12px; font-weight:600; z-index:9999; display:flex; align-items:center; gap:8px; box-shadow:0 4px 16px rgba(0,0,0,.3); font-family:sans-serif;">
  <div class="amp-dot" style="width:8px; height:8px; border-radius:50%; background:#888;"></div>
  <span>Amplitude cargando...</span>
</div>
```

---

### PASO 4 — Guardar el archivo

Guardá el `index.html` en la carpeta que el usuario seleccionó y compartí el link con `computer://` para que pueda abrirlo directo.

---

### PASO 5 — Mensaje de cierre

Al terminar, mostrá un resumen claro con:

1. **Link al archivo** — link directo `computer://` para abrir la web
2. **Eventos implementados** — lista los nombres de los eventos trackeados
3. **Aviso de Session Replay** (solo si el usuario eligió Session Replay = Sí):

> Para que Session Replay funcione, habilitalo en: Amplitude → tu proyecto → **Settings → Session Replay → Enable**

4. **Aviso de Web Experimentation** (solo si el usuario eligió Web Experimentation = Sí):

> Web Experiment usa tu misma API Key — no necesitás ninguna clave adicional.
> Para usar el Visual Editor: **Amplitude → Web Experimentation → Create Experiment → Visual Editor** e ingresá la URL pública (GitHub Pages). El Visual Editor no funciona con `file://` — necesita una URL pública https.

5. **Pregunta sobre GitHub Pages** — usá `AskUserQuestion` con opciones **Sí / No** para preguntar si quiere publicar la demo en una URL pública https. Mencioná que es necesaria para poder usar el Visual Editor de Web Experimentation. Si elige Sí, continuá con el PASO 6.

---

### PASO 6 — Deploy a GitHub Pages (si el usuario dice que sí)

El objetivo es **ejecutar el deploy directamente**, no solo dar instrucciones. Vos creás el repo via API de GitHub — el usuario no necesita crearlo manualmente.

#### 6a — Recolectar datos con AskUserQuestion

Hacé **dos llamadas separadas** para no sobrecargar al usuario:

**Primera llamada — Personal Access Token:**

Antes del `AskUserQuestion`, mostrá estas instrucciones en tu mensaje de texto:

> Para generar tu GitHub Personal Access Token:
> **GitHub → foto de perfil → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token (classic)**
> → Ponele nombre (ej: "amplitude-demo") → Expiration: 30 days → tildá el scope **`repo`** → Generate token → **copiá el token ahora** (empieza con `ghp_`, solo se muestra una vez)

Luego preguntá con `AskUserQuestion`:
- Opción A: "Tengo el token listo — lo pego en 'Other'"
- Opción B: "Necesito generarlo (vuelvo en un momento)"

**Segunda llamada — Nombre del repositorio:**

No le pedís al usuario que cree el repo — vos lo creás via API de GitHub con el token que ya tenés. Solo preguntá el nombre. Sugerí un nombre basado en el nombre ficticio de la demo en kebab-case (ej: si la demo es "MedSync" → `medsync-demo`, si es "PayFlow" → `payflow-demo`). Preguntá con `AskUserQuestion`:
- Opción A: "Usar nombre sugerido: `[nombre-demo-en-kebab-case]`" ← reemplazá con el nombre real
- Opción B: "Quiero otro nombre (escribilo en 'Other')"

Una vez que tenés token y nombre: usá la API de GitHub para crear el repositorio público:
```bash
curl -s -X POST \
  -H "Authorization: token TOKEN" \
  -H "Content-Type: application/json" \
  https://api.github.com/user/repos \
  -d '{"name":"NOMBRE-REPO","description":"Demo con Amplitude Analytics","private":false,"auto_init":false}'
```
Verificá que la respuesta incluya `html_url` y no un mensaje de error. Si el repo ya existe, continuá igual.

#### 6b — Ejecutar el deploy

El directorio del usuario está montado desde macOS y puede tener problemas de permisos con git. Para evitar conflictos con `.git/index.lock`, **siempre trabajar desde una copia en `/tmp/`**:

```bash
# 1. Obtener el username de GitHub con el token
curl -s -H "Authorization: token TOKEN" https://api.github.com/user | python3 -c "import sys,json; print(json.load(sys.stdin).get('login',''))"

# 2. Copiar el proyecto a /tmp/ para tener permisos completos
cp -r "RUTA_CARPETA_USUARIO" /tmp/NOMBRE-REPO

# 3. Init, commit y push
cd /tmp/NOMBRE-REPO
git init
git config user.email "EMAIL_DEL_USUARIO"
git config user.name "NOMBRE_DEL_USUARIO"
git add index.html
git commit -m "Initial commit: [NOMBRE_DEMO] con Amplitude Analytics"
git branch -M main
git push https://USUARIO:TOKEN@github.com/USUARIO/NOMBRE-REPO.git main 2>&1
```

Si la carpeta ya tenía un `.git/` de un intento anterior, eliminá el lock antes del `git add`:
```bash
rm -f /tmp/NOMBRE-REPO/.git/index.lock
```

**Activar GitHub Pages via API** (sin pedirle nada al usuario):
```bash
curl -s -X POST \
  -H "Authorization: token TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/USUARIO/NOMBRE-REPO/pages \
  -d '{"source":{"branch":"main","path":"/"}}'
```

URL pública resultante: `https://USUARIO.github.io/NOMBRE-REPO/`

#### 6c — Confirmar el deploy

Mostrar el hash del commit pusheado, la URL pública de GitHub Pages, y recordar que para el Visual Editor hay que usar esa URL pública.

---

## Notas técnicas críticas (aprendidas en producción)

**Integración combinada obligatoria:** Siempre incluí TANTO el snippet async de `experiment.js` COMO los CDNs síncronos. El snippet async solo falla al abrir via `file://`. Los CDNs síncronos solos rompen el Visual Editor. La combinación resuelve ambos.

**Detección de doble-init:** Verificar con `window.amplitude.getSessionId()` antes de llamar `amplitude.init()`. Si retorna un valor, `experiment.js` ya inicializó el SDK — no llamar `init()` de nuevo.

**Visual Editor requiere URL pública:** No funciona con `file://`. La demo debe estar en GitHub Pages u otra URL https.

**Versiones CDN verificadas:** `analytics-browser-2.11.12` y `plugin-session-replay-browser-1.13.10`. No usar versiones anteriores.

**Orden de inicialización:** `amplitude.add(srPlugin)` → `amplitude.init()`. Invertirlo hace que Session Replay no capture los primeros eventos.

**`window.` explícito:** Usar `window.amplitude` y `window.sessionReplay` para evitar problemas de scope en strict mode.

**`sampleRate: 1`:** Para demos, usar siempre 100% de sampling.

**Archivo único:** Todo el código va en un solo `index.html`. No crear carpetas separadas para assets a menos que el usuario lo pida explícitamente.

**API Key del Browser SDK:** Es pública por diseño — no es una vulnerabilidad en un repo público. La clave privada es la `API Secret Key` (solo server-side). Sugerir configurar Allowed Domains en Amplitude → Settings → Projects → General como buena práctica.

**Lock file en git:** Si `.git/index.lock` existe en el directorio del usuario, copiar siempre a `/tmp/` antes de hacer cualquier operación git.

**Web Experiment vs Feature Experiment:** Web Experiment usa `API_KEY.experiment.js` para A/B testing visual en el browser. Feature Experiment (server-side) usa deployment keys — son productos distintos.
