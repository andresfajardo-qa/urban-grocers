# Urban Grocers API – Resumen de bugs

Este documento resume los principales defectos encontrados durante las pruebas manuales de la API de Urban Grocers:

- `POST /api/v1/kits/:id/products`
- `POST /order-and-go/v1/delivery`

Los bugs se agrupan por comportamiento y se indican los casos de prueba y tickets Jira (UG1-xx) relacionados.

---

## BUG-01 – ID de Kit en formato string genera 500 en lugar de 404

**Endpoint:** `POST /api/v1/kits/:id/products`  
**Casos relacionados:** “Agregar producto a kit con ID del Kit en formato string ("abc")”  
**Jira:** UG1-1  
**Severidad:** Alta

**Descripción**  
Cuando el parámetro de ruta `kitId` se envía como string (por ejemplo `"abc"`), la API responde con `500 Internal Server Error`, lo que indica una excepción no controlada en el backend.

**Comportamiento esperado**  
- La petición debería rechazarse con `404 Not Found` (recurso inexistente) o `400 Bad Request` (formato inválido), sin que el servidor se caiga.

**Comportamiento actual**  
- Respuesta: `500 Internal Server Error`.

---

## BUG-02 – ID de Kit negativo / vacío / omitido devuelve códigos 5xx/4xx inconsistentes

**Endpoint:** `POST /api/v1/kits/:id/products`  
**Casos relacionados:**  
- ID negativo (-5) – UG1-2  
- ID vacío (`""`) – UG1-3  
- ID omitido – UG1-4  
**Severidad:** Alta

**Descripción**  
Varios valores inválidos de `kitId` no se manejan de forma coherente:

- ID negativo → `501 Internal Server Error`
- ID vacío → `405 Method Not Allowed`
- ID omitido → `406 Method Not Allowed`

**Comportamiento esperado**  
- Todos estos casos deberían responder con un **error de cliente coherente**, por ejemplo `404 Not Found` o `400 Bad Request`.

**Comportamiento actual**  
- Mezcla de errores de servidor / método (`501`, `405`, `406`), lo que complica el diagnóstico y evidencia falta de validaciones.

---

## BUG-03 – Valores inválidos de ID de producto devuelven 500 en lugar de 400

**Endpoint:** `POST /api/v1/kits/:id/products`  
**Casos relacionados:**  
- ID de producto string `"x123"` – UG1-5  
- ID de producto negativo (-1) – UG1-6  
- ID de producto vacío – UG1-7  
**Severidad:** Alta

**Descripción**  
Cuando `productsList[i].id` se envía como string, negativo o vacío, la API se cae con `500 Internal Server Error`.

**Comportamiento esperado**  
- La API debería devolver `400 Bad Request` con un mensaje de validación.

**Comportamiento actual**  
- Respuesta: `500 Internal Server Error`.

---

## BUG-04 – Falta el ID de producto y la API responde 200 OK

**Endpoint:** `POST /api/v1/kits/:id/products`  
**Caso relacionado:**  
- “Agregar producto a kit con ID válido omitiendo el parámetro de ID del Producto” – UG1-8  
**Severidad:** Alta

**Descripción**  
Si el objeto de producto se envía sin el campo `id`, la API igualmente responde `200 OK`.

**Comportamiento esperado**  
- La petición debería devolver `400 Bad Request`, ya que `productsList[i].id` es obligatorio.

**Comportamiento actual**  
- Respuesta: `200 OK`, permitiendo datos potencialmente inválidos.

---

## BUG-05 – Cantidades inválidas no se validan correctamente

**Endpoint:** `POST /api/v1/kits/:id/products`  
**Casos relacionados:**  
- Cantidad string `"cinco"` – UG1-9  
- Cantidad decimal `1.5` – UG1-10  
- Cantidad negativa `-2` – UG1-11  
- Cantidad vacía `""` – UG1-12  
- Cantidad omitida – UG1-13  
- Cantidad igual a `0` – UG1-14  
**Severidad:** Alta

**Descripción**  
Distintos valores inválidos de `quantity` producen respuestas inesperadas:

- Algunos casos devuelven `500 Internal Server Error`.
- Otros devuelven `200 OK` aun cuando la cantidad es lógicamente incorrecta (por ejemplo `-2` o `0`).

