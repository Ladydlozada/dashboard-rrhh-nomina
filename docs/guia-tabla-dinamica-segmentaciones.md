# Versión con tabla dinámica y segmentaciones nativas de Excel

El Dashboard del libro principal filtra con una lista desplegable y fórmulas. Esta versión usa, sobre los mismos datos, **tablas dinámicas, gráficos dinámicos y segmentaciones de datos** nativas de Excel. El resultado está en [`Dashboard_RRHH_Nomina_Segmentaciones.xlsx`](../Dashboard_RRHH_Nomina_Segmentaciones.xlsx); esta guía explica cómo se armó, por si quieres repetirlo.

Las segmentaciones se crean desde Excel: no se pueden generar de forma fiable por código, y un archivo armado a mano puede pedir "reparación" al abrirse.

![Segmentaciones en Excel](capturas/segmentaciones.png)

## Qué se construyó

Dos hojas nuevas sobre la tabla `TablaEmpleados` de la hoja `Datos`:

- **`Tablas_Pivot`**: el "motor". Cuatro tablas dinámicas.
- **`Dashboard_Segmentaciones`**: la presentación. Título, tarjetas de KPI, tres gráficos dinámicos y cuatro filtros (segmentaciones de `Departamento`, `Estado` y `Puesto`, más una escala de tiempo de `Fecha Ingreso` por años).

## Paso 1: tablas dinámicas

1. En `Datos`, haz clic en la tabla y elige **Insertar → Tabla dinámica → De tabla o rango → Nueva hoja de cálculo**. Llama a la hoja `Tablas_Pivot`.
2. **KPIs:** arrastra a **Valores** `ID Empleado` (Recuento), `Salario Mensual` (Suma) y `Salario Mensual` otra vez (Promedio). Renombra los encabezados a `Empleados`, `Costo nómina` y `Salario promedio`. Deja Filas, Columnas y Filtros vacíos.
3. Crea tres tablas más, también desde `TablaEmpleados` y en la misma hoja (**Hoja de cálculo existente**), para que compartan el origen:

| Tabla | Filas | Valores |
|---|---|---|
| Salario por departamento | `Departamento` | Promedio de `Salario Mensual` |
| Dotación por departamento | `Departamento` | Recuento de `ID Empleado` |
| Distribución salarial | `Salario Mensual` (agrupado) | Recuento de `ID Empleado` |

4. Para agrupar el salario: clic derecho sobre un valor de las filas → **Agrupar…** → Comenzar en `2000`, Terminar en `5000`, Por `500`.

## Paso 2: gráficos dinámicos

Con una celda de cada tabla seleccionada, **Analizar tabla dinámica → Gráfico dinámico**:

- Salario por departamento: columnas agrupadas.
- Dotación por departamento: anillos.
- Distribución salarial: columnas agrupadas.

Estilo: se ocultan los botones de campo, se quita la leyenda y el título "Total", y se pone un título propio. Las barras llevan `#0E7C7B`; la dona usa `#0E7C7B`, `#5B5F97`, `#F2A541`, `#E4572E` y `#76B041`, con etiquetas de datos.

El gráfico de dispersión de desempeño vs. salario no puede ser un gráfico dinámico (Excel no lo permite). Está en el Dashboard del libro principal y en la versión de Power BI.

## Paso 3: segmentaciones y escala de tiempo

1. Con una celda de la primera tabla seleccionada: **Analizar tabla dinámica → Insertar segmentación de datos** y marca `Departamento`, `Estado` y `Puesto`.
2. **Insertar escala de tiempo** con `Fecha Ingreso`, y cambia el nivel a **Años**.
3. **Conecta cada filtro a las cuatro tablas:** selecciona la segmentación → pestaña **Segmentación** (o **Escala de tiempo**) → **Conexiones de informe** → marca las cuatro tablas.
4. En `Estado`, selecciona **Activo** y **Licencia** (con Ctrl) para dejar fuera a las bajas, igual que en el Dashboard original.

## Paso 4: hoja de presentación

1. Crea la hoja `Dashboard_Segmentaciones` y apaga las líneas de cuadrícula.
2. Con el **Panel de selección** (Alt + F10), selecciona los filtros y los gráficos de `Tablas_Pivot`, córtalos (Ctrl + X) y pégalos en la hoja nueva. Siguen conectados a las tablas.
3. Las tarjetas son celdas combinadas con una fórmula `IMPORTARDATOSDINAMICOS` (en inglés, `GETPIVOTDATA`). Para crearla: escribe `=` y haz clic sobre cada cifra de la primera tabla. Excel la genera y sigue a los filtros. Formatos: `0` para empleados y `$#,##0` para costo y salario promedio.

## Comprobación

Con **Estado = Activo + Licencia** y los demás filtros sin tocar:

| KPI | Valor |
|---|---|
| Empleados | 120 |
| Costo de nómina | $404,470 |
| Salario promedio | $3,371 |

Con `Departamento = IT`: **37** empleados, **$140,720** y **$3,803**. Con `IT` y año de ingreso **2026**: **9** empleados, **$33,320** y **$3,702**.

![Segmentaciones filtradas por IT](capturas/segmentaciones-it.png)

## Diferencias con las otras versiones

- **Todo se filtra a la vez.** En las segmentaciones nativas, elegir un departamento también reduce los gráficos que comparan departamentos (la dona queda en un solo trozo). En Power BI se puede evitar con "Editar interacciones"; en tablas dinámicas, no.
- **Sin variación contra el mes anterior ni rotación.** Una tabla dinámica simple no sabe cuántos empleados había a fin de cada mes. Esa lógica está en el Dashboard con fórmulas y en `powerbi/medidas.dax`. Para llevarla a las segmentaciones habría que cargar las tablas al **Modelo de datos** y reutilizar esas medidas DAX.
