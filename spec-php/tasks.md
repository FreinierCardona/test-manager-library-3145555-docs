# Tareas — Manager Library (PHP + Laravel + React)

Cada tarea termina cuando su evidencia existe y se puede ejecutar (artículo XII de [`constitution.md`](constitution.md)). Las dependencias, no el número de tarea, determinan el orden ejecutable. Los casos de uso PHP y la SPA React permanecen en los repositorios `api` y `app`, respectivamente.

| # | Tarea | Depende de | Evidencia |
|---|---|---|---|
| T-01 | Esqueleto Laravel API y `GET /health` | — | `GET /api/v1/health` responde `{"status":"ok"}`; la aplicación arranca en el entorno local |
| T-02 | Migraciones PostgreSQL: seis tablas de negocio, FK, índices, checks e IDs de categorías | T-01 | Las consultas de `data-model.md` §10 coinciden con migraciones; `DB_SCHEMA=public`, `library` contiene solo sus seis tablas y no hay tablas starter de auth/session/cache/queue; no hay columnas `DEFAULT`, `created_at` ni `updated_at`; checks rechazan inventario fuera de rango, `inventory_version < 1` y documento vacío o con U+0020 exterior; cinco categorías tienen UUIDs fijos; la tabla técnica `public.migrations` queda fuera de `library` |
| T-03 | Value objects PHP: `Isbn`, `Quantity`, `DateRange` | T-01 | PHPUnit: ISBN solo 13 dígitos, cantidad positiva y fin de rango ≥ inicio |
| T-04 | Dominio `Book`: `Withdraw`, `Restock`, baja lógica | T-03 | Pruebas puras: stock nunca < 0 ni > total; baja marca `deleted_at` |
| T-05 | Dominio `Loan`/`LoanItem`: alta, devolución y estado calculado usando `Clock` | T-04 | PHPUnit con reloj fijo cubre `open`, `overdue`, `closed`, plazo 14 días, 1–3 líneas, sin libro repetido y devolución única |
| T-06 | Dominio `Member` y `User` | T-03 | Normaliza username; `document_number` recorta U+0020 exterior, conserva el resto, exige 1–30 caracteres y compara unicidad con distinción de mayúsculas; rechaza `admin` en alta de usuarios por API |
| T-07 | Puertos y casos de uso de autenticación: `LoginService`, `CreateLibrarianService` | T-06 | HU-01 y HU-02 pasan con fakes; token válido dura 8 h; credenciales erróneas no revelan cuál dato falló |
| T-08 | Puertos y casos de uso de libros: crear, buscar, editar, baja y portadas | T-04 | HU-03 y HU-04 pasan con fakes; cubre ISBN activo único, ajuste de existencias, portada nullable y baja con/sin préstamos abiertos |
| T-09 | Caso de uso `LendBooksService` | T-05, T-06, T-08 | HU-06 pasa con fakes: socio válido, usuario registrador, límite 3, vencidos, existencias, líneas únicas y snapshots congelados |
| T-10 | Casos de uso de devolución y vencidos | T-09 | HU-07 y HU-09 pasan con reloj fijo; devolver repetido → `ITEM_ALREADY_RETURNED`; días y orden coinciden con los criterios |
| T-11 | Caso de uso del reporte y puerto `ReportReader` | T-09 | HU-10 pasa; agrupa valores congelados, no muestra personas y rango vacío devuelve lista vacía |
| T-24 | Casos de uso de socios (crear, obtener, buscar, listar) | T-06 | HU-05 pasa con fakes: normalización, duplicado tras recorte, búsqueda por nombre/documento, paginación y sin eliminación |
| T-25 | Casos de uso de categorías y consultas de préstamos (obtener/listar) | T-05, T-06 | `GET /categories` entrega cinco categorías ordenadas; HU-08 pasa con filtros, estado calculado, orden, desempates y paginación |
| T-12 | Adaptadores de persistencia Laravel/Eloquent para libros, socios, usuarios y categorías | T-02, T-08, T-20, T-24, T-25 | Integration tests en PostgreSQL real verifican lecturas/escrituras, tablas cualificadas en `library`, `$timestamps = false`, soft delete, filtros, índices y `document_number`; no se consulta ni escribe en `users` convencional |
| T-13 | Persistencia transaccional de préstamos e inventario con `inventory_version` | T-12, T-09, T-25 | Dos peticiones concurrentes por el último ejemplar: una 201, otra 409 `BOOK_NOT_AVAILABLE`; conflicto con stock restante reintenta una vez; segundo cambio → `BOOK_CONCURRENTLY_MODIFIED`; todo error revierte préstamo, líneas y stock |
| T-14 | Adaptador `PasswordHasher` Argon2id y `TokenIssuer` JWT HS256 | T-07 | La base contiene solo hash; password/hash no aparece en logs/respuestas; firma, expiración, `sub`, `role` y rechazo de token inválido/caducado tienen pruebas |
| T-15 | Adaptador Laravel Filesystem para portadas | T-08 | PNG/JPG de 2 MB aceptado; 3 MB, MIME no permitido o contenido inválido → 422; no-portada deja `cover_key` nula; borrado de binario sigue el orden de retención |
| T-16 | Rutas Laravel, middleware, FormRequests, Resources y renderer de errores | T-10, T-11, T-12, T-13, T-14, T-15, T-24, T-25 | Laravel HTTP tests cubren cada ruta/rol/código del contrato; 404 binario de portada, error JSON uniforme, fechas UTC, paginación, orden y validaciones |
| T-17 | Composición con `LibraryServiceProvider` y aprovisionamiento idempotente de admin | T-02, T-12, T-13, T-14, T-16 | Con esquema migrado y solo categorías sembradas, arranca API y se autentica el admin guardado en `library."user"`; reinicio no duplica el usuario ni crea tablas de autenticación por defecto |
| T-18 | Migración de checks pendientes: `ck_book_total_positive`, `ck_user_role` | T-12 | Pruebas SQL PostgreSQL demuestran que escrituras inválidas fallan; `data-model.md` §4 concuerda |
| T-19 | Pruebas de arquitectura de artículos I–III | T-17 | Deptrac o regla automatizada equivalente falla ante dependencia Domain→Laravel/Adapters o binding de infraestructura fuera de `LibraryServiceProvider` |
| T-20 | `infra`: servicio de desarrollo PostgreSQL 16 vacío | T-01 | `docker compose up -d db` deja PostgreSQL saludable y sin tablas de negocio; migrations y semilla siguen siendo propiedad de `api` |
| T-21 | `tool`: sembrador por API en perfil demo aislado (≥40 libros, 10 socios, 15 préstamos, exactamente 3 vencidos) | T-17, T-26 | Crea 3 préstamos en T0 para 3 socios distintos; avanza reloj demo a T0 + 15 días; crea otros 12 para socios sin vencidos sin superar 3 abiertos; `/loans/overdue` devuelve exactamente 3; el reporte del rango cubre los préstamos; no ejecuta SQL |
| T-22 | `app`: SPA React — login, catálogo, socios, préstamo, devolución, vencidos y reporte | T-16, T-24, T-25 | Flujo funcional con `ApiClient`, Bearer JWT y manejo de error común; payloads respetan `snake_case`; no se implementa lógica de negocio solo en el navegador |
| T-23 | `page`: presentación pública estática | — | Abre sin la API y funciona en móvil/escritorio |
| T-26 | Reloj controlado solo para el perfil demo privado | T-05, T-17 | `DEMO_CLOCK_ENABLED=false` por defecto; solo red privada demo monta `POST /__test/clock/advance` fuera de `/api/v1`; el arranque de producción rechaza `true` y no registra la ruta |
| T-27 | Pruebas de React y E2E del stack | T-21, T-22, T-28 | Vitest/Testing Library cubren estados de carga, error, rol y formularios; Playwright recorre login → buscar libro → prestar → devolver → reporte contra Compose levantado |
| T-28 | `infra`: Compose de entrega con PostgreSQL, API Laravel y React | T-17, T-20, T-22 | Un comando levanta todos los servicios; en volumen vacío espera PostgreSQL saludable, corre `php artisan migrate --force`, crea el admin idempotentemente y deja `/api/v1/health` y frontend disponibles (RNF-5); `infra` no incluye SQL DDL |

## Definición de terminado

T-17 con evidencia completa el núcleo de backend. T-22 y T-27 completan el flujo React. T-18, T-19, T-21, T-23, T-26 y T-28 cubren robustez, despliegue y demo; T-24 y T-25 son prerequisitos funcionales del núcleo, no alcance opcional. Una tarea solo se cierra con su evidencia ejecutable y las reglas de negocio originales intactas.
