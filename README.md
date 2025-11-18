# Urban Grocers – Pruebas de API (Kits & Delivery)

Proyecto de portafolio del módulo **Tester Manual de Aplicaciones Web – TripleTen**.  
Este repositorio contiene las pruebas manuales realizadas sobre dos endpoints críticos de la API de Urban Grocers:

- `POST /api/v1/kits/:id/products`
- `POST /order-and-go/v1/delivery`

El objetivo fue validar reglas de negocio, códigos HTTP, manejo de errores y consistencia de las respuestas JSON.

---

## 🎯 Objetivo del proyecto

Asegurar que los endpoints cumplan correctamente con:

- Validación de IDs, cantidades y tipos de datos.
- Restricciones de negocio (máximo de productos, rangos permitidos).
- Cálculo correcto de:
  - `isItPossibleToDeliver`
  - `hostDeliveryCost`
  - `clientDeliveryCost`
- Manejo adecuado de datos inválidos y escenarios límite.

---

## 🧪 Alcance de las pruebas

Total de **56 casos de prueba**, distribuidos así:

### ✔ 20 casos – `POST /api/v1/kits/:id/products`
Incluyen:

- Casos positivos: agregar productos válidos, cantidades correctas, límites como 29–30 productos.
- Casos negativos: IDs inválidos (string, negativos, vacíos), productos omitidos, cantidades incorrectas (string, decimal, negativa, cero).
- Validación de códigos HTTP esperados (`200`, `400`, `404`).
- Identificación de inconsistencias donde la API devuelve respuestas inesperadas (`500`, `501`, `405`, `406`).

### ✔ 36 casos – `POST /order-and-go/v1/delivery`
Incluyen:

- Validación de la ventana horaria de entrega.
- Reglas para determinar `isItPossibleToDeliver`.
- Validación de tipos de datos incorrectos (string, vacío, negativo).
- Cálculo de costos según peso (`productsWeight`) y cantidad (`productsCount`).
- Casos límite para cada tramo de peso (0, 0.1, 2.9, 3.0, 3.1, 6.0, 6.1…).

Toda la documentación está en el archivo de casos de prueba del repositorio.

---

## 📁 Estructura del repositorio

```text
.
├─ README.md
├─ /screenshots           # Evidencias visuales del proyecto
├─ /test-cases            # Casos de prueba (Excel o PDF)
├─ /postman               # Colecciones de Postman
└─ /bug-summary           # Resumen de fallas encontradas
