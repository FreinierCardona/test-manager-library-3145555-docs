# Contrato de la API — Manager Library

El contrato HTTP es **independiente del lenguaje**: quien lo implemente en React + Laravel debe respetarlo igual que quien lo
implemente en Python o .NET. Qué hace cada operación está en [`spec.md`](spec.md); qué se guarda, en [`data-model.md`](data-model.md).

## 1. Convenciones

| Tema | Regla |
|---|---|
| Base | `/api/v1` |
| Formato | JSON, UTF-8. Nombres de campo en `snake_case` |
| Identificadores | UUID v4 como texto |
| Fechas | ISO 8601 en UTC, con `Z` (`2026-10-14T14:15:00Z`) |
| Dinero | No hay. El sistema no maneja importes |
| Autenticación | `Authorization: Bearer <token>` en todo salvo `POST /auth/login` y `GET /health` |
| Paginación | `?page=1&size=20`; por defecto `page=1` y `size=20`. `page` debe ser ≥ 1 y `size` debe estar entre 1 y 50; valores fuera de rango o no enteros responden 422 `VALIDATION_ERROR`. La respuesta es `{ "items": [...], "page": 1, "size": 20, "total": 134 }` |
| Idioma | Mensajes de error en español; campos y códigos en inglés |

### Cuerpo de error (único para todos los errores)

```json
{
  "code": "BOOK_NOT_AVAILABLE",
  "message": "No quedan ejemplares disponibles de «Cien años de soledad».",
  "details": [ { "field": "items[0].book_id", "issue": "sin ejemplares" } ]
}
```

`details` puede ir vacío. **Nunca** incluye trazas, SQL, rutas del servidor ni la contraseña.

### Códigos de estado

| Estado | Cuándo |
|---|---|
| 200 / 201 / 204 | Éxito / creado / sin contenido |
| 401 | Sin token, token inválido o vencido, credenciales inválidas |
| 403 | Rol insuficiente |
| 404 | No existe (o está dado de baja) |
| 409 | Conflicto de estado: duplicado, sin ejemplares, inventario modificado concurrentemente, ya devuelto, socio con vencidos, libro con préstamos abiertos |
| 422 | Validación: campo inválido, regla de entrada rota, rango de fechas invertido |

## 2. Códigos de error de negocio

| `code` | Estado | Origen |
|---|---|---|
| `INVALID_CREDENTIALS` | 401 | CA-01.2 |
| `FORBIDDEN_ROLE` | 403 | CA-01.3 |
| `NOT_FOUND` | 404 | Cualquier recurso |
| `USERNAME_TAKEN` | 409 | CA-02.2 |
| `ISBN_TAKEN` | 409 | CA-03.2 |
| `DOCUMENT_TAKEN` | 409 | CA-05.1 |
| `BOOK_HAS_OPEN_LOANS` | 409 | CA-03.5 |
| `BOOK_NOT_AVAILABLE` | 409 | CA-06.3 |
| `BOOK_CONCURRENTLY_MODIFIED` | 409 | La versión del inventario cambió dos veces durante el reintento de una mutación |
| `MEMBER_HAS_OVERDUE` | 409 | CA-06.5 |
| `ITEM_ALREADY_RETURNED` | 409 | CA-07.2 |
| `MEMBER_LIMIT_EXCEEDED` | 422 | CA-06.4 |
| `DUPLICATE_BOOK_IN_LOAN` | 422 | CA-06.2 |
| `VALIDATION_ERROR` | 422 | Cualquier campo inválido |
| `INVALID_DATE_RANGE` | 422 | CA-08.2 y CA-10.2 |

## 3. Endpoints

### 3.1 Autenticación y salud

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| POST | `/auth/login` | público | Devuelve token |
| GET | `/health` | público | `{ "status": "ok" }` |

`POST /auth/login`

```json
// petición
{ "username": "ana", "password": "********" }
// 200
{ "access_token": "eyJ...", "token_type": "Bearer", "expires_in": 28800, "role": "librarian" }
```

### 3.2 Usuarios

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| POST | `/users` | admin | Crea un bibliotecario |

```json
// petición  (role solo admite "librarian")
{ "username": "Ana ", "password": "********", "role": "librarian" }
// 201  (sin password ni password_hash)
{ "id": "…", "username": "ana", "role": "librarian" }
```

### 3.3 Categorías

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| GET | `/categories` | autenticado | Las cinco categorías, orden por nombre |

### 3.4 Libros

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| GET | `/books` | autenticado | Búsqueda: `q`, `category_id`, `page`, `size` |
| GET | `/books/{id}` | autenticado | Detalle |
| GET | `/books/{id}/cover` | autenticado | Devuelve los bytes de la portada con `Content-Type: image/jpeg` o `image/png` |
| POST | `/books` | admin | Crea |
| PUT | `/books/{id}` | admin | Reemplaza título, autor, categoría y total |
| DELETE | `/books/{id}` | admin | Baja lógica |
| PUT | `/books/{id}/cover` | admin | Sube portada (`multipart/form-data`, campo `file`) |
| DELETE | `/books/{id}/cover` | admin | Quita la portada |

Libro (respuesta):

```json
{
  "id": "…", "title": "Cien años de soledad", "author": "Gabriel García Márquez",
  "isbn": "9780307474728", "category": { "id": "…", "name": "Novela" },
  "total_copies": 4, "available_copies": 3, "cover_key": null
}
```

