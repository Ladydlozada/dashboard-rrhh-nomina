# Versión Power BI: mismo modelo de datos

Esta carpeta trae todo para armar en Power BI Desktop el mismo tablero del Excel, con el mismo modelo de datos y los mismos KPIs. Las segmentaciones de Power BI reemplazan al selector de departamento.

> **Estado:** informe terminado (`Dashboard_RRHH_Nomina.pbix`). Armé el informe en Power BI Desktop y comprobé las tarjetas contra los valores de control de más abajo, con todos los departamentos y con IT. Los valores de control salen de un cálculo independiente en Python.

## Qué hay en la carpeta

| Archivo | Para qué sirve |
|---|---|
| `datos/empleados.csv` | 132 registros: 120 vigentes y 12 bajas (igual que la hoja `Datos`) |
| `datos/pagos.csv` | Estado de pago por empleado y mes, en formato largo (igual que la hoja `Pagos`) |
| `datos/meses.csv` | Los seis meses del análisis, con inicio y fin de cada uno |
| `medidas.dax` | Columnas calculadas y medidas, equivalentes a las fórmulas del Excel |
| `tema.json` | Paleta de colores del dashboard |

## Paso 1: cargar los datos

1. **Inicio → Obtener datos → Texto o CSV**, y carga los tres archivos de `datos/`.
2. En cada uno, **Transformar datos** y revisa los tipos: fechas como *Fecha*, `Salario` como *Número entero* y `Rating` como *Número decimal*.
3. Para `Rating`: usa **Cambiar tipo → Usando configuración regional… → Inglés (Estados Unidos)**. Los CSV usan punto decimal, y con la configuración en español un `3.8` puede leerse mal.
4. **Cerrar y aplicar.**

## Paso 2: modelo

- **Una relación:** `Pagos[ID]` → `Empleados[ID]` (muchos a uno, filtro de `Empleados` hacia `Pagos`). Así, filtrar por departamento también filtra los pagos.
- **`Meses` va sin relaciones, a propósito.** Se usa como eje de los gráficos y las medidas leen de ahí el mes que corresponde. Si Power BI crea relaciones automáticas con ella, elimínalas.
- En `Meses`, ordena `Mes` por `Orden`.

## Paso 3: columnas calculadas y medidas

Abre `medidas.dax` y copia cada bloque en **Nueva columna** (las dos primeras, en `Empleados`) o **Nueva medida** (el resto). Después ordena `RangoSalarial` por `OrdenRango`.

Una regla importante del modelo: las medidas toman **el último mes del contexto**. En una tarjeta, sin filtro de mes, eso es octubre de 2026. En un gráfico con `Meses[Mes]` en el eje, es el mes de cada punto.

## Paso 4: tema

**Vista → Temas → Examinar temas** y elige `tema.json`.

## Paso 5: armar la página

| Zona | Visual | Campos |
|---|---|---|
| Arriba | 4 **segmentaciones** | `Departamento`, `Estado`, `Puesto` y `AnioIngreso` (como en la imagen de referencia) |
| Tarjetas (6) | **Tarjeta** | `Headcount`, `Costo Nómina`, `Salario Promedio`, `% Pago a Tiempo`, `Contrataciones`, `Rotación %` |
| Bajo cada tarjeta | Etiqueta de la variación | Medida `Etiqueta ...` correspondiente, con color por **Valor de campo** (`Color ...`) |
| Fila 1, izquierda | **Gráfico de columnas y líneas** | Eje: `Meses[Mes]`. Columnas: `Costo Nómina`. Línea: `Headcount` |
| Fila 1, centro | **Columnas** | Eje: `Departamento`. Valores: `Salario Prom Vigentes` |
| Fila 1, derecha | **Anillo** | Leyenda: `Departamento`. Valores: `Vigentes` |
| Fila 2, izquierda | **Columnas** | Eje: `RangoSalarial`. Valores: `Vigentes` |
| Fila 2, centro | **Dispersión** | X: `Rating`. Y: `Salario`. Detalles: `ID`. Filtro del visual: `Estado` distinto de `Baja`. Agrega **línea de tendencia** en el panel Análisis |
| Fila 2, derecha | **Cuadro de texto** o tarjeta | `Hallazgo Rotación` y `R2 Desempeño vs Salario` |

Para que los gráficos por departamento sigan mostrando los cinco departamentos al filtrar (como en el Excel), usa **Formato → Editar interacciones** y desactiva el filtro desde la segmentación de `Departamento` hacia esos dos visuales (barras y dona).

Medidas con valor vacío: `Bajas` lleva `+ 0` al final para que un departamento sin bajas muestre 0 y no "--".

Consejo: las medidas de `medidas.dax` se pueden cargar de una vez en la vista **Consulta DAX** (bloque `DEFINE MEASURE` y **Actualizar modelo con cambios**), en vez de copiarlas una por una.

## Valores de control

Con **ninguna segmentación aplicada**, tus tarjetas deben mostrar:

| Medida | Octubre (mes actual) | Septiembre (mes anterior) |
|---|---|---|
| Headcount | 120 | 118 |
| Costo Nómina | $404,470 | $393,260 |
| Salario Promedio | $3,371 | $3,333 |
| % Pago a Tiempo | 99.2% | 98.3% |
| Contrataciones | 6 | 4 |
| Rotación % | 3.3% (4 bajas) | 1.7% (2 bajas) |

`R2 Desempeño vs Salario` = **0.71**.

Con la segmentación **Departamento = IT**:

| Medida | Octubre | Septiembre |
|---|---|---|
| Headcount | 37 | 34 |
| Costo Nómina | $140,720 | $130,290 |
| Salario Promedio | $3,803 | $3,832 |
| % Pago a Tiempo | 100% | 97.2% |
| Contrataciones | 3 | 2 |
| Rotación % | 0.0% | 5.9% (2 bajas) |

Estas cifras son las mismas que da el Excel con el selector en "Todos" e "IT". Las comprobé con un cálculo independiente en Python que replica la lógica de las medidas.

## Si algo no coincide

- **`Rating` o `Salario` raros:** revisa la configuración regional del paso 1.
- **Tarjetas que suman todos los meses** (por ejemplo, 28 contrataciones): `Meses` quedó con una relación hacia otra tabla. Elimínala.
- **Los pagos no responden a la segmentación de departamento:** revisa que la relación `Pagos[ID]` → `Empleados[ID]` filtre de `Empleados` hacia `Pagos`.
- **Un error de DAX:** revisa que los nombres de tablas y columnas coincidan exactamente (`Empleados`, `Pagos`, `Meses`, `FechaBaja`, `MesInicio`, `InicioMes`, `FinMes`).

Cuando termines, guarda las capturas en `docs/capturas/` (por ejemplo `powerbi-dashboard.png`) y el archivo `.pbix` en esta carpeta.
