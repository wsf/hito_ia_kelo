# 4. Arquitectura

## Sistema de Base de Conocimiento con OpenClaw

### 4.1 Arquitectura General

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              ARQUITECTURA KB                                 │
└─────────────────────────────────────────────────────────────────────────────┘

    ┌─────────┐         ┌─────────┐         ┌─────────┐
    │  Admin  │         │ Usuario │         │  Admin  │
    │(Telegram)│        │(Telegram)│        │ (otro)  │
    └────┬────┘         └────┬────┘         └────┬────┘
         │                   │                   │
         └───────────────────┼───────────────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   Telegram API  │
                    └────────┬────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            OPENCLAW GATEWAY                                  │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         AGENTE (LLM)                                 │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────────┐ │   │
│  │  │   Router    │  │  KB Module  │  │  Converter  │  │  Governor  │ │   │
│  │  │  (intent)   │  │  (CRUD)     │  │  (PDF/XLS)  │  │  (rules)   │ │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └────────────┘ │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                      │                                      │
│                                      ▼                                      │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         WORKSPACE (Files)                            │   │
│  │                                                                      │   │
│  │   knowledge/          scripts/           config/         temp/       │   │
│  │   ├── INDEX.md        ├── convert-pdf.py ├── kb-admins.json         │   │
│  │   ├── GOVERNANCE.md   └── convert-xls.py └── settings.json          │   │
│  │   ├── CHANGELOG.md                                                   │   │
│  │   ├── _meta/                                                         │   │
│  │   │   └── documents.json                                             │   │
│  │   ├── _archive/                                                      │   │
│  │   └── [categorias]/                                                  │   │
│  │       └── *.md                                                       │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### 4.2 Componentes del Sistema

#### 4.2.1 Capa de Presentación (Telegram)

```
┌─────────────────────────────────────────┐
│           TELEGRAM INTERFACE            │
├─────────────────────────────────────────┤
│                                         │
│  Comandos Admin:          Archivos:     │
│  • /kb list               • PDF upload  │
│  • /kb status             • XLS upload  │
│  • /kb search             • Descarga    │
│  • /kb help                             │
│                                         │
│  Flujos interactivos:                   │
│  • Selección de categoría               │
│  • Input de tags                        │
│  • Confirmación de acciones             │
│                                         │
└─────────────────────────────────────────┘
```

#### 4.2.2 Capa de Lógica (OpenClaw Agent)

```
┌─────────────────────────────────────────────────────────────────┐
│                      MÓDULOS DEL AGENTE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │   ROUTER     │    │   AUTH       │    │   KB CRUD    │      │
│  │              │    │              │    │              │      │
│  │ • Detectar   │───▶│ • Validar ID │───▶│ • Create     │      │
│  │   intent     │    │ • Check rol  │    │ • Read       │      │
│  │ • Comando vs │    │ • Permisos   │    │ • Update     │      │
│  │   consulta   │    │              │    │ • Delete     │      │
│  └──────────────┘    └──────────────┘    └──────────────┘      │
│         │                                        │              │
│         │            ┌──────────────┐            │              │
│         │            │  CONVERTER   │            │              │
│         │            │              │            │              │
│         └───────────▶│ • PDF → MD   │◀───────────┘              │
│                      │ • XLS → MD   │                           │
│                      │ • Validate   │                           │
│                      └──────────────┘                           │
│                             │                                    │
│                             ▼                                    │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │  GOVERNOR    │    │   AUDITOR    │    │   INDEXER    │      │
│  │              │    │              │    │              │      │
│  │ • Prioridad  │    │ • Changelog  │    │ • Update     │      │
│  │ • Conflictos │    │ • Metadata   │    │   INDEX.md   │      │
│  │ • Estados    │    │ • History    │    │ • Sync meta  │      │
│  └──────────────┘    └──────────────┘    └──────────────┘      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### 4.2.3 Capa de Datos (Workspace)

```
┌─────────────────────────────────────────────────────────────────┐
│                     ESTRUCTURA DE DATOS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  knowledge/                                                      │
│  │                                                               │
│  ├── INDEX.md ─────────────────┐                                │
│  │   • Lista de documentos     │  Índice rápido                 │
│  │   • Categorías              │  para búsqueda                 │
│  │   • Descripciones breves    │                                │
│  │                             │                                │
│  ├── GOVERNANCE.md ────────────┤                                │
│  │   • Reglas de prioridad     │  Configuración                 │
│  │   • Estados válidos         │  de gobernanza                 │
│  │   • Resolución conflictos   │                                │
│  │                             │                                │
│  ├── CHANGELOG.md ─────────────┤                                │
│  │   • Historial de cambios    │  Auditoría                     │
│  │   • Quién, qué, cuándo      │                                │
│  │                             │                                │
│  ├── _meta/                    │                                │
│  │   └── documents.json ───────┤                                │
│  │       • Metadatos completos │  Base de datos                 │
│  │       • Versiones           │  en JSON                       │
│  │       • Tags, prioridades   │                                │
│  │                             │                                │
│  ├── _archive/ ────────────────┤                                │
│  │   └── *.md                  │  Documentos                    │
│  │       • Docs deprecados     │  archivados                    │
│  │                             │                                │
│  └── [categorias]/ ────────────┘                                │
│      └── *.md                     Documentos                    │
│          • Contenido real         activos                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

