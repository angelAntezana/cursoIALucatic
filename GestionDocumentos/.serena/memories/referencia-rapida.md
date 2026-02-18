# Referencia Rápida - GestionDocumentos

## Estado: PLANIFICACIÓN (sin código)

## Stack
- **Backend**: Java 21 + Spring Boot 3 + PostgreSQL 14+
- **Frontend**: Vue 3 + Vite + TypeScript
- **Auth**: JWT
- **Storage**: Local filesystem (`./_storage`)

## URLs
- Backend: `http://localhost:8080`
- Frontend: `http://localhost:5173`
- API Base: `http://localhost:8080/api/v1`
- Swagger: `http://localhost:8080/swagger-ui`

## Estructura de Proyecto (Futura)
```
backend/          → Spring Boot app (no creado)
frontend/         → Vue 3 app (no creado)
_storage/         → Archivos locales (no creado)
REQUISITOS.md     → ✓ Requisitos funcionales
Braninstorming.md → ✓ Especificación técnica
AGENTS.md         → ✓ Guías desarrollo
CLAUDE.md         → ✓ Instrucciones Claude
```

## Comandos de Desarrollo

### Backend (cuando exista)
```bash
./mvnw spring-boot:run    # Ejecutar
./mvnw test               # Tests
./mvnw clean install      # Build completo
```

### Frontend (cuando exista)
```bash
npm run dev         # Dev server
npm run test:unit   # Tests
npm run lint        # Linter
```

## Base de Datos (PostgreSQL)
- Database: `docsguard`
- User: `docsguard`
- Tablas: `app_user`, `document`, `doc_metadata`, `reminder`

## API Endpoints Principales

### Auth
- `POST /auth/register` - Registro
- `POST /auth/login` - Login (retorna JWT)

### Documents
- `POST /documents` - Upload (multipart)
- `GET /documents` - Lista con filtros
- `GET /documents/{id}` - Detalle
- `GET /documents/{id}/content` - Descargar
- `DELETE /documents/{id}` - Eliminar

### Metadata & Reminders
- `GET/PUT /documents/{id}/metadata`
- `POST /documents/{id}/reminders`
- `PATCH /reminders/{id}`

## Tipos de Documentos
`dni`, `pasaporte`, `permiso`, `seguro`, `factura`, `contrato`, `otros`

## Límites
- Tamaño archivo: ≤10MB
- Formatos: PDF, PNG, JPG, JPEG
- JWT expiración: 15-30 min
- Storage: `./_storage/{userId}/{uuid}.{ext}`

## Convenciones de Código

### Backend (Java)
- 4 espacios de indentación
- `camelCase` métodos/variables
- `PascalCase` clases
- `snake_case` columnas DB
- Packages: controller, service, domain, repository, dto, config, security

### Frontend (Vue)
- 2 espacios de indentación
- `PascalCase` nombres de componentes
- `kebab-case` props/events
- CSS scoped por componente

## Commits
Formato: `tipo(scope): descripción`
- `feat(backend): add document upload`
- `fix(frontend): handle empty files`
- `docs: update setup instructions`

## Seguridad Checklist
- [ ] BCrypt passwords
- [ ] JWT firmado
- [ ] Validación MIME + extensión
- [ ] Tamaño máximo archivo
- [ ] Filtrar por user_id
- [ ] CORS restringido
- [ ] No credentials en git

## Documentos a Consultar
1. **REQUISITOS.md** - Para entender funcionalidades MVP
2. **Braninstorming.md** - Para detalles técnicos y API
3. **AGENTS.md** - Para guías de código y testing
4. **CLAUDE.md** - Para principios y estándares
5. **`.specify/memory/constitution.md`** - Para principios inmutables

## MVP Scope
**IN**: Upload, clasificación básica, extracción fechas, alertas UI, dashboard, filtros, metadata, recordatorios  
**OUT**: Cloud, emails externos, OCR real, multi-user, cofres, firmas, OAuth

## Próximos Pasos
1. Crear backend/ con Spring Initializr
2. Crear frontend/ con Vite
3. Configurar PostgreSQL
4. Implementar auth JWT
5. Implementar endpoints documentos
6. Implementar UI básico