# Dashboard de Desempeño Comercial | Power BI

Análisis de ventas por producto, región y período | 2023-2024

## 📊 Descripción del Proyecto

Dashboard interactivo desarrollado en **Power BI** para monitorear el desempeño comercial de una organización. Incluye análisis de ventas, cumplimiento de metas y variación anual, con segmentación por producto y región geográfica.

**Período:** 2023-2024  
**Datos:** ~$1.23B en ventas  
**KPIs principales:** Cumplimiento (83%), Variación YoY (-12%), Venta Acumulada YTD

## 🎯 Características Principales

### Análisis Multidimensional
- **Por tiempo:** Evolución mensual de ventas vs. metas (líneas comparativas)
- **Por producto:** Performance de 7 líneas (Laptop, Monitor, Auriculares, Teclado, Webcam, Mouse)
- **Por región:** Distribución en 5 regiones (Sur, Occidente, Norte, Oriente, Centro)

### KPIs y Métricas
- Total de ventas y meta anual
- Porcentaje de cumplimiento
- Variación interanual (YoY)
- Venta acumulada año a la fecha (YTD)
- Promedio de venta por región/producto

### Interactividad
- Filtros dinámicos por año, mes y dimensiones
- Slicers para navegación ágil entre vistas
- Cross-filtering entre páginas

## 📁 Estructura del Repositorio

```
├── README.md                          # Este archivo
├── dashboard/                         # Capturas y vistas del BI
│   ├── portada.png                   # Página inicial
│   ├── evolucion_temporal.png        # Análisis mensual
│   ├── analisis_productos.png        # Performance por línea
│   ├── analisis_regiones.png         # Distribución geográfica
│   └── README.md                     # Guía de páginas
├── modelo/                           # Estructura del modelo de datos
│   ├── diagrama_modelo.png           # Esquema de relaciones
│   ├── tablas.md                     # Documentación de tablas
│   └── diccionario_datos.md          # Definición de columnas
├── medidas/                          # Código DAX de medidas
│   ├── medidas_ventas.dax            # KPIs de ventas
│   ├── medidas_metas.dax             # Cálculos de cumplimiento
│   ├── medidas_comparativas.dax      # Análisis YoY
│   └── README.md                     # Índice de medidas
├── documentacion/                    # Documentación técnica
│   ├── guia_tecnica.md               # Setup y uso
│   ├── diccionario_negocio.md        # Términos comerciales
│   └── consideraciones.md            # Limitaciones y mejoras
└── data/                             # Datos de ejemplo (opcional)
    └── sample_data.csv               # Muestra para reproducir
```

## 📐 Modelo de Datos

El modelo está compuesto por **5 tablas principales** con relaciones Star Schema:

- **dim_fecha:** Jerarquía temporal (año, mes, nombre mes, trimestre)
- **dim_producto:** Catálogo de líneas de producto
- **dim_region:** Geografía y estructura comercial
- **ventas_comerciales:** Tabla de hechos (transacciones)
- **metas_comerciales:** Objetivos por producto, región y período

[Ver diagrama completo →](./modelo/diagrama_modelo.png)

## 🧮 Medidas DAX Principales

### Ventas
```dax
total_ventas = CALCULATE(SUM(ventas_comerciales[monto_venta]))
```

### Metas
```dax
total_meta = CALCULATE(SUM(metas_comerciales[meta_venta]))
```

### Cumplimiento
```dax
pct_cumplimiento = DIVIDE(total_ventas, total_meta, 0)
```

### Variación Interanual
```dax
ventas_vs_anio_anterior = DIVIDE(ventas_actual - ventas_anterior, ventas_anterior, 0)
```

[Ver código DAX completo →](./medidas/)

## 🔧 Cómo Usar Este Repositorio

### 1. Explorar el Dashboard
Comienza por las capturas en `/dashboard` para entender la estructura visual.

### 2. Entender el Modelo
Revisa `/modelo/diagrama_modelo.png` y `tablas.md` para ver cómo están organizados los datos.

### 3. Estudiar las Medidas
En `/medidas` encontrarás el código DAX explicado por categoría. Usa estas como referencia para tus propios proyectos.

### 4. Replicar Localmente
Si tienes acceso al archivo `.pbix`:
- Descarga los datos desde la carpeta `/data` (si aplica)
- Abre el archivo en Power BI Desktop
- Las medidas están documentadas en cada tabla de medidas

## 🎓 Qué Puedes Aprender

✅ **Diseño de modelos:** Star Schema con tablas de dimensiones y hechos  
✅ **DAX avanzado:** CALCULATE, FILTER, TOTALYTD, variables, lógica condicional  
✅ **Storytelling con datos:** Narrativa visual en 4 páginas interconectadas  
✅ **UX/UI en BI:** Uso de slicers, narrativa y navegación  
✅ **Métricas comerciales:** Cumplimiento, variación YoY, análisis acumulado  

## 📊 Insights Clave del Análisis

- **Laptop** es el producto estrella con 117.5% de cumplimiento ($710M en ventas)
- **Mouse** es el rezagado con solo 21.7% de cumplimiento ($25.7M)
- **Sur y Occidente** lideran en volumen; Centro tiene menor desempeño
- **Variación interanual negativa (-12%)** señala presión comercial
- Picos de venta en **marzo y agosto** sugieren estacionalidad

## 🛠️ Tecnologías

- **Power BI Desktop** (modelo y visualizaciones)
- **DAX** (lenguaje de medidas)
- **Power Query** (transformación de datos)

## 📧 Contacto

Si tienes preguntas sobre este proyecto o quieres colaborar:
- **LinkedIn:** https://www.linkedin.com/in/ivostober/
- **Email:** ivo.stober@icloud.com

---

**Nota:** Este repositorio es una documentación del proyecto. El archivo `.pbix` original no está incluido (tamaño de archivo), pero toda la lógica está documentada en formato dax y en capturas.

**Última actualización:** Junio 2024
