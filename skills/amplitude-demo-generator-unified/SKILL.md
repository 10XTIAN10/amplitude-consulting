---
name: amplitude-demo-generator-unified
description: >
  Genera una web demo funcional con el Amplitude Unified SDK integrado (Analytics, Session Replay,
  Guides & Surveys, Web Experiment y Feature Experiment), diseñada para demos de producto por industria.
  Usa este skill SIEMPRE que el usuario mencione: "demo de amplitude unified", "unified sdk demo",
  "demo con todas las features de amplitude", "web demo con feature experiment", "demo con guides and surveys",
  "amplitude demo generator unified", o cualquier variante de querer crear una web funcional que use el
  Unified SDK de Amplitude (en vez del SDK clásico) para demostrar Analytics + Session Replay + Web Experiment +
  Feature Experiment + Guides & Surveys a una industria específica.
---

# Amplitude Demo Generator (Unified SDK)

Sos un especialista en Martech y Amplitude Analytics, con expertise específico en el **Unified SDK** de Amplitude. Tu misión es generar una web demo completamente funcional que implemente los 5 productos de Amplitude (Analytics, Session Replay, Guides & Surveys, Web Experiment, Feature Experiment) usando la instalación vía **Unified Script** (CDN, sin bundler), lista para mostrar a clientes de cualquier industria.

Esta skill es una variante de `amplitude-demo-generator`. La diferencia central está en el PASO 3 (instalación del SDK) y en que esta versión siempre genera, además del `index.html`, un documento `.md` de implementación funcional.

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
5. ¿Tenés una Deployment Key de Feature Experiment? (la API Key de Analytics NO sirve para esto — es una key distinta que se genera en Amplitude → Experiment → Feature Experiment → Deployments). Opciones: "Sí, la tengo" (pedila en "Other") / "No, que corra en modo local por ahora"
6. ¿Tu proyecto tiene Guides & Surveys habilitado en Amplitude → Settings → Guides & Surveys? (Sí / No / No sé)

**Tercera llamada — pre-armar el experimento (Feature Experiment, SIEMPRE se configura uno):**
7. ¿En qué flujo querés pre-armar un experimento simple de texto/color de botón? Dale opciones concretas armadas en base a lo que el usuario eligió en la pregunta 3, por ejemplo:
   - "CTA principal del landing" (ej: texto del botón "Empezar gratis")
   - "Botón del flujo profundo elegido" (ej: botón de confirmar en Checkout, o de continuar en Signup)
   - "Botón de la vista principal" (ej: CTA del dashboard/home)
8. ¿Qué tipo de cambio querés testear? Opciones: "Texto del botón" / "Color del botón"

No inventes ninguna key, ID o valor que el usuario no te haya dado. Si falta la Deployment Key, el experimento se implementa igual (nunca queda "sin configurar") pero corriendo en **modo local** — ver el detalle técnico en el PASO 3.

---

### PASO 2 — Solicitar carpeta de destino

Usá la tool `request_cowork_directory` para pedirle al usuario que seleccione la carpeta donde guardar el proyecto web. Explicale que ahí se van a crear dos archivos: `index.html` y un documento de implementación `.md`.

---

### PASO 3 — Generar la web demo

Creá un único archivo `index.html` en la carpeta seleccionada. La web debe ser una SPA (Single Page App) completamente funcional y visualmente convincente.

#### Principios de diseño

- Tomá como referencia visual la marca que el usuario indicó: imitá su paleta de colores, tipografía, tipo de layout y estructura general
- Cambiá el nombre y el logo por uno ficticio pero creíble
- La web tiene que verse profesional — no un esqueleto, sino algo que un cliente pueda ver y creer que es un producto real

#### Estructura de vistas (SPA)

Igual que en la skill clásica: Landing → Flujo especial profundo (mínimo 3 pasos) → Vista principal post-login.

#### Amplitude Unified SDK — integración correcta

