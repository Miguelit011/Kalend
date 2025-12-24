# ARCHITECTURE — Organización del Backend

## Objetivo
Mantener una arquitectura clara, entendible y mantenible,
adecuada para un desarrollador con poca experiencia en arquitecturas complejas,
pero suficientemente sólida para un SaaS real.

La organización prioriza:
- claridad
- agrupación por funcionalidad
- facilidad de navegación
- crecimiento progresivo

---

## Enfoque general

Arquitectura **por módulos funcionales (package-by-feature)**.

Cada módulo agrupa:
- entidades
- servicios
- repositorios
- controladores
relacionados con una misma funcionalidad del negocio.

---
## Módulos principales

### appointment
Todo lo relacionado con citas.

### business
Gestión del negocio (tenant).

### service
Servicios ofrecidos por el negocio.

### client
Clientes ligeros (sin login).

### notification
Comunicación externa.

### auth
Autenticación del negocio.

### common
Código compartido.

## Reglas de organización

- Cada módulo es responsable de su lógica
- Los controllers NO contienen lógica de negocio
- Los services contienen reglas y validaciones
- Los repositories solo acceden a la base de datos
- Nada de lógica repartida sin criterio

---

## Persistencia

- Base de datos: PostgreSQL
- ORM: JPA / Hibernate
- Migraciones: Flyway (fuente única de verdad)

Cada cambio de modelo implica:
- nueva migración Flyway
- actualización de entidades
- tests correspondientes

---

## Scheduler

Se usan tareas programadas para:
- expirar reservas PENDING_CONFIRMATION
- limpieza de tokens
- (futuro) recordatorios

Los schedulers viven en el módulo correspondiente
(ej. `appointment`).

---

## Principios clave

- Multi-tenant siempre explícito (`business_id`)
- Confirmación obligatoria en reservas online
- Una cita = un servicio
- Reglas de negocio antes que comodidad técnica
- Simplicidad antes que sobre-ingeniería