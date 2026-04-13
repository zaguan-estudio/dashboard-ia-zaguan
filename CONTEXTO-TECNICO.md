# Contexto técnico — Dashboard IA Zaguan
> Documento para David · Responsable de desarrollo IA Claude Code  
> Fecha: 13 de abril de 2026

---

## 1. Descripción del proyecto

Panel de control ejecutivo para la dirección de **Zaguan Estudio** (estudio de diseño y tecnología). El objetivo es ofrecer una visión en tiempo real del estado del negocio, guiada por inteligencia artificial, para facilitar la toma de decisiones estratégicas.

El sistema emula el ecosistema de herramientas IA (inspirado en Claude): tiene un módulo de **Visión** (objetivos estratégicos que la dirección redacta en texto conversacional) y **Conectores** (fuentes de datos externas modeladas), que juntos alimentan un panel ejecutivo inteligente.

---

## 2. Stack técnico

| Capa | Tecnología |
|---|---|
| Frontend | HTML5 + CSS custom (variables) + JS vanilla |
| Estructura | Un único fichero `index.html` (~3.600 líneas) |
| Sin build | No hay bundler, frameworks ni dependencias externas |
| Fuentes | Sistema (sans-serif) — no se cargan Google Fonts |
| Iconos | SVG inline (sin librería externa) |
| Control de versiones | Git — rama activa: `claude/build-executive-dashboard-UUvdV` |

> El proyecto es completamente autocontenido. Solo necesitas abrir `index.html` en el navegador para verlo funcionar.

---

## 3. Estructura de archivos

```
dashboard-ia-zaguan/
├── index.html            ← TODO el proyecto (HTML + CSS + JS)
├── panel-zaguan-v1.html  ← Versión anterior / referencia
└── CONTEXTO-TECNICO.md   ← Este documento
```

---

## 4. Arquitectura interna del fichero index.html

```
<head>
  <style>
    ├── Variables CSS (design tokens)
    ├── Reset y base
    ├── Layout: sidebar nav + main content
    ├── Componentes globales (badges, toggles, toast, banners IA)
    └── Estilos de cada módulo (sección propia por módulo)
  </style>
</head>

<body>
  ├── .sidebar                ← Navegación lateral fija
  ├── .main-content           ← Área de contenido principal
  │   └── section.section     ← Una por módulo (7 total)
  └── <script>
        ├── navigate()        ← Cambia sección activa
        ├── showToast()       ← Notificaciones toast globales
        ├── IA_MODULES[]      ← Array con datos de los banners IA
        ├── renderAIBanner()  ← Renderiza banners IA por sección
        └── Funciones por módulo (objSwitchTab, objSelectItem…)
      </script>
</body>
```

---

## 5. Módulos del panel (secciones)

| # | ID sección | Módulo | Estado |
|---|---|---|---|
| 1 | `section-resumen` | Resumen ejecutivo | Completo |
| 2 | `section-presupuestos` | Presupuestos | Completo |
| 3 | `section-proyectos` | Proyectos | Completo |
| 4 | `section-equipo` | Equipo | Completo |
| 5 | `section-finanzas` | Finanzas | Completo |
| 6 | `section-alertas` | Alertas IA | Completo |
| 7 | `section-objetivos` | Objetivos | Completo |

La navegación entre módulos funciona con `navigate(el)` en JS. Cada `.nav-item` tiene `data-section` con el ID de la sección destino.

---

## 6. Sistema de design tokens (CSS variables)

Todas las variables están declaradas en `:root`. Las más usadas:

```css
/* Fondos */
--bg-primary        /* fondo del body */
--bg-surface        /* cards, paneles */
--bg-elevated       /* inputs, hover states */

/* Textos */
--text-primary      /* títulos y valores */
--text-secondary    /* subtítulos */
--text-tertiary     /* labels, muted */

/* Bordes */
--border            /* separadores */
--border-strong     /* énfasis */

/* Colores semánticos (cada uno tiene su variante -bg y -border) */
--green / --green-bg / --green-border
--amber / --amber-bg / --amber-border
--red   / --red-bg   / --red-border
--blue  / --blue-bg  / --blue-border
--purple / ...

/* Tipografía */
--font-xs: 11px
--font-sm: 12px
--font-base: 13px
--font-md: 14px
--font-lg: 16px
--font-xl: 20px
```

---

## 7. Módulo Objetivos — arquitectura detallada

Es el módulo más complejo. Inspirado en la UI de Claude (Skills + MCP Servers). Layout de 3 columnas:

### Col 1 — Sidebar de configuración
- Clase: `.obj-sidebar`
- Dos tabs: `data-tab="vision"` y `data-tab="conectores"`
- JS: `objSwitchTab(el)` — cambia qué lista se muestra en Col 2