**Esta skill usa el Unified Script, NO la combinación clásica de `analytics-browser` + `plugin-session-replay-browser` + `experiment.js`.** Es un único script que ya incluye Analytics, Session Replay (opcional vía plugin) y Web Experiment por defecto. Guides & Surveys y Feature Experiment requieren piezas adicionales.

**En el `<head>` — este orden es obligatorio:**

```html
<!-- 1. Unified Script — incluye Analytics + Web Experiment por defecto -->
<script src="https://cdn.amplitude.com/script/API_KEY.js"></script>

<!-- 2. Guides & Surveys — requiere script separado por tamaño, va DESPUÉS del Unified Script -->
<script src="https://cdn.amplitude.com/script/API_KEY.engagement.js"></script>

<!-- 3. Feature Experiment — el Unified Script NO lo incluye (solo el paquete npm lo hace).
     Para un HTML estático sin bundler, se carga el bundle UMD standalone vía CDN (versión fija). -->
<script src="https://unpkg.com/@amplitude/experiment-js-client@1.21.3/dist/experiment.umd.js"></script>
```

**Por qué esta estructura:**
- El Unified Script (`API_KEY.js`) habilita Session Replay y Web Experiment por defecto — Web Experiment ya NO necesita el snippet anti-flicker viejo de `experiment.js`, el script único lo resuelve.
- Guides & Surveys queda excluido del Unified Script por tamaño de bundle — Amplitude requiere cargarlo aparte con `API_KEY.engagement.js` y registrarlo con `amplitude.add(window.engagement.plugin())`.
- Feature Experiment es código-based (feature flags), no vive en el Unified Script ni en el `.engagement.js`. El único paquete que lo incluye es el npm `@amplitude/unified`, que no aplica acá porque este demo es un HTML sin bundler. La alternativa validada es cargar el bundle UMD standalone del `experiment-js-client` (expone `window.Experiment`) directo desde unpkg, fijando la versión (`@1.21.3`) para reproducibilidad.

**Inicialización** — al final del `<body>`:

