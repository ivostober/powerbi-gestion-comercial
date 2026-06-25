# 🛠️ Guía Técnica del Proyecto

Instrucciones para entender, replicar y adaptar el dashboard en tu entorno.

## 📋 Tabla de Contenidos

1. [Requisitos](#requisitos)
2. [Estructura del Modelo](#estructura-del-modelo)
3. [Setup Inicial](#setup-inicial)
4. [Cómo Leer Este Proyecto](#cómo-leer-este-proyecto)
5. [Troubleshooting](#troubleshooting)

---

## 📦 Requisitos

### Software
- **Power BI Desktop** (versión 2.100 o superior)
  - [Descargar](https://powerbi.microsoft.com/es-es/desktop/)
- **Power Query** (incluido en Power BI)
- **Excel** (opcional, para datos de origen)

### Conocimiento
- **Intermedio:** Star Schema, relaciones 1:N
- **Intermedio:** Funciones DAX básicas (SUM, CALCULATE, DIVIDE)
- **Básico:** Visualizaciones en Power BI

### Datos
- Acceso a tablas fuente (CSV o conexión directa)
- Dimensiones: fecha, producto, región
- Hechos: ventas_comerciales, metas_comerciales

---

## 🏗️ Estructura del Modelo

### Arquitectura

```
┌─────────────────────────────────────────────┐
│           Capa de Presentación              │
│  (Reportes / Visualizaciones)               │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│         Capa de Medidas (DAX)               │
│  (_medidas: total_ventas, pct_cumplimiento) │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│      Capa Semántica (Modelo Power BI)       │
│  (Star Schema con dimensiones y hechos)     │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│         Capa de Datos Origen                │
│  (CSV, SQL Server, Excel, etc.)             │
└─────────────────────────────────────────────┘
```

### Tablas y Relaciones

**Dimensiones (de referencia):**
- `dim_fecha` → jerarquía temporal
- `dim_producto` → líneas de producto
- `dim_region` → geografía

**Hechos (transacciones):**
- `ventas_comerciales` → registro de ventas reales
- `metas_comerciales` → objetivos por período

**Tipo de modelo:** Star Schema (estrella)

---

## 🚀 Setup Inicial

### Paso 1: Preparar los Datos

1. **Obtener datos de origen:**
   - Solicitar export de `ventas_comerciales.csv`
   - Solicitar export de `metas_comerciales.csv`
   - Solicitar catálogo de `productos.csv`
   - Solicitar lista de `regiones.csv`

2. **Validar estructura (mínimo):**

   **ventas_comerciales:**
   ```
   id_venta | fecha | producto | region | cantidad | monto_venta
   1        | 2024-01-15 | Laptop | Sur | 5 | 5000000
   2        | 2024-01-15 | Mouse | Centro | 10 | 250000
   ```

   **metas_comerciales:**
   ```
   ano | mes | producto | region | meta_venta
   2024 | 1 | Laptop | Sur | 500000000
   2024 | 1 | Mouse | Sur | 50000000
   ```

3. **Cargar en Power BI:**
   - Archivo > Obtener datos > Texto/CSV
   - Selecciona cada archivo
   - Power Query abre automáticamente

### Paso 2: Modelado en Power BI

1. **Ir a Vista de Modelo (Pestaña Modelado)**

2. **Crear tabla dim_fecha:**
   ```
   En Power Query:
   - Nueva consulta > Rango vacío
   - Generar rango: 1/1/2023 a 31/12/2024
   - Agregar columnas derivadas: año, mes, nombre_mes, trimestre
   ```

3. **Crear relaciones:**
   ```
   dim_fecha --[fecha]--> ventas_comerciales
   dim_producto --[producto]--> ventas_comerciales
   dim_region --[region]--> ventas_comerciales
   
   dim_fecha --[ano, mes]--> metas_comerciales (si procede)
   ```

4. **Ocultar columnas técnicas:**
   ```
   Haz clic derecho > Ocultar en vista de reporte
   (guarda las columnas clave pero no las muestra)
   ```

### Paso 3: Crear Medidas

1. **Nueva tabla: `_medidas`**
   - Nueva tabla > Tabla vacía (solo para medidas)
   - No necesita datos

2. **Copiar medidas DAX:**
   - Abre archivo `medidas_dax.dax`
   - Copia cada medida
   - Pega en Power BI (Pestaña "Medidas")
   - Formula bar > enter

3. **Formato de medidas:**
   - Selecciona medida
   - Formato > Número (con decimales según corresponda)
   - Ejemplo: `pct_cumplimiento` > Porcentaje (2 dec.)

### Paso 4: Crear Visualizaciones

Replicas las 4 páginas del dashboard:

**Página 1: Portada**
- 5 tarjetas (Card)
  - total_ventas (grande, azul)
  - total_meta (mediano, gris)
  - pct_cumplimiento (grande, formato %)
  - variacion_anio_anterior (con símbolo ↓)
  - ventas_acumuladas_ytd

**Página 2: Evolución Temporal**
- Gráfico de línea
  - Eje X: nombre_mes
  - Eje Y: total_ventas
  - Leyenda: ano
- Slicers laterales: AÑO, MES

**Página 3: Análisis por Producto**
- Gráfico de barras horizontales
  - Eje Y: producto
  - Eje X: cumplimiento_producto
  - Línea de referencia: 100% (meta)
- Filtro: Producto (lista)

**Página 4: Análisis por Región**
- Gráfico de barras agrupadas
  - Eje X: region
  - Valor 1: total_ventas
  - Valor 2: total_meta
- Filtro: Región (lista)

---

## 📖 Cómo Leer Este Proyecto

### Para Aprender DAX

1. Empieza por la medida más simple:
   ```dax
   total_ventas = SUM(ventas_comerciales[monto_venta])
   ```
   → Esto suma TODOS los montos en contexto actual

2. Avanza a un DIVIDE seguro:
   ```dax
   pct_cumplimiento = DIVIDE([total_ventas], [total_meta], 0)
   ```
   → Ratio con manejo de error (retorna 0 si divide por 0)

3. Luego aprende CALCULATE:
   ```dax
   total_meta = CALCULATE(SUM(metas_comerciales[meta_venta]))
   ```
   → Suma que responde a contexto de filtros

4. Finalmente, variables complejas:
   ```dax
   VAR anio = IF(HASONEVALUE(...), VALUES(...), MAX(...))
   RETURN CALCULATE(...)
   ```

### Para Entender el Flujo de Datos

```
Datos Crudos (CSV)
       ↓
Power Query (limpiar, transformar)
       ↓
Modelo Power BI (relaciones, jerarquías)
       ↓
Medidas DAX (cálculos, lógica comercial)
       ↓
Visualizaciones (gráficos, tablas, tarjetas)
       ↓
Dashboard (narrativa, insights)
```

### Para Replicar Localmente

1. **Descargar código DAX:** Copia desde `medidas_dax.dax`
2. **Adaptar nombres:** Reemplaza si tus tablas se llaman diferente
3. **Validar relaciones:** Asegúrate que dimensiones → hechos en dirección correcta
4. **Testear medidas:** Filtra por un valor y verifica que suma sea correcta

---

## 🔍 Estructura de Carpetas Esperada

Si replicar localmente, organiza así:

```
mi_proyecto_powerbi/
├── datos_originales/
│   ├── ventas_comerciales.csv
│   ├── metas_comerciales.csv
│   ├── productos.csv
│   └── regiones.csv
│
├── powerbi/
│   ├── Informe_Desempeño_Comercial.pbix
│   └── Informe_Desempeño_Comercial.pbit (template)
│
├── documentacion/
│   ├── modelo_conceptual.pdf
│   ├── diccionario_datos.xlsx
│   └── guia_usuario.pdf
│
└── scripts/
    ├── limpiar_datos.py
    └── validar_integridad.sql
```

---

## 🧪 Validación del Modelo

Checklist para confirmar que tu modelo está correcto:

### Datos
- [ ] No hay valores NULL en columnas clave (fecha, producto, región)
- [ ] Fechas están en formato DATE (no texto)
- [ ] Montos son DECIMAL/CURRENCY (no texto)
- [ ] Catálogos (producto, región) no tienen duplicados

### Relaciones
- [ ] Cada relación es 1:N (dimensión → hecho)
- [ ] Dirección de filtro es correcta (dimensión → hecho)
- [ ] No hay relaciones circulares
- [ ] No hay cruce de muchos (N:N) a menos que sea intencional

### Medidas
- [ ] total_ventas retorna valor correcto
- [ ] pct_cumplimiento está entre 0 y 1 (o 0-100%)
- [ ] YTD responde a cambios de mes/año
- [ ] Variación YoY es negativa (-0.12) o positiva (0.15)

### Visualizaciones
- [ ] Filtros en una página NO afectan otras páginas (excepto navegación)
- [ ] Tarjetas muestran números sin decimales innecesarios
- [ ] Gráficos redondean a 1-2 decimales máximo
- [ ] Colores contrastan bien (accesibilidad)

---

## ❌ Troubleshooting

### Problema: Medida retorna #ERROR o #DIV/0!

**Causa común:** División por cero en `pct_cumplimiento`

**Solución:**
```dax
// ❌ Mal
pct_cumplimiento = [total_ventas] / [total_meta]

// ✅ Bien
pct_cumplimiento = DIVIDE([total_ventas], [total_meta], 0)
```

---

### Problema: YTD no acumula correctamente

**Causa común:** `dim_fecha` no está marcada como tabla de fechas

**Solución:**
1. Selecciona tabla `dim_fecha`
2. Pestaña "Modelado" > "Marcar como tabla de fechas"
3. Selecciona columna `fecha` como ID única

---

### Problema: Filtro no afecta a medida

**Causa común:** Relación no está configurada o dirección es incorrecta

**Solución:**
1. Ve a Vista de Modelo
2. Selecciona relación entre dim_producto y ventas_comerciales
3. Asegúrate que flecha apunta: dim_producto → ventas_comerciales
4. Hace filtrado cruzado: "Ambas direcciones" si lo requiere

---

### Problema: Medida lenta (tarda >5 segundos)

**Causa común:** FILTER sobre tabla de hechos sin índices

**Solución:**
```dax
// ❌ Lento
CALCULATE(SUM(...), FILTER(ventas_comerciales, ventas_comerciales[x] > 100))

// ✅ Rápido
VAR filtrado = FILTER(dim_producto, dim_producto[x] > 100)
RETURN CALCULATE(SUM(...), filtrado)
// (Filter sobre dimensión pequeña en lugar de hechos grandes)
```

---

### Problema: "El modelo no se puede procesar"

**Causa común:** Tabla de medidas contiene datos (debe estar vacía)

**Solución:**
1. Selecciona tabla `_medidas`
2. Editar consulta
3. Elimina cualquier paso de carga de datos
4. Cierra y aplica

---

## 📚 Recursos Adicionales

**Microsoft Docs:**
- [CALCULATE en DAX](https://learn.microsoft.com/es-es/dax/calculate-function-dax)
- [Relaciones en Power BI](https://learn.microsoft.com/es-es/power-bi/transform-model/desktop-relationships-understand)
- [Star Schema Design](https://learn.microsoft.com/es-es/power-bi/transform-model/star-schema)

**Comunidad:**
- Foro Power BI: `community.powerbi.com`
- Stack Overflow: Tag `power-bi` + `dax`
- Grupo Linkedin: "Power BI en Español"

**Libros (recomendados):**
- "The Definitive Guide to DAX" (Russo, Ferrari)
- "Power BI from Scratch" (Mitchell)

---

**Última actualización:** Junio 2024  
**Autor:** Proyecto Hard Skills
