# Estado Inicial del Proyecto - GestionDocumentos

**Fecha de registro**: 2026-02-17  
**Fase**: Planificación (sin código implementado)

## 1. Resumen del Proyecto

**GestionDocumentos** es una aplicación web local para gestionar documentos personales (DNI, facturas, contratos, permisos, seguros) destinada a familias y autónomos. El proyecto está en fase de planificación con documentación detallada, pero sin código implementado todavía.

### Objetivos del MVP
- Subida manual de documentos (PDF, PNG, JPG)
- Clasificación automática de documentos
- Extracción de fechas de caducidad y emisión
- Alertas de caducidad (solo UI, sin emails)
- Panel resumen con documentos organizados
- Todo funciona localmente (sin cloud)

## 2. Stack Tecnológico Definido

### Backend
- **Java 21** con **Spring Boot 3** (Maven)
- **PostgreSQL 14+** para base de datos
- **JWT** para autenticación
- **Flyway** para migraciones de base de datos
- **JUnit 5** para testing
- **Springdoc OpenAPI** para documentación API

### Frontend
- **Vue 3** con **Vite**
- **TypeScript** (recomendado)
- **Pinia** para gestión de estado
- **Vue Router** para navegación
- **Axios** para API calls
- **Vitest** para testing

### Almacenamiento
- Archivos guardados en carpeta local: `./_storage`
- Estructura: `./_storage/{userId}/{documentUuid}.{ext}`

## 3. Arquitectura del Sistema

```
[Vue 3 SPA] → [Spring Boot REST API] → [PostgreSQL Local]
                      ↓
                [Filesystem: ./_storage]
```

**Base URL API**: `http://localhost:8080/api/v1`  
**Frontend Dev**: `http://localhost:5173`

## 4. Modelo de Datos (PostgreSQL)

### Tablas principales

#### app_user
- `id` (UUID, PK)
- `email` (VARCHAR 255, UNIQUE)
- `password_hash` (VARCHAR 255)
- `created_at` (TIMESTAMPTZ)

#### document
- `id` (UUID, PK)
- `user_id` (UUID, FK → app_user)
- `filename` (VARCHAR 255)
- `mime` (VARCHAR 100)
- `size_bytes` (BIGINT)
- `storage_path` (TEXT)
- `checksum` (VARCHAR 64, SHA256)
- `created_at` (TIMESTAMPTZ)

#### doc_metadata
- `document_id` (UUID, PK/FK → document, CASCADE)
- `type` (VARCHAR 50: dni|pasaporte|permiso|seguro|factura|contrato|otros)
- `issuer` (VARCHAR 255)
- `person` (VARCHAR 255)
- `date_issue` (DATE)
- `date_expiry` (DATE)
- `amount` (NUMERIC 14,2)
- `currency` (VARCHAR 8)
- `tags` (JSONB)
- `confidence` (NUMERIC 3,2)

#### reminder
- `id` (UUID, PK)
- `document_id` (UUID, FK → document, CASCADE)
- `due_date` (DATE)
- `channel` (VARCHAR 20: email|none)
- `status` (VARCHAR 20: pending|sent|dismissed)
- `created_at` (TIMESTAMPTZ)

### Índices
- `idx_document_user` en `document(user_id)`
- `idx_metadata_type` en `doc_metadata(type)`
- `idx_metadata_expiry` en `doc_metadata(date_expiry)`

## 5. Contratos de API REST

### Autenticación
- `POST /auth/register` → 201 (email, password)
- `POST /auth/login` → 200 (retorna accessToken, expiresIn)

### Documentos
- `POST /documents` → 201 (multipart/form-data: file, opcional type)
- `GET /documents?query=&type=&from=&to=&page=&size=` → 200 (lista paginada)
- `GET /documents/{id}` → 200 (detalle con metadata)
- `GET /documents/{id}/content` → 200 (stream del archivo)
- `DELETE /documents/{id}` → 204

### Metadatos
- `GET /documents/{id}/metadata` → 200
- `PUT /documents/{id}/metadata` → 200

### Recordatorios
- `POST /documents/{id}/reminders` → 201 (due_date, channel)
- `GET /documents/{id}/reminders` → 200 (lista)
- `PATCH /reminders/{id}` → 200 (actualizar status)

**Autenticación**: Todas las rutas excepto `/auth/*` requieren `Authorization: Bearer <token>`

## 6. Estructura de Proyecto (Futura)

```
GestionDocumentos/
├── backend/                    # Spring Boot (no creado aún)
│   ├── src/main/java/com/example/docsguard/
│   │   ├── controller/        # REST endpoints
│   │   ├── service/           # Lógica de negocio
│   │   ├── domain/            # JPA entities
│   │   ├── repository/        # Data access
│   │   ├── dto/               # DTOs
│   │   ├── config/            # Configuración Spring
│   │   └── security/          # JWT filters, SecurityConfig
│   ├── src/main/resources/
│   │   ├── application.yml
│   │   └── db/migration/      # Flyway migrations
│   └── src/test/java/         # Tests JUnit
├── frontend/                   # Vue 3 app (no creado aún)
│   ├── src/
│   │   ├── components/
│   │   ├── views/
│   │   ├── stores/            # Pinia
│   │   ├── api/               # Axios clients
│   │   └── router/
│   └── .env.local
├── _storage/                   # Almacenamiento local (crear)
├── REQUISITOS.md               # ✓ Requisitos MVP
├── Braninstorming.md          # ✓ Especificación técnica detallada
├── AGENTS.md                   # ✓ Guías de desarrollo
└── CLAUDE.md                   # ✓ Instrucciones para Claude Code
```

