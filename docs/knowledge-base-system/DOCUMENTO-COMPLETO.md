# Sistema de Base de Conocimiento para OpenClaw

## 📋 Guía Completa de Implementación

Esta documentación describe cómo implementar un sistema de Base de Conocimiento (KB) integrado con OpenClaw, permitiendo a agentes de IA responder consultas basándose en documentación cargada por administradores.

---

## 📁 Índice

1. [Contexto](#1-contexto)
2. [Objetivos](#2-objetivos)
3. [Alcance](#3-alcance)
4. [Arquitectura](#4-arquitectura)
5. [Procesos](#5-procesos)
6. [Casos de Uso](#6-casos-de-uso)
7. [Tareas de Implementación](#7-tareas-de-implementación-con-openclaw)
8. [Casos de Prueba](#8-casos-de-prueba)

---

# 1. Contexto

## Sistema de Base de Conocimiento con OpenClaw

### 1.1 Descripción General

Este documento describe la implementación de un **Sistema de Base de Conocimiento (Knowledge Base - KB)** integrado con OpenClaw, que permite a agentes de IA acceder, gestionar y responder consultas basándose en documentación cargada por usuarios autorizados.

### 1.2 Problema a Resolver

Las organizaciones necesitan que sus asistentes de IA puedan:

- Responder preguntas basándose en documentación interna (manuales, guías, políticas)
- Mantener la información actualizada sin intervención técnica
- Garantizar consistencia en las respuestas
- Escalar sin requerir reentrenamiento del modelo

### 1.3 Solución Propuesta

Un sistema basado en archivos Markdown que:

1. **Almacena conocimiento** en archivos `.md` organizados por categorías
2. **Permite carga via Telegram** por usuarios autorizados
3. **Convierte automáticamente** PDFs y XLS a Markdown
4. **Mantiene gobernanza** con versionado, auditoría y control de acceso
5. **Resuelve ambigüedades** mediante reglas de prioridad configurables

### 1.4 Características del Entorno

| Aspecto | Especificación |
|---------|----------------|
| Volumen esperado | ~100 conversaciones/día |
| Concurrencia máxima | 10 usuarios simultáneos |
| Tamaño de documentos | ~5 páginas por documento |
| Cantidad de documentos | Variable (decenas a cientos) |
| Formatos de entrada | PDF, XLS/XLSX |
| Formato de almacenamiento | Markdown (.md) |
| Canal de interacción | Telegram (principal) |
| Plataforma | OpenClaw |

### 1.5 Stakeholders

| Rol | Responsabilidad |
|-----|-----------------|
| **Superadmin** | Configuración inicial, gestión de admins, resolución de conflictos |
| **Admin de contenido** | Carga, actualización y eliminación de documentos |
| **Usuario final** | Consulta de información via chat |
| **Desarrollador** | Implementación y mantenimiento del sistema |

### 1.6 Restricciones

- Sin base de datos externa (todo basado en archivos)
- Sin infraestructura adicional (solo OpenClaw + Telegram)
- Conversión de archivos debe ser automática
- Debe funcionar con la ventana de contexto del LLM (200K tokens)

### 1.7 Supuestos

- Los documentos fuente (PDF/XLS) están bien formateados
- Los admins tienen conocimiento básico de Telegram
- La información en los documentos no requiere actualización en tiempo real
- El volumen de documentos se mantendrá dentro de límites manejables (<500 docs)

---

# 2. Objetivos

## Sistema de Base de Conocimiento con OpenClaw

### 2.1 Objetivo General

Implementar un sistema de gestión de conocimiento que permita a agentes de OpenClaw responder consultas de usuarios basándose en documentación cargada y mantenida por administradores autorizados, sin requerir intervención técnica para actualizaciones de contenido.

---

### 2.2 Objetivos Específicos

#### 2.2.1 Gestión de Contenido

| ID | Objetivo | Métrica de Éxito |
|----|----------|------------------|
| O1 | Permitir carga de documentos PDF via Telegram | Admin puede cargar PDF en <2 minutos |
| O2 | Permitir carga de documentos XLS via Telegram | Admin puede cargar XLS en <2 minutos |
| O3 | Convertir automáticamente a formato Markdown | 100% de conversiones exitosas para docs estándar |
| O4 | Organizar documentos por categorías | Estructura de carpetas clara y navegable |
| O5 | Mantener índice actualizado automáticamente | INDEX.md siempre sincronizado |

#### 2.2.2 Gobernanza

| ID | Objetivo | Métrica de Éxito |
|----|----------|------------------|
| O6 | Controlar acceso por roles (superadmin/admin) | Solo autorizados pueden modificar KB |
| O7 | Versionar cambios en documentos | Historial completo en CHANGELOG.md |
| O8 | Registrar metadatos de cada documento | 100% de docs con metadata completa |
| O9 | Permitir deprecar/archivar documentos | Docs archivados no se usan en respuestas |
| O10 | Resolver ambigüedades con reglas claras | Reglas documentadas en GOVERNANCE.md |

#### 2.2.3 Consulta de Información

| ID | Objetivo | Métrica de Éxito |
|----|----------|------------------|
| O11 | Responder consultas usando la KB | >90% de respuestas usan KB cuando aplica |
| O12 | Identificar documento fuente correcto | Bot encuentra doc relevante en <3 seg |
| O13 | Citar fuente de la información | Respuestas incluyen referencia al doc |
| O14 | Manejar consultas sin info disponible | Bot indica claramente cuando no hay info |

#### 2.2.4 Operación

| ID | Objetivo | Métrica de Éxito |
|----|----------|------------------|
| O15 | Funcionar sin base de datos externa | Sistema 100% basado en archivos |
| O16 | Escalar a cientos de documentos | Performance estable con 500+ docs |
| O17 | Soportar concurrencia de 10 usuarios | Sin degradación con 10 simultáneos |
| O18 | Mantener logs de operación | Auditoría completa de acciones |

---

### 2.3 Objetivos No Funcionales

| Categoría | Objetivo | Especificación |
|-----------|----------|----------------|
| **Usabilidad** | Interfaz intuitiva | Admins no técnicos pueden operar |
| **Disponibilidad** | Alta disponibilidad | 99% uptime (dependiente de Telegram) |
| **Mantenibilidad** | Fácil de mantener | Documentación completa, código limpio |
| **Escalabilidad** | Crecimiento horizontal | Agregar categorías sin cambiar código |
| **Seguridad** | Control de acceso | Solo IDs autorizados pueden modificar |

---

### 2.4 Criterios de Aceptación Global

El sistema se considerará exitoso cuando:

- [ ] Un admin puede cargar un PDF y que esté disponible para consultas en <5 minutos
- [ ] Un admin puede actualizar/reemplazar un documento existente
- [ ] Un admin puede archivar un documento obsoleto
- [ ] El bot responde correctamente citando la fuente
- [ ] El bot maneja conflictos según reglas de prioridad
- [ ] Existe log completo de todas las operaciones
- [ ] La documentación permite a nuevos desarrolladores entender el sistema

---

### 2.5 Fuera de Alcance (Explícito)

Los siguientes elementos **NO** son objetivos de esta implementación:

- ❌ Búsqueda semántica con embeddings (RAG completo)
- ❌ Interfaz web para gestión
- ❌ Soporte para otros formatos (Word, PPT, etc.)
- ❌ Sincronización con sistemas externos
- ❌ Multi-idioma automático
- ❌ OCR para PDFs escaneados

---

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

# 5. Procesos

## Sistema de Base de Conocimiento con OpenClaw

### 5.1 Mapa de Procesos

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           MAPA DE PROCESOS KB                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  PROCESOS DE GESTIÓN                    PROCESOS DE OPERACIÓN               │
│  ┌─────────────────────┐                ┌─────────────────────┐             │
│  │ P1. Carga inicial   │                │ P6. Consulta simple │             │
│  │ P2. Actualización   │                │ P7. Consulta compleja│            │
│  │ P3. Sustitución     │                │ P8. Arbitraje       │             │
│  │ P4. Archivado       │                │                     │             │
│  │ P5. Restauración    │                │                     │             │
│  └─────────────────────┘                └─────────────────────┘             │
│                                                                              │
│  PROCESOS DE SOPORTE                    PROCESOS DE GOBERNANZA              │
│  ┌─────────────────────┐                ┌─────────────────────┐             │
│  │ P9. Conversión PDF  │                │ P12. Gestión admins │             │
│  │ P10. Conversión XLS │                │ P13. Auditoría      │             │
│  │ P11. Indexación     │                │ P14. Mantenimiento  │             │
│  └─────────────────────┘                └─────────────────────┘             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### 5.2 Procesos de Gestión

#### P1. Carga Inicial de Documento

**Trigger:** Admin envía archivo PDF/XLS por Telegram

**Actores:** Admin, Sistema

**Precondiciones:**
- Usuario está en lista de admins (kb-admins.json)
- Archivo es PDF o XLS válido
- Tamaño < límite permitido

**Flujo Principal:**

| # | Paso |
|---|------|
| 1 | Admin envía archivo por Telegram |
| 2 | Sistema valida que el usuario es admin |
| 3 | Sistema valida tipo y tamaño del archivo |
| 4 | Sistema descarga archivo a /temp |
| 5 | Sistema ejecuta conversión (PDF→MD o XLS→MD) |
| 6 | Sistema muestra preview del contenido convertido |
| 7 | Sistema pregunta categoría (lista de opciones) |
| 8 | Admin selecciona categoría |
| 9 | Sistema pregunta tags |
| 10 | Admin ingresa tags (o "ninguno") |
| 11 | Sistema muestra resumen y pide confirmación |
| 12 | Admin confirma |
| 13 | Sistema guarda archivo .md en knowledge/[categoria]/ |
| 14 | Sistema actualiza INDEX.md |
| 15 | Sistema actualiza documents.json con metadatos |
| 16 | Sistema registra en CHANGELOG.md |
| 17 | Sistema confirma éxito al admin |
| 18 | Sistema elimina archivo temporal |

**Flujos Alternativos:**

| Paso | Condición | Acción |
|------|-----------|--------|
| 2 | Usuario no es admin | Rechazar con mensaje "No autorizado" |
| 3 | Archivo inválido | Rechazar con mensaje de error específico |
| 5 | Conversión falla | Informar error, sugerir revisar formato |
| 12 | Admin cancela | Descartar todo, eliminar temporal |

**Postcondiciones:**
- Documento disponible en knowledge/
- INDEX.md actualizado
- documents.json actualizado
- CHANGELOG.md actualizado

---

#### P2. Actualización de Documento

**Trigger:** Admin quiere actualizar documento existente

**Flujo Principal:**

| # | Paso |
|---|------|
| 1 | Admin ejecuta /kb update o menciona actualizar |
| 2 | Sistema muestra lista de documentos actualizables |
| 3 | Admin selecciona documento |
| 4 | Sistema muestra info actual del documento |
| 5 | Sistema pide nuevo archivo |
| 6 | Admin envía nuevo archivo |
| 7 | Sistema convierte archivo |
| 8 | Sistema muestra diff o resumen de cambios |
| 9 | Sistema pide confirmación |
| 10 | Admin confirma |
| 11 | Sistema incrementa versión (X.Y → X.Y+1) |
| 12 | Sistema actualiza fecha_actualizacion |
| 13 | Sistema sobreescribe archivo .md |
| 14 | Sistema actualiza INDEX.md si cambió descripción |
| 15 | Sistema actualiza documents.json |
| 16 | Sistema registra en CHANGELOG.md |
| 17 | Sistema confirma éxito |

---

#### P3. Sustitución de Documento

**Trigger:** Admin quiere reemplazar documento por uno nuevo (ej: precios 2025 → 2026)

**Flujo Principal:**

| # | Paso |
|---|------|
| 1 | Admin ejecuta /kb replace o menciona reemplazar |
| 2 | Sistema muestra lista de documentos |
| 3 | Admin selecciona documento a reemplazar |
| 4 | Sistema pide nuevo archivo |
| 5 | Admin envía nuevo archivo |
| 6 | Sistema convierte archivo |
| 7 | Sistema pregunta si mantener nombre o crear nuevo |
| 8 | Admin decide |
| 9 | Sistema mueve documento viejo a _archive/ |
| 10 | Sistema cambia estado del viejo a "deprecado" |
| 11 | Sistema guarda documento nuevo |
| 12 | Sistema actualiza INDEX.md |
| 13 | Sistema actualiza documents.json (ambos docs) |
| 14 | Sistema registra sustitución en CHANGELOG.md |
| 15 | Sistema confirma éxito |

---

#### P4. Archivado de Documento

**Trigger:** Admin quiere desactivar documento obsoleto

**Flujo Principal:**

| # | Paso |
|---|------|
| 1 | Admin ejecuta /kb archive [documento] |
| 2 | Sistema muestra info del documento |
| 3 | Sistema pide confirmación |
| 4 | Admin confirma |
| 5 | Sistema mueve archivo a _archive/ |
| 6 | Sistema cambia estado a "deprecado" en documents.json |
| 7 | Sistema actualiza INDEX.md (remueve entrada) |
| 8 | Sistema registra en CHANGELOG.md |
| 9 | Sistema confirma éxito |

---

#### P5. Restauración de Documento

**Trigger:** Admin quiere recuperar documento archivado

**Flujo Principal:**

| # | Paso |
|---|------|
| 1 | Admin ejecuta /kb restore |
| 2 | Sistema lista documentos en _archive/ |
| 3 | Admin selecciona documento |
| 4 | Sistema pregunta categoría destino |
| 5 | Admin confirma categoría |
| 6 | Sistema mueve archivo de _archive/ a knowledge/[cat]/ |
| 7 | Sistema cambia estado a "activo" |
| 8 | Sistema actualiza INDEX.md |
| 9 | Sistema registra en CHANGELOG.md |
| 10 | Sistema confirma éxito |

---

### 5.3 Procesos de Operación

#### P6. Consulta Simple

**Trigger:** Usuario hace pregunta que requiere info de un documento

**Flujo Principal:**

| # | Paso |
|---|------|
| 1 | Usuario hace pregunta |
| 2 | Sistema lee INDEX.md |
| 3 | Sistema identifica documento(s) relevante(s) |
| 4 | Sistema lee documento específico |
| 5 | Sistema genera respuesta usando contenido |
| 6 | Sistema incluye referencia a fuente |
| 7 | Sistema envía respuesta al usuario |

---

#### P7. Consulta Compleja (múltiples documentos)

**Trigger:** Usuario hace pregunta que requiere info de varios documentos

**Flujo Principal:**

| # | Paso |
|---|------|
| 1 | Usuario hace pregunta |
| 2 | Sistema lee INDEX.md |
| 3 | Sistema identifica múltiples documentos relevantes |
| 4 | Sistema lee documents.json para verificar estados |
| 5 | Sistema filtra solo documentos con estado "activo" |
| 6 | Sistema lee cada documento necesario |
| 7 | Sistema sintetiza información de múltiples fuentes |
| 8 | Sistema incluye referencias a todas las fuentes |
| 9 | Sistema envía respuesta al usuario |

---

#### P8. Arbitraje de Información Conflictiva

**Trigger:** Múltiples documentos tienen información contradictoria

**Flujo Principal:**

| # | Paso |
|---|------|
| 1 | Sistema detecta información conflictiva entre documentos |
| 2 | Sistema lee GOVERNANCE.md para obtener reglas |
| 3 | Sistema lee documents.json para metadata de cada doc |
| 4 | Sistema aplica reglas de prioridad: |
| | a. Documento con estado "oficial" prevalece |
| | b. Si empate: documento con mayor prioridad numérica |
| | c. Si empate: documento más reciente |
| 5 | Sistema genera respuesta usando doc ganador |
| 6 | Sistema indica que había múltiples fuentes |
| 7 | Sistema cita documento utilizado y su fecha |

**Reglas de Prioridad (GOVERNANCE.md):**

```markdown
## Reglas de Arbitraje

1. Estado "oficial" > otros estados
2. prioridad numérica menor = más importante (1 > 2 > 3)
3. fecha_actualizacion más reciente gana empates
4. Si es info crítica (precios, horarios): siempre usar más reciente
5. Si no hay forma de decidir: informar al usuario y sugerir contactar admin
```

---

### 5.4 Procesos de Soporte

#### P9. Conversión PDF a Markdown

**Trigger:** Sistema necesita convertir PDF cargado

| # | Paso |
|---|------|
| 1 | Recibir ruta de archivo PDF |
| 2 | Ejecutar: pdftotext -layout input.pdf output.txt |
| 3 | Leer contenido de output.txt |
| 4 | Aplicar formateo Markdown: |
| | - Detectar títulos (líneas en mayúsculas o cortas) |
| | - Detectar listas (líneas con - o •) |
| | - Preservar tablas si es posible |
| 5 | Agregar header con metadata: |
| | - Título |
| | - Fecha de conversión |
| | - Archivo fuente |
| 6 | Retornar contenido Markdown |

**Script: scripts/convert-pdf.py**

```python
#!/usr/bin/env python3
import subprocess
import sys
import re
from datetime import datetime

def convert_pdf_to_markdown(pdf_path, output_path=None):
    # Extraer texto
    result = subprocess.run(
        ['pdftotext', '-layout', pdf_path, '-'],
        capture_output=True, text=True
    )
    
    if result.returncode != 0:
        raise Exception(f"Error en pdftotext: {result.stderr}")
    
    text = result.stdout
    
    # Formatear como Markdown
    lines = text.split('\n')
    md_lines = []
    
    # Header
    md_lines.append(f"<!-- Convertido de: {pdf_path} -->")
    md_lines.append(f"<!-- Fecha: {datetime.now().isoformat()} -->")
    md_lines.append("")
    
    for line in lines:
        # Detectar posibles títulos
        stripped = line.strip()
        if stripped.isupper() and len(stripped) > 3 and len(stripped) < 60:
            md_lines.append(f"## {stripped.title()}")
        elif stripped.startswith('•') or stripped.startswith('-'):
            md_lines.append(f"- {stripped[1:].strip()}")
        else:
            md_lines.append(line)
    
    content = '\n'.join(md_lines)
    
    if output_path:
        with open(output_path, 'w') as f:
            f.write(content)
    
    return content

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Uso: convert-pdf.py <archivo.pdf> [salida.md]")
        sys.exit(1)
    
    pdf_path = sys.argv[1]
    output_path = sys.argv[2] if len(sys.argv) > 2 else None
    
    result = convert_pdf_to_markdown(pdf_path, output_path)
    if not output_path:
        print(result)
```

---

#### P10. Conversión XLS a Markdown

**Script: scripts/convert-xls.py**

```python
#!/usr/bin/env python3
import pandas as pd
import sys
from datetime import datetime

def convert_xls_to_markdown(xls_path, output_path=None):
    # Leer todas las hojas
    xlsx = pd.ExcelFile(xls_path)
    
    md_lines = []
    md_lines.append(f"<!-- Convertido de: {xls_path} -->")
    md_lines.append(f"<!-- Fecha: {datetime.now().isoformat()} -->")
    md_lines.append("")
    
    for sheet_name in xlsx.sheet_names:
        df = pd.read_excel(xlsx, sheet_name=sheet_name)
        
        md_lines.append(f"## {sheet_name}")
        md_lines.append("")
        
        # Convertir DataFrame a tabla Markdown
        md_lines.append(df.to_markdown(index=False))
        md_lines.append("")
    
    content = '\n'.join(md_lines)
    
    if output_path:
        with open(output_path, 'w') as f:
            f.write(content)
    
    return content

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Uso: convert-xls.py <archivo.xlsx> [salida.md]")
        sys.exit(1)
    
    xls_path = sys.argv[1]
    output_path = sys.argv[2] if len(sys.argv) > 2 else None
    
    result = convert_xls_to_markdown(xls_path, output_path)
    if not output_path:
        print(result)
```

---

#### P11. Indexación Automática

**Trigger:** Se agrega, actualiza o elimina un documento

| # | Paso |
|---|------|
| 1 | Recibir evento (create/update/delete) |
| 2 | Leer INDEX.md actual |
| 3 | Leer documents.json actual |
| 4 | Según evento: |
| | - CREATE: agregar entrada al índice |
| | - UPDATE: actualizar descripción si cambió |
| | - DELETE: remover entrada del índice |
| 5 | Regenerar INDEX.md ordenado por categoría |
| 6 | Guardar INDEX.md |

---

### 5.5 Procesos de Gobernanza

#### P12. Gestión de Admins

**Solo superadmin puede ejecutar**

```
Agregar admin:
/kb admin add <telegram_id> <nombre>

Remover admin:
/kb admin remove <telegram_id>

Listar admins:
/kb admin list
```

---

#### P13. Auditoría

**Registro automático en CHANGELOG.md:**

```markdown
## 2026-03-11

### 14:30 UTC
- **[NUEVO]** asanas/inversiones.md v1.0
  - Autor: Ana (7194252326)
  - Fuente: inversiones-completo.pdf
  - Tags: inversiones, avanzado

### 10:15 UTC
- **[ACTUALIZADO]** admin/precios.md v1.0 → v1.1
  - Autor: Ana (7194252326)
  - Cambios: Actualización de precios de paquetes

## 2026-03-10

### 18:00 UTC
- **[ARCHIVADO]** admin/precios-2025.md
  - Autor: Ale (1007231414)
  - Razón: Reemplazado por precios-2026.md
```

---

#### P14. Mantenimiento Periódico

**Tareas sugeridas (manual o via cron):**

| Frecuencia | Tarea |
|------------|-------|
| Diaria | Verificar integridad de INDEX.md vs archivos reales |
| Semanal | Revisar documentos en estado "borrador" |
| Mensual | Revisar documentos sin actualizaciones en 6+ meses |
| Trimestral | Limpiar _archive/ de docs muy antiguos |

---

# 6. Casos de Uso

## Sistema de Base de Conocimiento con OpenClaw

### 6.1 Diagrama General de Casos de Uso

```
                                    ┌─────────────────────────────┐
                                    │    SISTEMA KB OpenClaw      │
                                    └─────────────────────────────┘
                                                  │
        ┌─────────────────────────────────────────┼─────────────────────────────────────────┐
        │                                         │                                         │
        ▼                                         ▼                                         ▼
┌───────────────┐                       ┌───────────────┐                         ┌───────────────┐
│   SUPERADMIN  │                       │     ADMIN     │                         │   USUARIO     │
└───────┬───────┘                       └───────┬───────┘                         └───────┬───────┘
        │                                       │                                         │
        │ UC01: Gestionar admins                │ UC03: Cargar documento                  │ UC08: Consultar info
        │ UC02: Configurar reglas               │ UC04: Actualizar documento              │ UC09: Pedir aclaración
        │ UC03-07: (todos los de admin)         │ UC05: Reemplazar documento              │
        │                                       │ UC06: Archivar documento                │
        │                                       │ UC07: Listar/buscar documentos          │
        │                                       │                                         │
        └───────────────────────────────────────┴─────────────────────────────────────────┘
```

---

### 6.2 Casos de Uso Detallados

#### UC01: Gestionar Administradores

| Campo | Descripción |
|-------|-------------|
| **ID** | UC01 |
| **Nombre** | Gestionar Administradores |
| **Actor Principal** | Superadmin |
| **Descripción** | Permite agregar, remover y listar administradores del sistema |
| **Precondiciones** | El usuario tiene rol de superadmin |
| **Trigger** | Superadmin ejecuta comando /kb admin |

**Flujo Principal:**

```
1. Superadmin ejecuta /kb admin [acción]
2. Sistema valida rol de superadmin
3. Según acción:
   
   [add]
   3a. Sistema solicita Telegram ID
   3b. Sistema solicita nombre
   3c. Sistema agrega a kb-admins.json
   3d. Sistema confirma: "Admin [nombre] agregado"
   
   [remove]
   3a. Sistema muestra lista de admins
   3b. Superadmin selecciona admin
   3c. Sistema solicita confirmación
   3d. Sistema remueve de kb-admins.json
   3e. Sistema confirma: "Admin [nombre] removido"
   
   [list]
   3a. Sistema muestra lista de admins con roles
```

**Flujos Alternativos:**

| Paso | Condición | Acción |
|------|-----------|--------|
| 2 | No es superadmin | "No autorizado para gestionar admins" |
| 3b | ID ya existe | "Este usuario ya es admin" |
| 3b | ID inválido | "ID de Telegram inválido" |

---

#### UC02: Configurar Reglas de Gobernanza

| Campo | Descripción |
|-------|-------------|
| **ID** | UC02 |
| **Nombre** | Configurar Reglas de Gobernanza |
| **Actor Principal** | Superadmin |
| **Descripción** | Permite modificar reglas de prioridad y arbitraje |
| **Precondiciones** | El usuario tiene rol de superadmin |
| **Trigger** | Superadmin ejecuta /kb governance |

---

#### UC03: Cargar Documento

| Campo | Descripción |
|-------|-------------|
| **ID** | UC03 |
| **Nombre** | Cargar Documento |
| **Actor Principal** | Admin |
| **Descripción** | Permite subir un PDF o XLS y convertirlo a documento de KB |
| **Precondiciones** | El usuario tiene rol de admin |
| **Trigger** | Admin envía archivo por Telegram |

**Flujo Principal:**

```
1. Admin envía archivo PDF o XLS por Telegram
2. Sistema detecta archivo
3. Sistema valida que el usuario es admin
4. Sistema valida tipo de archivo (PDF/XLS)
5. Sistema valida tamaño (< 10MB PDF, < 5MB XLS)
6. Sistema descarga archivo a /temp
7. Sistema ejecuta conversión:
   - PDF: pdftotext → formato MD
   - XLS: pandas → tablas MD
8. Sistema muestra preview (primeras 20 líneas)
9. Sistema pregunta: "¿En qué categoría va este documento?"
   y muestra opciones:
   1. Asanas
   2. Pranayama
   3. Anatomía
   4. Admin
   5. Otra (especificar)
10. Admin selecciona opción
11. Sistema pregunta: "¿Tags? (separados por coma, o 'ninguno')"
12. Admin ingresa tags
13. Sistema genera nombre de archivo (slug del título)
14. Sistema muestra resumen y pide confirmación
15. Admin confirma
16. Sistema guarda archivo en knowledge/[categoria]/[nombre].md
17. Sistema actualiza INDEX.md
18. Sistema actualiza documents.json
19. Sistema registra en CHANGELOG.md
20. Sistema confirma: "✅ Documento cargado exitosamente"
```

**Ejemplo de Interacción:**

```
Admin: [envía "manual-inversiones.pdf"]

Bot: 📄 Recibí "manual-inversiones.pdf"
     Procesando... ✅
     
     Preview:
     ─────────────────
     # Inversiones en Yoga
     
     Las inversiones son posturas donde 
     la cabeza queda por debajo del corazón...
     ─────────────────
     
     ¿En qué categoría va?
     1. Asanas
     2. Pranayama
     3. Anatomía
     4. Admin
     5. Otra

Admin: 1

Bot: ¿Tags? (separados por coma, o "ninguno")

Admin: inversiones, avanzado, fuerza

Bot: 📋 Resumen:
     - Archivo: manual-inversiones.md
     - Categoría: asanas/
     - Tags: inversiones, avanzado, fuerza
     - Estado: activo
     
     ¿Confirmar? (Sí/No)

Admin: Sí

Bot: ✅ ¡Listo!
     📁 knowledge/asanas/manual-inversiones.md
     📝 Agregado al INDEX.md
     📊 Registrado en documents.json
```

---

#### UC04: Actualizar Documento

| Campo | Descripción |
|-------|-------------|
| **ID** | UC04 |
| **Nombre** | Actualizar Documento |
| **Actor Principal** | Admin |
| **Descripción** | Permite actualizar contenido de documento existente |
| **Precondiciones** | Documento existe y está activo |
| **Trigger** | Admin ejecuta /kb update |

---

#### UC05: Reemplazar Documento

| Campo | Descripción |
|-------|-------------|
| **ID** | UC05 |
| **Nombre** | Reemplazar Documento |
| **Actor Principal** | Admin |
| **Descripción** | Archiva documento existente y lo reemplaza por uno nuevo |
| **Precondiciones** | Documento existente está activo |
| **Trigger** | Admin ejecuta /kb replace |

---

#### UC06: Archivar Documento

| Campo | Descripción |
|-------|-------------|
| **ID** | UC06 |
| **Nombre** | Archivar Documento |
| **Actor Principal** | Admin |
| **Descripción** | Mueve documento a archivo, ya no se usa en respuestas |
| **Precondiciones** | Documento existe y está activo |
| **Trigger** | Admin ejecuta /kb archive |

---

#### UC07: Listar y Buscar Documentos

| Campo | Descripción |
|-------|-------------|
| **ID** | UC07 |
| **Nombre** | Listar y Buscar Documentos |
| **Actor Principal** | Admin |
| **Descripción** | Permite ver documentos disponibles y buscar por criterios |
| **Trigger** | Admin ejecuta /kb list o /kb search |

**Flujo Principal - Listar:**

```
1. Admin ejecuta /kb list
2. Sistema muestra:

   📚 Base de Conocimiento
   
   asanas/ (3 documentos)
   ├── posturas-de-pie.md (v2.0) ✅
   ├── inversiones.md (v1.0) ✅
   └── secuencias-basicas.md (v1.2) ✅
   
   pranayama/ (1 documento)
   └── respiracion-ujjayi.md (v1.0) ✅
   
   admin/ (2 documentos)
   ├── precios.md (v1.1) ✅
   └── horarios.md (v1.0) ✅
   
   📁 Archivados: 2 documentos
```

---

#### UC08: Consultar Información

| Campo | Descripción |
|-------|-------------|
| **ID** | UC08 |
| **Nombre** | Consultar Información |
| **Actor Principal** | Usuario Final |
| **Descripción** | Usuario hace pregunta, sistema responde usando KB |
| **Trigger** | Usuario envía mensaje con pregunta |

**Flujo Principal:**

```
1. Usuario pregunta: "¿Cuáles son los beneficios de sirsasana?"
2. Sistema lee INDEX.md
3. Sistema identifica documento relevante: asanas/inversiones.md
4. Sistema lee documents.json para verificar estado (activo)
5. Sistema lee asanas/inversiones.md
6. Sistema genera respuesta usando contenido
7. Sistema responde:

   Los beneficios de Sirsasana (postura sobre la cabeza) incluyen:
   
   - Mejora la circulación cerebral
   - Fortalece hombros y brazos
   - Estimula la glándula pineal
   - Calma el sistema nervioso
   
   📖 Fuente: Manual de Inversiones
```

---

#### UC09: Resolver Información Conflictiva

| Campo | Descripción |
|-------|-------------|
| **ID** | UC09 |
| **Nombre** | Resolver Información Conflictiva |
| **Actor Principal** | Sistema (automático) |
| **Descripción** | Sistema resuelve conflictos entre documentos |
| **Trigger** | Consulta encuentra información contradictoria |

---

### 6.3 Matriz de Casos de Uso vs Roles

| Caso de Uso | Superadmin | Admin | Usuario |
|-------------|:----------:|:-----:|:-------:|
| UC01: Gestionar admins | ✅ | ❌ | ❌ |
| UC02: Configurar reglas | ✅ | ❌ | ❌ |
| UC03: Cargar documento | ✅ | ✅ | ❌ |
| UC04: Actualizar documento | ✅ | ✅ | ❌ |
| UC05: Reemplazar documento | ✅ | ✅ | ❌ |
| UC06: Archivar documento | ✅ | ✅ | ❌ |
| UC07: Listar/buscar docs | ✅ | ✅ | ❌ |
| UC08: Consultar información | ✅ | ✅ | ✅ |
| UC09: Resolver conflictos | Auto | Auto | Auto |

---

# 7. Tareas de Implementación con OpenClaw

## Sistema de Base de Conocimiento con OpenClaw

### 7.1 Resumen de Tareas

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ROADMAP DE IMPLEMENTACIÓN                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  FASE 1: Setup Base (2-3 días)                                              │
│  ├── T01: Estructura de carpetas                                            │
│  ├── T02: Archivos de configuración                                         │
│  └── T03: Scripts de conversión                                             │
│                                                                              │
│  FASE 2: Lógica del Agente (3-4 días)                                       │
│  ├── T04: Instrucciones en AGENTS.md                                        │
│  ├── T05: Comandos /kb                                                      │
│  └── T06: Flujo de carga de documentos                                      │
│                                                                              │
│  FASE 3: Gobernanza (2 días)                                                │
│  ├── T07: Sistema de roles                                                  │
│  ├── T08: Auditoría automática                                              │
│  └── T09: Reglas de arbitraje                                               │
│                                                                              │
│  FASE 4: Testing y Documentación (2 días)                                   │
│  ├── T10: Casos de prueba                                                   │
│  └── T11: Guía de usuario                                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### 7.2 Fase 1: Setup Base

#### T01: Crear Estructura de Carpetas

**Objetivo:** Establecer la estructura de archivos del sistema KB

**Acciones:**

```bash
# Ejecutar en el workspace del agente
mkdir -p knowledge/_meta
mkdir -p knowledge/_archive
mkdir -p knowledge/asanas
mkdir -p knowledge/pranayama
mkdir -p knowledge/anatomia
mkdir -p knowledge/admin
mkdir -p scripts
mkdir -p config
mkdir -p temp
```

**Archivos a crear:**

```
workspace/
├── knowledge/
│   ├── INDEX.md
│   ├── GOVERNANCE.md
│   ├── CHANGELOG.md
│   ├── _meta/
│   │   └── documents.json
│   ├── _archive/
│   ├── asanas/
│   ├── pranayama/
│   ├── anatomia/
│   └── admin/
├── scripts/
│   ├── convert-pdf.py
│   └── convert-xls.py
├── config/
│   └── kb-admins.json
└── temp/
```

---

#### T02: Crear Archivos de Configuración

**Archivo: knowledge/INDEX.md**

```markdown
# Índice de Base de Conocimiento

> Última actualización: [fecha]

## Categorías

### 📁 asanas/
*Documentos sobre posturas de yoga*

| Documento | Descripción | Tags |
|-----------|-------------|------|
| (vacío) | | |

### 📁 pranayama/
*Documentos sobre técnicas de respiración*

| Documento | Descripción | Tags |
|-----------|-------------|------|
| (vacío) | | |

### 📁 anatomia/
*Documentos sobre anatomía aplicada al yoga*

| Documento | Descripción | Tags |
|-----------|-------------|------|
| (vacío) | | |

### 📁 admin/
*Documentos administrativos (precios, horarios, políticas)*

| Documento | Descripción | Tags |
|-----------|-------------|------|
| (vacío) | | |

---

## Estadísticas
- Total documentos: 0
- Documentos activos: 0
- Documentos archivados: 0
```

**Archivo: knowledge/GOVERNANCE.md**

```markdown
# Gobernanza de Base de Conocimiento

## 1. Roles y Permisos

### Superadmin
- Gestionar administradores
- Configurar reglas de gobernanza
- Todas las acciones de Admin

### Admin
- Cargar documentos
- Actualizar documentos
- Archivar documentos
- Listar y buscar documentos

### Usuario
- Consultar información

## 2. Estados de Documentos

| Estado | Descripción | Se usa en respuestas |
|--------|-------------|---------------------|
| `activo` | Documento vigente | ✅ Sí |
| `borrador` | En preparación | ❌ No |
| `revision` | Pendiente de aprobación | ❌ No |
| `deprecado` | Archivado/obsoleto | ❌ No |

## 3. Reglas de Prioridad

Cuando hay información conflictiva entre documentos:

1. **Estado "oficial"** prevalece sobre otros estados
2. **Prioridad numérica** menor = más importante (1 > 2 > 3)
3. **Fecha más reciente** gana en caso de empate
4. **Información crítica** (precios, horarios): siempre usar el más reciente

## 4. Categorías Válidas

- `asanas` - Posturas de yoga
- `pranayama` - Técnicas de respiración
- `anatomia` - Anatomía aplicada
- `admin` - Información administrativa

## 5. Convenciones de Nombres

- Archivos: `nombre-descriptivo.md` (kebab-case)
- Sin espacios, sin caracteres especiales
- Máximo 50 caracteres

## 6. Retención

- Documentos archivados se mantienen 1 año
- Después de 1 año, se pueden eliminar permanentemente
```

**Archivo: knowledge/CHANGELOG.md**

```markdown
# Changelog - Base de Conocimiento

> Registro de todos los cambios en la base de conocimiento.

---

## [Fecha de inicio]

### Inicialización
- Sistema KB inicializado
- Estructura de carpetas creada
- Archivos de configuración establecidos

---

<!-- Nuevas entradas se agregan arriba de esta línea -->
```

**Archivo: knowledge/_meta/documents.json**

```json
{
  "_metadata": {
    "version": "1.0",
    "created": "2026-03-11",
    "description": "Registro de metadatos de todos los documentos"
  },
  "documents": {}
}
```

**Archivo: config/kb-admins.json**

```json
{
  "_metadata": {
    "version": "1.0",
    "description": "Lista de administradores autorizados"
  },
  "superadmins": [
    {
      "telegram_id": "1007231414",
      "nombre": "Ale",
      "agregado": "2026-03-11"
    }
  ],
  "admins": [
    {
      "telegram_id": "7194252326",
      "nombre": "Ana",
      "agregado": "2026-03-11",
      "categorias_permitidas": ["asanas", "pranayama", "anatomia", "admin"]
    }
  ]
}
```

---

#### T03: Crear Scripts de Conversión

**Archivo: scripts/convert-pdf.py**

```python
#!/usr/bin/env python3
"""
Convierte PDF a Markdown para el sistema KB de OpenClaw.
Uso: python convert-pdf.py <archivo.pdf> [salida.md]
"""

import subprocess
import sys
import re
import os
from datetime import datetime

def extract_text_from_pdf(pdf_path):
    """Extrae texto del PDF usando pdftotext."""
    try:
        result = subprocess.run(
            ['pdftotext', '-layout', pdf_path, '-'],
            capture_output=True,
            text=True,
            timeout=60
        )
        
        if result.returncode != 0:
            raise Exception(f"pdftotext error: {result.stderr}")
        
        return result.stdout
    
    except FileNotFoundError:
        raise Exception("pdftotext no encontrado. Instalar: apt install poppler-utils")
    except subprocess.TimeoutExpired:
        raise Exception("Timeout al procesar PDF")

def format_as_markdown(text, source_file):
    """Convierte texto plano a Markdown con formato básico."""
    lines = text.split('\n')
    md_lines = []
    
    # Header con metadata
    md_lines.append("---")
    md_lines.append(f"fuente: {os.path.basename(source_file)}")
    md_lines.append(f"convertido: {datetime.now().strftime('%Y-%m-%d %H:%M')}")
    md_lines.append("---")
    md_lines.append("")
    
    in_list = False
    
    for line in lines:
        stripped = line.strip()
        
        # Saltar líneas vacías múltiples
        if not stripped:
            if md_lines and md_lines[-1] != "":
                md_lines.append("")
            continue
        
        # Detectar posibles títulos (líneas cortas en mayúsculas)
        if (stripped.isupper() and 
            3 < len(stripped) < 60 and 
            not any(c.isdigit() for c in stripped[:3])):
            md_lines.append(f"## {stripped.title()}")
            continue
        
        # Detectar subtítulos (terminan en :)
        if stripped.endswith(':') and len(stripped) < 50:
            md_lines.append(f"### {stripped}")
            continue
        
        # Detectar listas
        if stripped.startswith(('•', '-', '*', '·')):
            md_lines.append(f"- {stripped[1:].strip()}")
            in_list = True
            continue
        
        # Detectar listas numeradas
        if re.match(r'^\d+[\.\)]\s', stripped):
            md_lines.append(stripped)
            in_list = True
            continue
        
        # Línea normal
        if in_list and not stripped.startswith((' ', '\t')):
            in_list = False
        
        md_lines.append(stripped)
    
    return '\n'.join(md_lines)

def convert_pdf_to_markdown(pdf_path, output_path=None):
    """Función principal de conversión."""
    if not os.path.exists(pdf_path):
        raise FileNotFoundError(f"Archivo no encontrado: {pdf_path}")
    
    # Extraer texto
    text = extract_text_from_pdf(pdf_path)
    
    if not text.strip():
        raise Exception("El PDF no contiene texto extraíble (¿es escaneado?)")
    
    # Formatear como Markdown
    markdown = format_as_markdown(text, pdf_path)
    
    # Guardar si se especificó salida
    if output_path:
        with open(output_path, 'w', encoding='utf-8') as f:
            f.write(markdown)
        return f"Guardado en: {output_path}"
    
    return markdown

def main():
    if len(sys.argv) < 2:
        print("Uso: python convert-pdf.py <archivo.pdf> [salida.md]")
        print("Si no se especifica salida, imprime a stdout")
        sys.exit(1)
    
    pdf_path = sys.argv[1]
    output_path = sys.argv[2] if len(sys.argv) > 2 else None
    
    try:
        result = convert_pdf_to_markdown(pdf_path, output_path)
        print(result)
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)

if __name__ == "__main__":
    main()
```

**Archivo: scripts/convert-xls.py**

```python
#!/usr/bin/env python3
"""
Convierte XLS/XLSX a Markdown para el sistema KB de OpenClaw.
Uso: python convert-xls.py <archivo.xlsx> [salida.md]
"""

import sys
import os
from datetime import datetime

try:
    import pandas as pd
except ImportError:
    print("Error: pandas no instalado. Ejecutar: pip install pandas openpyxl")
    sys.exit(1)

def convert_xls_to_markdown(xls_path, output_path=None):
    """Convierte archivo Excel a Markdown."""
    if not os.path.exists(xls_path):
        raise FileNotFoundError(f"Archivo no encontrado: {xls_path}")
    
    # Leer archivo
    try:
        xlsx = pd.ExcelFile(xls_path)
    except Exception as e:
        raise Exception(f"Error al leer archivo: {e}")
    
    md_lines = []
    
    # Header con metadata
    md_lines.append("---")
    md_lines.append(f"fuente: {os.path.basename(xls_path)}")
    md_lines.append(f"convertido: {datetime.now().strftime('%Y-%m-%d %H:%M')}")
    md_lines.append(f"hojas: {len(xlsx.sheet_names)}")
    md_lines.append("---")
    md_lines.append("")
    
    # Procesar cada hoja
    for sheet_name in xlsx.sheet_names:
        df = pd.read_excel(xlsx, sheet_name=sheet_name)
        
        # Título de la hoja
        md_lines.append(f"## {sheet_name}")
        md_lines.append("")
        
        # Limpiar datos
        df = df.fillna('')
        
        # Convertir a tabla Markdown
        if not df.empty:
            # Headers
            headers = '| ' + ' | '.join(str(col) for col in df.columns) + ' |'
            separator = '|' + '|'.join(['---' for _ in df.columns]) + '|'
            
            md_lines.append(headers)
            md_lines.append(separator)
            
            # Filas
            for _, row in df.iterrows():
                row_str = '| ' + ' | '.join(str(val) for val in row.values) + ' |'
                md_lines.append(row_str)
        else:
            md_lines.append("*(Hoja vacía)*")
        
        md_lines.append("")
    
    content = '\n'.join(md_lines)
    
    # Guardar si se especificó salida
    if output_path:
        with open(output_path, 'w', encoding='utf-8') as f:
            f.write(content)
        return f"Guardado en: {output_path}"
    
    return content

def main():
    if len(sys.argv) < 2:
        print("Uso: python convert-xls.py <archivo.xlsx> [salida.md]")
        print("Si no se especifica salida, imprime a stdout")
        sys.exit(1)
    
    xls_path = sys.argv[1]
    output_path = sys.argv[2] if len(sys.argv) > 2 else None
    
    try:
        result = convert_xls_to_markdown(xls_path, output_path)
        print(result)
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)

if __name__ == "__main__":
    main()
```

---

### 7.3 Fase 2: Lógica del Agente

#### T04: Actualizar AGENTS.md con Instrucciones KB

**Agregar a AGENTS.md:**

```markdown
## Sistema de Base de Conocimiento (KB)

### Ubicación de archivos KB
- Índice: `knowledge/INDEX.md`
- Reglas: `knowledge/GOVERNANCE.md`
- Metadatos: `knowledge/_meta/documents.json`
- Admins: `config/kb-admins.json`
- Documentos: `knowledge/[categoria]/*.md`

### Cuando un usuario hace una pregunta

1. Leer `knowledge/INDEX.md` para identificar documentos relevantes
2. Verificar estado del documento en `documents.json` (solo usar "activo")
3. Leer el documento específico
4. Si hay conflictos, aplicar reglas de `GOVERNANCE.md`
5. Responder citando la fuente

### Cuando un admin envía un archivo

1. Verificar que el usuario está en `config/kb-admins.json`
2. Descargar archivo a `temp/`
3. Convertir usando scripts:
   - PDF: `python scripts/convert-pdf.py temp/archivo.pdf`
   - XLS: `python scripts/convert-xls.py temp/archivo.xlsx`
4. Mostrar preview y pedir categoría
5. Guardar en `knowledge/[categoria]/[nombre].md`
6. Actualizar `INDEX.md`, `documents.json`, `CHANGELOG.md`

### Comandos /kb disponibles

| Comando | Descripción | Roles |
|---------|-------------|-------|
| `/kb list` | Listar documentos | Admin+ |
| `/kb search [término]` | Buscar en KB | Admin+ |
| `/kb status` | Estado general | Admin+ |
| `/kb update` | Actualizar documento | Admin+ |
| `/kb archive [doc]` | Archivar documento | Admin+ |
| `/kb admin list` | Listar admins | Superadmin |
| `/kb admin add` | Agregar admin | Superadmin |
| `/kb help` | Ayuda de comandos | Admin+ |
```

---

### 7.4 Fase 3: Gobernanza

#### T07: Implementar Sistema de Roles

**Lógica de validación:**

```python
def verificar_permiso(telegram_id, accion):
    admins = leer_json('config/kb-admins.json')
    
    # Superadmin puede todo
    for sa in admins['superadmins']:
        if sa['telegram_id'] == str(telegram_id):
            return True
    
    # Admin tiene permisos limitados
    acciones_admin = ['list', 'search', 'status', 'upload', 'update', 'archive']
    
    for a in admins['admins']:
        if a['telegram_id'] == str(telegram_id):
            return accion in acciones_admin
    
    return False
```

---

### 7.5 Checklist de Implementación

```markdown
## Checklist General

### Fase 1: Setup Base
- [ ] T01: Estructura de carpetas creada
- [ ] T02: Archivos de configuración creados
  - [ ] INDEX.md
  - [ ] GOVERNANCE.md
  - [ ] CHANGELOG.md
  - [ ] documents.json
  - [ ] kb-admins.json
- [ ] T03: Scripts de conversión
  - [ ] convert-pdf.py funcional
  - [ ] convert-xls.py funcional
  - [ ] Dependencias instaladas

### Fase 2: Lógica del Agente
- [ ] T04: AGENTS.md actualizado con instrucciones KB
- [ ] T05: Comandos /kb implementados
- [ ] T06: Flujo de carga funcional

### Fase 3: Gobernanza
- [ ] T07: Sistema de roles funcional
- [ ] T08: Auditoría automática
- [ ] T09: Reglas de arbitraje implementadas

### Fase 4: Testing
- [ ] T10: Todos los casos de prueba pasados
- [ ] T11: Documentación de usuario completa
```

---

# 8. Casos de Prueba

## Sistema de Base de Conocimiento con OpenClaw

### 8.1 Estrategia de Testing

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PIRÁMIDE DE TESTING                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                            ┌─────────┐                                      │
│                            │  E2E    │  ← Flujos completos via Telegram     │
│                           ─┴─────────┴─                                     │
│                          ┌─────────────┐                                    │
│                          │ Integración │  ← Componentes juntos              │
│                        ──┴─────────────┴──                                  │
│                       ┌───────────────────┐                                 │
│                       │     Unitarios     │  ← Scripts, funciones           │
│                     ──┴───────────────────┴──                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### 8.2 Casos de Prueba Unitarios

#### CPU01: Conversión PDF a Markdown

| # | Entrada | Resultado Esperado | Estado |
|---|---------|-------------------|--------|
| 1 | PDF válido con texto | Markdown con contenido | ⬜ |
| 2 | PDF con tablas | Tablas preservadas en MD | ⬜ |
| 3 | PDF con listas | Listas formateadas correctamente | ⬜ |
| 4 | PDF vacío | Error: "no contiene texto" | ⬜ |
| 5 | PDF escaneado (imagen) | Error: "no contiene texto extraíble" | ⬜ |
| 6 | Archivo no existe | Error: "Archivo no encontrado" | ⬜ |
| 7 | Archivo no es PDF | Error de pdftotext | ⬜ |
| 8 | PDF muy grande (>10MB) | Procesa correctamente (o timeout) | ⬜ |

#### CPU02: Conversión XLS a Markdown

| # | Entrada | Resultado Esperado | Estado |
|---|---------|-------------------|--------|
| 1 | XLS con una hoja | MD con tabla única | ⬜ |
| 2 | XLS con múltiples hojas | MD con secciones por hoja | ⬜ |
| 3 | XLS con celdas vacías | Celdas vacías manejadas | ⬜ |
| 4 | XLS con fórmulas | Valores calculados (no fórmulas) | ⬜ |
| 5 | Archivo XLSX | Funciona igual que XLS | ⬜ |
| 6 | Archivo no existe | Error apropiado | ⬜ |
| 7 | Archivo corrupto | Error manejado | ⬜ |

#### CPU03: Validación de Admin

| # | Usuario | Acción | Resultado Esperado | Estado |
|---|---------|--------|-------------------|--------|
| 1 | Superadmin (1007231414) | /kb list | Permitido | ⬜ |
| 2 | Superadmin | /kb admin add | Permitido | ⬜ |
| 3 | Admin (7194252326) | /kb list | Permitido | ⬜ |
| 4 | Admin | /kb admin add | Denegado | ⬜ |
| 5 | Usuario random | /kb list | Denegado | ⬜ |
| 6 | Usuario random | Subir PDF | Denegado | ⬜ |

---

### 8.3 Casos de Prueba de Integración

#### CPI01: Flujo Completo de Carga

**Pasos:**

```
1. Admin envía PDF por Telegram
2. Sistema responde pidiendo categoría
3. Admin selecciona "asanas"
4. Sistema pide tags
5. Admin ingresa "inversiones, avanzado"
6. Sistema muestra resumen
7. Admin confirma
8. Verificar:
   - [ ] Archivo existe en knowledge/asanas/
   - [ ] INDEX.md actualizado
   - [ ] documents.json actualizado
   - [ ] CHANGELOG.md tiene entrada
   - [ ] Mensaje de confirmación enviado
```

#### CPI02: Consulta con Documento Existente

**Setup:**
```
Crear documento: knowledge/asanas/inversiones.md
Contenido: información sobre sirsasana
Actualizar INDEX.md
```

**Pasos:**

```
1. Usuario pregunta: "¿Cuáles son los beneficios de sirsasana?"
2. Verificar:
   - [ ] Respuesta incluye información del documento
   - [ ] Respuesta cita la fuente
   - [ ] Respuesta es coherente con el contenido
```

---

### 8.4 Casos de Prueba E2E

#### CPE01: Ciclo de Vida Completo de Documento

**Escenario:**

```
PARTE 1: CREACIÓN
1. Admin (Ana) abre Telegram
2. Envía archivo: "precios-2026.pdf"
3. Selecciona categoría: admin
4. Ingresa tags: precios, 2026
5. Confirma
6. ✅ Documento creado

PARTE 2: CONSULTA
7. Usuario pregunta: "¿Cuánto cuesta el paquete mensual?"
8. ✅ Sistema responde con info del documento

PARTE 3: ACTUALIZACIÓN
9. Admin envía nuevo archivo con precios actualizados
10. Ejecuta /kb update
11. Selecciona precios-2026.md
12. Sube nueva versión
13. ✅ Documento actualizado a v1.1

PARTE 4: ARCHIVADO
14. Admin ejecuta /kb archive precios-2026.md
15. Confirma
16. ✅ Documento archivado

PARTE 5: VERIFICACIÓN
17. Usuario pregunta sobre precios
18. ✅ Sistema indica que no hay info disponible
19. Verificar CHANGELOG.md tiene todo el historial
```

---

### 8.5 Checklist de Validación Final

```markdown
## Validación Final

### Funcionalidad Core
- [ ] Carga de PDF funciona
- [ ] Carga de XLS funciona
- [ ] Conversión produce MD legible
- [ ] INDEX.md se actualiza automáticamente
- [ ] documents.json se actualiza automáticamente
- [ ] CHANGELOG.md registra operaciones

### Comandos
- [ ] /kb list muestra documentos
- [ ] /kb search encuentra documentos
- [ ] /kb status muestra resumen
- [ ] /kb update permite actualizar
- [ ] /kb archive funciona
- [ ] /kb help muestra ayuda

### Seguridad
- [ ] Solo admins pueden gestionar KB
- [ ] Solo superadmin puede gestionar admins
- [ ] Usuarios normales no pueden modificar

### Consultas
- [ ] Consultas usan documentos de KB
- [ ] Se cita la fuente
- [ ] Arbitraje funciona correctamente
- [ ] Documentos archivados no se usan

### Integración
- [ ] No afecta funcionalidades existentes
- [ ] Telegram funciona correctamente
- [ ] Archivos se guardan en lugares correctos
```

---

*Documento completo | Versión: 1.0 | Fecha: 2026-03-11*
