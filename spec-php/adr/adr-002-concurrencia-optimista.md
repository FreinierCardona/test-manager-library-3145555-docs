# ADR-002 · Concurrencia optimista sobre los ejemplares

- **Estado:** aceptada
- **Contexto:** dos bibliotecarios pueden prestar el **último ejemplar** del mismo libro al mismo tiempo (RNF-2). Si ambos leen
  `available_copies = 1` y ambos escriben, el sistema presta un libro que no tiene.
- **Decisión:** control de concurrencia **optimista** mediante `book.inventory_version` (`bigint`, inicia en 1). Toda mutación de
  existencias actualiza la fila con `WHERE id = :id AND inventory_version = :expected_version` e incrementa el token en la misma
  escritura. El préstamo y sus líneas se guardan en la misma
  transacción que los descuentos; cualquier conflicto o error revierte la operación completa.
- **Conflicto:** ante una versión obsoleta, el adaptador vuelve a leer y revalida el inventario, y reintenta una vez. Si ya no quedan
  ejemplares, responde **409 `BOOK_NOT_AVAILABLE`**. Si la versión vuelve a cambiar y aún hay existencias, responde **409
  `BOOK_CONCURRENTLY_MODIFIED`** para que el cliente pueda reintentar la solicitud completa. La **última barrera** es el `CHECK`
  `ck_book_available_range` (`0 <= available_copies <= total_copies`), complementado por `ck_book_inventory_version_positive`.
- **Alternativas descartadas:** bloqueo pesimista de la fila (`SELECT … FOR UPDATE`) en cada préstamo: serializa toda la biblioteca por
  un caso raro; contador en memoria: no sobrevive a más de un proceso.
- **Consecuencias:** `inventory_version` forma parte del modelo físico y no lleva `DEFAULT`; la aplicación lo inicializa en 1.
  Si cualquiera de los `CHECK` salta, **algo escribió fuera del adaptador**: es un defecto, no un caso de negocio. Devolver dos veces
  la misma línea no suma dos veces porque `returned_at` ya está lleno (CA-07.2).
- **Cómo se verifica:** prueba concurrente con dos peticiones simultáneas sobre un libro con un ejemplar: exactamente una recibe 201
  y la otra 409 `BOOK_NOT_AVAILABLE`; una prueba adicional fuerza un conflicto optimista con existencias restantes y comprueba el
  único reintento y la respuesta `BOOK_CONCURRENTLY_MODIFIED` si vuelve a cambiar la versión.
