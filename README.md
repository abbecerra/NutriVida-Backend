# NutriVida — Backend (Node.js + Express + PostgreSQL)

API REST del sistema NutriVida, entrega EP3 de Desarrollo Full Stack 2.

## Equipo
Matías Lazo Núñez · Esteban Alonso Toloza Araneda · Abraham Becerra Muñoz.

## Enlaces
- **Swagger UI (Elastic IP indicada por el equipo):** http://54.243.53.206:3001/api-docs/
- **OpenAPI JSON:** http://54.243.53.206:3001/api-docs.json
- **Health:** http://54.243.53.206:3001/api/v1/health
- **Repositorio público del frontend:** PENDIENTE DE CREAR / PEGAR URL REAL.
- Repositorio conjunto de origen: https://github.com/matias-nu/NutriVida.P2

La dirección Swagger solo estará accesible si la instancia EC2 está encendida, la API escucha en 3001 y el grupo de seguridad permite el acceso. El equipo comprobó HTTP 200 en localhost y la anterior IP pública; verificar de nuevo la nueva Elastic IP antes del envío.

## Decisiones técnicas
- Express 5 para rutas REST y respuestas JSON.
- PostgreSQL con `pg.Pool`, usando parámetros SQL para valores de entrada.
- Conexión a Amazon RDS mediante certificado CA `global-bundle.pem` (no se incluye en ZIP por seguridad).
- Autenticación JWT (vigencia 1 hora), hash de contraseñas con bcryptjs (factor 12), middleware de roles y comprobación de usuario activo.
- Baja lógica de usuarios, servicios y planes mediante `activo=FALSE`.
- Swagger UI y OpenAPI definidos en `swagger.js`.

## Variables de entorno privadas
`PORT` (opcional, predeterminado 3001), `DB_HOST`, `DB_PORT` (predeterminado 5432), `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `JWT_SECRET`.

El archivo `backend/db.js` requiere el certificado `global-bundle.pem` en la carpeta backend. **No publicar `.env`, contraseñas, claves ni tokens.**

## Instalación y ejecución
1. Instalar Node.js y npm.
2. Ejecutar `npm ci` dentro de `backend`.
3. Configurar variables de entorno privadas y el certificado CA de RDS.
4. Ejecutar `npm start` (equivale a `node index.js`).
5. Consultar `http://localhost:3001/api/v1/health` y `http://localhost:3001/api-docs/` desde el servidor.

## Entidades y relaciones (modelo lógico deducido del código)
- `roles` (id, nombre) → `usuarios.rol_id`.
- `usuarios` (id, nombre, correo, password_hash, rol_id, activo) → pacientes y nutricionistas.
- `servicios` → `citas.servicio_id`.
- `planes_nutricionales` → catálogo de planes.
- `citas` → `paciente_id` y `nutricionista_id` referencian `usuarios.id`; `servicio_id` referencia `servicios.id`.

**Límite:** no se incluyó el DDL de PostgreSQL; el diagrama lógico es una inferencia de las consultas, no una exportación de RDS. El código transforma el error PostgreSQL 23505 en HTTP 409 al crear una cita; la restricción física que origina ese error no está en el ZIP.

## Endpoints principales
Prefijo `/api/v1`:
- Autenticación: `POST /auth/registro`, `POST /auth/login`, `GET /auth/perfil`.
- Usuarios (Administrador): `GET /usuarios`, `GET /usuarios/:id`, `POST /usuarios`, `PUT /usuarios/:id`, `DELETE /usuarios/:id`.
- Servicios y planes: `GET` público; `POST`, `PUT /:id`, `DELETE /:id` solo Administrador.
- Nutricionistas: `GET /nutricionistas`, `GET /nutricionistas/:id/horarios?fecha=AAAA-MM-DD`.
- Citas: `GET /citas`, `POST /citas` (Paciente), `PATCH /citas/:id/estado`.

La documentación detallada está en `05_Documentacion_API_NutriVida_EP3` dentro del paquete de entrega.

## Diagrama de secuencia: agendamiento
1. React consulta servicios, nutricionistas y horarios.
2. Paciente autenticado envía `POST /api/v1/citas` con JWT.
3. Middleware verifica token y rol Paciente.
4. Express valida fecha, profesional y servicio; consulta/inserta en PostgreSQL.
5. API responde `201` en éxito, `400` por datos inválidos o `409` si PostgreSQL emite 23505.
6. React muestra confirmación o error.

## Estado del trabajo
Implementados en código: JWT, roles, usuarios CRUD, servicios CRUD, planes CRUD, consultas de nutricionistas, listado y creación de citas, cambio de estado, Swagger. No hay rutas `GET /citas/:id`, `PUT /citas/:id` ni `DELETE /citas/:id` en el ZIP revisado. No se ejecutaron pruebas automatizadas nuevas del backend; `npm test` no contiene tests reales.

## Seguridad
La URL pública actual usa HTTP y expone la API completa en el puerto 3001. Es una configuración temporal de demostración, no recomendada para producción. No enviar credenciales reales por HTTP sin TLS.
