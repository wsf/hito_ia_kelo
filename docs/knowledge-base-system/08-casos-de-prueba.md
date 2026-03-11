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

| ID | CPU01 |
|----|-------|
| **Nombre** | Conversión PDF a Markdown |
| **Componente** | scripts/convert-pdf.py |
| **Precondiciones** | poppler-utils instalado |

**Casos:**

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

**Script de prueba:**

```bash
#!/bin/bash
# test-convert-pdf.sh

echo "=== Tests de convert-pdf.py ==="

# Test 1: PDF válido
echo "Test 1: PDF válido"
python scripts/convert-pdf.py tests/samples/documento-valido.pdf tests/output/test1.md
if [ $? -eq 0 ] && [ -s tests/output/test1.md ]; then
    echo "✅ PASS"
else
    echo "❌ FAIL"
fi

# Test 2: PDF no existe
echo "Test 2: PDF no existe"
python scripts/convert-pdf.py tests/samples/no-existe.pdf 2>&1 | grep -q "no encontrado"
if [ $? -eq 0 ]; then
    echo "✅ PASS"
else
    echo "❌ FAIL"
fi

# Agregar más tests...
```

---

#### CPU02: Conversión XLS a Markdown

| ID | CPU02 |
|----|-------|
| **Nombre** | Conversión XLS a Markdown |
| **Componente** | scripts/convert-xls.py |
| **Precondiciones** | pandas, openpyxl instalados |

**Casos:**

| # | Entrada | Resultado Esperado | Estado |
|---|---------|-------------------|--------|
| 1 | XLS con una hoja | MD con tabla única | ⬜ |
| 2 | XLS con múltiples hojas | MD con secciones por hoja | ⬜ |
| 3 | XLS con celdas vacías | Celdas vacías manejadas | ⬜ |
| 4 | XLS con fórmulas | Valores calculados (no fórmulas) | ⬜ |
| 5 | Archivo XLSX | Funciona igual que XLS | ⬜ |
| 6 | Archivo no existe | Error apropiado | ⬜ |
| 7 | Archivo corrupto | Error manejado | ⬜ |

---

#### CPU03: Validación de Admin

| ID | CPU03 |
|----|-------|
| **Nombre** | Validación de permisos de admin |
| **Componente** | Lógica del agente |

**Casos:**

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

| ID | CPI01 |
|----|-------|
| **Nombre** | Carga de documento PDF completa |
| **Componentes** | Telegram, Agente, Scripts, Archivos |

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

**Datos de prueba:**

```
Archivo: test-inversiones.pdf
Categoría: asanas
Tags: inversiones, avanzado
Nombre esperado: test-inversiones.md
```

---

#### CPI02: Consulta con Documento Existente

| ID | CPI02 |
|----|-------|
| **Nombre** | Consulta usando KB |
| **Componentes** | Agente, INDEX.md, Documentos |

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

#### CPI03: Arbitraje entre Documentos

| ID | CPI03 |
|----|-------|
| **Nombre** | Resolución de conflicto |
| **Componentes** | Agente, GOVERNANCE.md, documents.json |

**Setup:**
```
Crear 2 documentos con info conflictiva:
- admin/precios-febrero.md: "Clase: $4500"
- admin/precios-marzo.md: "Clase: $5000"

documents.json:
- precios-febrero: prioridad 1, fecha 2026-02-01
- precios-marzo: prioridad 1, fecha 2026-03-01
```

**Pasos:**

```
1. Usuario pregunta: "¿Cuánto cuesta una clase?"
2. Verificar:
   - [ ] Sistema usa documento más reciente
   - [ ] Respuesta: "$5000"
   - [ ] Menciona fuente (precios-marzo)
```

---

### 8.4 Casos de Prueba E2E (End-to-End)

#### CPE01: Ciclo de Vida Completo de Documento

| ID | CPE01 |
|----|-------|
| **Nombre** | Ciclo completo: crear, actualizar, archivar |
| **Duración** | ~10 minutos |

**Escenario:**

```
PARTE 1: CREACIÓN
─────────────────
1. Admin (Ana) abre Telegram
2. Envía archivo: "precios-2026.pdf"
3. Selecciona categoría: admin
4. Ingresa tags: precios, 2026
5. Confirma
6. ✅ Documento creado

PARTE 2: CONSULTA
─────────────────
7. Usuario pregunta: "¿Cuánto cuesta el paquete mensual?"
8. ✅ Sistema responde con info del documento

PARTE 3: ACTUALIZACIÓN
─────────────────────
9. Admin envía nuevo archivo con precios actualizados
10. Ejecuta /kb update
11. Selecciona precios-2026.md
12. Sube nueva versión
13. ✅ Documento actualizado a v1.1

PARTE 4: ARCHIVADO
─────────────────
14. Admin ejecuta /kb archive precios-2026.md
15. Confirma
16. ✅ Documento archivado

PARTE 5: VERIFICACIÓN
────────────────────
17. Usuario pregunta sobre precios
18. ✅ Sistema indica que no hay info disponible
19. Verificar CHANGELOG.md tiene todo el historial
```

**Checklist de Verificación:**

