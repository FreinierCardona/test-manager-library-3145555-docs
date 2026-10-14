# Plan técnico — Manager Library (PHP + Laravel + React)

Cómo se construye el producto definido en [`spec.md`](spec.md) y [`constitution.md`](constitution.md) con el stack del reto. El modelo vive únicamente en [`data-model.md`](data-model.md), y el contrato HTTP en [`api-contract.md`](api-contract.md). Este spec describe ambos repositorios ejecutables: API Laravel y SPA React; no cambia historias ni endpoints.

## 1. Decisiones técnicas

| # | Decisión | Elección | Por qué |
|---|---|---|---|
| D-01 | Backend | PHP 8.3 o superior y Laravel 12.x; versiones exactas fijadas en `composer.lock` | Runtime compatible con el stack solicitado; dependencias reproducibles |
| D-02 | Frontend | React 19.x, TypeScript 5.x y Vite; versiones exactas fijadas en el lockfile del repo `app` | SPA independiente que consume JSON por HTTP; no se usa Blade para las pantallas de operación |
| D-03 | Persistencia | PostgreSQL 16; Laravel Database/Eloquent solo en adaptadores; migraciones Laravel crean las seis tablas cualificadas en `library`; modelos usan `$timestamps = false` | Evita tablas/columnas implícitas del scaffolding y conserva el único dueño del esquema (ADR-001) |
| D-04 | Concurrencia | `book.inventory_version` y actualización condicional atómica + `CHECK` de rango (ADR-002) | Protege el último ejemplar y conserva el contrato de error |
| D-05 | Fechas | Instantes UTC en `timestamptz`; puerto `Clock`, implementado con reloj del sistema o reloj fijo de pruebas/demo | Calcula vencimientos sin modificar la API normal ni esperar 14 días en pruebas |
| D-06 | Reporte | Consulta agregada PostgreSQL detrás de `ReportReader`; agrupa por datos congelados | Una sola consulta y resultados históricos estables (ADR-004) |
| D-07 | Objetos de valor | `Isbn`, `Quantity`, `DateRange`; `document_number` es texto opaco validado según CA-05.1 | No se asigna identidad ni formato nacional al documento |
| D-08 | Portadas | Laravel Filesystem: disco local en desarrollo y almacenamiento compatible S3 en producción; PostgreSQL guarda `cover_key` | El binario no se almacena en la base ni se filtra una ruta física |
| D-09 | Contraseñas | `PasswordHasher` usa Laravel `Hash` configurado con `HASH_DRIVER=argon2id` (PHP `password_hash()`); la disponibilidad del driver se verifica al inicio | Hash moderno, coste configurable, ninguna contraseña en claro persistida |
| D-10 | Token | `TokenIssuer` usa `firebase/php-jwt` para emitir/verificar JWT HS256 de 8 h, con `sub`, `role`, `iat` y `exp`; `JWT_SECRET` tiene al menos 32 bytes aleatorios | Cumple el contrato Bearer sin introducir sesiones Laravel en las rutas de la API |
| D-11 | React/API | Cliente HTTP centralizado con `VITE_API_BASE_URL`; todos los payloads siguen `snake_case` y el contrato JSON | Mantiene desacoplados `app` y `api`; CORS permite solo el origen configurado |
| D-12 | Pruebas | PHPUnit para dominio/aplicación; Laravel HTTP/feature tests y PostgreSQL para integración; Vitest + Testing Library para React y Playwright para flujos E2E | Evidencia por capa; casos críticos se prueban contra PostgreSQL real |
| D-13 | Composición | `LibraryServiceProvider` enlaza puertos y adaptadores; se registra desde `bootstrap/providers.php` | Punto único de composición coherente con la constitución |

La emisión y validación JWT se encapsula detrás de `TokenIssuer`/middleware. `firebase/php-jwt` se fija en Composer y se somete a revisión de CVE; los controladores no codifican ni verifican tokens directamente.

## 2. Configuración por entorno

| Variable | Obligatoria | Ejemplo/uso |
|---|---|---|
| `APP_ENV` | sí | `local`, `testing` o `production` |
| `APP_KEY` | sí | Clave Laravel, generada fuera del repositorio; distinta de `JWT_SECRET` |
| `DB_CONNECTION` | sí | `pgsql` |
| `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD` | sí | Host, puerto, base y usuario PostgreSQL |
| `DB_SCHEMA` | sí | `public` (`search_path`); el DDL y los modelos cualifican las tablas de negocio como `library.<tabla>` |
| `HASH_DRIVER` | sí | `argon2id`; el arranque falla si PHP no tiene disponible ese driver |
| `JWT_SECRET` | sí | Secreto aleatorio ≥ 32 bytes; nunca en el repo |
| `ADMIN_USERNAME`, `ADMIN_PASSWORD` | sí al aprovisionar admin | Secretos del entorno; no se imprimen en logs |
| `COVERS_DISK` | no | `local` en desarrollo; disco privado S3 en producción |
| `CORS_ALLOWED_ORIGINS` | sí | Origen exacto del frontend, por ejemplo `http://localhost:5173` en local |
| `DEMO_CLOCK_ENABLED` | no | `false`; solo `true` en perfil demo aislado y rechazado en producción |
| `VITE_API_BASE_URL` | sí en el repo `app` | URL pública de `/api/v1`; no contiene secretos |

`APP_KEY` y `JWT_SECRET` son secretos distintos. `VITE_*` se compila dentro del frontend y nunca puede contener credenciales privadas.

## 3. Capas y responsabilidades

- **React (`app`)**: pantallas y formularios, rutas protegidas, estado de autenticación, presentación de errores y cliente HTTP. No replica reglas de negocio como disponibilidad o autorización.
- **Laravel HTTP (`api`)**: rutas versionadas, autenticación Bearer, autorización por rol, `FormRequest` para validación de forma y un manejador global que traduce errores al cuerpo único.
- **Application**: casos de uso Laravel-agnósticos, puertos y transacciones de aplicación. No retorna modelos Eloquent.
- **Domain**: reglas e invariantes puras de préstamo, libro, socio y usuario.
- **Adapters**: repositorios Eloquent/PostgreSQL, filesystem de portadas, reloj del sistema, hash y JWT.
- **Infra**: Compose provisiona la base vacía y servicios; no crea tablas ni semilla.
- **Tool**: sembrador PHP/CLI que autentica y usa únicamente la API pública; el cambio de reloj se hace solo por el canal demo privado de T-26.

## 4. Orden de construcción

Contrato/diseño confirmado → esqueleto API y SPA → motor PostgreSQL vacío de desarrollo en paralelo → migraciones → dominio PHP → casos de uso con fakes → adaptadores Eloquent/seguridad/storage → rutas JSON y autorización → componentes/flujo React → Compose integral → sembrador demo → pruebas de contrato, concurrencia y punta a punta. El DAG y la evidencia están en [`tasks.md`](tasks.md).
