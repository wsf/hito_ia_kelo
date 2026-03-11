# Prompt de Implementación - Sistema de Base de Conocimiento

## Instrucciones para OpenClaw

Copia y pega este prompt completo en una nueva sesión de OpenClaw para implementar el sistema de Base de Conocimiento.

---

## 🚀 PROMPT COMPLETO

```
Necesito que implementes un Sistema de Base de Conocimiento (KB) completo para este agente de OpenClaw. El sistema debe permitir:

1. **Cargar documentos PDF y XLS** via Telegram (para admins)
2. **Convertir automáticamente** esos documentos a Markdown
3. **Responder consultas de usuarios** (via WhatsApp y Telegram) usando la información de la KB
4. **Gestionar versiones y auditoría** de todos los documentos
5. **Resolver conflictos** cuando hay información contradictoria entre documentos

## ESPECIFICACIONES TÉCNICAS

### Volumen esperado:
- ~100 conversaciones/día
- Concurrencia máxima: 10 usuarios
- Documentos: ~5 páginas cada uno, hasta 500 documentos

### Canales:
- **WhatsApp**: Canal principal para consultas de clientes
- **Telegram**: Canal para administración y carga de documentos

### Roles:
- **Superadmin**: Gestiona admins, configura reglas, todo lo demás
- **Admin**: Carga, actualiza, archiva documentos
- **Usuario**: Solo consulta información

## TAREAS A REALIZAR

### FASE 1: Crear estructura de carpetas

Ejecuta estos comandos:

```bash
mkdir -p knowledge/_meta
mkdir -p knowledge/_archive
mkdir -p knowledge/general
mkdir -p scripts
mkdir -p config
mkdir -p temp
```

### FASE 2: Crear archivos de configuración

#### 2.1 Crear knowledge/INDEX.md

```markdown
# Índice de Base de Conocimiento

> Última actualización: [fecha actual]

## Categorías

### 📁 general/
*Documentos generales*

| Documento | Descripción | Tags |
|-----------|-------------|------|
| (vacío) | | |

---

## Estadísticas
- Total documentos: 0
- Documentos activos: 0
- Documentos archivados: 0
```

#### 2.2 Crear knowledge/GOVERNANCE.md

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

1. **Prioridad numérica** menor = más importante (1 > 2 > 3)
2. **Fecha más reciente** gana en caso de empate
3. **Información crítica** (precios, horarios): siempre usar el más reciente

## 4. Convenciones de Nombres

- Archivos: `nombre-descriptivo.md` (kebab-case)
- Sin espacios, sin caracteres especiales
- Máximo 50 caracteres
```

#### 2.3 Crear knowledge/CHANGELOG.md

```markdown
# Changelog - Base de Conocimiento

> Registro de todos los cambios en la base de conocimiento.

---

## [Fecha actual]

### Inicialización
- Sistema KB inicializado
- Estructura de carpetas creada
- Archivos de configuración establecidos

---
```

#### 2.4 Crear knowledge/_meta/documents.json

```json
{
  "_metadata": {
    "version": "1.0",
    "created": "[fecha actual]",
    "description": "Registro de metadatos de todos los documentos"
  },
  "documents": {}
}
```

#### 2.5 Crear config/kb-admins.json

Pregúntame los Telegram IDs de los admins para configurar este archivo:

```json
{
  "_metadata": {
    "version": "1.0",
    "description": "Lista de administradores autorizados"
  },
  "superadmins": [
    {
      "telegram_id": "[PREGUNTAR]",
      "nombre": "[PREGUNTAR]",
      "agregado": "[fecha actual]"
    }
  ],
  "admins": []
}
```

### FASE 3: Crear scripts de conversión

#### 3.1 Crear scripts/convert-pdf.py

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
    lines = text.split('\n')
    md_lines = []
    
    md_lines.append("---")
    md_lines.append(f"fuente: {os.path.basename(source_file)}")
    md_lines.append(f"convertido: {datetime.now().strftime('%Y-%m-%d %H:%M')}")
    md_lines.append("---")
    md_lines.append("")
    
    for line in lines:
        stripped = line.strip()
        if not stripped:
            if md_lines and md_lines[-1] != "":
                md_lines.append("")
            continue
        if (stripped.isupper() and 3 < len(stripped) < 60 and 
            not any(c.isdigit() for c in stripped[:3])):
            md_lines.append(f"## {stripped.title()}")
            continue
        if stripped.endswith(':') and len(stripped) < 50:
            md_lines.append(f"### {stripped}")
            continue
        if stripped.startswith(('•', '-', '*', '·')):
            md_lines.append(f"- {stripped[1:].strip()}")
            continue
        if re.match(r'^\d+[\.\)]\s', stripped):
            md_lines.append(stripped)
            continue
        md_lines.append(stripped)
    
    return '\n'.join(md_lines)