- [ ] Documento creado correctamente
- [ ] INDEX.md reflejó creación
- [ ] documents.json tiene metadata v1.0
- [ ] CHANGELOG tiene entrada de creación
- [ ] Consulta funcionó correctamente
- [ ] Actualización incrementó versión
- [ ] CHANGELOG tiene entrada de actualización
- [ ] Archivado movió a _archive/
- [ ] INDEX.md ya no lista el documento
- [ ] Estado cambió a "deprecado"
- [ ] Consulta post-archivo no usa el documento

---

#### CPE02: Gestión de Admins

| ID | CPE02 |
|----|-------|
| **Nombre** | Gestión completa de administradores |
| **Actor** | Superadmin |

**Escenario:**

```
1. Superadmin ejecuta /kb admin list
   ✅ Ve lista actual de admins

2. Superadmin ejecuta /kb admin add
   - Ingresa ID: 12345678
   - Ingresa nombre: "Nuevo Admin"
   ✅ Admin agregado

3. Verificar que nuevo admin puede ejecutar /kb list
   ✅ Funciona

4. Superadmin ejecuta /kb admin remove 12345678
   ✅ Admin removido

5. Verificar que ex-admin NO puede ejecutar /kb list
   ✅ Denegado
```

---

#### CPE03: Manejo de Errores

| ID | CPE03 |
|----|-------|
| **Nombre** | Respuesta a errores comunes |

**Casos:**

| # | Acción | Error Esperado | Estado |
|---|--------|----------------|--------|
| 1 | Usuario no-admin intenta /kb list | "No autorizado" | ⬜ |
| 2 | Admin sube archivo .doc | "Solo PDF y XLS" | ⬜ |
| 3 | Admin sube PDF de 50MB | "Archivo muy grande" | ⬜ |
| 4 | Admin intenta archivar doc inexistente | "Documento no encontrado" | ⬜ |
| 5 | Consulta sin documentos relevantes | Respuesta apropiada sin error | ⬜ |

---

### 8.5 Casos de Prueba de Regresión

#### CPR01: Compatibilidad con Funcionalidad Existente

Verificar que el sistema KB no afecta funcionalidades previas:

| # | Funcionalidad | Verificación | Estado |
|---|---------------|--------------|--------|
| 1 | Respuestas de yoga a usuarios | Siguen funcionando | ⬜ |
| 2 | Mensajes para Ana | Se guardan correctamente | ⬜ |
| 3 | Memoria de alumnos | Persiste y funciona | ⬜ |
| 4 | Comandos existentes | No interfieren con /kb | ⬜ |

---

### 8.6 Casos de Prueba de Carga

#### CPC01: Múltiples Documentos

| Escenario | Acción | Métrica | Criterio |
|-----------|--------|---------|----------|
| 50 documentos | /kb list | Tiempo respuesta | < 3 seg |
| 100 documentos | Consulta usuario | Tiempo respuesta | < 5 seg |
| 10 categorías | Navegación | Usabilidad | Clara |

---

### 8.7 Datos de Prueba

#### Archivos de muestra necesarios:

```
tests/
├── samples/
│   ├── documento-valido.pdf        # PDF simple con texto
│   ├── documento-tablas.pdf        # PDF con tablas
│   ├── documento-listas.pdf        # PDF con listas
│   ├── documento-vacio.pdf         # PDF sin contenido
│   ├── documento-escaneado.pdf     # PDF imagen (para test de error)
│   ├── planilla-simple.xlsx        # Excel una hoja
│   ├── planilla-multiple.xlsx      # Excel varias hojas
│   └── archivo-invalido.txt        # Para test de tipo inválido
└── output/
    └── (archivos generados por tests)
```

#### Contenido sugerido para documento-valido.pdf:

```
MANUAL DE INVERSIONES EN YOGA

Introducción

Las inversiones son posturas donde la cabeza queda por debajo del corazón.

BENEFICIOS

• Mejora la circulación cerebral
• Fortalece los hombros
• Calma el sistema nervioso

POSTURAS PRINCIPALES

1. Sirsasana (sobre la cabeza)
2. Sarvangasana (sobre los hombros)
3. Adho Mukha Vrksasana (parado de manos)

CONTRAINDICACIONES

- Hipertensión no controlada
- Glaucoma
- Lesiones cervicales
```

---

### 8.8 Template de Reporte de Pruebas

```markdown
# Reporte de Pruebas KB
Fecha: [YYYY-MM-DD]
Ejecutado por: [Nombre]
Versión: [X.X]

## Resumen

| Tipo | Total | Pasados | Fallidos | Pendientes |
|------|-------|---------|----------|------------|
| Unitarios | X | X | X | X |
| Integración | X | X | X | X |
| E2E | X | X | X | X |

## Detalle de Fallos

### [ID del caso]
- **Descripción:** 
- **Resultado esperado:**
- **Resultado obtenido:**
- **Evidencia:**
- **Severidad:** Alta/Media/Baja

## Notas

[Observaciones adicionales]

## Siguiente Paso

[Qué hacer con los fallos]
```

---

### 8.9 Checklist de Validación Final

Antes de dar por completada la implementación:

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

*Documento: 08-casos-de-prueba.md | Versión: 1.0 | Fecha: 2026-03-11*
