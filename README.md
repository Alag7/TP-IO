# Optimización de Distribución de Productos Lácteos

**TP Investigación Operativa 2026**  
**Integrantes:** Alaguibe, Muñoz, Pozzo, Zeballos

---

## Descripción del Problema

Este proyecto modela y resuelve un problema de **programación lineal** orientado a la optimización de costos de transporte en la cadena de distribución de una empresa láctea argentina. El modelo determina la cantidad óptima de cada producto a enviar desde cada planta productora hacia cada centro de distribución, minimizando los costos totales de transporte y procesamiento, sujeto a restricciones de capacidad productiva, disponibilidad de materia prima y satisfacción de la demanda.

---

## Archivos del Proyecto

| Archivo | Descripción |
|---|---|
| `DATASOURCE.xlsx` | Parámetros de entrada del modelo (conjuntos, capacidades, costos, demandas) |
| `opt-proccess-lng.lng` | Modelo de optimización en lenguaje LINGO |
| `RESULTADOS.xlsx` | Solución óptima generada por el solver |

---

## Estructura del Modelo (`opt-proccess-lng.lng`)

El modelo está escrito en **LINGO** e importa sus datos directamente desde `DATASOURCE.xlsx` mediante la función `@OLE`. Los resultados se exportan automáticamente a `RESULTADOS.xlsx`.

### Variables de Decisión

- **`X(I, J, K)`** — Cantidad del producto `I` enviada desde la planta `J` al centro de distribución `K`.
- **`Y(J)`** — Litros de leche procesados (materia prima utilizada) en la planta `J`.
- **`LECHE_DESCREMADA_SF(J)`** — Litros de leche descremada enviados desde la planta `J` (Buenos Aires o Córdoba) hacia Santa Fe para ser procesados allí.

### Función Objetivo

Minimizar el costo total:

```
MIN = Σ COSTO(I,J,K) × X(I,J,K)  +  Σ COSTO_LD_SF(J) × LECHE_DESCREMADA_SF(J)
```

### Restricciones

- **Balance de leche procesada:** la materia prima consumida en cada planta debe ser igual a la suma ponderada de todos los productos enviados desde esa planta (usando factores de conversión litros-de-leche por unidad de producto).
- **Balance de leche descremada en Santa Fe:** la leche descremada recibida en P3 debe ser igual a la utilizada para producir LPD y LFD en esa planta.
- **Sin desperdicio de crema/manteca:** la crema separada en BA y Córdoba debe transformarse completamente en CREMA o MANTECA.
- **Disponibilidad de materia prima:** el total de leche procesada no puede superar el 98% de la leche disponible en el período (1.168.344.890 litros, con estacionalidad de octubre).
- **Satisfacción de demanda:** cada producto debe llegar en cantidad suficiente a cada centro de distribución.
- **Capacidades de planta:** se respetan los límites de producción de leche fluida, leche en polvo, crema, manteca y queso para cada planta.

---

## Datos de Entrada (`DATASOURCE.xlsx`)

### Plantas y Centros de Distribución

| Planta | Ubicación |
|---|---|
| P1 | Buenos Aires |
| P2 | Córdoba |
| P3 | Santa Fe |

| Centro | Ubicación |
|---|---|
| C1 | Capital Federal |
| C2 | Rosario |
| C3 | Mendoza |
| C4 | Tucumán |

### Productos

| Código | Producto | Factor de Conversión (lt/u) | Refrigerado |
|---|---|---|---|
| LFE | Leche Fluida Entera | 1.063 | Sí |
| LFD | Leche Fluida Descremada | 1.063 | Sí |
| LPE | Leche en Polvo Entera | 8.755 | No |
| LPD | Leche en Polvo Descremada | 12.25 | No |
| CREMA | Crema | 12.25 | Sí |
| MANTECA | Manteca | 34.028 | Sí |
| QB | Queso Blanco | 7.245 | Sí |
| QMB | Queso Muy Blando | 2.395 | Sí |

### Capacidades de Producción (unidades/período)

| Planta | Leche Fluida | Leche en Polvo | Crema | Manteca | Queso |
|---|---|---|---|---|---|
| P1 – Buenos Aires | 1.200.000 | 200.000 | 150.000 | 70.000 | 300.000 |
| P2 – Córdoba | 1.100.000 | 120.000 | 200.000 | 90.000 | 250.000 |
| P3 – Santa Fe | 1.350.000 | 110.000 | — | — | 250.000 |

> Santa Fe no produce Crema ni Manteca. La leche descremada sobrante de Buenos Aires y Córdoba se envía a Santa Fe para complementar su producción de LPD y LFD.

### Demanda Total País (referencia)

| Producto | Demanda total (unidades) |
|---|---|
| Leche Fluida Entera | 40.458.660 |
| Leche Fluida Descremada | 20.842.340 |
| Leche en Polvo Entera | 2.957.000 |
| Leche en Polvo Descremada | 1.009.270 |
| Crema | 7.171.000 lt |
| Manteca | 1.333.000 |
| Queso Blanco | 16.193.300 |
| Queso Muy Blando | 5.209.870 |

Los valores de demanda por centro se calculan distribuyendo la demanda nacional con las siguientes proporciones: Capital Federal 45%, Rosario 25%, Mendoza 18%, Tucumán 12%.

---

## Resultados Óptimos (`RESULTADOS.xlsx`)

### Producción por Planta

| Planta | Leche Procesada (litros) |
|---|---|
| P1 – Buenos Aires | 4.480.622,53 |
| P2 – Córdoba | 2.262.609,34 |
| P3 – Santa Fe | 1.490.206,41 |
| **Total** | **8.233.438,28** |

### Leche Descremada Enviada a Santa Fe

| Origen | Litros enviados a P3 |
|---|---|
| P1 – Buenos Aires | 89.559,05 |
| P2 – Córdoba | 153.989,62 |

### Distribución de Envíos (selección de flujos activos)

Cada planta abastece preferentemente a los centros de menor costo de transporte:

- **P1 (Buenos Aires)** → C1 Capital Federal (todos los productos)
- **P2 (Córdoba)** → C3 Mendoza y C4 Tucumán
- **P3 (Santa Fe)** → C2 Rosario

---

## Cómo Ejecutar el Modelo

1. Abrir el archivo `opt-proccess-lng.lng` en **LINGO**.
2. Verificar que `DATASOURCE.xlsx` y `RESULTADOS.xlsx` se encuentren en el **mismo directorio** que el archivo `.lng` (el modelo los referencia con rutas relativas mediante `@OLE`).
3. Ejecutar el solver (Solve / `Ctrl+S`).
4. Los resultados se escriben automáticamente en las hojas de `RESULTADOS.xlsx`:
   - `RESULTADOS ENVIOS (X)` — cantidades a enviar por producto, planta y centro.
   - `RESULTADOS PRODUCCION (Y)` — leche procesada por planta.
   - `RESULTADOS ENVIOS A SF (LD)` — leche descremada enviada a Santa Fe.

---

## Dependencias

- **LINGO** (cualquier versión con soporte `@OLE`)
- **Microsoft Excel** (para la lectura/escritura de `DATASOURCE.xlsx` y `RESULTADOS.xlsx`)
- Sistema operativo Windows (requerido por el conector OLE)