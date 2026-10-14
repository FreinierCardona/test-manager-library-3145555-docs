# Constitución — Manager Library (PHP + Laravel + React)

Principios innegociables de la variante del reto. Las historias, reglas, modelo y contrato HTTP permanecen definidos por [`spec.md`](spec.md), [`data-model.md`](data-model.md) y [`api-contract.md`](api-contract.md), idénticos a las variantes de referencia. Este documento fija cómo se implementan en PHP/Laravel y React.

Cada artículo incluye cómo se verifica. Una implementación que lo incumpla debe corregirse antes de dar por terminada la tarea.

---

## Artículo I — El dominio no depende de Laravel

`api/app/Domain/` contiene entidades, value objects, enums y reglas puras de PHP. No importa `Illuminate`, Eloquent, controladores, requests HTTP, adaptadores ni servicios externos. Usa tipos estándar de PHP, incluidos `DateTimeImmutable`, y no realiza I/O.

**Verificación:** una prueba de arquitectura comprueba que `Domain` no depende de `Application`, `Adapters`, `Providers` ni namespaces `Illuminate`.

## Artículo II — La aplicación declara los puertos

`api/app/Application/` contiene casos de uso y puertos definidos como interfaces por quienes los consumen. Los casos de uso no dependen de Eloquent, de clases HTTP ni de detalles de PostgreSQL. Las entradas/salidas entre capas usan DTOs explícitos; las entidades Eloquent no cruzan el límite HTTP.

**Verificación:** pruebas de aplicación ejecutan fakes de repositorios, reloj, almacenamiento, hash y emisión/validación de tokens sin arrancar Laravel ni PostgreSQL.

## Artículo III — Un único punto de composición

`api/app/Providers/LibraryServiceProvider.php` es el punto de composición: enlaza puertos con adaptadores. Laravel lo registra en `api/bootstrap/providers.php`. Los controladores no crean repositorios ni adaptadores; reciben casos de uso por inyección de dependencias.

**Verificación:** prueba de arquitectura y revisión de bindings confirman que los adaptadores de la biblioteca solo se registran en ese provider.

## Artículo IV — SOLID se demuestra con límites

| Principio | Aplicación concreta | Verificación |
|---|---|---|
| **SRP** | Un caso de uso resuelve una acción; `LendBooksService` no es controlador ni repositorio | La lógica HTTP y SQL vive fuera del servicio de aplicación |
| **OCP** | Cambiar almacenamiento local por S3 no modifica dominio ni casos de uso | Se añade un adaptador y se enlaza en composición |
| **LSP** | Los fakes sustituyen a adaptadores sin cambiar los resultados esperados | Casos de aplicación pasan con fakes y con integración real |
| **ISP** | Puertos pequeños (`Clock`, `PasswordHasher`, `TokenIssuer`, `CoverStorage`) | Los implementadores no dejan operaciones sin uso |
| **DIP** | Aplicación define los puertos que necesita | Las interfaces consumidoras viven en `Application` |

## Artículo V — Las migraciones de Laravel son dueñas del esquema

`infra` provisiona PostgreSQL 16 vacío. Todo DDL, incluidas tablas, índices parciales, restricciones y semilla de categorías, vive en `api/database/migrations/`. Las migraciones de Laravel pueden usar `DB::statement()` cuando el schema builder no exprese con claridad un `CHECK` o índice parcial de PostgreSQL.

**Verificación:** `infra` no contiene `CREATE TABLE`; al migrar una base vacía, las consultas de [`data-model.md`](data-model.md) reflejan exactamente el esquema declarado.

## Artículo VI — La base protege las invariantes que puede expresar

Cada `CHECK`, `UNIQUE`, FK e índice declarados como **motor** se crean en migraciones, también si la regla tiene validación previa en `FormRequest` o en el dominio. Una validación HTTP no sustituye una restricción de persistencia. Las reglas imposibles de expresar como `CHECK` se mantienen en aplicación/transacción y se etiquetan como tales.

**Verificación:** las consultas de [`data-model.md`](data-model.md) §10 coinciden con las migraciones; hay pruebas de integración que intentan violar las restricciones.

## Artículo VII — El estado derivado no se almacena

El estado de préstamo (`open`, `overdue`, `closed`) y `days_late` se calculan con el puerto `Clock`; no se añaden columnas para esos valores. Las consultas de Laravel devuelven el mismo resultado que la regla del modelo y del contrato.

**Verificación:** el esquema no contiene columnas `status` ni `days_late`; pruebas con reloj fijo cubren vencimiento, devolución y cierre.

## Artículo VIII — Las líneas conservan los valores prestados

Al registrar un préstamo, `loan_item` conserva `book_title`, `isbn` y `category_name` de ese instante. El reporte agregado consulta esos campos congelados, no el catálogo actual. Una relación Eloquent con `book` no puede reemplazar esos valores históricos.

**Verificación:** una prueba renombra un libro/categoría después del préstamo y confirma que el reporte mantiene los valores originales.

## Artículo IX — Los secretos no se versionan ni se exponen

`ADMIN_PASSWORD`, `JWT_SECRET` y las credenciales de servicios proceden del entorno. `api/.env` está ignorado; solo se versiona `.env.example` con placeholders. Fuera del entorno local, la API se sirve exclusivamente por HTTPS/TLS. Las contraseñas solo llegan en los cuerpos de login/alta, nunca se almacenan en claro ni aparecen en logs o respuestas. La API persiste únicamente el hash Argon2id. El hash JWT HS256 autentica el token; no cifra sus claims.

**Verificación:** `gitleaks` en CI; pruebas inspeccionan logs, payloads, respuestas y persistencia; el perfil de producción rechaza el reloj demo y exige transporte HTTPS en el borde.

## Artículo X — Gana PostgreSQL

Si el modelo, las migraciones y PostgreSQL discrepan, se corrige el documento o la migración para que describa el esquema aprobado. No se permite un esquema manual distinto en cada entorno.

## Artículo XI — Idioma

Clases, métodos, tablas, columnas, rutas y códigos permanecen en inglés. La documentación de producto y los mensajes de error se escriben en español. La interfaz React usa español para los mensajes visibles.

## Artículo XII — Una tarea termina con evidencia

Una tarea de [`tasks.md`](tasks.md) está completa cuando su evidencia existe y puede ejecutarse, no cuando el endpoint solo está registrado o la pantalla solo está maquetada. Las pruebas de API y de React deben apuntar a los criterios de aceptación del SDD.