```javascript
var _ampReady = false;
var _featureExperimentReady = false;

(function initAmplitude() {
  try {
    // Session Replay: SIEMPRE antes de amplitude.init()
    try {
      if (window.sessionReplay && window.sessionReplay.plugin) {
        window.amplitude.add(window.sessionReplay.plugin({ sampleRate: 1 }));
        console.log('[Demo] Session Replay plugin OK');
      }
    } catch(srErr) {
      console.warn('[Demo] Session Replay error (non-fatal):', srErr.message);
    }

    // Guides & Surveys
    try {
      if (window.engagement && window.engagement.plugin) {
        window.amplitude.add(window.engagement.plugin());
        console.log('[Demo] Guides & Surveys plugin OK');
      }
    } catch(gsErr) {
      console.warn('[Demo] Guides & Surveys error (non-fatal):', gsErr.message);
    }

    window.amplitude.init('API_KEY', {
      fetchRemoteConfig: true,
      autocapture: {
        attribution: true,
        fileDownloads: true,
        formInteractions: true,
        pageViews: true,
        sessions: true,
        elementInteractions: true,
        networkTracking: true,
        webVitals: true,
        frustrationInteractions: true,
      },
    });

    _ampReady = true;
    console.log('[Demo] Amplitude Unified SDK initialized');
  } catch(e) {
    console.error('[Demo] Amplitude init FAILED:', e);
  }
})();

// ===== Feature Experiment — SIEMPRE queda configurado y corriendo =====
// Basado en la doc oficial (experiment-js-client): variant(key, fallback) y la opción
// fallbackVariant SIEMPRE devuelven un valor utilizable, incluso sin Deployment Key.
// Con Deployment Key -> remote evaluation real. Sin ella -> modo local (documentado como tal,
// nunca se simula ni se etiqueta como "real" evaluación de Amplitude).
var EXPERIMENT_FLAG_KEY = 'FLAG_KEY_ELEGIDO'; // debe coincidir con el flag/experimento creado en Amplitude
var DEPLOYMENT_KEY = ''; // completar con la Deployment Key real cuando el usuario la tenga

var FALLBACK_VARIANT = { value: 'control', payload: { buttonText: 'TEXTO_ORIGINAL' } };
var DEMO_VARIANT      = { value: 'treatment', payload: { buttonText: 'TEXTO_ALTERNATIVO' } };
var _experimentVariant = FALLBACK_VARIANT;
var experimentClient = null;

(function initFeatureExperiment() {
  try {
    if (!window.Experiment) {
      console.warn('[Demo] Experiment SDK no cargó (revisar script de unpkg)');
      return;
    }

    if (DEPLOYMENT_KEY) {
      // Remote evaluation real — requiere que el flag EXPERIMENT_FLAG_KEY exista en Amplitude
      experimentClient = window.Experiment.initializeWithAmplitudeAnalytics(DEPLOYMENT_KEY, {
        fallbackVariant: FALLBACK_VARIANT
      });
      experimentClient.fetch().then(function () {
        _experimentVariant = experimentClient.variant(EXPERIMENT_FLAG_KEY, FALLBACK_VARIANT);
        _featureExperimentReady = true;
        applyExperimentVariant();
        updateBadge();
      });
    } else {
      // Modo LOCAL: sin red, sin Deployment Key. Split determinístico 50/50 entre
      // FALLBACK_VARIANT y DEMO_VARIANT SOLO para poder previsualizar ambos casos en la demo.
      // Esto NO es evaluación real de Amplitude — se documenta explícitamente como tal.
      _experimentVariant = (Math.random() < 0.5) ? FALLBACK_VARIANT : DEMO_VARIANT;
      _featureExperimentReady = true; // "activo en modo local"
      applyExperimentVariant();
      updateBadge();
      console.warn('[Demo] Feature Experiment en modo LOCAL (sin Deployment Key). Ver AMPLITUDE-IMPLEMENTATION.md para pasar a remote evaluation real.');
    }
  } catch (feErr) {
    console.warn('[Demo] Feature Experiment error (non-fatal):', feErr.message);
  }
})();

function applyExperimentVariant() {
  var btn = document.getElementById('experiment-cta-btn'); // el botón elegido en PASO 1, preguntas 7-8
  if (btn && _experimentVariant && _experimentVariant.payload) {
    if (_experimentVariant.payload.buttonText) btn.textContent = _experimentVariant.payload.buttonText;
    if (_experimentVariant.payload.buttonColor) btn.style.background = _experimentVariant.payload.buttonColor;
  }
  track('Feature Flag Exposed', { flag_key: EXPERIMENT_FLAG_KEY, variant: _experimentVariant.value });
}
```

**Reglas para esta parte, sin excepción:**
- El experimento SIEMPRE se implementa y SIEMPRE resuelve una variante — nunca queda "sin armar", tenga o no tenga Deployment Key el usuario.
- El flag key (`EXPERIMENT_FLAG_KEY`) y los dos valores de payload (texto o color original vs. alternativo) se definen según las respuestas 7 y 8 del PASO 1 — nunca inventes un flag key genérico sin sentido para la industria, usá algo descriptivo (ej: `signup-cta-copy`, `checkout-button-color`).
- Si no hay Deployment Key, el modo local queda clarísimamente etiquetado en consola y en el `.md` — nunca se hace pasar por evaluación real de Amplitude.
- Si el usuario dijo "No sé" o "No" sobre Guides & Surveys estar habilitado en su proyecto, igual dejá el código del engagement plugin — es no invasivo y no rompe nada si la feature no está habilitada del lado de Amplitude.

#### Eventos a trackear — mínimo 30, ideal 30-50

Implementá un helper `track(eventName, props)` y usalo en todos los puntos de interacción de las vistas y flujos generados. La cantidad final depende de la industria y los flujos elegidos, pero como piso cubrí estas categorías (no son eventos fijos — adaptalos a la industria/flujo real, pero mantené la cobertura):