### 4.3 Flujos de Datos

#### 4.3.1 Flujo de Carga de Documento

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│  Admin  │    │Telegram │    │OpenClaw │    │Converter│    │Workspace│
└────┬────┘    └────┬────┘    └────┬────┘    └────┬────┘    └────┬────┘
     │              │              │              │              │
     │ Envía PDF    │              │              │              │
     │─────────────▶│              │              │              │
     │              │ Archivo      │              │              │
     │              │─────────────▶│              │              │
     │              │              │ Valida admin │              │
     │              │              │─────────────▶│              │
     │              │              │              │              │
     │              │              │ PDF → MD     │              │
     │              │              │─────────────▶│              │
     │              │              │              │ Convierte    │
     │              │              │◀─────────────│              │
     │              │              │              │              │
     │              │ ¿Categoría?  │              │              │
     │◀─────────────│◀─────────────│              │              │
     │              │              │              │              │
     │ "asanas"     │              │              │              │
     │─────────────▶│─────────────▶│              │              │
     │              │              │              │              │
     │              │              │ Guarda .md   │              │
     │              │              │─────────────────────────────▶│
     │              │              │              │              │
     │              │              │ Actualiza    │              │
     │              │              │ INDEX.md     │              │
     │              │              │─────────────────────────────▶│
     │              │              │              │              │
     │              │              │ Actualiza    │              │
     │              │              │ documents.json              │
     │              │              │─────────────────────────────▶│
     │              │              │              │              │
     │              │              │ Log en       │              │
     │              │              │ CHANGELOG.md │              │
     │              │              │─────────────────────────────▶│
     │              │              │              │              │
     │              │ ✅ Listo     │              │              │
     │◀─────────────│◀─────────────│              │              │
     │              │              │              │              │
```

#### 4.3.2 Flujo de Consulta

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│ Usuario │    │Telegram │    │OpenClaw │    │Workspace│
└────┬────┘    └────┬────┘    └────┬────┘    └────┬────┘
     │              │              │              │
     │ Pregunta     │              │              │
     │─────────────▶│─────────────▶│              │
     │              │              │              │
     │              │              │ Lee INDEX.md │
     │              │              │─────────────▶│
     │              │              │◀─────────────│
     │              │              │              │
     │              │              │ Identifica   │
     │              │              │ doc(s)       │
     │              │              │              │
     │              │              │ Lee doc.md   │
     │              │              │─────────────▶│
     │              │              │◀─────────────│
     │              │              │              │
     │              │              │ Si conflicto:│
     │              │              │ Lee GOVERNANCE│
     │              │              │─────────────▶│
     │              │              │◀─────────────│
     │              │              │              │
     │              │              │ Genera       │
     │              │              │ respuesta    │
     │              │              │              │
     │ Respuesta    │              │              │
     │◀─────────────│◀─────────────│              │
     │ + fuente     │              │              │
     │              │              │              │
```

---

### 4.4 Modelo de Datos

