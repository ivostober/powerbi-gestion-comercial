# 📋 Documentación de Tablas

## dim_fecha (Dimensión Temporal)

**Tipo:** Dimensión | **Registros:** 365+ días  
**Propósito:** Jerarquía temporal para análisis por período

### Columnas

| Columna | Tipo | Descripción |
|---------|------|-------------|
| `fecha` | Date | Clave primaria (PK). Fecha calendario |
| `ano` | Integer | Año (2023, 2024) |
| `ano_mes` | String | Código YYYYmm para ordenamiento |
| `mes` | Integer | Mes (1-12) |
| `nombre_mes` | String | Nombre localizado (enero, febrero...) |
| `trimestre` | String | Q1, Q2, Q3, Q4 |

### Jerarquía Recomendada
```
año → trimestre → mes → nombre_mes → fecha
```

---

## dim_producto (Dimensión Producto)

**Tipo:** Dimensión | **Registros:** 7 líneas  
**Propósito:** Catálogo de productos para análisis de portafolio

### Columnas

| Columna | Tipo | Descripción |
|---------|------|-------------|
| `producto` | String | Clave primaria (PK). Nombre de línea (Laptop, Monitor, etc.) |

### Valores Válidos
- Laptop (Principal, 117.5% cumplimiento)
- Monitor
- Auriculares
- Teclado
- Webcam
- Mouse (Rezagado, 21.7%)
- Tedado (sic)

---

## dim_region (Dimensión Región)

**Tipo:** Dimensión | **Registros:** 5 regiones  
**Propósito:** Estructura geográfica para análisis regional

### Columnas

| Columna | Tipo | Descripción |
|---------|------|-------------|
| `region` | String | Clave primaria (PK). Nombre geográfico |

### Valores Válidos
- **Sur** (Mejor desempeño, $251.3M)
- **Occidente** ($258.4M)
- **Norte** ($256.1M)
- **Oriente** ($244.1M)
- **Centro** (Menor desempeño, $190.4M)

---

## ventas_comerciales (Tabla de Hechos - Transacciones)

**Tipo:** Fact Table | **Registros:** Miles de transacciones  
**Propósito:** Registro de todas las ventas reales

### Columnas

| Columna | Tipo | Descripción | Relación |
|---------|------|-------------|----------|
| `id_venta` | Integer | Clave primaria (PK) | - |
| `fecha` | Date | Clave foránea (FK) | dim_fecha |
| `producto` | String | Clave foránea (FK) | dim_producto |
| `region` | String | Clave foránea (FK) | dim_region |
| `cantidad` | Integer | Unidades vendidas | - |
| `monto_venta` | Decimal | Venta en $ (principal métrica) | - |
| `precio_unitario` | Decimal | Precio por unidad | - |
| `vendedor` | String | Nombre del comercial | - |

### Hechos Agregables
- **monto_venta:** SUM, AVERAGE
- **cantidad:** COUNT, SUM

---

## metas_comerciales (Tabla de Hechos - Objetivos)

**Tipo:** Fact Table | **Registros:** Metas mensuales por producto/región  
**Propósito:** Registro de objetivos para comparación vs. real

### Columnas

| Columna | Tipo | Descripción | Relación |
|---------|------|-------------|----------|
| `meta_venta` | Decimal | Objetivo en $ (principal métrica) | - |
| `ano` | Integer | Año | dim_fecha |
| `mes` | Integer | Mes | dim_fecha |
| `producto` | String | Clave foránea (FK) | dim_producto |
| `region` | String | Clave foránea (FK) | dim_region |

### Características
- Desagregable por producto y región
- Permite cálculos de cumplimiento dinámicos

---

## _medidas (Tabla de Medidas - Solo en Power BI)

**Tipo:** Tabla de cálculos | **Ubicación:** Modelo implícito  
**Propósito:** Almacenar todas las medidas DAX para reutilización

### Medidas Contenidas
```
✓ total_ventas
✓ total_meta
✓ pct_cumplimiento
✓ ventas_acumuladas_ytd
✓ ventas_vs_anio_anterior
```

[Ver código completo en `/medidas`]

---

## 🔗 Diagrama de Relaciones

```
┌──────────────┐
│  dim_fecha   │
└──────┬───────┘
       │ (fecha)
       │
       ├────────────────┬────────────────┐
       │                │                │
       ▼                ▼                ▼
┌──────────────────┐ ┌──────────────────┐
│ ventas_comerciales│ │metas_comerciales │
└──────┬───────────┘ └──────┬───────────┘
       │                    │
       ├─(producto)         ├─(producto)
       │                    │
       ├─(region)           ├─(region)
       │
       └─(ano, mes para FECHA en metas)

┌──────────────────┐      ┌──────────────────┐
│  dim_producto    │      │   dim_region     │
└──────────────────┘      └──────────────────┘
```

### Relaciones Principales

| De | A | En | Cardinalidad |
|----|---|----|--------------|
| dim_fecha | ventas_comerciales | fecha | 1:N |
| dim_producto | ventas_comerciales | producto | 1:N |
| dim_region | ventas_comerciales | region | 1:N |
| dim_fecha | metas_comerciales | ano, mes | 1:N |
| dim_producto | metas_comerciales | producto | 1:N |
| dim_region | metas_comerciales | region | 1:N |

**Tipo de relaciones:** One-to-Many (1:N) | Dirección: De dimensión → Hecho

---

## 📊 Cardinalidad y Distribución

| Tabla | Registros | Crecimiento | Densidad |
|-------|-----------|-------------|----------|
| dim_fecha | 730 | Fija (2 años) | 100% |
| dim_producto | 7 | Fija | 100% |
| dim_region | 5 | Fija | 100% |
| ventas_comerciales | ~50,000 | Diario | 68% |
| metas_comerciales | ~840 | Mensual | 95% |

---

## 🎯 Recomendaciones de Índices (SQL)

Si los datos provienen de una base de datos relacional:

```sql
-- Índices recomendados para desempeño
CREATE INDEX idx_ventas_fecha ON ventas_comerciales(fecha);
CREATE INDEX idx_ventas_producto ON ventas_comerciales(producto);
CREATE INDEX idx_ventas_region ON ventas_comerciales(region);
CREATE INDEX idx_metas_ano_mes ON metas_comerciales(ano, mes);
```

---

## 🔄 Ciclo de Actualización

- **dim_fecha:** No requiere actualizaciones (tabla estática)
- **dim_producto:** Actualización anual o bajo demanda
- **dim_region:** Actualización bajo demanda
- **ventas_comerciales:** Diaria (preferentemente noche)
- **metas_comerciales:** Mensual (día 1 del mes)

---

**Última revisión:** Junio 2026  
**Compatibilidad:** Power BI 2.100+
