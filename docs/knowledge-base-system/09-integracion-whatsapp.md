# 9. Integración con WhatsApp

## Sistema de Base de Conocimiento con OpenClaw

### 9.1 Descripción General

Este documento describe la hoja de ruta para integrar **WhatsApp** como canal principal de comunicación entre los usuarios (clientes/contactos) y el sistema de Base de Conocimiento. Las consultas de los clientes se realizarán a través de WhatsApp, y el agente de OpenClaw responderá utilizando la información almacenada en la KB.

---

### 9.2 Arquitectura con WhatsApp

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      ARQUITECTURA KB + WHATSAPP                              │
└─────────────────────────────────────────────────────────────────────────────┘

    ┌─────────┐         ┌─────────┐         ┌─────────┐
    │ Cliente │         │  Admin  │         │  Admin  │
    │(WhatsApp)│        │(WhatsApp)│        │(Telegram)│
    └────┬────┘         └────┬────┘         └────┬────┘
         │                   │                   │
         │                   │                   │
         ▼                   ▼                   ▼
    ┌─────────┐         ┌─────────┐         ┌─────────┐
    │WhatsApp │         │WhatsApp │         │Telegram │
    │Business │         │Business │         │Bot API  │
    │  API    │         │  API    │         │         │
    └────┬────┘         └────┬────┘         └────┬────┘
         │                   │                   │
         └───────────────────┼───────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            OPENCLAW GATEWAY                                  │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         AGENTE (LLM)                                 │   │
