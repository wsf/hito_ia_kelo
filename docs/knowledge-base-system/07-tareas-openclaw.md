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

---

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

**Verificación:**
- [ ] Todas las carpetas existen
- [ ] Permisos correctos (lectura/escritura)

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

**Verificación:**
- [ ] INDEX.md creado y formateado
- [ ] GOVERNANCE.md con reglas definidas
- [ ] CHANGELOG.md inicializado
- [ ] documents.json válido (JSON sintaxis OK)
- [ ] kb-admins.json con admins iniciales

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

**Verificación:**
- [ ] Scripts creados con permisos de ejecución
- [ ] Dependencias instaladas (poppler-utils, pandas, openpyxl)
- [ ] Test con PDF de prueba
- [ ] Test con XLS de prueba

---

### 7.3 Fase 2: Lógica del Agente

---

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

### Validación de admins

```javascript
// Pseudocódigo para validar admin
function esAdmin(telegramId) {
    const admins = leer('config/kb-admins.json');
    return admins.superadmins.some(a => a.telegram_id === telegramId) ||
           admins.admins.some(a => a.telegram_id === telegramId);
}

function esSuperadmin(telegramId) {
    const admins = leer('config/kb-admins.json');
    return admins.superadmins.some(a => a.telegram_id === telegramId);
}
```
```

---

#### T05: Implementar Comandos /kb

**Agregar lógica para cada comando en el comportamiento del agente:**

```markdown
### /kb list

Cuando recibo `/kb list`:

1. Verificar que el usuario es admin
2. Leer `knowledge/INDEX.md`
3. Leer `knowledge/_meta/documents.json` para estados
4. Mostrar lista formateada:
   ```
   📚 Base de Conocimiento
   
   asanas/ (X documentos)
   ├── documento1.md (vX.X) ✅
   └── documento2.md (vX.X) ✅
   
   [etc...]
   
   📁 Archivados: X documentos
   ```

### /kb search [término]

1. Verificar admin
2. Buscar en INDEX.md (títulos, descripciones)
3. Buscar en documents.json (tags)
4. Mostrar resultados ordenados por relevancia

### /kb status

1. Verificar admin
2. Contar documentos por categoría y estado
3. Mostrar resumen:
   ```
   📊 Estado de la Base de Conocimiento
   
   Total: X documentos
   ├── Activos: X
   ├── Borrador: X
   └── Archivados: X
   
   Por categoría:
   ├── asanas: X
   ├── pranayama: X
   └── admin: X
   
   Última actualización: [fecha]
   ```
```

---

#### T06: Implementar Flujo de Carga

**Agregar a AGENTS.md:**

```markdown
### Flujo de carga de documento

Cuando un admin envía un archivo (PDF o XLS):

**Paso 1: Validación**
```
- Verificar que es admin (config/kb-admins.json)
- Verificar tipo de archivo (PDF o XLS)
- Verificar tamaño (< 10MB)
```

**Paso 2: Conversión**
```bash
# Para PDF
python scripts/convert-pdf.py temp/archivo.pdf temp/preview.md

# Para XLS  
python scripts/convert-xls.py temp/archivo.xlsx temp/preview.md
```

**Paso 3: Preview**
```
Mostrar primeras 20 líneas del contenido convertido
```

**Paso 4: Categorización**
```
Preguntar:
"¿En qué categoría va este documento?"
1. asanas
2. pranayama
3. anatomia
4. admin
5. Otra (especificar)
```

**Paso 5: Tags**
```
Preguntar:
"¿Tags? (separados por coma, o 'ninguno')"
```

**Paso 6: Confirmación**
```
Mostrar resumen y pedir confirmación
```

**Paso 7: Guardado**
```python
# Pseudocódigo
archivo_destino = f"knowledge/{categoria}/{nombre_slug}.md"
guardar(contenido_md, archivo_destino)

# Actualizar INDEX.md
agregar_a_indice(archivo_destino, descripcion, tags)

# Actualizar documents.json
metadata = {
    "titulo": titulo,
    "version": "1.0",
    "fecha_creacion": hoy(),
    "autor": nombre_admin,
    "estado": "activo",
    "tags": tags,
    "archivo_original": nombre_archivo_original
}
actualizar_documents_json(archivo_destino, metadata)

# Registrar en CHANGELOG
log = f"[NUEVO] {archivo_destino} v1.0 - Autor: {nombre_admin}"
agregar_a_changelog(log)
```