`POST /books` (petición): `title`, `author`, `isbn`, `category_id`, `total_copies`. Los disponibles los calcula el servidor.
`cover_key` es una clave opaca y no se expone como ruta de almacenamiento. `GET /books/{id}/cover` devuelve el binario de la portada; si el libro no existe, está dado de baja o no tiene portada, responde 404 con el cuerpo de error único (`NOT_FOUND`).

### 3.5 Socios

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| GET | `/members` | admin, librarian | Lista y busca: `q`, `page`, `size` |
| GET | `/members/{id}` | admin, librarian | Detalle con `active_loans` (conteo sin devolver) |
| POST | `/members` | admin, librarian | Crea: `full_name`, `document_number` |

`document_number` es obligatorio; se recorta el espacio ASCII U+0020 de los extremos y debe quedar entre 1 y 30 caracteres. Se conserva como texto opaco, incluidos caso, puntuación y ceros iniciales; no se aplica validación de formato nacional. La unicidad se evalúa con comparación sensible a mayúsculas sobre el valor recortado; repetido: 409 `DOCUMENT_TAKEN`.

### 3.6 Préstamos

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| POST | `/loans` | admin, librarian | Registra un préstamo |
| GET | `/loans` | admin, librarian | Lista: `from`, `to`, `status`, `member_id`, `page`, `size`; fechas UTC inclusivas, `status` admite `open`, `overdue` o `closed`; orden `loaned_at` descendente y `id` ascendente |
| GET | `/loans/{id}` | admin, librarian | Detalle con líneas y estado |
| POST | `/loans/{loan_id}/items/{item_id}/return` | admin, librarian | Devuelve una línea |
| GET | `/loans/overdue` | admin, librarian | Líneas vencidas: `page`, `size`; orden `days_late` descendente, `due_at` ascendente e `item_id` ascendente |

En `GET /loans`, `from` y `to` son fechas `YYYY-MM-DD` interpretadas en UTC e inclusivas sobre la fecha UTC de `loaned_at`; ambos parámetros son opcionales. Si se envían ambos y `from > to`, responde 422 `INVALID_DATE_RANGE`. Un `status` distinto de `open`, `overdue` o `closed` responde 422 `VALIDATION_ERROR`.

`POST /loans`

```json
// petición  (1 a 3 elementos, sin libros repetidos)
{ "member_id": "…", "items": [ { "book_id": "…" }, { "book_id": "…" } ] }
// 201
{
  "id": "…", "member_id": "…", "loaned_at": "2026-10-14T14:15:00Z", "status": "open",
  "items": [
    { "id": "…", "book_id": "…", "book_title": "Cien años de soledad", "isbn": "9780307474728",
      "category_name": "Novela", "due_at": "2026-10-28T14:15:00Z", "returned_at": null }
  ]
}
```

`status` es calculado: `open` (hay líneas sin devolver y ninguna vencida), `overdue` (alguna sin devolver y vencida), `closed` (todas devueltas).

`POST /loans/{loan_id}/items/{item_id}/return` → 200 con la línea (`returned_at` lleno).

`GET /loans/overdue` → respuesta paginada estándar; cada elemento de `items` es `{ "loan_id", "item_id", "book_title", "member_name", "due_at", "days_late" }`. Se incluyen líneas sin devolver con `due_at` anterior a ahora UTC. `days_late` es el número entero de periodos completos de 24 horas transcurridos desde `due_at` y puede ser 0 durante las primeras 24 horas de vencimiento. Aquí sí se muestra el nombre del socio, porque es un listado operativo; **el reporte del §3.7 no**.

### 3.7 Reporte

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| GET | `/reports/most-borrowed` | admin | `from` y `to` obligatorios en formato `YYYY-MM-DD`, fechas UTC inclusivas sobre `loaned_at` |

El reporte consulta el intervalo de instantes `loaned_at >= from 00:00:00 UTC` y `loaned_at < (to + 1 día) 00:00:00 UTC`. Si `from > to`, responde 422 `INVALID_DATE_RANGE`.

```json
{
  "from": "2026-10-01", "to": "2026-10-31",
  "rows": [
    { "book_id": "…", "book_title": "Cien años de soledad", "category_name": "Novela", "times_borrowed": 12 }
  ]
}
```

Una fila por `book_id + book_title + category_name` congelados. Orden: `times_borrowed` descendente, luego título, `book_id` y `category_name` ascendentes para desempatar. Sin socios ni bibliotecarios.

## 4. Seguridad

| Aspecto | Regla |
|---|---|
| Token | JWT firmado, 8 horas, con `sub` y `role`. Sin datos personales dentro |
| Transporte | Fuera del entorno local, la API se expone por HTTPS/TLS; las contraseñas solo se envían en el cuerpo de `POST /auth/login` y `POST /users` |
| Contraseñas | Solo se persiste el hash; la contraseña no se almacena en claro ni aparece en logs o respuestas; el hash nunca sale en ninguna respuesta |
| Autorización | Se evalúa en cada endpoint por rol; un 403 no revela si el recurso existe |
| CORS | Solo el origen del frontend configurado por entorno |
| Subida de portada | Solo `image/jpeg` y `image/png`, máx. 2 MB; el tipo se valida por contenido, no por extensión |
