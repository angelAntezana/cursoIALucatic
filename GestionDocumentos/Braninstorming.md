
# 📄 Requisitos y Guía Local — Gestor de Documentos Personales  
**Backend:** Java 21 + Spring Boot 3 (Maven)  
**Frontend:** Vue 3 (SPA con Vite)  
**Base de datos:** PostgreSQL  
**Almacenamiento de archivos:** Carpeta local en tu máquina  
**Despliegue:** Todo en local (Dev)

---

## 0) Objetivo

Construir un **MVP local** de una app web para **gestionar documentos personales** con:
- Login (JWT).
- API REST para documentos, metadatos y recordatorios.
- SPA en Vue 3.
- PostgreSQL local.
- Subida de archivos a **una carpeta local** (p. ej. `./_storage`).

---

## 1) Alcance funcional (MVP)

- **Autenticación**: registro e inicio de sesión con JWT.
- **Documentos**:
  - Subida de PDF/JPG/PNG a carpeta local.
  - Listado, detalle (con metadatos), borrado.
  - Búsqueda simple (texto) + filtros por tipo/rango de fechas.
- **Metadatos**: lectura/edición básica (tipo, emisor, fechas, importe).
- **Recordatorios**: creación/consulta (simulación de envío en logs).
- **Notificaciones**: simuladas (se registran en log; sin correo real).
- **Exportación**: (opcional MVP+1).

> **Fuera de alcance en MVP**: OCR/IA real (se deja stub), cofres/compartición, firmas, integraciones externas.

---

## 2) Arquitectura local

``
[Vue 3 SPA (Vite)]  →  [Spring Boot API (JWT)]
├─ Auth (Login/Registro)
├─ Document Service (CRUD + almacenamiento local)
├─ Metadata Service (PostgreSQL)
├─ Reminder Service (jobs en memoria)
└─ (Stub) OCR/IA
[PostgreSQL Local]  [Carpeta local para archivos: ./_storage]

---

## 3) Modelo de datos

### Entidades
- **User**
  - `id (uuid)`, `email (unique)`, `password_hash`, `created_at`
- **Document**
  - `id (uuid)`, `user_id (fk)`, `filename`, `mime`, `size_bytes`, `storage_path`, `checksum (sha256)`, `created_at`
- **DocMetadata**
  - `document_id (pk/fk)`, `type (dni|pasaporte|permiso|seguro|factura|contrato|otros)`, `issuer`, `person`, `date_issue`, `date_expiry`, `amount`, `currency`, `tags (jsonb)`, `confidence`
- **Reminder**
  - `id (uuid)`, `document_id (fk)`, `due_date`, `channel (email|none)`, `status (pending|sent|dismissed)`, `created_at`

### SQL (Flyway V1)
```sql
CREATE TABLE app_user (
  id UUID PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE document (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES app_user(id),
  filename VARCHAR(255) NOT NULL,
  mime VARCHAR(100) NOT NULL,
  size_bytes BIGINT NOT NULL,
  storage_path TEXT NOT NULL,
  checksum VARCHAR(64),
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE doc_metadata (
  document_id UUID PRIMARY KEY REFERENCES document(id) ON DELETE CASCADE,
  type VARCHAR(50) NOT NULL,
  issuer VARCHAR(255),
  person VARCHAR(255),
  date_issue DATE,
  date_expiry DATE,
  amount NUMERIC(14,2),
  currency VARCHAR(8),
  tags JSONB,
  confidence NUMERIC(3,2)
);

CREATE TABLE reminder (
  id UUID PRIMARY KEY,
  document_id UUID NOT NULL REFERENCES document(id) ON DELETE CASCADE,
  due_date DATE NOT NULL,
  channel VARCHAR(20) NOT NULL,
  status VARCHAR(20) NOT NULL DEFAULT 'pending',
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_document_user ON document(user_id);
CREATE INDEX idx_metadata_type ON doc_metadata(type);
CREATE INDEX idx_metadata_expiry ON doc_metadata(date_expiry);

```


## 4) Contratos de la API (REST)

Base URL: /api/v1
Auth: Bearer JWT en Authorization: Bearer <token>

Auth

