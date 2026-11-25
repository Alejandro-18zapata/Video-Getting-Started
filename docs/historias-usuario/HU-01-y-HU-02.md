# Historias de Usuario — CrediYa (borrador HU1 y HU2)

> Fuente: `Enunciado del reto.docx` + `Historias de Usuario.docx`.
> Los cambios marcados como **[Propuesto]** son ajustes sugeridos para revisar juntos; el resto respeta el contenido original.

---

## Historia de Usuario: 1

### Nombre: Registrar usuarios en el sistema

**Descripción de negocio:**
Como administrador/asesor del sistema, quiero registrar un nuevo solicitante proporcionando sus datos personales básicos, incluyendo nombres y apellidos por separado, **[Propuesto: credenciales de acceso y rol]** para mantener un registro claro y ordenado de los clientes potenciales **[Propuesto: y permitir su ingreso posterior al sistema]**.

**Descripción técnica:**

- Microservicio `AUTENTICACION` con WebFlux.
- Endpoint `POST /api/v1/usuarios`.
- Debe seguir los principios de la arquitectura hexagonal, separando el dominio de la infraestructura.
- La persistencia debe ser transaccional sobre base de datos relacional (RDS).
  - **[Propuesto]** Usar `TransactionalOperator` de R2DBC en lugar de `@Transactional`, para no romper el modelo reactivo (`@Transactional` es bloqueante).
- Se deben agregar logs de traza para monitorear **[Propuesto: con `correlationId` propagado]**.
- Toda excepción debe ser manejada y procesada, para que el consumidor de la API no reciba mensajes inesperados.
  - **[Propuesto]** Manejo centralizado con `@RestControllerAdvice`: `400` para validaciones, `409` para correo duplicado y `500` controlado para errores no previstos.
- **[Propuesto]** Documentar el endpoint con OpenAPI/Swagger e incluir tests unitarios (requisitos del enunciado).
- **[Propuesto]** Responder `201 Created` con el `id` del usuario registrado, sin devolver la clave en la respuesta.

**Criterios de aceptación:**

- Se debe poder registrar un nuevo solicitante proporcionando sus datos personales: `nombres`, `apellidos`, `fecha_nacimiento`, `direccion`, `telefono`, `correo_electronico` y `salario_base`.
- El sistema debe validar que los campos `nombres`, `apellidos`, `correo_electronico` y `salario_base` no sean nulos ni vacíos (considerando también cadenas solo con espacios).
- El sistema debe validar que el `correo_electronico` proporcionado no esté previamente registrado por otro solicitante.
- El sistema debe validar el formato correcto de los datos: `salario_base` numérico entre `0` y `15.000.000`, y `correo_electronico` con formato de email válido.
- Una vez registrado exitosamente, la información del solicitante debe quedar guardada permanentemente en la base de datos.
- **[Propuesto]** El registro debe incluir `clave` (almacenada cifrada, nunca en texto plano) y `rol` (`SOLICITANTE`, `ASESOR` o `ADMIN`), necesarios para la HU3.
- **[Propuesto]** `fecha_nacimiento` no puede ser futura; `telefono` y `direccion` no pueden estar vacíos y deben tener un formato válido.
- **[Propuesto]** El correo se normaliza (minúsculas y sin espacios periféricos) antes de validar duplicados.
- **[Propuesto]** Solo un usuario autenticado con rol `ADMIN` o `ASESOR` puede invocar este endpoint (la validación efectiva se implementará en HU3).

---

## Historia de Usuario: 2

### Nombre: Registrar una solicitud de préstamo

**Descripción de negocio:**
Como cliente, quiero enviar mi solicitud de préstamo con la información necesaria (monto y plazo deseado, además del tipo de préstamo) para que CrediYa pueda evaluarla. **[Propuesto: La solicitud queda asociada al cliente autenticado.]**

**Descripción técnica:**

- Microservicio `SOLICITUDES` con WebFlux.
- Endpoint `POST /api/v1/solicitud`.
- El endpoint debe validar la información del cliente y del préstamo.
  - **[Propuesto]** Verificar de forma síncrona que el cliente existe en `AUTENTICACION` (comunicación entre microservicios).
- El código debe manejar las transacciones de forma reactiva, aprovechando las ventajas de WebFlux.
  - **[Propuesto]** Usar `TransactionalOperator` de R2DBC, coherente con HU1 y con la persistencia en RDS.
- Se deben agregar logs de traza para monitorear **[Propuesto: con `correlationId` propagado]**.
- Toda excepción debe ser manejada y procesada, para que el consumidor de la API no reciba mensajes inesperados.
  - **[Propuesto]** Manejo centralizado con `@RestControllerAdvice`: `400` validaciones, `404` tipo de préstamo/cliente no encontrado, `500` controlado.
- **[Propuesto]** Si el tipo de préstamo tiene habilitada la opción `validacion_automatica`, publicar un evento en SQS para la lambda `CAPACIDAD_DE_ENDEUDAMIENTO` ( HU7 ).
- **[Propuesto]** Guardar un snapshot de la `tasa_interes` del tipo de préstamo, necesario para el listado de la HU4.
- **[Propuesto]** Documentar el endpoint con OpenAPI/Swagger e incluir tests unitarios.

**Criterios de aceptación:**

- Se puede enviar una solicitud de crédito que incluya información del cliente (`documento de identidad`) y los detalles del préstamo (`monto`, `plazo`), además del tipo de préstamo que se quiere hacer.
- La solicitud se registra automáticamente con un estado inicial de `"Pendiente de revisión"`.
- El sistema valida que el tipo de préstamo seleccionado sea uno de los tipos de préstamo existentes.
- **[Propuesto]** `monto > 0` y `plazo > 0`, respetando los rangos definidos para el tipo de préstamo seleccionado.
- **[Propuesto]** El cliente debe existir y la solicitud solo puede crearse para el cliente autenticado (validación que se reforzará en HU3).
- **[Propuesto]** Responder `201 Created` con `solicitudId`, `estado` inicial y `fechaCreacion`.
- **[Propuesto]** Unificar los estados en un enum: `PENDIENTE_REVISION`, `APROBADA`, `RECHAZADA`, `REVISION_MANUAL` (usados por HU4, HU6 y HU7).

---

## Puntos a decidir juntos

1. **HU1:** ¿Incluimos desde ya `clave` y `rol` en el registro? Son necesarios para que HU3 pueda hacer login; si no, HU3 quedaría bloqueada.
2. **HU1:** ¿Quién puede registrar: solo `ADMIN/ASESOR`, o también permite auto-registro de `SOLICITANTE`? El enunciado habla de "solicitantes" y "roles", conviene definirlo.
3. **HU2:** ¿La identificación del cliente es por `documento_identidad`, por `usuarioId` o por ambos? Recomendación: `usuarioId` como clave y `documento_identidad` como dato de negocio.
4. **HU2:** ¿La publicación del evento SQS para validación automática se hace ya en esta historia o se difiere a HU7? Recomendación: dejar el "hook" preparado en HU2 y desarrollar la lambda en HU7.
5. **General:** ¿Los rangos de `monto`/`plazo` y la `tasa_interes` salen de un catálogo de "tipos de préstamo"? No aparece como HU numerada, pero el enunciado sí lo pide.
6. **General:** ¿Normalizamos los estados a un único enum en español o en inglés? Hay variantes en el documento (`Pendiente de revisión`, `Revision manual`, `REVISION MANUAL`, `Aprobada`).
