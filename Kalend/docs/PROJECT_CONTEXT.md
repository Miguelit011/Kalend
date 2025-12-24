# PROJECT CONTEXT — SaaS de Gestión de Citas

## Visión general
Esta aplicación es un SaaS de gestión de citas para negocios locales
(peluquerías, clínicas, centros de estética, talleres, gimnasios, etc.).

El sistema permite:
- Reservas online SIN crear cuenta (cliente)
- Gestión de agenda y citas desde un panel privado (negocio)
- Confirmación obligatoria por enlace enviado al teléfono (WhatsApp/SMS)
- Convivencia de citas online y citas presenciales
- Control de abusos SIN usar IP (solo teléfono + reglas de negocio)

La aplicación es multi-tenant: un solo sistema sirve a múltiples negocios.
Todos los datos están aislados por `business_id`.

---

## Actores

### Cliente (no autenticado)
- No tiene cuenta ni contraseña
- Se identifica solo por teléfono (por negocio)
- Reserva citas, confirma y cancela mediante enlaces
- Puede tener varias citas para distintos servicios y días

### Negocio (autenticado)
- Usuarios internos: OWNER y STAFF
- Acceden a un panel privado
- Gestionan servicios, horarios y citas
- Pueden crear citas presenciales manualmente

---

## Conceptos clave del dominio

### Business (Negocio)
- Identifica al tenant
- Tiene un `business_id` interno y un `businessSlug` público (URL)
- Configura reglas del negocio:
  - margen de cancelación
  - límite de citas activas por teléfono
  - tiempo de expiración de reservas pendientes

### Service (Servicio)
- Lo que el negocio ofrece
- Campos clave:
  - nombre
  - duración
  - buffer antes/después
  - activo/inactivo
- Un servicio inactivo no se puede reservar online

### Client (Cliente ligero)
- Identificado por `business_id + phone`
- No es un usuario autenticado
- Guarda historial y control:
  - no_show_count
  - late_cancel_count
  - blocked_until
- Un cliente puede tener múltiples citas

### Appointment (Cita)
Regla central:
> Una cita = un servicio = un bloque de tiempo

- Un cliente puede tener muchas citas
- Cada cita corresponde a un único servicio
- Campos clave:
  - service_id
  - client_id
  - start_time / end_time
  - status

### AppointmentToken (Token)
- Permite acciones sin login
- Siempre asociado a una única cita
- Tipos:
  - CONFIRM (confirmar cita, uso único, corta duración)
  - MANAGE (ver/cancelar cita, duración media)

---

## Estados de cita

Estados posibles:
- PENDING_CONFIRMATION
- CONFIRMED
- CANCELLED
- COMPLETED
- NO_SHOW
- EXPIRED

Transiciones válidas:
- Crear reserva online → PENDING_CONFIRMATION
- Confirmar por enlace → CONFIRMED
- Expirar automáticamente → EXPIRED
- Cancelar (cliente o staff) → CANCELLED
- Completar (staff) → COMPLETED
- No asistencia (staff) → NO_SHOW

Reglas:
- Una cita expirada no puede confirmarse
- Una cita completada o no-show no puede cancelarse
- Solo CONFIRMED bloquea disponibilidad real

---

## Flujo de reserva online

1. Cliente elige servicio
2. Consulta huecos disponibles
3. Introduce nombre y teléfono
4. Se crea cita en PENDING_CONFIRMATION
5. Se envía enlace por WhatsApp/SMS
6. El cliente confirma → cita pasa a CONFIRMED

Las reservas pendientes expiran automáticamente si no se confirman.

---

## Citas presenciales (staff)

- Se crean desde el panel del negocio
- Son el mismo tipo de entidad `Appointment`
- Se crean directamente como CONFIRMED
- No requieren token ni confirmación
- Bloquean agenda igual que una cita online

---

## Disponibilidad

- Los huecos NO se almacenan
- Se calculan dinámicamente según:
  - horarios semanales
  - excepciones (festivos, vacaciones)
  - duración y buffers del servicio
  - citas CONFIRMED existentes

---

## Reglas anti-abuso (sin IP)

- Límite de citas activas por teléfono y negocio
  - Cita activa = PENDING_CONFIRMATION + CONFIRMED
- Expiración rápida de reservas pendientes
- Bloqueos temporales por:
  - no-shows repetidos
  - cancelaciones tardías frecuentes

---

## Fuera de alcance (MVP)

- Pagos online
- Recordatorios automáticos
- IA
- Multi-idioma
- Marketplace o búsqueda global