POST /auth/register → 201
Body: { "email": "user@mail.com", "password": "******" }
POST /auth/login → 200
Body: { "email": "user@mail.com", "password": "******" }
Resp: { "accessToken": "…", "expiresIn": 1800 }

Documentos

POST /documents → 201
multipart/form-data: file, opcional type
Resp: { id, filename, mime, size, created_at }
GET /documents?query=&type=&from=&to=&page=&size= → 200
Resp: [{ id, filename, type, issuer, date_expiry, created_at }]
GET /documents/{id} → 200
Resp: { id, metadata, downloadUrl?: "/api/v1/documents/{id}/content" }
GET /documents/{id}/content → 200 (stream del archivo)
DELETE /documents/{id} → 204

Metadatos

GET /documents/{id}/metadata → 200 { ... }
PUT /documents/{id}/metadata → 200 { ... }

Recordatorios

POST /documents/{id}/reminders → 201
Body: { "due_date": "2026-03-10", "channel": "none" }
GET /documents/{id}/reminders → 200 [{ ... }]
PATCH /reminders/{id} → 200
Body: { "status": "dismissed" }

Errores (formato uniforme)

{
  "timestamp": "2026-02-17T08:00:00Z",
  "status": 400,
  "error": "Bad Request",
  "code": "VALIDATION_ERROR",
  "message": "El tipo no es válido",
  "path": "/api/v1/documents"
}


## 5) Requisitos no funcionales (local)

Rendimiento: ≤ 10 MB por archivo (configurable).
Seguridad:

Hash de contraseñas con BCrypt.
JWT expiración 15–30 min.
Validación de entrada con Bean Validation.
Autorización por propietario (filtrar por user_id).


Subidas seguras:

Validar MIME y extensión (pdf|png|jpg|jpeg).
Ruta de almacenamiento fuera del webroot del frontend.


Logs: sin PII; simular notificaciones escribiendo a log.
Observabilidad: Actuator /health.
i18n: ES.

## 6) Estructura de repos
repo/
  backend/
    src/main/java/com/example/docsguard/...
    src/main/resources/
      application.yml
      db/migration/V1__init.sql
    pom.xml
  frontend/
    src/...
    index.html
    package.json
    vite.config.ts
  _storage/     # carpeta local para archivos (montada por backend)

## 7) Backend (Spring Boot + Maven)
    7.1 Dependencias (pom.xml)
    
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>docsguard</artifactId>
  <version>0.1.0</version>
  <properties>
    <java.version>21</java.version>
    <spring.boot.version>3.3.0</spring.boot.version>
  </properties>
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>${spring.boot.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
  <dependencies>
    <!-- Core -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <!-- Security/JWT -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt-api</artifactId>
      <version>0.11.5</version>
    </dependency>
    <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt-impl</artifactId>
      <version>0.11.5</version>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt-jackson</artifactId>
      <version>0.11.5</version>
      <scope>runtime</scope>
    </dependency>
    <!-- Data -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
      <groupId>org.postgresql</groupId>
      <artifactId>postgresql</artifactId>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>org.flywaydb</groupId>
      <artifactId>flyway-core</artifactId>
    </dependency>
    <!-- Utils -->
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>
    <!-- Docs -->
    <dependency>
      <groupId>org.springdoc</groupId>
      <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
      <version>2.5.0</version>
    </dependency>
    <!-- Tests -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>

## 7.2) Configuración (application.yml)
server:
  port: 8080

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/docsguard
    username: docsguard
    password: secret
  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
  flyway:
    locations: classpath:db/migration

app:
  jwt:
    secret: "cambia_esta_clave_super_secreta"
    expires-minutes: 30
  storage:
    base-path: "./_storage"   # carpeta local para archivos
  uploads:
    max-mb: 10
  cors:
    allowed-origins: "http://localhost:5173"

## 8) Frontend (Vue 3 + Vite)
## 8.1) Creación del proyecto

cd frontend
npm create vite@latest
# Project name: docsguard-web
# Framework: Vue
# Variant: TypeScript (recomendado) o JavaScript
cd docsguard-web
npm install
npm install axios pinia vue-router element-plus


## 8.2) Estructura recomendada
src/
  api/              # axios instances y clientes
  stores/           # pinia stores (auth, documents)
  views/            # páginas (Login, Register, Documents, DocumentDetail)
  components/       # componentes (UploadDialog, MetadataForm, ReminderList)
  router/           # rutas
  assets/

## 8.3) Variables de entorno (.env)
VITE_API_BASE=http://localhost:8080/api/v1


8.4 Rutas (router)

/login, /register, /documents, /documents/:id,

8.5 Flujo de autenticación

Guardar accessToken (Pinia + localStorage).
Incluir Authorization: Bearer <token> en axios por interceptor.
Proteger rutas (si no token → /login).


9) Almacenamiento local de archivos

