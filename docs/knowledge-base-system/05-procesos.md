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

```
┌────┬────────────────────────────────────────────────────────────┐
│ #  │ Paso                                                       │
├────┼────────────────────────────────────────────────────────────┤
│ 1  │ Admin envía archivo por Telegram                          │
│ 2  │ Sistema valida que el usuario es admin                    │
│ 3  │ Sistema valida tipo y tamaño del archivo                  │
│ 4  │ Sistema descarga archivo a /temp                          │
│ 5  │ Sistema ejecuta conversión (PDF→MD o XLS→MD)              │
│ 6  │ Sistema muestra preview del contenido convertido          │
│ 7  │ Sistema pregunta categoría (lista de opciones)            │
│ 8  │ Admin selecciona categoría                                │
│ 9  │ Sistema pregunta tags                                     │
│ 10 │ Admin ingresa tags (o "ninguno")                          │
│ 11 │ Sistema muestra resumen y pide confirmación               │
│ 12 │ Admin confirma                                            │
│ 13 │ Sistema guarda archivo .md en knowledge/[categoria]/      │
│ 14 │ Sistema actualiza INDEX.md                                │
│ 15 │ Sistema actualiza documents.json con metadatos            │
│ 16 │ Sistema registra en CHANGELOG.md                          │
│ 17 │ Sistema confirma éxito al admin                           │
│ 18 │ Sistema elimina archivo temporal                          │
└────┴────────────────────────────────────────────────────────────┘
```

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

```
┌────┬────────────────────────────────────────────────────────────┐
│ #  │ Paso                                                       │
├────┼────────────────────────────────────────────────────────────┤
│ 1  │ Admin ejecuta /kb update o menciona actualizar            │
│ 2  │ Sistema muestra lista de documentos actualizables         │
│ 3  │ Admin selecciona documento                                │
│ 4  │ Sistema muestra info actual del documento                 │
│ 5  │ Sistema pide nuevo archivo                                │
│ 6  │ Admin envía nuevo archivo                                 │
│ 7  │ Sistema convierte archivo                                 │
│ 8  │ Sistema muestra diff o resumen de cambios                 │
│ 9  │ Sistema pide confirmación                                 │
│ 10 │ Admin confirma                                            │
│ 11 │ Sistema incrementa versión (X.Y → X.Y+1)                  │
│ 12 │ Sistema actualiza fecha_actualizacion                     │
│ 13 │ Sistema sobreescribe archivo .md                          │
│ 14 │ Sistema actualiza INDEX.md si cambió descripción          │
│ 15 │ Sistema actualiza documents.json                          │
│ 16 │ Sistema registra en CHANGELOG.md                          │
│ 17 │ Sistema confirma éxito                                    │
└────┴────────────────────────────────────────────────────────────┘
```

---

#### P3. Sustitución de Documento

**Trigger:** Admin quiere reemplazar documento por uno nuevo (ej: precios 2025 → 2026)

**Flujo Principal:**

```
┌────┬────────────────────────────────────────────────────────────┐
│ #  │ Paso                                                       │
├────┼────────────────────────────────────────────────────────────┤
│ 1  │ Admin ejecuta /kb replace o menciona reemplazar           │
│ 2  │ Sistema muestra lista de documentos                       │
│ 3  │ Admin selecciona documento a reemplazar                   │
│ 4  │ Sistema pide nuevo archivo                                │
│ 5  │ Admin envía nuevo archivo                                 │
│ 6  │ Sistema convierte archivo                                 │
│ 7  │ Sistema pregunta si mantener nombre o crear nuevo         │
│ 8  │ Admin decide                                              │
│ 9  │ Sistema mueve documento viejo a _archive/                 │
│ 10 │ Sistema cambia estado del viejo a "deprecado"             │
│ 11 │ Sistema guarda documento nuevo                            │
│ 12 │ Sistema actualiza INDEX.md                                │
│ 13 │ Sistema actualiza documents.json (ambos docs)             │
│ 14 │ Sistema registra sustitución en CHANGELOG.md              │
│ 15 │ Sistema confirma éxito                                    │
└────┴────────────────────────────────────────────────────────────┘
```

---

#### P4. Archivado de Documento

**Trigger:** Admin quiere desactivar documento obsoleto

**Flujo Principal:**

```
┌────┬────────────────────────────────────────────────────────────┐
│ #  │ Paso                                                       │
├────┼────────────────────────────────────────────────────────────┤
│ 1  │ Admin ejecuta /kb archive [documento]                     │
│ 2  │ Sistema muestra info del documento                        │
│ 3  │ Sistema pide confirmación                                 │
│ 4  │ Admin confirma                                            │
│ 5  │ Sistema mueve archivo a _archive/                         │
│ 6  │ Sistema cambia estado a "deprecado" en documents.json     │
│ 7  │ Sistema actualiza INDEX.md (remueve entrada)              │
│ 8  │ Sistema registra en CHANGELOG.md                          │
│ 9  │ Sistema confirma éxito                                    │
└────┴────────────────────────────────────────────────────────────┘
```

---

#### P5. Restauración de Documento

**Trigger:** Admin quiere recuperar documento archivado

**Flujo Principal:**

