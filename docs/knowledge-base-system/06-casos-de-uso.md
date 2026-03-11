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

---

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

**Postcondiciones:**
- kb-admins.json actualizado
- Cambio registrado en CHANGELOG.md

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

**Flujo Principal:**

```
1. Superadmin ejecuta /kb governance [acción]
2. Sistema valida rol
3. Según acción:
   
   [show]
   3a. Sistema muestra GOVERNANCE.md actual
   
   [priority]
   3a. Sistema pregunta nuevo orden de prioridades
   3b. Superadmin define orden
   3c. Sistema actualiza GOVERNANCE.md
   
   [rule add]
   3a. Sistema pregunta descripción de regla
   3b. Sistema agrega a GOVERNANCE.md
```

**Postcondiciones:**
- GOVERNANCE.md actualizado
- Cambio registrado en CHANGELOG.md

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
11. Si seleccionó "Otra":
    11a. Sistema pide nombre de categoría
    11b. Admin ingresa nombre
12. Sistema pregunta: "¿Tags? (separados por coma, o 'ninguno')"
13. Admin ingresa tags
14. Sistema genera nombre de archivo (slug del título)
15. Sistema muestra resumen:
    ```
    📋 Resumen:
    - Archivo: [nombre].md
    - Categoría: [categoría]/
    - Tags: [tag1, tag2, ...]
    - Estado: activo
    
    ¿Confirmar? (Sí/No)
    ```
16. Admin confirma
17. Sistema guarda archivo en knowledge/[categoria]/[nombre].md
18. Sistema actualiza INDEX.md
19. Sistema actualiza documents.json
20. Sistema registra en CHANGELOG.md
21. Sistema confirma: "✅ Documento cargado exitosamente"
22. Sistema elimina archivo temporal
```

**Flujos Alternativos:**

| Paso | Condición | Acción |
|------|-----------|--------|
| 3 | No es admin | "No autorizado" |
| 4 | Tipo inválido | "Solo se permiten PDF y XLS" |
| 5 | Muy grande | "Archivo excede límite de [X]MB" |
| 7 | Conversión falla | "Error al procesar archivo. Verificar formato." |
| 14 | Nombre ya existe | Preguntar si actualizar o usar nombre alternativo |
| 16 | Admin cancela | Descartar todo, eliminar temporal |

**Postcondiciones:**
- Documento guardado en knowledge/
- INDEX.md actualizado
- documents.json actualizado
- CHANGELOG.md actualizado

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

**Flujo Principal:**

```
1. Admin ejecuta /kb update
2. Sistema muestra lista de documentos:
   ```
   Documentos actualizables:
   1. asanas/posturas-de-pie.md (v2.0)
   2. asanas/inversiones.md (v1.0)
   3. admin/precios.md (v1.1)
   ```
3. Admin selecciona documento (número o nombre)
4. Sistema muestra info actual:
   ```
   📄 asanas/inversiones.md
   - Versión: 1.0
   - Última actualización: 2026-03-10
   - Tags: inversiones, avanzado
   ```
5. Sistema pide: "Enviá el nuevo archivo"
6. Admin envía archivo
7. Sistema convierte archivo
8. Sistema muestra cambios (si es posible)
9. Sistema pide confirmación
10. Admin confirma
11. Sistema:
    - Incrementa versión (1.0 → 1.1)
    - Actualiza fecha_actualizacion
    - Sobreescribe archivo .md
    - Actualiza documents.json
    - Registra en CHANGELOG.md
12. Sistema confirma: "✅ Documento actualizado a v1.1"
```

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

**Flujo Principal:**

```
1. Admin ejecuta /kb replace
2. Sistema muestra documentos
3. Admin selecciona documento a reemplazar
4. Sistema confirma: "¿Reemplazar admin/precios-2025.md?"
5. Admin confirma
6. Sistema pide nuevo archivo
7. Admin envía archivo
8. Sistema convierte
9. Sistema pregunta: 
   "¿Nombre del nuevo documento?"
   - Sugerido: precios-2026.md
10. Admin confirma o modifica nombre
11. Sistema:
    - Mueve viejo a _archive/precios-2025.md
    - Cambia estado del viejo a "deprecado"
    - Guarda nuevo documento
    - Actualiza INDEX.md y documents.json
    - Registra sustitución en CHANGELOG.md
12. Sistema confirma reemplazo
```

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

**Flujo Principal:**

```
1. Admin ejecuta /kb archive [nombre]
2. Sistema busca documento
3. Sistema muestra info y pide confirmación:
   ```
   ⚠️ Archivar documento:
   - asanas/inversiones.md
   - Versión: 1.0
   - Creado: 2026-03-10
   
   El documento ya no se usará en respuestas.
   ¿Confirmar? (Sí/No)
   ```
4. Admin confirma
5. Sistema:
    - Mueve a _archive/
    - Cambia estado a "deprecado"
    - Actualiza INDEX.md (remueve)
    - Registra en CHANGELOG.md
6. Sistema confirma: "✅ Documento archivado"
```

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
   ```
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
```

**Flujo Principal - Buscar:**

```
1. Admin ejecuta /kb search [término]
2. Sistema busca en INDEX.md y documents.json
3. Sistema muestra resultados:
   ```
   🔍 Resultados para "inversiones":
   
   1. asanas/inversiones.md
      Tags: inversiones, avanzado, fuerza
      
   2. asanas/posturas-de-pie.md
      Menciona: "preparación para inversiones"
   ```
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
   ```
   Los beneficios de Sirsasana (postura sobre la cabeza) incluyen:
   
   - Mejora la circulación cerebral
   - Fortalece hombros y brazos
   - Estimula la glándula pineal
   - Calma el sistema nervioso
   
   📖 Fuente: Manual de Inversiones
   ```
```

**Flujos Alternativos:**

| Paso | Condición | Acción |
|------|-----------|--------|
| 3 | No encuentra documento | Responder con conocimiento general + indicar que no hay doc específico |
| 3 | Múltiples documentos | Leer todos y sintetizar |
| 4 | Documento deprecado | Buscar alternativa activa |

---

#### UC09: Resolver Información Conflictiva

| Campo | Descripción |
|-------|-------------|
| **ID** | UC09 |
| **Nombre** | Resolver Información Conflictiva |
| **Actor Principal** | Sistema (automático) |
| **Descripción** | Sistema resuelve conflictos entre documentos |
| **Trigger** | Consulta encuentra información contradictoria |

**Flujo Principal:**

```
1. Usuario pregunta: "¿Cuánto cuesta la clase individual?"
2. Sistema encuentra 2 documentos con precios diferentes:
   - admin/precios-marzo.md: "$5000"
   - admin/precios-febrero.md: "$4500"
3. Sistema lee GOVERNANCE.md para reglas
4. Sistema lee documents.json para metadata
5. Sistema aplica reglas:
   - precios-marzo: prioridad 1, fecha 2026-03-01
   - precios-febrero: prioridad 1, fecha 2026-02-01
   - Regla: info de precios → usar más reciente
6. Sistema responde usando precios-marzo:
   ```
   La clase individual tiene un costo de $5000.
   
   📖 Fuente: Lista de precios (actualizado marzo 2026)
   ```
```

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

*Documento: 06-casos-de-uso.md | Versión: 1.0 | Fecha: 2026-03-11*
