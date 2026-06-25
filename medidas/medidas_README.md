# 🧮 Medidas DAX - Índice Completo

Documentación de todas las medidas utilizadas en el dashboard "Informe de Desempeño Comercial".

## 📑 Índice de Medidas

### Medidas de Ventas
- [`total_ventas`](#1-total_ventas) — Suma de todas las ventas
- [`promedio_venta_producto`](#6-promedio_venta_producto) — Ticket promedio

### Medidas de Metas
- [`total_meta`](#2-total_meta) — Suma de objetivos comerciales
- [`pct_cumplimiento`](#3-pct_cumplimiento) — % cumplimiento de metas

### Medidas Temporales
- [`ventas_acumuladas_ytd`](#4-ventas_acumuladas_ytd) — Acumulado año a la fecha
- [`ventas_vs_anio_anterior`](#5-variación-interanual-yoy) — Variación YoY
- [`variacion_mensual`](#7-variación-mensual-mom) — Variación MoM

### Medidas por Dimensión
- [`cumplimiento_producto`](#8-cumplimiento-por-producto) — Cumplimiento por línea
- [`cumplimiento_region`](#9-cumplimiento-por-región) — Cumplimiento geográfico

### Validación
- [`meses_con_meta`](#10-contador-de-meses-con-meta) — Control de cobertura de datos

---

## 💡 Cómo Usar Este Material

### Para Estudiantes
1. Empieza por `total_ventas` y `total_meta` (conceptos básicos)
2. Luego avanza a `pct_cumplimiento` (DIVIDE)
3. Sigue con medidas temporales (CALCULATE, DATEADD)
4. Finalmente, medidas complejas con variables (VAR)

### Para Desarrolladores
1. Copia el código completo desde `medidas_dax.dax`
2. Adapta los nombres de tablas/columnas a tu modelo
3. Ajusta contextos de filtro según tu arquitectura
4. Mantén la estructura con tabla `_medidas` dedicada

### Para Portafolio
1. Explica qué hace cada medida (impacto de negocio)
2. Muestra cómo se usa en visualizaciones
3. Destaca la lógica compleja (VAR, HASONEVALUE)
4. Comenta sobre rendimiento y optimización

---

## 🎯 Contexto de Uso en el Dashboard

### Página 1: Portada (RESUMEN EJECUTIVO)
```
Muestra KPIs principales:
[total_ventas] — $1,230.3M (grande, destacado)
[total_meta] — $1,482.1M (referencia)
[pct_cumplimiento] — 83% (rojo si <80%, verde si ≥85%)
[ventas_vs_anio_anterior] — -12% (rojo por caída)
```

### Página 2: Evolución Temporal
```
Gráfico de línea (mes a mes):
X: nom_mes (enero a diciembre)
Y1: [total_ventas] (año 2024, línea azul)
Y2: [total_ventas] (año 2023, línea rosa)
Slicer: AÑO (2023, 2024)
```

### Página 3: Productos
```
Gráfico de barras horizontales por línea:
Y: producto (Laptop, Monitor, etc.)
X1: [cumplimiento_producto] (color azul, barras)
Línea referencia: [promedio_venta_producto]
Filtro: Año, Mes
```

### Página 4: Regiones
```
Gráfico de barras agrupadas:
X: region (Sur, Occidente, etc.)
Y1: [total_ventas] (azul oscuro)
Y2: [total_meta] (rosa)
Filtro: Año, Mes
```

---

## 🔧 Adaptación de Medidas a Tu Modelo

Si tu modelo tiene nombres diferentes, actualiza así:

```dax
// ❌ Nombres estándares (proyecto actual)
SUM(ventas_comerciales[monto_venta])

// ✅ Adaptado a otro modelo
SUM(FactVentas[SalesAmount])
```

**Cambios comunes:**
- `ventas_comerciales` → `FactSales`, `tbFactVentas`, `Transacciones`
- `monto_venta` → `amount`, `sales_amount`, `importe`
- `meta_venta` → `target`, `goal_amount`, `meta`
- `dim_fecha` → `DimDate`, `Calendario`, `Fechas`

---

## ⚡ Optimización y Performance

### Medidas Pesadas (evitar exceso)
- ❌ `FILTER` sobre tabla de hechos grande
- ❌ Múltiples `CALCULATE` anidados sin `VAR`
- ❌ Comparativas YoY sin `DATEADD`

### Medidas Ligeras (recomendadas)
- ✅ `SUM` directo sobre hechos
- ✅ Variables para reutilización
- ✅ `CALCULATE` con contexto específico
- ✅ `DIVIDE` para ratios seguros

### Ejemplo de Optimización
```dax
// ❌ Lento
medida_lenta = 
CALCULATE(
    SUM(tabla[X]),
    FILTER(tabla, tabla[Y] > 100)
)

// ✅ Rápido
medida_rapida = 
VAR filtrado = FILTER(tabla, tabla[Y] > 100)
RETURN
    CALCULATE(SUM(tabla[X]), filtrado)
```

---

## 📚 Recursos para Profundizar

### Conceptos Clave en Nuestras Medidas

| Función | Uso en Este Proyecto | Aprende Más |
|---------|---------------------|------------|
| `CALCULATE` | Cambiar contexto de filtro | `total_meta` |
| `DIVIDE` | División segura | `pct_cumplimiento` |
| `TOTALYTD` | Acumulado año a fecha | `ventas_acumuladas_ytd` |
| `DATEADD` | Desplazar períodos | `variacion_mensual` |
| `VAR` | Variables para legibilidad | `ventas_vs_anio_anterior` |
| `HASONEVALUE` | Contexto condicional | `total_meta` |
| `VALUES` | Obtener valor filtrado | `total_meta` |
| `DISTINCTCOUNT` | Contar únicos | `promedio_venta_producto` |
| `FILTER` | Filtrar tablas | `cumplimiento_producto` |

---

## 🧪 Testing de Medidas

Verifica que tus medidas funcionen correctamente:

```dax
// Test 1: ¿Suma correctamente?
ASSERT([total_ventas] = 1230.3M, "Error en suma de ventas")

// Test 2: ¿Responde a filtros?
// En visualización, filtra por producto "Laptop"
// Verifica que [total_ventas] solo sumar Laptop (~$710M)

// Test 3: ¿YTD calcula correctamente?
// Filtra mes = diciembre
// [ventas_acumuladas_ytd] debe ≈ [total_ventas] (acumulado anual)

// Test 4: ¿División por cero?
// Filtra producto con meta = 0
// [pct_cumplimiento] debe retornar 0, no #DIV/0!
```

---

## 📝 Buenas Prácticas

### 1. Nombre las medidas claramente
```dax
// ❌ Poco claro
medida1 = SUM(...)

// ✅ Descriptivo
total_ventas_por_producto = SUM(...)
```

### 2. Usa comentarios para lógica compleja
```dax
// Obtiene metas solo para el año seleccionado
// Si no hay año seleccionado, toma el máximo
VAR anio_seleccionado = 
    IF(HASONEVALUE(...))
```

### 3. Organiza medidas por categoría
```
_medidas.total_ventas        (Grupo: Ventas)
_medidas.cumplimiento_producto (Grupo: Cumplimiento)
_medidas.variacion_mensual   (Grupo: Comparativas)
```

### 4. Documental fórmulas complejas
```dax
// Uso: Comparativa de desempeño interanual
// Contexto: Se filtra automáticamente por año, mes, región
// Limitaciones: Requiere datos del año anterior
medida_compleja = ...
```

---

## 🚀 Próximos Pasos

### Mejoras Sugeridas
1. **Medida de Meta Flexible** — Permitir ajuste dinámico de metas
2. **Forecast** — Proyección de ventas basada en tendencia
3. **Anomalías** — Detectar desviaciones significativas
4. **Benchmark** — Comparativa contra historial de mejores meses

### Ejemplo: Medida Mejorada
```dax
// Proyección simple basada en promedio YTD
ventas_proyectadas = 
VAR promedio_mensual = [total_ventas] / MONTH(TODAY())
VAR meses_restantes = 12 - MONTH(TODAY())
VAR proyeccion = [ventas_acumuladas_ytd] + (promedio_mensual * meses_restantes)
RETURN proyeccion
```

---

## 📧 FAQ

**P: ¿Por qué algunas medidas usan VAR?**  
R: VAR mejora legibilidad y performance al reutilizar cálculos.

**P: ¿Qué es HASONEVALUE?**  
R: Detecta si hay exactamente un valor en el contexto de filtro. Útil para lógica condicional.

**P: ¿Puedo cambiar los nombres de las medidas?**  
R: Sí, pero actualiza todas las referencias en visualizaciones y en medidas que las usen.

**P: ¿Cómo manejo datos faltantes?**  
R: DIVIDE(..., 0) retorna 0 en lugar de error. Para otros casos, usa COALESCE o IFERROR.

---

**Última actualización:** Junio 2024  
**Compatibilidad:** Power BI 2.100+ | DAX 2.0+  
**Autor:** Proyecto Hard Skills