**Comportamiento esperado**  
- Todas las cantidades inválidas o no positivas deberían devolver `400 Bad Request`.

**Comportamiento actual**  
- Mezcla de respuestas `500` y `200 OK`, lo que indica falta de validaciones y manejo de errores.

---

## BUG-06 – Valores inválidos de deliveryTime se aceptan en lugar de devolver 400

**Endpoint:** `POST /order-and-go/v1/delivery`  
**Casos relacionados:**  
- deliveryTime string, negativo, cero, vacío – UG1-15, UG1-16, UG1-17, UG1-18  
**Severidad:** Alta

**Descripción**  
Cuando `deliveryTime` se envía como texto, negativo, cero o vacío, la API igualmente responde `200 OK` y en algunos casos marca `isItPossibleToDeliver = true`.

**Comportamiento esperado**  
- `400 Bad Request` con un mensaje de error de validación sobre `deliveryTime`.

**Comportamiento actual**  
- `200 OK` con una respuesta de negocio normal, enmascarando problemas de entrada.

---

## BUG-07 – Valores inválidos de productsCount se aceptan en lugar de devolver 400

**Endpoint:** `POST /order-and-go/v1/delivery`  
**Casos relacionados:**  
- productsCount string, negativo, cero, vacío – UG1-19, UG1-20, UG1-21, UG1-22  
**Severidad:** Media–Alta

**Descripción**  
Valores de `productsCount` que deberían ser inválidos (texto, negativos, 0, vacíos) no son rechazados; la API responde con `200 OK`.

**Comportamiento esperado**  
- `400 Bad Request` indicando que el recuento de productos es inválido.

**Comportamiento actual**  
- `200 OK` con cálculo normal.

---

## BUG-08 – Valores inválidos de productsWeight se aceptan en lugar de devolver 400

**Endpoint:** `POST /order-and-go/v1/delivery`  
**Casos relacionados:**  
- productsWeight string, negativo, cero, vacío – UG1-26, UG1-27, UG1-28, UG1-29  
**Severidad:** Media–Alta

**Descripción**  
Valores inválidos de `productsWeight` se procesan como válidos y la API responde `200 OK`.

**Comportamiento esperado**  
- `400 Bad Request` cuando el peso es no numérico, negativo, cero o vacío.

**Comportamiento actual**  
- `200 OK` con respuesta estándar de entrega.

---

## BUG-09 – isItPossibleToDeliver y clientDeliveryCost no respetan las reglas de negocio

**Endpoint:** `POST /order-and-go/v1/delivery`  
**Casos relacionados:**  
- deliveryTime = 7, 23 – UG1-33, UG1-34  
**Severidad:** Alta

**Descripción**  
Para horarios que deberían estar fuera de la ventana de entrega, la API sigue devolviendo `isItPossibleToDeliver = true` y un `clientDeliveryCost` mayor que 0.

**Comportamiento esperado**  
- `isItPossibleToDeliver = false` y `clientDeliveryCost = 0` cuando no es posible realizar la entrega.

**Comportamiento actual**  
- `isItPossibleToDeliver = true` y costos distintos de cero, en contradicción con los requisitos.

---

## BUG-10 – Cálculo incorrecto de hostDeliveryCost / clientDeliveryCost en ciertos rangos de peso y cantidad

**Endpoint:** `POST /order-and-go/v1/delivery`  
**Casos relacionados:**  
- Pesos 3.1, 3.2, 6.1 – UG1-23, UG1-24, UG1-25  
- Combinaciones con contador y peso (5 productos & 3.1 kg, 5 & 6.1 kg, 16 & 2.0 kg) – UG1-30, UG1-31, UG1-32  
**Severidad:** Alta

**Descripción**  
Los valores calculados de `hostDeliveryCost` y `clientDeliveryCost` no coinciden con las reglas tarifarias esperadas para ciertos valores límite y combinaciones de peso + cantidad.

**Comportamiento esperado**  
- Que los costos sigan las reglas de precio definidas para cada rango y combinación.

**Comportamiento actual**  
- La API devuelve costos menores o diferentes a los esperados (por ejemplo `hostDeliveryCost = 3` en lugar de 5, o `clientDeliveryCost = 0` en lugar de 9).

---