```
┌────┬────────────────────────────────────────────────────────────┐
│ #  │ Paso                                                       │
├────┼────────────────────────────────────────────────────────────┤
│ 1  │ Admin ejecuta /kb restore                                 │
│ 2  │ Sistema lista documentos en _archive/                     │
│ 3  │ Admin selecciona documento                                │
│ 4  │ Sistema pregunta categoría destino                        │
│ 5  │ Admin confirma categoría                                  │
│ 6  │ Sistema mueve archivo de _archive/ a knowledge/[cat]/     │
│ 7  │ Sistema cambia estado a "activo"                          │
│ 8  │ Sistema actualiza INDEX.md                                │
│ 9  │ Sistema registra en CHANGELOG.md                          │
│ 10 │ Sistema confirma éxito                                    │
└────┴────────────────────────────────────────────────────────────┘
```

---

### 5.3 Procesos de Operación

#### P6. Consulta Simple

**Trigger:** Usuario hace pregunta que requiere info de un documento

**Flujo Principal:**

```
┌────┬────────────────────────────────────────────────────────────┐
│ #  │ Paso                                                       │
├────┼────────────────────────────────────────────────────────────┤
│ 1  │ Usuario hace pregunta                                     │
│ 2  │ Sistema lee INDEX.md                                      │
│ 3  │ Sistema identifica documento(s) relevante(s)              │
│ 4  │ Sistema lee documento específico                          │
│ 5  │ Sistema genera respuesta usando contenido                 │
│ 6  │ Sistema incluye referencia a fuente                       │
│ 7  │ Sistema envía respuesta al usuario                        │
└────┴────────────────────────────────────────────────────────────┘
```

---

#### P7. Consulta Compleja (múltiples documentos)

**Trigger:** Usuario hace pregunta que requiere info de varios documentos

**Flujo Principal:**

```
┌────┬────────────────────────────────────────────────────────────┐
│ #  │ Paso                                                       │
├────┼────────────────────────────────────────────────────────────┤
│ 1  │ Usuario hace pregunta                                     │
│ 2  │ Sistema lee INDEX.md                                      │
│ 3  │ Sistema identifica múltiples documentos relevantes        │
│ 4  │ Sistema lee documents.json para verificar estados         │
│ 5  │ Sistema filtra solo documentos con estado "activo"        │
│ 6  │ Sistema lee cada documento necesario                      │
│ 7  │ Sistema sintetiza información de múltiples fuentes        │
│ 8  │ Sistema incluye referencias a todas las fuentes           │
│ 9  │ Sistema envía respuesta al usuario                        │
└────┴────────────────────────────────────────────────────────────┘
```

---

#### P8. Arbitraje de Información Conflictiva

**Trigger:** Múltiples documentos tienen información contradictoria

**Flujo Principal:**

```
┌────┬────────────────────────────────────────────────────────────┐
│ #  │ Paso                                                       │
├────┼────────────────────────────────────────────────────────────┤
│ 1  │ Sistema detecta información conflictiva entre documentos  │
│ 2  │ Sistema lee GOVERNANCE.md para obtener reglas             │
│ 3  │ Sistema lee documents.json para metadata de cada doc      │
│ 4  │ Sistema aplica reglas de prioridad:                       │
│    │   a. Documento con estado "oficial" prevalece             │
│    │   b. Si empate: documento con mayor prioridad numérica    │
│    │   c. Si empate: documento más reciente                    │
│ 5  │ Sistema genera respuesta usando doc ganador               │
│ 6  │ Sistema indica que había múltiples fuentes                │
│ 7  │ Sistema cita documento utilizado y su fecha               │
└────┴────────────────────────────────────────────────────────────┘
```

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

```
┌────┬────────────────────────────────────────────────────────────┐
│ #  │ Paso                                                       │
├────┼────────────────────────────────────────────────────────────┤
│ 1  │ Recibir ruta de archivo PDF                               │
│ 2  │ Ejecutar: pdftotext -layout input.pdf output.txt          │
│ 3  │ Leer contenido de output.txt                              │
│ 4  │ Aplicar formateo Markdown:                                │
│    │   - Detectar títulos (líneas en mayúsculas o cortas)      │
│    │   - Detectar listas (líneas con - o •)                    │
│    │   - Preservar tablas si es posible                        │
│ 5  │ Agregar header con metadata:                              │
│    │   - Título                                                │
│    │   - Fecha de conversión                                   │
│    │   - Archivo fuente                                        │
│ 6  │ Retornar contenido Markdown                               │
└────┴────────────────────────────────────────────────────────────┘
```

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

```
┌────┬────────────────────────────────────────────────────────────┐
│ #  │ Paso                                                       │
├────┼────────────────────────────────────────────────────────────┤
│ 1  │ Recibir evento (create/update/delete)                     │
│ 2  │ Leer INDEX.md actual                                      │
│ 3  │ Leer documents.json actual                                │
│ 4  │ Según evento:                                             │
│    │   - CREATE: agregar entrada al índice                     │
│    │   - UPDATE: actualizar descripción si cambió              │
│    │   - DELETE: remover entrada del índice                    │
│ 5  │ Regenerar INDEX.md ordenado por categoría                 │
│ 6  │ Guardar INDEX.md                                          │
└────┴────────────────────────────────────────────────────────────┘
```

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

*Documento: 05-procesos.md | Versión: 1.0 | Fecha: 2026-03-11*