- **Navegación/vistas**: `Page Viewed`, `Nav Link Clicked`, `CTA Clicked`, `Empty State Viewed`
- **Signup/Onboarding** (si aplica): un evento por paso (`Signup Step Viewed`, `Signup Credentials Entered`, `Signup Personal Data Entered`, `Signup OTP Requested`, `Signup OTP Entered`, `Signup Completed`, `Signup Error`)
- **Checkout/Transacción** (si aplica): `Checkout Started`, evento por paso, `Item Added To Cart`, `Payment Method Selected`, `Order Confirmed`, `Order Error`
- **Búsqueda/descubrimiento**: `Search Performed`, `Search Cleared`, `Filter Applied`, `Item Clicked`
- **Vista principal / dashboard**: `Home Viewed`, eventos de interacción específicos de la industria (ej. `Balance Viewed` en fintech, `Course Viewed` en edtech)
- **Cuenta/perfil**: `Profile Menu Opened`, `Profile Viewed`, `Notification Clicked`, `Logout Clicked`
- **Feature Experiment** (si hay Deployment Key): `Feature Flag Exposed` con `{ flag_key, variant }` cuando el fetch resuelve

Documentá cada evento final (nombre exacto + cuándo se dispara + propiedades) en el PASO 3.5.

#### Identificación de usuario

Igual que la skill clásica — al completar signup:

```javascript
try {
  if (window.amplitude && window.amplitude.Identify) {
    var identifyEvent = new window.amplitude.Identify();
    identifyEvent.set('email', email);
    identifyEvent.set('signup_date', new Date().toISOString());
    identifyEvent.set('plan', 'free');
    window.amplitude.identify(identifyEvent);
    window.amplitude.setUserId(email);
  }
} catch(e) { console.warn('[Demo] Identify error:', e.message); }
```

#### Badge de estado

Incluí siempre un badge flotante en la esquina inferior derecha que refleje el estado real de los 5 productos (Analytics, Session Replay, Guides & Surveys, Web Experiment, Feature Experiment) — no solo "Amplitude activo/offline" genérico como en la skill clásica.

---

### PASO 3.5 — Generar el documento de implementación `.md`

Creá un archivo `AMPLITUDE-IMPLEMENTATION.md` en la misma carpeta, con esta estructura:

```markdown
# Implementación de Amplitude — [Nombre Demo]

## Datos del proyecto
| Campo | Valor |
|---|---|
| Proyecto Amplitude (API Key) | [primeros 8 caracteres]... |
| Repositorio GitHub | [URL del repo, si se hizo deploy — si no, "No aplica"] |
| URL pública (GitHub Pages) | [URL pública, si se hizo deploy — si no, "No aplica"] |
| Rama / deploy | [ej: `main`, deploy directo desde raíz] |

## SDK utilizado
Unified Script (CDN, sin bundler) — API Key: [primeros 8 caracteres]...

## Productos activos
| Producto | Estado | Cómo está implementado |
|---|---|---|
| Analytics | Activo | Unified Script + autocapture completo |
| Session Replay | Activo | `sessionReplay.plugin({ sampleRate: 1 })` |
| Guides & Surveys | Activo (requiere habilitarlo en el proyecto) | `API_KEY.engagement.js` + `engagement.plugin()` |
| Web Experiment | Activo | Incluido por defecto en el Unified Script |
| Feature Experiment | Activo — [modo LOCAL / modo REMOTE con Deployment Key] | `experiment-js-client@1.21.3` (UMD, vía unpkg) — flag key: `[EXPERIMENT_FLAG_KEY]` |

## Experimento pre-armado (Feature Experiment)
- **Flujo/botón**: [cuál eligió el usuario en la pregunta 7]
- **Tipo de cambio**: [texto / color, según pregunta 8]
- **Flag key**: `[EXPERIMENT_FLAG_KEY]`
- **Variantes**: `control` = [valor original] / `treatment` = [valor alternativo]
- **Modo actual**: [LOCAL (split 50/50 en el browser, sin Deployment Key) / REMOTE (evaluación real vía Amplitude)]

### Cómo pasar de modo LOCAL a REMOTE (evaluación real)
1. Ir a **Amplitude → Experiment → Feature Experiment → Deployments** y copiar (o crear) la Deployment Key del proyecto
2. Crear un Flag/Experimento en Amplitude con el key EXACTO `[EXPERIMENT_FLAG_KEY]` y dos variantes: `control` y `treatment`
3. En `index.html`, completar `var DEPLOYMENT_KEY = '...'` con la key real
4. Recargar la demo — a partir de ahí `experiment.fetch()` trae la asignación real desde Amplitude, dejando de usar el split local

## Eventos trackeados (N eventos)
| Evento | Cuándo se dispara | Propiedades |
|---|---|---|
| ... | ... | ... |

## Estructura de la web
[Descripción de las vistas y flujos]

## Cómo activar lo que falta
[Instrucciones puntuales — ej. cómo habilitar Guides & Surveys]
```

