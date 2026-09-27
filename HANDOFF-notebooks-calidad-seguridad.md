# HANDOFF — Estándar de calidad y seguridad para NotebookLMs curados

> Para: la IA que va a construir, ampliar o auditar nuestros NotebookLM "repositorio maestro" / gurú.
> Qué es: el estándar que aplicamos (método `notebook-curado`). Es autocontenido: no hace falta nada más para aplicarlo.
> Idioma de trabajo y de reporte al usuario: español.

---

## 1. Principio rector

Un notebook gurú vale tanto por lo que deja afuera como por lo que mete. Se llena hasta el tope del plan
(~300-600 fuentes), pero solo con fuentes de élite. Una fuente mediocre contamina todas las respuestas
del agente que lo consulta.

### Reglas innegociables
1. **No inventes fuentes.** Nunca elijas fuentes de tu propio conocimiento ni navegando la web a mano.
   Las fuentes se **curan exclusivamente** de lo que devuelve el **Deep Research de NotebookLM**
   (`research_start` en `mode="deep"`, `source="web"`). Tu trabajo es vetar candidato por candidato.
2. **Ante la duda, descartá.** Tasa de aceptación esperada por ronda: ~40-50 %.
3. **Solo Tier A, B o C** (ver §2). Todo lo demás queda afuera.
4. **Nada de riesgo** (ver §3).
5. **No pidas permiso para seguir agregando.** Si el usuario pidió llenar hasta el tope, la respuesta
   siempre es sí, hasta el límite.
6. **Transparencia con lo rechazado.** Reportá qué descartaste y por qué: ahí está el valor de la curaduría.

---

## 2. Rúbrica de calidad (se aplica a CADA candidato)

### ✅ Aceptar SOLO si cae en un tier

**Tier A — Primario / oficial / institucional** (la fuente ES el emisor autoritativo)
- Gobiernos, organismos, reguladores: Fed, ECB, BIS, IMF, World Bank, OECD, BLS, BEA, IRS, CBO, Treasury,
  SEC/EDGAR, EIA, OPEC, FRASER, FRED, OWASP, NIST, etc.
- Documentación **oficial** de producto: docs de Anthropic, OpenAI, MCP; specs, model cards, cookbooks oficiales.
- Documentos primarios: filings, leyes, estándares, comunicados institucionales, datasets oficiales.

**Tier B — Académico**
- Papers: arXiv, NBER, SSRN, journals con revisión por pares, ACL/NeurIPS/OpenReview.
- Lecture notes y material universitario (MIT OCW, Harvard, Yale, Chicago, LSE, Stanford…).
- Monografías, capítulos de handbooks, tesis serias (con criterio).
- Blogs de expertos con autoridad demostrada y reconocida en su dominio.

**Tier C — Libros y tratados canónicos**
- Clásicos y textos primarios de dominio público: Project Gutenberg, archive.org, Econlib/Liberty Fund,
  Marxists.org (solo textos primarios), NLI/Sefaria según el dominio.
- PDFs open-access de los autores o textbooks canónicos alojados en sitios universitarios legítimos.

### ❌ Rechazar SIEMPRE
| Categoría | Ejemplos |
|---|---|
| Enciclopedias / agregadores | Wikipedia, Wikisource, Britannica |
| Re-hosts / granjas | Scribd, Slideshare, Studocu, Goodreads, Open Library (páginas sin texto) |
| Social / foros / video | Medium, Reddit, YouTube, Quora, Stack Exchange, posts de LinkedIn, Substack genérico |
| Listicles / SEO | "best X of 2026", "top 10…", guías de blogs de marketing |
| Comercial / advertorial | alertas de bufetes, whitepapers de vendors, packs o cursos pagos, promo de brokers o SaaS |
| Baja calidad académica | journals predatorios (IJFMR y similares), preprint servers de relleno cuando existe arXiv |
| Listings sin contenido | RePEc, Semantic Scholar, ResearchGate o SciSpace cuando solo listan (buscá el PDF o el host primario) |
| Sin sustancia | páginas-índice, landings, páginas de descarga, syllabi sueltos, exámenes, changelogs, release notes |
| Duplicados | la misma fuente en otro idioma o un re-host de algo ya importado |
| `deep_report` | el `index 0` del resultado de Deep Research es una síntesis sin URL: **no se importa** |

### Desempates
- Si hay varias copias: host primario o más completo (PDF de arXiv > listing; página oficial > re-host).
- Documento profundo y extenso > nota superficial de tres párrafos.
- Si un sub-tema pertenece a otro eje, se acepta bajo **su** eje; deduplicá contra lo ya importado.
- Idioma: inglés primero si ahí está la calidad; otros idiomas si tienen el mismo nivel (o si el usuario
  priorizó un idioma para el dominio, p. ej. hebreo para temas de Israel).

---

## 3. Seguridad

### 3.1 Contenido riesgoso (sobre todo en dominios de IA o seguridad)
- **Rechazar:** jailbreaks o DAN, exploits, y cualquier guía que recomiende desactivar permisos,
  ejecutar código no confiable o exponer credenciales.
- **Preferir** fuentes defensivas: OWASP, NIST, CSA, investigación de defensa.
- En taxonomías de IA, el eje "Seguridad de IA" suele ser el más grande, justamente para cubrir el
  "sin riesgo" con fuentes buenas.