│  │                                                                      │   │
│  │  • Recibe mensajes de WhatsApp y Telegram                           │   │
│  │  • Identifica canal de origen                                       │   │
│  │  • Consulta KB para responder                                       │   │
│  │  • Responde por el mismo canal                                      │   │
│  │                                                                      │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                      │                                      │
│                                      ▼                                      │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         WORKSPACE (Files)                            │   │
│  │                                                                      │   │
│  │   knowledge/              config/                                    │   │
│  │   ├── INDEX.md            ├── kb-admins.json                        │   │
│  │   ├── [categorias]/*.md   └── whatsapp-config.json                  │   │
│  │   └── ...                                                            │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### 9.3 Canales y Roles

| Canal | Usuarios | Funcionalidades |
|-------|----------|-----------------|
| **WhatsApp** | Clientes/Contactos | Consultas a la KB, preguntas generales |
| **WhatsApp** | Admins (opcional) | Gestión de KB, comandos /kb |
| **Telegram** | Admins | Gestión de KB, carga de documentos |

---

### 9.4 Opciones de Integración WhatsApp

#### Opción 1: WhatsApp Business API (Oficial)

**Requisitos:**
- Cuenta de WhatsApp Business verificada
- Proveedor de API (Meta Business Suite, Twilio, MessageBird, etc.)
- Número de teléfono dedicado

**Pros:**
- ✅ Oficial y confiable
- ✅ Soporte para mensajes masivos
- ✅ Verificación de empresa (badge verde)
- ✅ Webhooks robustos

**Contras:**
- ❌ Requiere aprobación de Meta
- ❌ Costo por mensaje
- ❌ Setup más complejo

**Proveedores recomendados:**

| Proveedor | Costo aprox. | Complejidad |
|-----------|--------------|-------------|
| Twilio | $0.005-0.05/msg | Media |
| MessageBird | $0.004-0.04/msg | Media |
| 360Dialog | $0.003-0.03/msg | Media |
| Meta Cloud API | Gratis (límites) | Alta |

---

#### Opción 2: WhatsApp Web Bridge (No oficial)

**Requisitos:**
- Número de WhatsApp regular
- Librería como `whatsapp-web.js` o `Baileys`
- Servidor siempre encendido

**Pros:**
- ✅ Gratis
- ✅ Setup rápido
- ✅ Sin aprobación requerida

**Contras:**
- ❌ Puede violar ToS de WhatsApp
- ❌ Riesgo de ban
- ❌ Menos estable
- ❌ Requiere escanear QR periódicamente

---

#### Opción 3: OpenClaw WhatsApp Plugin (Recomendada)

OpenClaw tiene soporte nativo para WhatsApp via configuración.

**Configuración en OpenClaw:**

```yaml
# config.yaml
channels:
  whatsapp:
    enabled: true
    provider: "twilio"  # o "360dialog", "messagebird", "cloud-api"
    account_sid: "ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
    auth_token: "your_auth_token"
    phone_number: "+1234567890"
    webhook_path: "/webhooks/whatsapp"
```

---

### 9.5 Hoja de Ruta de Implementación

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ROADMAP INTEGRACIÓN WHATSAPP                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  FASE 1: Preparación (1-2 días)                                             │
│  ├── W01: Elegir proveedor de WhatsApp API                                  │
│  ├── W02: Crear cuenta y obtener credenciales                               │
│  └── W03: Registrar número de WhatsApp Business                             │
│                                                                              │
│  FASE 2: Configuración OpenClaw (1 día)                                     │
│  ├── W04: Configurar canal WhatsApp en OpenClaw                             │
│  ├── W05: Configurar webhook para recibir mensajes                          │
│  └── W06: Probar envío/recepción de mensajes                                │
│                                                                              │
│  FASE 3: Integración con KB (1-2 días)                                      │
│  ├── W07: Actualizar AGENTS.md con instrucciones WhatsApp                   │
│  ├── W08: Definir flujo de consultas desde WhatsApp                         │
│  └── W09: Configurar respuestas con citación de fuentes                     │
│                                                                              │
│  FASE 4: Control de Acceso (1 día)                                          │
│  ├── W10: Definir quién puede consultar (todos vs whitelist)                │
│  ├── W11: Configurar admins de WhatsApp (si aplica)                         │
│  └── W12: Implementar rate limiting                                         │
│                                                                              │
│  FASE 5: Testing y Go-Live (1-2 días)                                       │
│  ├── W13: Pruebas con usuarios reales                                       │
│  ├── W14: Ajustes de tono y respuestas                                      │
│  └── W15: Go-live y monitoreo                                               │
│                                                                              │
│  TOTAL ESTIMADO: 5-8 días                                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### 9.6 Tareas Detalladas

#### FASE 1: Preparación

##### W01: Elegir Proveedor de WhatsApp API

**Criterios de selección:**

| Criterio | Peso | Opciones a evaluar |
|----------|------|-------------------|
| Costo por mensaje | Alto | Comparar 3+ proveedores |
| Facilidad de integración | Alto | SDK disponible, documentación |
| Confiabilidad | Alto | SLA, uptime histórico |
| Soporte | Medio | Tiempo de respuesta, idioma |
| Funcionalidades extra | Bajo | Templates, analytics, etc. |

**Acción:** Completar matriz de decisión y seleccionar proveedor.

##### W02: Crear Cuenta y Obtener Credenciales

**Para Twilio (ejemplo):**

1. Crear cuenta en https://www.twilio.com
2. Ir a Console → Messaging → WhatsApp
3. Completar registro de WhatsApp Business
4. Obtener:
   - Account SID
   - Auth Token
   - WhatsApp Phone Number

**Para Meta Cloud API:**

1. Crear cuenta en https://business.facebook.com
2. Crear app en https://developers.facebook.com
3. Agregar producto "WhatsApp"
4. Obtener:
   - Phone Number ID
   - Access Token
   - Business Account ID

##### W03: Registrar Número de WhatsApp Business

- Verificar número de teléfono
- Configurar perfil de negocio (nombre, logo, descripción)
- Esperar aprobación (24-72 horas)

---

#### FASE 2: Configuración OpenClaw

##### W04: Configurar Canal WhatsApp en OpenClaw

**Archivo de configuración:**

```yaml
# En la configuración de OpenClaw
channels:
  whatsapp:
    enabled: true
    provider: "twilio"
    credentials:
      account_sid: "${TWILIO_ACCOUNT_SID}"
      auth_token: "${TWILIO_AUTH_TOKEN}"
      phone_number: "+5491234567890"
    settings:
      typing_indicator: true
      read_receipts: true
      max_message_length: 4096
```

##### W05: Configurar Webhook

**URL del webhook:**
```
https://tu-dominio.com/api/webhooks/whatsapp
```

**Configurar en el proveedor:**
- URL de callback
- Verificación de token
- Eventos a recibir: `messages`, `status`

##### W06: Probar Envío/Recepción

**Test básico:**
1. Enviar mensaje desde WhatsApp al número del bot
2. Verificar que llega al webhook
3. Verificar que el agente responde
4. Verificar que la respuesta llega al usuario

---

#### FASE 3: Integración con KB

##### W07: Actualizar AGENTS.md

**Agregar sección:**

```markdown
## Consultas desde WhatsApp

### Identificación del canal
- Si el mensaje viene de WhatsApp, el campo `channel` será "whatsapp"
- El `sender_id` será el número de teléfono del usuario

### Flujo de consulta desde WhatsApp
1. Usuario envía pregunta por WhatsApp
2. Leer INDEX.md para identificar documentos relevantes
3. Leer documento(s) necesario(s)
4. Generar respuesta clara y concisa (límite 4096 caracteres)
5. Incluir fuente de manera breve
6. Enviar respuesta

### Formato de respuestas WhatsApp
- Usar emojis con moderación
- Mantener mensajes concisos
- Dividir respuestas largas en múltiples mensajes si es necesario
- Incluir fuente al final: "📖 Fuente: [nombre del documento]"

### Limitaciones WhatsApp
- No se pueden enviar archivos MD
- No hay formato Markdown completo (solo negrita *texto* e itálica _texto_)
- Máximo 4096 caracteres por mensaje
```

##### W08: Definir Flujo de Consultas

```
┌─────────┐         ┌─────────┐         ┌─────────┐
│ Cliente │         │OpenClaw │         │   KB    │
│WhatsApp │         │  Agent  │         │  Files  │
└────┬────┘         └────┬────┘         └────┬────┘
     │                   │                   │
     │ "¿Cuánto cuesta   │                   │
     │  una clase?"      │                   │
     │──────────────────▶│                   │
     │                   │                   │
     │                   │ Lee INDEX.md      │
     │                   │──────────────────▶│
     │                   │◀──────────────────│
     │                   │                   │
     │                   │ Lee admin/precios │
     │                   │──────────────────▶│
     │                   │◀──────────────────│
     │                   │                   │
     │ "La clase         │                   │
     │  individual       │                   │
     │  cuesta $5000.    │                   │
     │  📖 Lista de      │                   │
     │  precios 2026"    │                   │
     │◀──────────────────│                   │
     │                   │                   │
```

##### W09: Respuestas con Citación

**Template de respuesta:**

```
[Respuesta a la consulta]

📖 Fuente: [Nombre del documento]
```

**Ejemplo:**

```
La clase individual tiene un costo de $5000.

Los paquetes disponibles son:
• 4 clases: $18.000
• 8 clases: $32.000
• Mensual libre: $45.000

📖 Fuente: Lista de precios 2026
```

---

#### FASE 4: Control de Acceso

##### W10: Definir Política de Acceso

**Opción A: Acceso abierto (recomendado para atención al cliente)**

Cualquier número puede consultar la KB.

```json
{
  "whatsapp": {
    "access": "open",
    "rate_limit": {
      "messages_per_minute": 5,
      "messages_per_hour": 50
    }
  }
}
```

**Opción B: Whitelist (para grupos cerrados)**

Solo números autorizados pueden consultar.

```json
{
  "whatsapp": {
    "access": "whitelist",
    "allowed_numbers": [
      "+5491155551234",
      "+5491155555678"
    ]
  }
}
```

##### W11: Configurar Admins de WhatsApp

**Agregar a kb-admins.json:**

```json
{
  "superadmins": [
    {
      "telegram_id": "1007231414",
      "whatsapp_number": "+5491155551234",
      "nombre": "Ale"
    }
  ],
  "admins": [
    {
      "telegram_id": "7194252326",
      "whatsapp_number": "+5491155555678",
      "nombre": "Ana"
    }
  ]
}
```

##### W12: Rate Limiting

**Configuración recomendada:**

| Métrica | Límite | Acción al exceder |
|---------|--------|-------------------|
| Mensajes/minuto | 5 | Respuesta: "Por favor esperá un momento..." |
| Mensajes/hora | 50 | Respuesta: "Alcanzaste el límite de consultas..." |
| Mensajes/día | 200 | Bloqueo temporal con aviso |

---

#### FASE 5: Testing y Go-Live

##### W13: Pruebas con Usuarios Reales

**Checklist de pruebas:**

| # | Escenario | Verificar | Estado |
|---|-----------|-----------|--------|
| 1 | Consulta simple | Respuesta correcta con fuente | ⬜ |
| 2 | Consulta sin info en KB | Respuesta apropiada | ⬜ |
| 3 | Consulta múltiples docs | Sintetiza información | ⬜ |
| 4 | Mensaje muy largo | Se divide correctamente | ⬜ |
| 5 | Emoji y caracteres especiales | Se manejan bien | ⬜ |
| 6 | Rate limit | Se aplica correctamente | ⬜ |
| 7 | Admin ejecuta /kb | Funciona por WhatsApp | ⬜ |

##### W14: Ajustes de Tono

**Guía de tono para WhatsApp:**

| Aspecto | Recomendación |
|---------|---------------|
| Saludo | Cálido pero breve: "¡Hola! 👋" |
| Extensión | Conciso, máximo 3-4 párrafos |
| Emojis | 1-3 por mensaje, relevantes |
| Formalidad | Semi-formal, cercano |
| Cierre | Ofrecer más ayuda: "¿Necesitás algo más?" |

##### W15: Go-Live y Monitoreo

**Checklist de lanzamiento:**

- [ ] Webhook funcionando correctamente
- [ ] Número de WhatsApp verificado
- [ ] Perfil de negocio completo
- [ ] KB poblada con documentos
- [ ] Rate limiting configurado
- [ ] Alertas configuradas para errores
- [ ] Logs habilitados

**Métricas a monitorear:**

| Métrica | Herramienta | Frecuencia |
|---------|-------------|------------|
| Mensajes recibidos/día | Analytics | Diario |
| Tiempo de respuesta promedio | Logs | Diario |
| Errores de entrega | Webhook logs | Real-time |
| Consultas sin respuesta KB | Logs | Semanal |

---

### 9.7 Configuración Multi-Canal

**Archivo: config/channels.json**

```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "provider": "twilio",
      "purpose": "customer_queries",
      "features": {
        "kb_queries": true,
        "kb_management": false,
        "file_upload": false
      }
    },
    "telegram": {
      "enabled": true,
      "purpose": "admin_management",
      "features": {
        "kb_queries": true,
        "kb_management": true,
        "file_upload": true
      }
    }
  },
  "routing": {
    "default_response_channel": "same_as_input",
    "admin_notifications": "telegram",
    "error_alerts": "telegram"
  }
}
```

---

### 9.8 Diferencias de Funcionalidad por Canal

| Funcionalidad | WhatsApp | Telegram |
|---------------|:--------:|:--------:|
| Consultar KB | ✅ | ✅ |
| Subir PDF/XLS | ❌ | ✅ |
| Comandos /kb | ⚠️ Limitado | ✅ |
| Recibir archivos | ❌ | ✅ |
| Formato Markdown | ⚠️ Básico | ✅ |
| Botones inline | ❌ | ✅ |
| Rate limiting | ✅ Estricto | ⚠️ Relajado |

---

### 9.9 Ejemplo de Conversación WhatsApp

```
┌─────────────────────────────────────────┐
│ 📱 WhatsApp                             │
├─────────────────────────────────────────┤
│                                         │
│ Cliente (10:30):                        │
│ Hola, quería saber los horarios de      │
│ las clases de yoga                      │
│                                         │
│ Bot (10:30):                            │
│ ¡Hola! 👋                               │
│                                         │
│ Los horarios de clases son:             │
│                                         │
│ 🧘‍♀️ *Lunes y Miércoles*                 │
│ • 9:00 - Hatha suave                    │
│ • 18:00 - Hatha dinámico                │
│                                         │
│ 🧘‍♀️ *Martes y Jueves*                   │
│ • 10:00 - Yoga restaurativo             │
│ • 19:00 - Vinyasa                       │
│                                         │
│ 🧘‍♀️ *Sábados*                           │
│ • 10:00 - Clase abierta                 │
│                                         │
│ 📖 Fuente: Horarios 2026                │
│                                         │
│ ¿Necesitás más información? 😊          │
│                                         │
│ Cliente (10:31):                        │
│ Y cuánto sale?                          │
│                                         │
│ Bot (10:31):                            │
│ Los precios son:                        │
│                                         │
│ • Clase suelta: $5.000                  │
│ • Pack 4 clases: $18.000                │
│ • Pack 8 clases: $32.000                │
│ • Mensual libre: $45.000                │
│                                         │
│ 📖 Fuente: Lista de precios 2026        │
│                                         │
│ ¿Te interesa algún paquete? 🙏          │
│                                         │
└─────────────────────────────────────────┘
```

---

### 9.10 Troubleshooting

| Problema | Causa posible | Solución |
|----------|---------------|----------|
| No llegan mensajes | Webhook mal configurado | Verificar URL y token |
| Respuestas lentas | Timeout de API | Aumentar timeout, revisar logs |
| Mensajes cortados | Excede 4096 chars | Dividir en múltiples mensajes |
| Error de autenticación | Credenciales inválidas | Renovar token de API |
| Ban de WhatsApp | Uso no autorizado | Migrar a API oficial |

---

### 9.11 Costos Estimados

**Para 100 conversaciones/día:**

| Proveedor | Costo mensual estimado |
|-----------|----------------------|
| Twilio | $50-150 USD |
| MessageBird | $40-120 USD |
| 360Dialog | $30-100 USD |
| Meta Cloud API | $0-50 USD (con límites) |

*Nota: Costos varían según país y tipo de mensaje (session vs template)*

---

### 9.12 Checklist Final de Implementación

```markdown
## Checklist WhatsApp Integration

### Preparación
- [ ] Proveedor seleccionado
- [ ] Cuenta creada y verificada
- [ ] Número de WhatsApp Business activo
- [ ] Credenciales obtenidas

### Configuración
- [ ] Canal configurado en OpenClaw
- [ ] Webhook funcionando
- [ ] Prueba de envío/recepción OK

### Integración KB
- [ ] AGENTS.md actualizado
- [ ] Flujo de consultas definido
- [ ] Formato de respuestas configurado

### Control de Acceso
- [ ] Política de acceso definida
- [ ] Rate limiting configurado
- [ ] Admins configurados (si aplica)

### Go-Live
- [ ] Pruebas con usuarios completadas
- [ ] Tono y respuestas ajustados
- [ ] Monitoreo configurado
- [ ] Documentación actualizada
```

---

*Documento: 09-integracion-whatsapp.md | Versión: 1.0 | Fecha: 2026-03-11*
