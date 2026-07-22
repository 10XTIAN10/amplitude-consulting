# Claude Skills — Amplitude Consulting

Skills para Claude Desktop que automatizan tareas frecuentes de consultoría en Amplitude.

## ¿Cómo instalar una skill?

1. Abrí Claude Desktop → **Settings → Skills**
2. Creá una carpeta con el nombre de la skill (ej: `amplitude-demo-generator-unified`)
3. Copiá el archivo `SKILL.md` dentro de esa carpeta
4. Reiniciá Claude Desktop
5. Activá la skill con las frases indicadas en cada SKILL.md

---

## Skills disponibles

### 🌐 /amplitude-demo-generator-unified

Genera una web demo funcional con el Amplitude Unified SDK integrado: Analytics, Session Replay, Guides & Surveys, Web Experiment y Feature Experiment, adaptada a cualquier industria y marca de referencia.

**Activar con:** `"demo de amplitude unified"`, `"unified sdk demo"`, `"demo con todas las features de amplitude"`, `"web demo con feature experiment"`, `"demo con guides and surveys"`

**Requiere:** Claude Desktop + Cowork + API Key de Amplitude (+ Deployment Key opcional para Feature Experiment en modo remoto)

📁 [`/skills/amplitude-demo-generator-unified/SKILL.md`](./amplitude-demo-generator-unified/SKILL.md)

---

### 🌱 /amplitude-data-seeder

Lee el tracking plan del proyecto vía MCP, genera eventos sintéticos con funnels realistas y los carga vía Batch API.

**Activar con:** `"generar datos demo"`, `"seed eventos amplitude"`, `"poblar proyecto amplitude"`, `"datos sintéticos amplitude"`

**Requiere:** Claude Desktop + MCP de Amplitude conectado + API Key + API Secret

📁 [`/skills/amplitude-data-seeder/SKILL.md`](./amplitude-data-seeder/SKILL.md)

---

## Más información

Playbook completo: [Amplitude Consulting Playbook](https://10xtian10.github.io/amplitude-consulting/)

Autor: [Cristian Vallon](https://www.linkedin.com/in/cristian-vallon/)
