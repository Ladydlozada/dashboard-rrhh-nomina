# Versión con tabla dinámica y segmentaciones nativas de Excel

El Dashboard del libro filtra con una lista desplegable y fórmulas. Esta guía arma, sobre los mismos datos, una segunda hoja con **tablas dinámicas, gráficos dinámicos y segmentaciones de datos** nativas de Excel, como en la imagen de referencia. Toma unos 20 minutos.

Las segmentaciones tienen que crearse desde Excel: no se pueden generar de forma fiable por código, y un archivo armado a mano puede pedir "reparación" al abrirse.

## Qué vas a construir

Una hoja nueva, **Dashboard_Pivot**, con:

- **4 filtros:** segmentaciones de `Departamento`, `Estado` y `Puesto`, más una escala de tiempo para `Fecha Ingreso`.
- **3 cifras de KPI:** empleados, costo de nómina y salario promedio.
- **3 gráficos dinámicos:** salario promedio por departamento, dotación por departamento y distribución salarial.

Todos responden a los mismos filtros a la vez.

## Paso 1: primera tabla dinámica (KPIs)

1. Abre `Dashboard_RRHH_Nomina.xlsx` y ve a la hoja **Datos**. Haz clic en cualquier celda de la tabla `TablaEmpleados`.
2. **Insertar → Tabla dinámica → De tabla o rango → Nueva hoja de cálculo.** Ponle de nombre **Dashboard_Pivot**.
3. En la lista de campos, arrastra a **Valores**:
   - `ID Empleado` (queda como *Recuento*).
   - `Salario Mensual` dos veces: una como *Suma* y la otra cambiada a *Promedio* (clic en el campo → **Configuración de campo de valor → Resumir valores por → Promedio**).
4. Formato de número: `$#,##0` para la suma y el promedio.

## Paso 2: tablas dinámicas para los gráficos

Crea tres tablas más. Todas se crean desde la misma `TablaEmpleados`, así que **comparten los mismos datos de origen**, que es lo que permite que una segmentación las controle a todas.

| Tabla | Filas | Valores |
|---|---|---|
| Salario por departamento | `Departamento` | Promedio de `Salario Mensual` |
| Dotación por departamento | `Departamento` | Recuento de `ID Empleado` |
| Distribución salarial | `Salario Mensual` (agrupado) | Recuento de `ID Empleado` |

Para agrupar el salario: clic derecho sobre un valor de las filas → **Agrupar…** → Comenzar en `2000`, Terminar en `5000`, Por `500`.

Colócalas una al lado de otra en la hoja, con espacio entre ellas para que no se pisen al filtrar.

## Paso 3: gráficos dinámicos

Haz clic en cada tabla y usa **Análisis de tabla dinámica → Gráfico dinámico**:

- Salario por departamento: **columnas agrupadas**.
- Dotación por departamento: **anillos**.
- Distribución salarial: **columnas agrupadas**, con el ancho de separación en 15%.

Usa los mismos colores del dashboard: `#0E7C7B`, `#5B5F97`, `#F2A541`, `#E4572E`, `#76B041`.

El gráfico de dispersión de desempeño vs. salario no se puede hacer como gráfico dinámico (Excel no lo permite). Déjalo en la hoja Dashboard original.

## Paso 4: segmentaciones

1. Haz clic en la primera tabla dinámica: **Análisis de tabla dinámica → Insertar segmentación de datos**. Marca `Departamento`, `Estado` y `Puesto`.
2. Para el año de ingreso: **Análisis de tabla dinámica → Insertar escala de tiempo** y elige `Fecha Ingreso`. Configúrala en **Años**.
3. **Conecta cada filtro a todas las tablas.** Selecciona la segmentación → pestaña **Segmentación → Conexiones de informe** y marca las cuatro tablas. Repite con la escala de tiempo.
4. En la segmentación de `Estado`, selecciona **Activo** y **Licencia** (con Ctrl) para dejar fuera a las bajas, igual que en el dashboard original.

## Paso 5: tarjetas de KPI

En celdas sueltas arriba de la hoja, escribe `=` y haz clic sobre cada total de la primera tabla dinámica: Excel genera una fórmula `GETPIVOTDATA` que sigue a los filtros. Dales el formato de tarjeta del dashboard (fondo blanco, cifra grande, franja de color).

## Comprobación

Con **Estado = Activo + Licencia** y los demás filtros sin tocar, las cifras deben ser:

| KPI | Valor |
|---|---|
| Empleados | 120 |
| Costo de nómina | $404,470 |
| Salario promedio | $3,371 |

Son las mismas que muestra la hoja Dashboard con el selector en "Todos". Con `Departamento = IT` deben dar 37 empleados, $140,720 y $3,803.

## Qué no cubre esta versión

Las tablas dinámicas simples no calculan la **variación contra el mes anterior** ni la **rotación**, porque eso necesita saber cuántos empleados había a fin de cada mes. Esa lógica sí está en el Dashboard con fórmulas y en `powerbi/medidas.dax`. Si quieres esos indicadores también con segmentaciones en Excel, puedes cargar las tablas al **Modelo de datos** y reutilizar las medidas DAX de esa carpeta.

## Al terminar

Guarda el libro como `Dashboard_RRHH_Nomina_Segmentaciones.xlsx`, una captura como `docs/capturas/segmentaciones.png`, y súbelos al repositorio.