### Col 2 — Lista de ítems
- Clase: `.obj-list-pane`
- **Lista Visión** (`#obj-list-vision`): 5 áreas estratégicas — Global, Financiero, Presupuestos, Proyectos, Equipo
- **Lista Conectores** (`#obj-list-conectores`): Asana, Holded, Airtable (conectados) + Slack, Gmail (no conectados)
- Cada `.obj-list-item` tiene `data-item="vision-global"` etc.
- JS: `objSelectItem(el)` — activa el panel de detalle correspondiente

### Col 3 — Panel de detalle
- Clase: `.obj-detail-pane`
- Cada panel tiene ID `obj-detail-{data-item}` — ej: `obj-detail-vision-global`
- Se muestra/oculta con clase `.active`
- **Paneles Visión**: toggle on/off + textarea editable + botón guardar → `objSaveVision(area)` → `showToast()`
- **Paneles Conector activo**: URL endpoint + dot de estado + chips de datos que alimenta + lista de herramientas disponibles
- **Paneles Conector inactivo**: aviso de estado + botón "Conectar" + chips de datos en gris

---

## 8. Banners IA (Zaguán IA)

Cada módulo tiene un banner de análisis IA. Se renderizan dinámicamente con:

```js
const IA_MODULES = [
  { section: 'resumen',      title: '...', body: '...', ... },
  { section: 'presupuestos', title: '...', ... },
  // ...
];

function renderAIBanner(sectionKey) { ... }
```

Los banners tienen:
- Título del insight generado por IA
- Cuerpo con el análisis
- Botón "Actualizar estado" (ancla bottom-right)
- Un aside con métricas numéricas

---

## 9. Sistema de alertas

Módulo `section-alertas` con grupos de prioridad:

```
Críticas (rojo)
Avisos    (ámbar)
Info      (azul)
```

Cada alerta tiene botón de dismiss. Cuando se descarta toda la fila del grupo, el grupo se oculta automáticamente. Las alertas críticas también aparecen en el widget de `section-resumen`.

---

## 10. Módulo Resumen — estructura

El resumen ejecutivo contiene:

- **Health Score global** — número grande + tendencia
- **5 widget cards** de módulos: Presupuestos, Proyectos, Equipo, Finanzas *(el widget de Objetivos fue eliminado)*
- Cada card: badge de estado (OK / Atención / Revisar), métricas clave, barras de progreso, tip IA
- **Banner Zaguán IA** — análisis ejecutivo del día
- **Tabla de alertas críticas** — con navegación directa al módulo

---

## 11. Navegación y estado

```js
function navigate(el) {
  // 1. Quita clase .active de todos los .nav-item y section.section
  // 2. Activa el nav-item clickado
  // 3. Muestra la sección con data-section coincidente
  // 4. Renderiza el banner IA correspondiente
}
```

El título del header superior (`#page-title`) se actualiza con `SECTION_TITLES[key]`.

---

## 12. Convenciones de nombrado CSS

| Prefijo | Módulo |
|---|---|
| `rsm-` | Resumen |
| `pres-` | Presupuestos |
| `proy-` | Proyectos |
| `eq-` | Equipo |
| `fin-` | Finanzas |
| `obj-` | Objetivos |
| `ai-` | Banners IA (globales) |
| `alert-` | Alertas |

---

## 13. Fuentes de datos modeladas (Conectores)

| Conector | Estado | Datos | Módulos |
|---|---|---|---|
| Asana | Conectado | Proyectos activos, tareas, fechas, carga de equipo | Proyectos, Equipo |
| Holded | Conectado | Facturación, gastos, cobros, presupuestos | Finanzas, Presupuestos |
| Airtable | Conectado | Pipeline comercial, leads, clientes | Presupuestos, Resumen |
| Slack | No conectado | Mensajes, alertas de proyecto | Equipo, Alertas |
| Gmail | No conectado | Respuestas clientes, aprobaciones | Presupuestos, Resumen |

> Los datos son actualmente estáticos (mock). La integración real requeriría conectar cada API en backend.

---

## 14. Cómo replicar el proyecto

```bash
# 1. Clonar el repositorio
git clone <url-del-repo>
cd dashboard-ia-zaguan

# 2. Cambiar a la rama de desarrollo
git checkout claude/build-executive-dashboard-UUvdV

# 3. Abrir en el navegador
open index.html
# o en Linux:
xdg-open index.html
```

No hay dependencias npm, ni servidor necesario. Funciona abriendo el fichero directamente.

---

## 15. Próximos pasos previstos

- [ ] Conectar APIs reales (Asana, Holded, Airtable) vía backend o edge functions
- [ ] Persistencia de la Visión estratégica (localStorage o base de datos)
- [ ] Sistema de autenticación (la dirección accede con credenciales)
- [ ] Exportación de informes PDF por módulo
- [ ] Modo oscuro / claro (las variables CSS ya están preparadas para ello)
- [ ] Notificaciones push cuando una alerta crítica se dispara

---

*Generado automáticamente por Claude Code · Sesión de desarrollo Zaguan IA*
