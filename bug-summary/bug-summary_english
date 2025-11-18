# Urban Grocers API – Bug Summary

This document summarizes the main defects found while testing the Urban Grocers API:

- `POST /api/v1/kits/:id/products`
- `POST /order-and-go/v1/delivery`

Bugs are grouped by behavior. Each group references the original test cases and Jira tickets (UG1-xx).

---

## BUG-01 – String Kit ID causes 500 instead of 404

**Endpoint:** `POST /api/v1/kits/:id/products`  
**Related test case(s):** “Agregar producto a kit con ID del Kit en formato string ("abc")”  
**Jira:** UG1-1  
**Severity:** High

**Description**  
When the `kitId` path parameter is sent as a string value (e.g. `"abc"`), the API returns `500 Internal Server Error`. This indicates an unhandled exception on the backend.

**Expected behavior**  
- The request should be rejected with `404 Not Found` (non-existing resource) or `400 Bad Request` (invalid format), without crashing the server.

**Actual behavior**  
- Response: `500 Internal Server Error`.

---

## BUG-02 – Negative / empty / missing Kit ID returns 5xx/4xx inconsistent codes

**Endpoint:** `POST /api/v1/kits/:id/products`  
**Related test cases:**  
- Kit ID negative (-5) – UG1-2  
- Kit ID empty (`""`) – UG1-3  
- Kit ID omitted – UG1-4  
**Severity:** High

**Description**  
Several invalid `kitId` values are not handled consistently:

- Negative ID → `501 Internal Server Error`
- Empty ID → `405 Method Not Allowed`
- Missing ID → `406 Method Not Allowed`

**Expected behavior**  
- All these cases should be rejected with a **consistent client error**, typically `404 Not Found` or `400 Bad Request`.

**Actual behavior**  
- Different server / method errors (`501`, `405`, `406`), which makes debugging difficult and suggests poor input validation.

---

## BUG-03 – Invalid Product ID values return 500 instead of 400

**Endpoint:** `POST /api/v1/kits/:id/products`  
**Related test cases:**  
- Product ID as string `"x123"` – UG1-5  
- Product ID negative (-1) – UG1-6  
- Product ID empty – UG1-7  
**Severity:** High

**Description**  
When `productsList[i].id` is sent as string, negative, or empty, the API crashes with `500 Internal Server Error`.

**Expected behavior**  
- The API should return `400 Bad Request` with a validation error message.

**Actual behavior**  
- Response: `500 Internal Server Error`.

---

## BUG-04 – Missing Product ID in productsList is accepted (200 OK)

**Endpoint:** `POST /api/v1/kits/:id/products`  
**Related test case:**  
- “Agregar producto a kit con ID válido omitiendo el parámetro de ID del Producto” – UG1-8  
**Severity:** High

**Description**  
If the product object is sent without an `id` field, the API still returns `200 OK`.

**Expected behavior**  
- Request should be rejected with `400 Bad Request` because `productsList[i].id` is required.

**Actual behavior**  
- Response: `200 OK`, which may create inconsistent or invalid data on the server side.

---

## BUG-05 – Invalid quantity values are not validated correctly

**Endpoint:** `POST /api/v1/kits/:id/products`  
**Related test cases:**  
- Quantity as string `"cinco"` – UG1-9  
- Quantity as decimal `1.5` – UG1-10  
- Quantity negative `-2` – UG1-11  
- Quantity empty `""` – UG1-12  
- Quantity omitted – UG1-13  
- Quantity equal to `0` – UG1-14  
**Severity:** High

**Description**  
Different invalid `quantity` values produce unexpected responses:

- Some cases return `500 Internal Server Error`.
- Others return `200 OK` even when the value is logically invalid (e.g. `-2`, `0`).

**Expected behavior**  
- All invalid or non-positive quantities should be rejected with `400 Bad Request`.

**Actual behavior**  
- Mix of `500` errors and `200 OK` responses, indicating missing validation and error handling.

---

## BUG-06 – Invalid deliveryTime values are accepted instead of returning 400

**Endpoint:** `POST /order-and-go/v1/delivery`  
**Related test cases:**  
- deliveryTime as string, negative, zero, empty – UG1-15, UG1-16, UG1-17, UG1-18  
**Severity:** High

**Description**  
When `deliveryTime` is sent as a string, negative number, zero, or empty value, the API still returns a normal business response (`200 OK`) and even sets `isItPossibleToDeliver = true`.

**Expected behavior**  
- `400 Bad Request` with an error payload describing the invalid `deliveryTime`.

**Actual behavior**  
- `200 OK` with a normal response body, hiding invalid input issues.

---

## BUG-07 – Invalid productsCount values are accepted instead of returning 400

**Endpoint:** `POST /order-and-go/v1/delivery`  
**Related test cases:**  
- productsCount as string, negative, zero, empty – UG1-19, UG1-20, UG1-21, UG1-22  
**Severity:** Medium–High

**Description**  
`productsCount` values that should be invalid (string, negative, 0, empty) are not rejected; the API returns a successful `200 OK` response.

**Expected behavior**  
- `400 Bad Request` with a clear validation error.

**Actual behavior**  
- `200 OK` and normal calculation, which can lead to incorrect logistics decisions.

---

## BUG-08 – Invalid productsWeight values are accepted instead of returning 400

**Endpoint:** `POST /order-and-go/v1/delivery`  
**Related test cases:**  
- productsWeight as string, negative, zero, empty – UG1-26, UG1-27, UG1-28, UG1-29  
**Severity:** Medium–High

**Description**  
Invalid `productsWeight` values are processed as if they were valid, and the API still returns `200 OK`.

**Expected behavior**  
- `400 Bad Request` when the weight is non-numeric, negative, zero or empty.

**Actual behavior**  
- `200 OK` with a regular delivery calculation.

---

## BUG-09 – isItPossibleToDeliver and clientDeliveryCost do not follow business rules

**Endpoint:** `POST /order-and-go/v1/delivery`  
**Related test cases:**  
- deliveryTime = 7, 23 – UG1-33, UG1-34  
**Severity:** High

**Description**  
For times that should be outside the delivery window, the API still returns `isItPossibleToDeliver = true` and non-zero `clientDeliveryCost`.

**Expected behavior**  
- `isItPossibleToDeliver = false` and `clientDeliveryCost = 0` for non-deliverable time slots.

**Actual behavior**  
- `isItPossibleToDeliver = true` and client delivery cost greater than 0, contradicting the requirements.

---

## BUG-10 – hostDeliveryCost / clientDeliveryCost miscalculated for certain weight and count ranges

**Endpoint:** `POST /order-and-go/v1/delivery`  
**Related test cases:**  
- Weight 3.1, 3.2, 6.1 – UG1-23, UG1-24, UG1-25  
- Combinations with productsCount and productsWeight (e.g. 5 items & 3.1kg, 5 items & 6.1kg, 16 items & 2.0kg) – UG1-30, UG1-31, UG1-32  
**Severity:** High

**Description**  
The calculated `hostDeliveryCost` and `clientDeliveryCost` do not match the expected tariff rules for boundary and combined values of weight and product count.

**Expected behavior**  
- Costs should follow the documented pricing rules for each range and combination.

**Actual behavior**  
- The API returns lower or different costs than expected (e.g. `hostDeliveryCost` = 3 instead of 5, or `clientDeliveryCost` = 0 instead of 9).

---
