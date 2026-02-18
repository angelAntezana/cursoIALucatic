<!-- Requirements for MVP "Gestión de documentos personales" -->

# Requisitos del MVP: Gestión de documentos personales

## Visión general
- App web local destinada a familias y autónomos que necesitan organizar y supervisar sus documentos personales críticos (DNI, facturas, contratos, permisos, seguros).
- Backend en Java con Spring Boot, frontend en Vue.js y base de datos PostgreSQL.
- Priorizar un MVP sencillo: solo funcionalidades esenciales sin almacenamiento externo ni despliegue.

## Funcionalidades principales
1. **Organización automática**  
   - Subida manual de documentos (PDF, imágenes) por parte del usuario.  
   - Clasificación automática según palabras clave, metadatos y etiquetas (DNI, factura, contrato, permiso, seguro).
   - Almacenamiento en PostgreSQL con referencias al fichero físico guardado localmente.

2. **Extracción de fechas importantes**  
   - Procesamiento simple de texto/OCR para detectar fechas de vencimiento o emisión.  
   - Indexación de todas las fechas relevantes (emisión, caducidad, renovación).

3. **Alertas de caducidad**  
   - Monitorización periódica (diaria o al abrir la app) de las fechas extraídas.  
   - Notificaciones internas (UI) que muestren documentos próximos a vencer (DNI, permisos, seguros).  
   - Filtrado por intervalo (7, 30 días) configurable en la UI.

4. **Panel resumen**  
   - Vista principal con lista de documentos organizados por categoría.  
   - Indicadores rápidos de caducidades próximas y documentos recientes.

## Casos de uso
1. **Familiar**: La cabeza de familia sube el DNI del menor y recibe alerta 30 días antes de la caducidad; revisa facturas y contratos vigentes desde el panel familiar.
2. **Autónomo**: Administra contratos y facturas desde el panel, visualiza próximas fechas de vencimiento de permisos y seguros, y reordena recibos por cliente.
3. **Usuario general**: Busca un documento por tipo o fecha, ve la cronología de subidas y confirma que no hay caducidades pendientes.

## Límites del MVP
- Todo se ejecuta localmente: la app, el procesamiento de documentos y la base de datos PostgreSQL local (no hay nube ni sincronización).  
- No hay autentificación avanzada ni integración con terceros (correo, SMS o API externas).  
- OCR es básico y offline: se apoya en bibliotecas que deben instalarse localmente y se limita a textos claros.  
- Escala mínima: no se contempla gestión multi-usuario ni soporte para múltiples dispositivos sincronizados.  
- Notificaciones solo en interfaz; no se mandan correos ni mensajes externos.

## Estructura técnica propuesta
- Backend: Spring Boot con endpoints REST para subir documentos, clasificar, listar y reportar caducidades.  
- Frontend: Vue.js (sin servidor externo) con formularios de subida, vistas de tabla y tarjetas de alertas.  
- PostgreSQL: tablas para documentos, categorías, fechas detectadas y alertas.  
- Almacenamiento de archivos: carpeta local indexada desde la app backend y referenciada en la base de datos.

## Prioridades para el MVP
1. Subida y clasificación automática de documentos locales.  
2. Extracción de fechas básicas y almacenamiento estructurado.  
3. Panel de alertas sin notificaciones externas.  
4. Interfaz Vue con listas y filtros mínimos.  
5. Despliegue local, documentación de cómo levantar la app y la base de datos.

## Tareas inmediatas
- Configurar Spring Boot y PostgreSQL local.  
- Crear endpoints REST para upload, listar y alertas.  
- Desarrollar views Vue para subir documentos, ver lista y ver alertas.  
- Implementar lógica simple de extracción de fechas y alerta en backend.  
- Documentar pasos para correr la app en local (README complementario).
