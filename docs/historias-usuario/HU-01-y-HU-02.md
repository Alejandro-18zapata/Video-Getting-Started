# Historias de Usuario — CrediYa (HU1 y HU2)

> Fuente: `Historias de Usuario.docx` y `Enunciado del reto.docx`.

---

## Historia de Usuario: 1

**Nombre:** Registrar usuarios en el sistema

**Descripción de negocio:**
Como administrador/asesor del sistema, quiero registrar un nuevo usuario proporcionando sus datos personales básicos, incluyendo nombres y apellidos por separado, para mantener un registro claro y ordenado de los clientes potenciales.

**Descripción técnica:**

- Microservicio `AUTENTICACION` con WebFlux.
- Endpoint `POST /api/v1/usuarios`.
- Debe seguir los principios de la arquitectura hexagonal, separando el dominio de la infraestructura.
- La persistencia de datos debe ser transaccional para garantizar la atomicidad de la operación de guardado (ej. usando anotaciones `@Transactional`).
- Se deben agregar logs de traza para monitorear.
- Toda excepción que se produzca debe ser manejada y procesada, para que el usuario que usa la API no le lleguen mensajes inesperados.

**Criterios de aceptación:**

- Se debe poder registrar un nuevo solicitante proporcionando sus datos personales: `nombres`, `apellidos`, `fecha_nacimiento`, `direccion`, `telefono`, `correo_electronico` y `salario_base`.
- El sistema debe validar que los campos `nombres`, `apellidos`, `correo_electronico` y `salario_base` no sean nulos o vacíos.
- El sistema debe validar que el `correo_electronico` proporcionado no esté previamente registrado por otro solicitante.
- El sistema debe validar el formato correcto de los datos, como por ejemplo, que el `salario_base` sea un valor numérico (esté entre 0 y 15000000) y el `correo_electronico` tenga un formato de email válido.
- Una vez registrado exitosamente, la información del solicitante debe quedar guardada permanentemente en la base de datos.

---

## Historia de Usuario: 2

**Nombre:** Registrar una solicitud de préstamo

**Descripción de negocio:**
Como cliente, quiero enviar mi solicitud de préstamo con la información necesaria (monto y plazo deseado) para que CrediYa pueda evaluarla.

**Descripción técnica:**

- Microservicio `SOLICITUDES` con WebFlux.
- Endpoint `POST /api/v1/solicitud`.
- El endpoint debe validar la información del cliente y del préstamo.
- El código debe manejar las transacciones de forma reactiva, aprovechando las ventajas de WebFlux.
- Se deben agregar logs de traza para monitorear.
- Toda excepción que se produzca debe ser manejada y procesada, para que el usuario que usa la API no le lleguen mensajes inesperados.

**Criterios de aceptación:**

- Se puede enviar una solicitud de crédito que incluya información del cliente (documento de identidad) y los detalles del préstamo (`monto`, `plazo`), el tipo de préstamo que se quiere hacer.
- La solicitud se registra automáticamente con un estado inicial de "Pendiente de revisión".
- El sistema valida que el tipo de préstamo seleccionado sea uno de los tipos de préstamo existentes.