#### 4.4.1 documents.json Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "patternProperties": {
    "^[a-z0-9-/]+\\.md$": {
      "type": "object",
      "required": ["titulo", "version", "fecha_creacion", "estado"],
      "properties": {
        "titulo": {
          "type": "string",
          "description": "Título legible del documento"
        },
        "version": {
          "type": "string",
          "pattern": "^\\d+\\.\\d+$",
          "description": "Versión semántica (major.minor)"
        },
        "fecha_creacion": {
          "type": "string",
          "format": "date",
          "description": "Fecha ISO de creación"
        },
        "fecha_actualizacion": {
          "type": "string",
          "format": "date",
          "description": "Fecha ISO de última actualización"
        },
        "autor": {
          "type": "string",
          "description": "Nombre o ID del autor"
        },
        "estado": {
          "type": "string",
          "enum": ["activo", "borrador", "revision", "deprecado"],
          "description": "Estado actual del documento"
        },
        "prioridad": {
          "type": "integer",
          "minimum": 1,
          "maximum": 10,
          "description": "Prioridad para resolución de conflictos (1=más alta)"
        },
        "tags": {
          "type": "array",
          "items": {"type": "string"},
          "description": "Etiquetas para búsqueda"
        },
        "descripcion": {
          "type": "string",
          "description": "Descripción breve del contenido"
        },
        "archivo_original": {
          "type": "string",
          "description": "Nombre del archivo fuente (PDF/XLS)"
        },
        "notas": {
          "type": "string",
          "description": "Notas adicionales"
        }
      }
    }
  }
}
```

#### 4.4.2 Ejemplo documents.json

```json
{
  "asanas/posturas-de-pie.md": {
    "titulo": "Manual de Posturas de Pie",
    "version": "2.1",
    "fecha_creacion": "2026-01-15",
    "fecha_actualizacion": "2026-03-10",
    "autor": "Ana",
    "estado": "activo",
    "prioridad": 1,
    "tags": ["asanas", "principiantes", "de-pie", "alineación"],
    "descripcion": "Guía completa de asanas de pie con instrucciones de alineación",
    "archivo_original": "posturas-de-pie-v2.pdf",
    "notas": "Actualizado con correcciones de Virabhadrasana"
  },
  "admin/precios-2026.md": {
    "titulo": "Lista de Precios 2026",
    "version": "1.0",
    "fecha_creacion": "2026-03-01",
    "autor": "Ana",
    "estado": "activo",
    "prioridad": 1,
    "tags": ["precios", "admin", "clases"],
    "descripcion": "Precios actualizados de clases y paquetes",
    "archivo_original": "precios-marzo-2026.xlsx"
  }
}
```

#### 4.4.3 kb-admins.json Schema

```json
{
  "superadmins": [
    {
      "telegram_id": "1007231414",
      "nombre": "Ale",
      "permisos": ["*"]
    }
  ],
  "admins": [
    {
      "telegram_id": "7194252326",
      "nombre": "Ana",
      "permisos": ["create", "update", "archive", "list"]
    }
  ],
  "categorias_permitidas": {
    "7194252326": ["asanas", "pranayama", "admin"]
  }
}
```

---

### 4.5 Integraciones

```
┌─────────────────────────────────────────────────────────────────┐
│                      INTEGRACIONES                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐                       ┌─────────────┐          │
│  │  Telegram   │◀─────────────────────▶│  OpenClaw   │          │
│  │  Bot API    │   Mensajes, archivos  │  Gateway    │          │
│  └─────────────┘                       └──────┬──────┘          │
│                                               │                  │
│                                               │                  │
│  ┌─────────────┐                       ┌──────▼──────┐          │
│  │  pdftotext  │◀──────────────────────│   Agent     │          │
│  │ (poppler)   │   exec: conversión    │   Tools     │          │
│  └─────────────┘                       └──────┬──────┘          │
│                                               │                  │
│  ┌─────────────┐                              │                  │
│  │  Python     │◀─────────────────────────────┘                  │
│  │  scripts    │   exec: pandas, openpyxl                       │
│  └─────────────┘                                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

### 4.6 Consideraciones de Seguridad

| Aspecto | Implementación |
|---------|----------------|
| Autenticación | Telegram ID (nativo) |
| Autorización | Validación contra kb-admins.json |
| Validación de archivos | Verificar tipo MIME, tamaño |
| Sanitización | Limpiar nombres de archivo |
| Auditoría | Log de todas las operaciones |
| Aislamiento | Un workspace por agente |

---

### 4.7 Consideraciones de Performance

| Aspecto | Estrategia |
|---------|------------|
| Lectura de archivos | Índice pequeño, docs separados |
| Conversión | Asíncrona, una a la vez |
| Contexto LLM | Cargar solo docs necesarios |
| Concurrencia | Sin locks (archivos independientes) |
| Cache | No requerido (archivos son rápidos) |

---

*Documento: 04-arquitectura.md | Versión: 1.0 | Fecha: 2026-03-11*