Carpeta base configurada en application.yml: ./_storage.
Estructura de guardado: ./_storage/{userId}/{documentUuid}.{ext}.
Nota: el backend debe stream del archivo en GET /documents/{id}/content y no exponer rutas de sistema.

10) Docker (opcional para DB)
Para mantener todo local, puedes ejecutar solo PostgreSQL con Docker y correr backend/frontend en tu host:


# docker-compose.yml (solo DB)
version: "3.9"
services:
  db:
    image: postgres:14
    environment:
      POSTGRES_DB: docsguard
      POSTGRES_USER: docsguard
      POSTGRES_PASSWORD: secret
    ports: ["5432:5432"]
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
Alternativa: instala PostgreSQL localmente y crea la DB/usuario con las mismas credenciales que en application.yml.

11) Comandos de desarrollo
11.1 Backend


# En repo/backend
./mvnw spring-boot:run
# o
mvn spring-boot:run

11.2 Frontend

# En repo/frontend/docsguard-web
npm run dev
# Abrir: http://localhost:5173


12) Criterios de aceptación (MVP)

Puedo registrarme e iniciar sesión y el token protege las rutas.
Puedo subir un documento (PDF/JPG/PNG ≤ 10 MB).
Veo el listado y puedo filtrar por tipo y fechas.
Puedo ver y editar metadatos.
Puedo crear/listar recordatorios; al llegar la fecha se marca sent (simulación en logs).
Puedo descargar/visualizar el archivo (/content).
Puedo eliminar un documento (borra archivo y registros).
Todos los endpoints responden con errores uniformes.


13) Validaciones clave

Extensiones permitidas: pdf|png|jpg|jpeg.
Verificación de MIME en servidor.
Tamaño máximo configurable (por defecto 10 MB).
Sanitizar filename y renombrar a UUID.
Límite de paginación por defecto (p. ej. 20).

14) Seguridad local (mínima viable)

BCrypt para contraseñas.
JWT firmado con secreto seguro (no subir a VCS).
CORS restringido a http://localhost:5173.
Autorización por user_id en cada consulta de recursos.
Logs sin datos sensibles.


15) Roadmap corto (MVP+1)

Refresh tokens + logout server-side.
Export ZIP + JSON de metadatos.
Envío real de emails (SMTP local tipo MailHog).
OCR/IA real (servicio externo) y pipeline asíncrono.
Detección de duplicados por checksum.
Pruebas de integración con Testcontainers.

16) Notas de implementación (sugerencias)

Controladores finos + Services con lógica + Repos (JPA).
Manejador global de errores (@ControllerAdvice) devolviendo el modelo de error.
Filtro JWT en Spring Security.
Actuator para health (/actuator/health).
Springdoc para documentar contratos y facilitar pruebas.


17) Checklist de arranque

 Crear carpeta repo/_storage con permisos de escritura.
 Levantar PostgreSQL (Docker o local).
 Configurar application.yml con credenciales.
 Ejecutar backend (mvn spring-boot:run) – verificar /swagger-ui.
 Ejecutar frontend (npm run dev) – probar login/registro.
 Subir documento de prueba y verificar que se guarda en ./_storage.

18) Ejemplos de requests (curl)

# Registro
curl -X POST http://localhost:8080/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@mail.com","password":"Secret123!"}'

# Login
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@mail.com","password":"Secret123!"}'

# Subida (reemplaza <TOKEN> y ruta de archivo)
curl -X POST http://localhost:8080/api/v1/documents \
  -H "Authorization: Bearer <TOKEN>" \
  -F "file=@/ruta/a/archivo.pdf"

# Lista
curl -H "Authorization: Bearer <TOKEN>" \
  "http://localhost:8080/api/v1/documents?type=factura&page=0&size=20"
