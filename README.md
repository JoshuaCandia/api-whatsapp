# WhatsApp Chatbot & Sistema de Agentes

<div align="center">

![WhatsApp](https://img.shields.io/badge/WhatsApp-25D366?style=for-the-badge&logo=whatsapp&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-2D3748?style=for-the-badge&logo=prisma&logoColor=white)
![Socket.io](https://img.shields.io/badge/Socket.io-010101?style=for-the-badge&logo=socket.io&logoColor=white)
![Google Cloud](https://img.shields.io/badge/Google_Cloud-4285F4?style=for-the-badge&logo=google-cloud&logoColor=white)

**Sistema inteligente de atención al cliente vía WhatsApp con gestión de vendedores multi-sucursal**

</div>

---

## ¿Qué hace?

Este sistema combina un **chatbot conversacional** con un **panel de gestión para vendedores**, permitiendo atender clientes de WhatsApp de forma automatizada y escalable.

### El Bot

- **Responde automáticamente** consultas frecuentes (horarios, ubicación, envíos, formas de pago)
- **Guía al cliente** paso a paso para solicitar cotizaciones
- **Detecta intenciones** y deriva a un vendedor cuando es necesario
- **Maneja multimedia** (imágenes, videos, documentos, ubicaciones)
- **Soporta mensajes interactivos** (botones, listas de opciones)

### El Sistema de Agentes

- **4 niveles jerárquicos**: Vendedor → Encargado → Zonal → Gerencia
- **Control de acceso granular**: cada rol ve solo lo que le corresponde
- **Asignación inteligente**: conversaciones se asignan según disponibilidad
- **Chat en tiempo real**: WebSockets para actualizaciones instantáneas
- **Multi-sucursal**: organizado por zonas y tiendas

---

## Flujos Conversacionales

### Flujo de Información

El cliente puede consultar sin intervención humana:

```
Cliente: "Hola, ¿qué horarios tienen?"
Bot: 🕐 Horarios de atención:
     📅 Lunes a Viernes: 8:00 a 18:00 hs
     📅 Sábados: 8:00 a 13:00 hs

     ¿Necesitas información sobre otro tema?
     [Ubicación] [Envíos] [Formas de pago] [Contacto]
```

**Temas disponibles:**
- Horarios de atención
- Ubicación y cómo llegar
- Envíos y delivery
- Formas de pago y financiación
- Garantía y devoluciones
- Contacto

### Flujo de Cotización

Guía completa para presupuestos:

```
1. ¿Qué producto te interesa?
   [Cerámicos] [Porcelanato] [Vinílicos] [Azulejos] [Mosaicos]

2. Contame los detalles (medidas, color, modelo)

3. ¿Qué cantidad necesitás?

4. ¿Cómo indicamos tu ubicación?
   [Enviar ubicación GPS] [Elegir zona manualmente]

5. ¿Cómo te contactamos?
   [Por este WhatsApp] [Por email] [Llamarme]

6. ¡Listo! Un vendedor te contactará en breve
```

---

## Arquitectura

```
┌─────────────────────────────────────────────────────────────┐
│                       CLIENTE                                │
│                    (WhatsApp User)                           │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                  META WEBHOOK API                            │
│              (WhatsApp Cloud Platform)                       │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    API SERVER                                │
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Webhook    │  │   Agents    │  │     WebSocket       │  │
│  │ Controller  │  │     API     │  │      Server         │  │
│  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘  │
│         │                │                     │             │
│         ▼                ▼                     ▼             │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                  APPLICATION LAYER                       ││
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────┐  ││
│  │  │   Bot    │ │  Intent  │ │   Flow   │ │   Agent    │  ││
│  │  │ Service  │ │ Detector │ │ Manager  │ │  Service   │  ││
│  │  └──────────┘ └──────────┘ └──────────┘ └────────────┘  ││
│  └─────────────────────────────────────────────────────────┘│
│                           │                                  │
│                           ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                   DOMAIN LAYER                           ││
│  │    Customer • Conversation • Message • Agent • Store     ││
│  └─────────────────────────────────────────────────────────┘│
│                           │                                  │
│                           ▼                                  │
│  ┌──────────────┐  ┌────────────┐  ┌──────────────────────┐ │
│  │  PostgreSQL  │  │    GCS     │  │   WhatsApp Cloud     │ │
│  │   (Prisma)   │  │  Storage   │  │       API            │ │
│  └──────────────┘  └────────────┘  └──────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Stack Tecnológico

| Capa | Tecnología |
|------|------------|
| Runtime | Node.js + TypeScript |
| Framework | Express.js v5 |
| Base de Datos | PostgreSQL + Prisma ORM |
| Tiempo Real | Socket.IO |
| Almacenamiento | Google Cloud Storage |
| Autenticación | JWT + bcrypt |
| Mensajería | WhatsApp Cloud API (Meta) |

---

## Estructura del Proyecto

```
src/
├── chatbot/                    # Lógica del bot
│   ├── application/
│   │   ├── flows/              # Flujos conversacionales
│   │   │   ├── info.flow.ts
│   │   │   └── quotation.flow.ts
│   │   └── services/
│   │       ├── bot.service.ts
│   │       ├── intent-detector.service.ts
│   │       └── flow-manager.service.ts
│   └── infrastructure/
│       └── repositories/
│
├── agents/                     # Sistema de vendedores
│   ├── domain/
│   │   └── entities/agent.entity.ts
│   ├── application/
│   │   └── services/agent-conversation.service.ts
│   └── infrastructure/
│       ├── controllers/agent.controller.ts
│       ├── middleware/auth.middleware.ts
│       └── routes/agent.routes.ts
│
├── messaging/                  # Envío de mensajes
│   ├── domain/
│   │   └── entities/message.entity.ts
│   └── infrastructure/
│       └── adapters/whatsapp-cloud.adapter.ts
│
├── webhook/                    # Recepción de mensajes
│   └── infrastructure/
│       └── controllers/webhook.controller.ts
│
└── shared/
    ├── config/env.config.ts
    └── infrastructure/
        ├── database/prisma.service.ts
        ├── storage/gcs.service.ts
        └── websocket/socket.service.ts
```

---

## Instalación

### Requisitos

- Node.js 18+
- PostgreSQL 14+
- Cuenta de Meta Business (WhatsApp Cloud API)
- Proyecto en Google Cloud (para storage)

### Pasos

```bash
# Clonar el repo
git clone https://github.com/JoshuaCandia/api-whatsapp.git
cd api-whatsapp

# Instalar dependencias
npm install

# Configurar variables de entorno
cp .env.example .env
# Editar .env con tus credenciales

# Crear base de datos
npx prisma migrate dev

# Cargar datos iniciales (zonas, tiendas, agentes de prueba)
npx prisma db seed

# Iniciar en desarrollo
npm run dev
```

---

## Configuración

### Variables de Entorno

```env
# Servidor
PORT=3000

# Meta/WhatsApp
META_ACCESS_TOKEN=tu_token_de_acceso
META_PHONE_NUMBER_ID=tu_numero_de_telefono
META_VERIFY_TOKEN=token_para_verificar_webhook

# Base de Datos
DATABASE_URL="postgresql://user:password@localhost:5432/chatbot"

# JWT
JWT_SECRET=tu_secreto_super_seguro
JWT_EXPIRES_IN=24h

# Google Cloud Storage
GOOGLE_APPLICATION_CREDENTIALS=credentials/tu-archivo.json
GCS_PROJECT_ID=tu-proyecto
GCS_PUBLIC_BUCKET_NAME=tu-bucket-publico
GCS_PRIVATE_BUCKET_NAME=tu-bucket-privado
```

### Configurar Webhook en Meta

1. Ir a [Meta for Developers](https://developers.facebook.com)
2. Crear/seleccionar tu app
3. Agregar el producto "WhatsApp"
4. En Configuración > Webhooks:
   - URL: `https://tu-dominio.com/webhook`
   - Token: el mismo que pusiste en `META_VERIFY_TOKEN`
   - Suscribirse a: `messages`

---

## API Endpoints

### Autenticación

```http
POST /api/agents/auth/login
Content-Type: application/json

{
  "email": "vendedor@example.com",
  "password": "123456"
}
```

Respuesta:
```json
{
  "agent": {
    "id": "clx...",
    "name": "Juan Vendedor",
    "role": "SELLER",
    "store": { "name": "Tienda 1 - Belgrano" }
  },
  "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

### Conversaciones

```http
# Obtener conversaciones en espera
GET /api/agents/conversations/waiting
Authorization: Bearer <token>

# Obtener mis conversaciones asignadas
GET /api/agents/conversations/mine
Authorization: Bearer <token>

# Ver detalle de conversación con mensajes
GET /api/agents/conversations/:id
Authorization: Bearer <token>

# Asignar conversación
POST /api/agents/conversations/:id/assign
Authorization: Bearer <token>

# Enviar mensaje
POST /api/agents/conversations/:id/messages
Authorization: Bearer <token>
Content-Type: application/json

{
  "message": "Hola! Te paso la cotización..."
}

# Resolver conversación
POST /api/agents/conversations/:id/resolve
Authorization: Bearer <token>
```

### WebSocket Events

```javascript
// Conectar
const socket = io('http://localhost:3000', {
  auth: { token: 'tu_jwt_token' }
});

// Eventos que recibís
socket.on('message:new', (data) => {
  // Nuevo mensaje de cliente
});

socket.on('conversation:waiting', (data) => {
  // Nueva conversación esperando asignación
});

socket.on('conversation:assigned', (data) => {
  // Conversación fue asignada
});
```

---

## Roles y Permisos

| Rol | Acceso | Descripción |
|-----|--------|-------------|
| `SELLER` | Sus propias conversaciones | Vendedor de piso |
| `MANAGER` | Todas las de su tienda | Encargado de tienda |
| `ZONE_SUPERVISOR` | Todas las de su zona | Supervisor zonal |
| `REGIONAL_MANAGER` | Todas | Gerencia regional |

---

## Usuarios de Prueba

Después de ejecutar el seed:

```
GERENCIA
└─ gerente@example.com / 123456

ZONALES
├─ zonal.norte@example.com / 123456
├─ zonal.sur@example.com / 123456
└─ zonal.oeste@example.com / 123456

ENCARGADOS
├─ encargado.r1@example.com / 123456 (Tienda Belgrano)
└─ encargado.r2@example.com / 123456 (Tienda Palermo)

VENDEDORES
├─ vendedor1.r1@example.com / 123456
├─ vendedor2.r1@example.com / 123456
└─ vendedor1.r2@example.com / 123456
```

---

## Scripts Disponibles

```bash
npm run dev       # Desarrollo con hot-reload
npm run build     # Compilar TypeScript
npm start         # Producción
npm run seed      # Cargar datos iniciales
npm run migrate   # Ejecutar migraciones
```

---

## Próximas Features

- [ ] Dashboard web para supervisores
- [ ] Envío de imágenes desde agentes
- [ ] Transcripción de notas de voz
- [ ] Analytics y reportes
- [ ] Integración con CRM
- [ ] Templates de mensajes aprobados por Meta

---

## Licencia

Desarrollado para una empresa de retail. Todos los derechos reservados.

---

<div align="center">

Desarrollado con mass cafe y mass paciencia

</div>
# api-whatsapp