**Paso 8: Confirmación**
```
"✅ Documento cargado exitosamente"
```
```

---

### 7.4 Fase 3: Gobernanza

---

#### T07: Implementar Sistema de Roles

**Lógica de validación:**

```markdown
### Validación de permisos

Antes de ejecutar cualquier comando /kb:

1. Obtener telegram_id del mensaje
2. Leer config/kb-admins.json
3. Verificar rol:

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

### Mensajes de error

- No autorizado: "❌ No tenés permisos para esta acción"
- Solo superadmin: "❌ Esta acción requiere permisos de superadmin"
```

---

#### T08: Implementar Auditoría Automática

**Función para registrar en CHANGELOG:**

```markdown
### Registro automático

Cada operación debe registrarse en CHANGELOG.md:

```python
def registrar_cambio(tipo, archivo, autor, detalles=""):
    from datetime import datetime
    
    fecha = datetime.now().strftime("%Y-%m-%d")
    hora = datetime.now().strftime("%H:%M UTC")
    
    entrada = f"""
### {hora}
- **[{tipo}]** {archivo}
  - Autor: {autor}
  - {detalles}
"""
    
    # Leer CHANGELOG actual
    changelog = leer('knowledge/CHANGELOG.md')
    
    # Buscar sección de hoy o crearla
    if f"## {fecha}" not in changelog:
        # Agregar nueva sección de fecha
        insertar_despues_de("# Changelog", f"\n## {fecha}\n", changelog)
    
    # Agregar entrada bajo la fecha de hoy
    insertar_despues_de(f"## {fecha}", entrada, changelog)
    
    guardar('knowledge/CHANGELOG.md', changelog)
```

### Tipos de registro

| Tipo | Cuándo |
|------|--------|
| NUEVO | Documento creado |
| ACTUALIZADO | Documento modificado |
| ARCHIVADO | Documento movido a _archive |
| RESTAURADO | Documento recuperado de _archive |
| ELIMINADO | Documento eliminado permanentemente |
| CONFIG | Cambio en configuración/gobernanza |
```

---

#### T09: Implementar Reglas de Arbitraje

**Lógica para resolver conflictos:**

```markdown
### Resolución de conflictos

Cuando hay múltiples documentos con información sobre el mismo tema:

```python
def resolver_conflicto(documentos):
    """
    documentos: lista de rutas a documentos relevantes
    Retorna: documento que debe prevalecer
    """
    
    # Leer metadata de cada documento
    meta = leer_json('knowledge/_meta/documents.json')
    governance = leer('knowledge/GOVERNANCE.md')
    
    docs_info = []
    for doc in documentos:
        if doc in meta['documents']:
            docs_info.append({
                'ruta': doc,
                'estado': meta['documents'][doc].get('estado', 'activo'),
                'prioridad': meta['documents'][doc].get('prioridad', 5),
                'fecha': meta['documents'][doc].get('fecha_actualizacion', 
                         meta['documents'][doc].get('fecha_creacion'))
            })
    
    # Filtrar solo activos
    docs_activos = [d for d in docs_info if d['estado'] == 'activo']
    
    if not docs_activos:
        return None
    
    # Ordenar por prioridad (menor = mejor), luego por fecha (más reciente = mejor)
    docs_ordenados = sorted(docs_activos, 
                            key=lambda x: (x['prioridad'], -parse_date(x['fecha'])))
    
    return docs_ordenados[0]['ruta']
```

### Comunicar al usuario

Cuando se usa arbitraje, informar:

```
[Respuesta basada en el documento más actualizado]

📖 Fuente: [nombre del documento] (actualizado [fecha])
ℹ️ Nota: Existe información adicional en otros documentos.
```
```

---

### 7.5 Fase 4: Testing y Documentación

Ver documento **08-casos-de-prueba.md** para casos de prueba detallados.

---

### 7.6 Checklist de Implementación

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
  - [ ] /kb list
  - [ ] /kb search
  - [ ] /kb status
  - [ ] /kb help
  - [ ] /kb update
  - [ ] /kb archive
  - [ ] /kb admin (solo superadmin)
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

*Documento: 07-tareas-openclaw.md | Versión: 1.0 | Fecha: 2026-03-11*