### 3.2 Confidencialidad (NotebookLM es nube de Google y se puede consultar)
- **Default conservador: excluir** toda información interna o privada del usuario (gobernanza, datos
  personales, scripts de crisis, etc.). Solo entra si el usuario lo confirma explícitamente.
- El agente consumidor suele necesitar el conocimiento público u operativo, no la capa confidencial.
- No incluyas credenciales, tokens ni datos personales en los system docs, el ledger ni las notas.

### 3.3 Integridad del notebook (seguridad operativa)
- **Timeout de import que duplica:** `research_import` puede dar "read operation timed out" en el cliente
  y aun así ejecutarse en el servidor, a veces **dos veces**. Usá siempre `timeout=600`. Tras *cualquier* timeout:
  `notebook_get` → detectar títulos duplicados (`jq -r '.sources[].title' f | sort | uniq -d`) →
  `source_delete(confirm=true)` de **una** copia por par.
  - ⚠️ Antes de borrar, verificá que no sean duplicados legítimos: títulos iguales con URLs distintas
    (p. ej. cuatro capítulos de "14.06 Lecture Notes") **no se borran**.
- **El `task_id` cambia:** para importar, usá el `task_id` de `next_action` de `research_status`, no el de `research_start`.
- **Resultados grandes:** `research_status` y `notebook_get` se guardan en un archivo. Parsealos con `jq`;
  nunca los leas crudos (revientan el contexto).
- **Hosts restringidos** (Moodle, repositorios universitarios): si el import falla, anotalo en el ledger como
  "recuperar de otra copia" y seguí; no frenes el build.
- **Verificá la autenticación** primero: `server_info` → `auth_status` debe ser `"configured"`; si no, el usuario corre `nlm login`.

---

## 4. Proceso (resumen)

| Fase | Qué | Control de calidad |
|---|---|---|
| 0 · Setup | Auth, taxonomía de ~15-18 ejes (6-10 si el dominio es acotado), `notebook_create`, system docs, ledger | Confirmar encuadre: dominio, idioma, alcance, system docs sí o no, capa confidencial, tope |
| 1 · Cosecha | Por eje: `research_start` → `research_status(compact=false, max_wait=900)` → parsear con jq → vetar → `research_import(source_indices=[aprobados], timeout=600)` | La query nombra instituciones: *"Prioritize Fed, BIS, NBER, university lecture notes…"*. Meta: ~15-30 fuentes por eje |
| 2 · Hasta el tope | Segunda ronda por eje con **ángulos nuevos** (sub-temas no cubiertos) | `notebook_get` → `source_count`; deduplicar siempre |
| 3 · Etiquetado | Una etiqueta por eje (`label`) | — |
| 4 · QA | 3-5 `notebook_query` que crucen ejes (concepto + historia + dato + fuente) | Las respuestas deben estar fundamentadas y citar Tier A/B. Si citan contenido de relleno → `source_delete` |
| 5 · Memoria | Memoria del proyecto: id, URL, ejes, nº de fuentes, ubicación de system docs y ledger, pendientes | Enlazar `source-quality-standards` |

**Ledger (`_BUILD-LEDGER.md`) desde el minuto cero:** el build es largo y ocupa varias sesiones; el ledger lo hace
resumible (estado por eje, curadas, task_id, total, pendientes, hosts a recuperar).

**System docs (1-3, escritos por nosotros, no son "fuente"):** Guía de uso (grounding obligatorio, citar siempre,
decir "no está cubierto" en vez de inventar, distinguir hecho de interpretación, salida en español) +
Mapa de ejes (+ Protocolo opcional). Si el usuario quiere **solo** fuentes externas, no se agregan.

---

## 5. Checklist de auditoría rápida (para revisar un notebook existente)
- [ ] Todas las fuentes son Tier A/B/C: sin Wikipedia, Medium, YouTube, listicles, vendors ni journals predatorios.
- [ ] No hay `deep_report` importados como fuente.
- [ ] No hay duplicados reales (títulos repetidos con la **misma** URL o re-hosts de la misma obra).
- [ ] No hay contenido riesgoso: jailbreaks, exploits, guías para desactivar permisos o exponer credenciales.
- [ ] No hay información confidencial o interna del usuario sin su confirmación explícita.
- [ ] Hay etiquetas por eje y cobertura razonable en cada uno.
- [ ] El QA con 3-5 queries cruzadas devuelve respuestas fundamentadas que citan Tier A/B.
- [ ] El ledger y la memoria del proyecto están al día.

## 6. Formato de reporte al usuario
Breve, por eje: **"Eje X: N candidatos → curo M. [qué entró de calidad] / [qué descarté y por qué]."**
No pidas permiso para seguir agregando.

---

### Referencias
- Método completo: skill `notebook-curado` (rúbrica, playbook operativo, taxonomías de ejemplo, plantillas de
  system docs y ledger, scripts `parse_candidates.sh` y `check_dupes.sh`).
- Estándar del usuario: memoria `source-quality-standards`. Notebooks ya construidos con este método:
  `brand-book-salonline-notebook`, `maxwell-economia-notebook`, `guru-ia-notebook`.
  *(Esas memorias no estaban disponibles en la sesión que generó este handoff; este documento recoge
  el estándar tal como figura en la skill.)*