## 7. Documentación Disponible

### REQUISITOS.md
- Visión general del MVP
- Funcionalidades principales (organización, extracción fechas, alertas)
- Casos de uso (familia, autónomo, usuario general)
- Límites del MVP (local-only, sin OCR real, sin cloud)
- Tareas inmediatas

### Braninstorming.md
- Arquitectura detallada
- Modelo de datos completo con SQL
- Contratos API REST con ejemplos curl
- Configuración backend (pom.xml, application.yml)
- Configuración frontend (Vite, dependencias)
- Requisitos no funcionales (seguridad, validaciones)
- Roadmap MVP+1

### AGENTS.md
- Estructura de módulos
- Comandos de build/test/desarrollo
- Convenciones de código (Java 4 spaces, Vue 2 spaces)
- Guías de testing
- Convenciones de commits (Conventional Commits)
- Tips de seguridad

### CLAUDE.md
- Principios de desarrollo (constitution-based)
- Comandos de desarrollo
- Estructura del proyecto
- Coding standards (camelCase backend, kebab-case frontend)
- Testing standards
- Commit conventions
- Security checklist
- Integración con Speckit

## 8. Configuración de Desarrollo

### Backend (cuando se cree)
```bash
./mvnw clean install       # Build + tests
./mvnw spring-boot:run     # Ejecutar backend
./mvnw test                # Solo tests
```

### Frontend (cuando se cree)
```bash
npm install                # Instalar dependencias
npm run dev                # Dev server
npm run lint               # Linter
npm run test:unit          # Tests
```

### Base de Datos
```bash
# Opción 1: Docker Compose
docker compose up -d

# Opción 2: PostgreSQL local
# Database: docsguard
# User: docsguard
# Password: (en application-local.yml)
```

## 9. Requisitos No Funcionales

### Seguridad
- BCrypt para passwords
- JWT con expiración 15-30 min
- Validación de entrada (Bean Validation)
- Autorización por user_id
- Validación MIME y extensión de archivos
- Tamaño máximo archivo: 10MB
- CORS restringido a localhost:5173

### Validaciones de Subida
- Extensiones permitidas: pdf|png|jpg|jpeg
- Verificación MIME en servidor
- Sanitización de filename
- Renombrado a UUID

## 10. Alcance del MVP

### Incluido
✓ Registro e inicio de sesión (JWT)  
✓ Subida de documentos (PDF/JPG/PNG)  
✓ Clasificación automática básica  
✓ Extracción de fechas (stub, sin OCR real)  
✓ Listado con filtros (texto, tipo, fechas)  
✓ Vista/edición de metadatos  
✓ Creación/listado de recordatorios  
✓ Alertas de caducidad (solo UI)  
✓ Descarga de archivos  
✓ Eliminación de documentos  

### Excluido del MVP
✗ Cloud storage o sincronización  
✗ Emails/SMS externos  
✗ OCR/IA real (solo stub)  
✗ Multi-usuario o multi-dispositivo  
✗ Cofres, compartición, firmas  
✗ Exportación avanzada  
✗ Autenticación avanzada (OAuth, 2FA)  

## 11. Principios de Constitución del Proyecto

Según `.specify/memory/constitution.md`, el proyecto sigue estos principios inmutables:

1. **API-First Architecture**
2. **Security by Design**
3. **Test-Driven Quality**
4. **Data Integrity & Migrations**
5. **Local-First Development**
6. **Clear Separation of Concerns**
7. **Pragmatic Simplicity (YAGNI)**

## 12. Estado Actual de Archivos

```
✓ REQUISITOS.md          - Requisitos funcionales MVP
✓ Braninstorming.md      - Especificación técnica completa
✓ AGENTS.md              - Guías de desarrollo
✓ CLAUDE.md              - Instrucciones Claude Code
✗ backend/               - No creado
✗ frontend/              - No creado
✗ _storage/              - No creado
✗ docker-compose.yml     - No creado
```

## 13. Próximos Pasos Sugeridos

1. Crear estructura de directorios (backend/, frontend/, _storage/)
2. Configurar proyecto Spring Boot con Maven
3. Configurar proyecto Vue 3 con Vite
4. Implementar migraciones Flyway (V1__init.sql)
5. Configurar PostgreSQL local o Docker
6. Implementar autenticación JWT
7. Implementar endpoints de documentos
8. Implementar frontend básico (login, registro, upload)
9. Testing de integración
10. Documentación de deployment local

## 14. Ejemplos de Uso (Futuros)

### Registro
```bash
curl -X POST http://localhost:8080/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@mail.com","password":"Secret123!"}'
```

### Login
```bash
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@mail.com","password":"Secret123!"}'
```

### Subida de documento
```bash
curl -X POST http://localhost:8080/api/v1/documents \
  -H "Authorization: Bearer <TOKEN>" \
  -F "file=@/ruta/a/archivo.pdf"
```

### Listar documentos
```bash
curl -H "Authorization: Bearer <TOKEN>" \
  "http://localhost:8080/api/v1/documents?type=factura&page=0&size=20"
```

---

**Conclusión**: El proyecto está completamente especificado y documentado, listo para comenzar la implementación. Todas las decisiones arquitectónicas, tecnológicas y de diseño están tomadas y documentadas.