# 3. Alcance

## Sistema de Base de Conocimiento con OpenClaw

### 3.1 Alcance Funcional

#### 3.1.1 Funcionalidades Incluidas

```
┌─────────────────────────────────────────────────────────────────┐
│                    SISTEMA DE KNOWLEDGE BASE                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │   CARGA DE      │  │   GESTIÓN DE    │  │   CONSULTA DE   │ │
│  │   CONTENIDO     │  │   CONTENIDO     │  │   INFORMACIÓN   │ │
│  ├─────────────────┤  ├─────────────────┤  ├─────────────────┤ │
│  │ • Upload PDF    │  │ • Actualizar    │  │ • Buscar info   │ │
│  │ • Upload XLS    │  │ • Reemplazar    │  │ • Responder     │ │
│  │ • Conversión MD │  │ • Archivar      │  │ • Citar fuente  │ │
│  │ • Categorizar   │  │ • Restaurar     │  │ • Arbitrar      │ │
│  │ • Etiquetar     │  │ • Versionar     │  │                 │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │   GOBERNANZA    │  │   AUDITORÍA     │  │   ADMIN         │ │
│  ├─────────────────┤  ├─────────────────┤  ├─────────────────┤ │
│  │ • Control acceso│  │ • Changelog     │  │ • Listar docs   │ │
│  │ • Roles/permisos│  │ • Metadatos     │  │ • Ver estados   │ │
│  │ • Prioridades   │  │ • Historial     │  │ • Reportes      │ │
│  │ • Reglas        │  │ • Trazabilidad  │  │ • Comandos /kb  │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

### 3.2 Módulos del Sistema

#### Módulo 1: Carga de Contenido

| Funcionalidad | Descripción | Prioridad |
|---------------|-------------|-----------|
| Upload PDF via Telegram | Admin envía PDF, bot lo recibe | Alta |
| Upload XLS via Telegram | Admin envía XLS, bot lo recibe | Alta |
| Conversión PDF → Markdown | Extracción de texto y formateo | Alta |
| Conversión XLS → Markdown | Tablas a formato MD | Alta |
| Asignación de categoría | Selección interactiva | Alta |
| Asignación de tags | Input libre, separado por comas | Media |
| Validación de formato | Verificar que el archivo es válido | Alta |
| Preview antes de publicar | Mostrar extracto del contenido | Media |

#### Módulo 2: Gestión de Contenido

| Funcionalidad | Descripción | Prioridad |
|---------------|-------------|-----------|
| Actualizar documento | Subir nueva versión | Alta |
| Reemplazar documento | Deprecar viejo + crear nuevo | Alta |
| Archivar documento | Mover a _archive/, estado deprecado | Alta |
| Restaurar documento | Recuperar de _archive/ | Baja |
| Editar metadatos | Cambiar tags, prioridad, etc. | Media |
| Cambiar categoría | Mover entre carpetas | Media |

#### Módulo 3: Consulta de Información

| Funcionalidad | Descripción | Prioridad |
|---------------|-------------|-----------|
| Búsqueda en índice | Consultar INDEX.md | Alta |
| Lectura de documento | Cargar MD específico | Alta |
| Respuesta con contexto | Usar info de KB en respuesta | Alta |
| Citación de fuente | Mencionar documento origen | Alta |
| Manejo de ambigüedades | Aplicar reglas de GOVERNANCE.md | Alta |
| Respuesta sin info | Indicar que no hay datos | Alta |

#### Módulo 4: Gobernanza

| Funcionalidad | Descripción | Prioridad |
|---------------|-------------|-----------|
| Control de acceso | Validar ID de Telegram | Alta |
| Roles (superadmin/admin) | Diferentes permisos | Alta |
| Reglas de prioridad | Definir qué doc prevalece | Alta |
| Estados de documento | activo/borrador/deprecado/revision | Alta |

#### Módulo 5: Auditoría

| Funcionalidad | Descripción | Prioridad |
|---------------|-------------|-----------|
| Changelog automático | Log de cada operación | Alta |
| Metadatos de documento | Fecha, autor, versión | Alta |
| Historial de consultas | Qué docs se usan más | Baja |

#### Módulo 6: Administración

| Funcionalidad | Descripción | Prioridad |
|---------------|-------------|-----------|
| Comando /kb list | Listar documentos | Alta |
| Comando /kb status | Estado general de KB | Alta |
| Comando /kb search | Buscar en índice | Media |
| Comando /kb help | Ayuda de comandos | Alta |

---

### 3.3 Alcance Técnico

#### 3.3.1 Stack Tecnológico

| Componente | Tecnología |
|------------|------------|
| Plataforma | OpenClaw |
| Canal | Telegram Bot API |
| Almacenamiento | Sistema de archivos (workspace) |
| Formato de datos | Markdown, JSON |
| Conversión PDF | pdftotext (poppler-utils) |
| Conversión XLS | Python + pandas/openpyxl |
| Runtime | Node.js (OpenClaw) |

#### 3.3.2 Estructura de Archivos

```
workspace/
├── knowledge/
│   ├── INDEX.md
│   ├── GOVERNANCE.md
│   ├── CHANGELOG.md
│   ├── _meta/
│   │   └── documents.json
│   ├── _archive/
│   ├── [categoria-1]/
│   │   └── *.md
│   └── [categoria-n]/
│       └── *.md
├── scripts/
│   ├── convert-pdf.py
│   └── convert-xls.py
└── config/
    └── kb-admins.json
```

---

### 3.4 Exclusiones Explícitas

| Excluido | Razón |
|----------|-------|
| RAG con embeddings | Complejidad innecesaria para el volumen |
| Base de datos SQL/NoSQL | Mantener simplicidad |
| Interfaz web | Telegram es suficiente |
| OCR para PDFs escaneados | Requiere infraestructura adicional |
| Formatos Word/PPT | Fuera del requerimiento inicial |
| Multi-tenancy | Un workspace por cliente |
| Sincronización cloud | No requerido |
| Búsqueda full-text avanzada | Índice manual es suficiente |

---

### 3.5 Límites del Sistema

| Aspecto | Límite | Justificación |
|---------|--------|---------------|
| Documentos totales | 500 máx recomendado | Performance del índice |
| Tamaño por documento | 50 páginas máx | Contexto del LLM |
| Tamaño archivo PDF | 10 MB máx | Procesamiento |
| Tamaño archivo XLS | 5 MB máx | Procesamiento |
| Categorías | 20 máx recomendado | Navegabilidad |
| Admins | 10 máx | Gestión de permisos |
| Concurrencia | 10 usuarios | Especificación inicial |

---

### 3.6 Dependencias Externas

| Dependencia | Tipo | Crítica |
|-------------|------|---------|
| Telegram Bot API | Servicio | Sí |
| OpenClaw Gateway | Plataforma | Sí |
| poppler-utils | Librería | Sí |
| Python 3.x | Runtime | Sí |
| pandas | Librería Python | Sí |
| openpyxl | Librería Python | Sí |

---

### 3.7 Entregables

| Entregable | Descripción |
|------------|-------------|
| Estructura de carpetas | knowledge/ configurado |
| Scripts de conversión | convert-pdf.py, convert-xls.py |
| Archivos de configuración | kb-admins.json, GOVERNANCE.md |
| Documentación técnica | Este documento y relacionados |
| Casos de prueba | Suite de tests documentada |
| Guía de usuario | Manual para admins |

---

*Documento: 03-alcance.md | Versión: 1.0 | Fecha: 2026-03-11*
