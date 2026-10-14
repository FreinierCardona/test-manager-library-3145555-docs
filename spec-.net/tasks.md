# Tareas — Manager Library (.NET)

Cada tarea termina cuando su **evidencia** existe y se puede ejecutar (artículo XII). Estado inicial de todas: **pendiente**.

| # | Tarea | Depende de | Evidencia |
|---|---|---|---|
| T-01 | Esqueleto del proyecto y `GET /health` | — | `GET /health` responde `{"status":"ok"}`; el contenedor arranca con un comando |
| T-02 | Migración inicial: seis tablas, FK, índices y restricciones, incluida `book.inventory_version` y `ck_member_document_number_trimmed`, y las cinco categorías sembradas | T-01 | Las consultas de `data-model.md` §10 devuelven lo documentado; ninguna columna con `DEFAULT`; los `CHECK` rechazan versiones < 1 y documentos vacíos o con U+0020 exterior |
| T-03 | Dominio: `Isbn`, `Quantity`, `DateRange` (objetos de valor) | T-01 | Pruebas unitarias: ISBN de 13 dígitos, cantidad > 0, fin ≥ inicio |
| T-04 | Dominio: `Book` (`Withdraw`, `Restock`, baja lógica) | T-03 | Pruebas: nunca < 0, nunca > total |
| T-05 | Dominio: `Loan` (`AddItem`, `ReturnItem`, estado calculado, 14 días, máx. 3) | T-04 | Pruebas con un `Clock` falso: `open`, `overdue`, `closed` |
| T-06 | Dominio: `Member` y `User` (normalización de nombre, rol válido; `document_number` opaco) | T-03 | `"Ana "` queda `ana`; documento `" 001-23 "` queda `"001-23"`; vacío y más de 30 caracteres → 422; el formato nacional no se altera ni se valida; rol `admin` por API rechazado |
| T-07 | Puertos y casos de uso de **autenticación** (`LoginService`, `CreateLibrarianService`) con fakes | T-06 | HU-01 y HU-02 pasan contra fakes |
| T-08 | Puertos y casos de uso de **libros** (crear, buscar, editar, baja, portada) con fakes | T-04 | HU-03 y HU-04 pasan contra fakes |
| T-09 | Caso de uso **prestar** (`LendBooksService`) con fakes | T-05, T-06, T-08 | HU-06 completa: límites, vencidos, sin ejemplares |
| T-10 | Caso de uso **devolver** y **vencidos** | T-09 | HU-07 y HU-09; devolver dos veces → `ITEM_ALREADY_RETURNED` |
| T-11 | Caso de uso **reporte de más prestados** y su puerto de lectura | T-09 | HU-10; el título viejo se conserva tras renombrar (ADR-004) |
| T-12 | Adaptador de persistencia: repositorios de libro, socio, usuario y categoría | T-02, T-08, T-24, T-25 | Pruebas de integración contra PostgreSQL real; documento recortado y unicidad persistida |
| T-13 | Adaptador de persistencia: transacción de préstamos e inventario con concurrencia optimista por `inventory_version`, más consultas de préstamos | T-12, T-09, T-25 | Dos solicitudes simultáneas por el último ejemplar: una 201 y otra 409 `BOOK_NOT_AVAILABLE`; un conflicto con existencias disponibles se reintenta una vez y, si vuelve a cambiar, responde 409 `BOOK_CONCURRENTLY_MODIFIED`; cualquier fallo revierte préstamo, líneas y existencias (RNF-2); búsquedas/listados cumplen filtros y orden de HU-08 |
| T-14 | Adaptador de seguridad: hash de claves y token JWT | T-07 | La contraseña no se persiste en claro ni aparece en logs o respuestas; la base contiene solo el hash (RNF-3) |
| T-15 | Adaptador de portadas (disco) | T-08 | Subir PNG de 2 MB ok; 3 MB → 422; sin portada → `cover_key` nulo |
| T-16 | Adaptador HTTP: rutas de [`api-contract.md`](api-contract.md), cuerpo de error único, autorización por rol | T-10, T-11, T-12, T-13, T-14, T-15, T-24, T-25 | Pruebas de contrato recorren todas las rutas y códigos de §2; verifican descarga de portada y 404; prueban límites UTC inclusivos, rango invertido, estados admitidos, orden/desempates, paginación válida y rechazo de valores inválidos; el despliegue no local expone la API solo por HTTPS (RNF-3) |
| T-17 | Composición y arranque (admin inicial desde entorno) | T-02, T-12, T-13, T-14, T-16 | El sistema arranca con esquema migrado y solo las cinco categorías sembradas antes de crear el admin; el login funciona y no hay libros, socios ni préstamos |
| T-18 | Bajar a la base las invariantes **solo dominio**: `ck_book_total_positive`, `ck_user_role` | T-12 | Un `INSERT` por `psql` que las viole falla; `data-model.md` §4 se actualiza |
| T-19 | Pruebas de arquitectura (artículos I, II y III) | T-17 | NetArchTest pasa y falla si se introduce una importación prohibida |
| T-20 | `infra`: `docker compose up` levanta motor, API y frontend | T-17 | Un comando; el motor comienza sin esquema y la migración de T-02 crea tablas y categorías antes de iniciar la aplicación |
| T-21 | `tool`: sembrador de demostración por la API en perfil aislado (≥ 40 libros, 10 socios, 15 préstamos, exactamente 3 vencidos) | T-17, T-26 | Crea 3 préstamos con el reloj en T0 por la API; llama al control de T-26 para avanzar a T0 + 15 días; crea otros 12 por la API solo para socios sin vencidos y distribuidos de modo que nadie supere 3 préstamos abiertos; `/loans/overdue` devuelve exactamente 3 líneas y el reporte tiene filas; producción no expone el control del reloj |
| T-22 | `app`: login, catálogo, socios, prestar, devolver, vencidos, reporte | T-16, T-24, T-25 | Recorrido manual de punta a punta en la feria |
| T-23 | `page`: sitio público estático de presentación | — | Se abre sin la API; correcto en móvil y escritorio |
| T-24 | Casos de uso de socios (crear, obtener, buscar y listar) con fakes | T-06 | HU-05 pasa contra fakes: normalización de documento, duplicado (incluidos valores que solo difieren en espacios exteriores), búsqueda por nombre/documento, paginación y ausencia de eliminación |
| T-25 | Casos de uso de categorías y consultas de préstamos (obtener/listar) con fakes | T-05, T-06 | `GET /categories` retorna cinco categorías ordenadas; HU-08 pasa contra fakes con filtros, estado calculado, orden, desempates y paginación |
| T-26 | Reloj controlable exclusivo del perfil demo aislado | T-05, T-17 | `DEMO_CLOCK_ENABLED` es falso por defecto; solo en la red privada del perfil demo habilita `POST /__test/clock/advance` con `{ "days": 15 }`, fuera de `/api/v1`; una prueba confirma que producción rechaza `DEMO_CLOCK_ENABLED=true` y no monta la ruta |

## Definición de terminado de la prueba

Quien llegue a T-17 con evidencia tiene el **núcleo** hecho. T-18, T-19, T-20, T-21, T-22, T-23 y T-26 son lo que separa un trabajo bueno de uno sobresaliente. T-24 y T-25 son prerrequisitos del núcleo porque completan historias obligatorias. Se evalúa
el desempeño con SDD, no la cantidad de tareas: una tarea con evidencia vale más que tres a medias.