---

### PASO 4 — Guardar los archivos

Guardá `index.html` y `AMPLITUDE-IMPLEMENTATION.md` en la carpeta que el usuario seleccionó y compartí los links con `computer://`.

---

### PASO 5 — Mensaje de cierre

Mostrá un resumen con: link al `index.html`, link al `.md`, lista de eventos implementados, y qué productos quedaron activos vs. pendientes de configuración del lado del usuario (Deployment Key, habilitar Guides & Surveys, etc.).

Preguntá con `AskUserQuestion` (Sí/No) si quiere publicar la demo en GitHub Pages.

---

### PASO 6 — Deploy a GitHub Pages (si el usuario dice que sí)

Idéntico al PASO 6 de la skill clásica `amplitude-demo-generator` (recolección de token, creación de repo vía API, copia a `/tmp/`, push, activación de Pages vía API). No dupliques la lógica — replicá ese mismo procedimiento.

**Después de confirmar el deploy**, volvé a abrir el `AMPLITUDE-IMPLEMENTATION.md` y completá la sección "Datos del proyecto" con el repo real y la URL pública de GitHub Pages, y volvé a subir ese archivo actualizado al repo (commit + push). Si el usuario no pidió deploy, dejá esos dos campos como "No aplica" en el `.md` y no los inventes.

---

## Notas técnicas críticas (aprendidas en producción)

**Unified Script incluye Web Experiment por defecto:** no hace falta el snippet anti-flicker separado que usaba la skill clásica con `experiment.js`.

**Guides & Surveys es opt-out por tamaño, no por defecto en el script CDN:** siempre requiere el script `.engagement.js` aparte, a diferencia del paquete npm `@amplitude/unified` donde sí viene por defecto.

**Feature Experiment NO tiene versión CDN oficial de Amplitude** — la vía validada es el bundle UMD standalone de `@amplitude/experiment-js-client` en unpkg, fijando versión. Requiere Deployment Key (no confundir con la API Key de Analytics) para evaluación remota real.

**El experimento SIEMPRE queda armado y corriendo, con o sin Deployment Key** — usando el método oficial `variant(key, fallback)` y la opción `fallbackVariant` del config (documentados en la SDK reference de `experiment-js-client`). Sin Deployment Key, se usa un split local 50/50 explícitamente etiquetado como modo LOCAL — nunca se inventa ni se simula una Deployment Key, y nunca se hace pasar el modo local por evaluación real de Amplitude.

**Orden de plugins:** `sessionReplay.plugin()` y `engagement.plugin()` van con `amplitude.add()` ANTES de `amplitude.init()`.

**Versión pinneada del experiment-js-client:** usar siempre `@1.21.3` (o la versión que se haya validado explícitamente) — nunca `@latest`, para reproducibilidad del demo.

**Archivo único:** todo el código va en un solo `index.html`, salvo el documento `.md` de implementación que es un archivo aparte.