def convert_pdf_to_markdown(pdf_path, output_path=None):
    if not os.path.exists(pdf_path):
        raise FileNotFoundError(f"Archivo no encontrado: {pdf_path}")
    
    text = extract_text_from_pdf(pdf_path)
    
    if not text.strip():
        raise Exception("El PDF no contiene texto extraíble")
    
    markdown = format_as_markdown(text, pdf_path)
    
    if output_path:
        with open(output_path, 'w', encoding='utf-8') as f:
            f.write(markdown)
        return f"Guardado en: {output_path}"
    
    return markdown

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Uso: python convert-pdf.py <archivo.pdf> [salida.md]")
        sys.exit(1)
    
    pdf_path = sys.argv[1]
    output_path = sys.argv[2] if len(sys.argv) > 2 else None
    
    try:
        result = convert_pdf_to_markdown(pdf_path, output_path)
        print(result)
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)
```

#### 3.2 Crear scripts/convert-xls.py

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
    if not os.path.exists(xls_path):
        raise FileNotFoundError(f"Archivo no encontrado: {xls_path}")
    
    try:
        xlsx = pd.ExcelFile(xls_path)
    except Exception as e:
        raise Exception(f"Error al leer archivo: {e}")
    
    md_lines = []
    md_lines.append("---")
    md_lines.append(f"fuente: {os.path.basename(xls_path)}")
    md_lines.append(f"convertido: {datetime.now().strftime('%Y-%m-%d %H:%M')}")
    md_lines.append(f"hojas: {len(xlsx.sheet_names)}")
    md_lines.append("---")
    md_lines.append("")
    
    for sheet_name in xlsx.sheet_names:
        df = pd.read_excel(xlsx, sheet_name=sheet_name)
        md_lines.append(f"## {sheet_name}")
        md_lines.append("")
        df = df.fillna('')
        
        if not df.empty:
            headers = '| ' + ' | '.join(str(col) for col in df.columns) + ' |'
            separator = '|' + '|'.join(['---' for _ in df.columns]) + '|'
            md_lines.append(headers)
            md_lines.append(separator)
            for _, row in df.iterrows():
                row_str = '| ' + ' | '.join(str(val) for val in row.values) + ' |'
                md_lines.append(row_str)
        else:
            md_lines.append("*(Hoja vacía)*")
        md_lines.append("")
    
    content = '\n'.join(md_lines)
    
    if output_path:
        with open(output_path, 'w', encoding='utf-8') as f:
            f.write(content)
        return f"Guardado en: {output_path}"
    
    return content

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Uso: python convert-xls.py <archivo.xlsx> [salida.md]")
        sys.exit(1)
    
    xls_path = sys.argv[1]
    output_path = sys.argv[2] if len(sys.argv) > 2 else None
    
    try:
        result = convert_xls_to_markdown(xls_path, output_path)
        print(result)
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)
```

### FASE 4: Actualizar AGENTS.md

Agrega esta sección al archivo AGENTS.md del workspace:

```markdown
## Sistema de Base de Conocimiento (KB)

### Ubicación de archivos KB
- Índice: `knowledge/INDEX.md`
- Reglas: `knowledge/GOVERNANCE.md`
- Changelog: `knowledge/CHANGELOG.md`
- Metadatos: `knowledge/_meta/documents.json`
- Admins: `config/kb-admins.json`
- Documentos: `knowledge/[categoria]/*.md`

### Cuando un usuario hace una pregunta (WhatsApp o Telegram)

1. Leer `knowledge/INDEX.md` para identificar documentos relevantes
2. Verificar estado del documento en `documents.json` (solo usar "activo")
3. Leer el documento específico
4. Si hay conflictos, aplicar reglas de `GOVERNANCE.md`
5. Responder citando la fuente: "📖 Fuente: [nombre del documento]"

### Cuando un admin envía un archivo PDF o XLS

1. Verificar que el usuario está en `config/kb-admins.json`
2. Descargar archivo a `temp/`
3. Convertir usando scripts:
   - PDF: `python scripts/convert-pdf.py temp/archivo.pdf temp/preview.md`
   - XLS: `python scripts/convert-xls.py temp/archivo.xlsx temp/preview.md`
4. Mostrar preview (primeras 20 líneas)
5. Preguntar categoría
6. Preguntar tags
7. Pedir confirmación
8. Guardar en `knowledge/[categoria]/[nombre].md`
9. Actualizar `INDEX.md`, `documents.json`, `CHANGELOG.md`
10. Confirmar éxito

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

### Validación de permisos

Antes de cualquier comando /kb o carga de archivo:
1. Obtener ID del usuario (telegram_id o whatsapp_number)
2. Leer `config/kb-admins.json`
3. Verificar si está en superadmins o admins
4. Si no está autorizado: "❌ No tenés permisos para esta acción"

### Formato de respuestas con KB

Para WhatsApp (máximo 4096 caracteres):
- Respuesta concisa
- Emojis moderados
- Fuente al final: "📖 Fuente: [documento]"

Para Telegram:
- Puede usar Markdown completo
- Puede ser más extenso
- Incluir fuente

### Registro de cambios (CHANGELOG)

Cada operación en la KB debe registrarse:
- [NUEVO] documento creado
- [ACTUALIZADO] documento modificado
- [ARCHIVADO] documento movido a _archive
- Incluir: fecha, hora, autor, detalles
```

### FASE 5: Verificar dependencias

Ejecuta estos comandos para verificar/instalar dependencias:

```bash
# Verificar pdftotext
which pdftotext || echo "FALTA: apt install poppler-utils"

# Verificar Python y pandas
python3 -c "import pandas; print('pandas OK')" || echo "FALTA: pip install pandas openpyxl"
```

### FASE 6: Prueba inicial

1. Crea un documento de prueba en `knowledge/general/bienvenida.md`:

```markdown
---
titulo: Mensaje de Bienvenida
version: 1.0
fecha_creacion: [fecha actual]
estado: activo
tags: [bienvenida, inicio]
---

# Bienvenida

Este es el sistema de Base de Conocimiento.

## ¿Cómo funciona?

Los usuarios pueden hacer preguntas y el sistema responderá usando la información almacenada en los documentos.

## Contacto

Para más información, contactar al administrador.
```

2. Actualiza `knowledge/_meta/documents.json` con la metadata del documento.

3. Actualiza `knowledge/INDEX.md` con la entrada del documento.

4. Prueba haciendo una pregunta: "¿Cómo funciona el sistema?"

---

## PREGUNTAS QUE NECESITO QUE ME RESPONDAS

Antes de empezar, necesito que me digas:

1. **¿Cuál es el Telegram ID del superadmin?**
2. **¿Cuál es el nombre del superadmin?**
3. **¿Hay otros admins para agregar ahora?** (Telegram ID y nombre de cada uno)
4. **¿Qué categorías de documentos necesitás?** (ej: productos, servicios, precios, FAQ, etc.)
5. **¿El canal de WhatsApp ya está configurado en OpenClaw?**

---

Una vez que me respondas, empiezo con la implementación paso a paso.
```

---

## 📋 Instrucciones de Uso

1. **Abrí una nueva sesión de OpenClaw** para el agente donde querés implementar el sistema
2. **Copiá todo el contenido** dentro del bloque de código de arriba (desde "Necesito que implementes..." hasta "...empiezo con la implementación paso a paso.")
3. **Pegalo como mensaje** en la nueva sesión
4. **Respondé las preguntas** que te haga el agente
5. El agente irá creando todos los archivos y configuraciones

---

## ⏱️ Tiempo Estimado

- Implementación básica: 30-60 minutos
- Con personalización de categorías: 1-2 horas
- Incluyendo pruebas: 2-3 horas

---

## ✅ Resultado Esperado

Al finalizar, tendrás:

```
workspace/
├── knowledge/
│   ├── INDEX.md
│   ├── GOVERNANCE.md
│   ├── CHANGELOG.md
│   ├── _meta/
│   │   └── documents.json
│   ├── _archive/
│   └── [categorias]/
│       └── *.md
├── scripts/
│   ├── convert-pdf.py
│   └── convert-xls.py
├── config/
│   └── kb-admins.json
├── temp/
└── AGENTS.md (actualizado)
```

Y el agente podrá:
- ✅ Recibir consultas de usuarios por WhatsApp/Telegram
- ✅ Responder usando la información de la KB
- ✅ Permitir a admins cargar documentos PDF/XLS
- ✅ Convertir automáticamente a Markdown
- ✅ Mantener versionado y auditoría

---

*Documento: PROMPT-IMPLEMENTACION.md | Versión: 1.0 | Fecha: 2026-03-11*
