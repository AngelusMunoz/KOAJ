# KOAJ — Pipeline ETL y Análisis Comercial

Proyecto académico de análisis de datos orientado a la construcción de un flujo completo de **extracción, transformación, validación, modelado y carga de datos**, seguido de su explotación en **PostgreSQL y Power BI**.

El proyecto parte de un archivo CSV con información transaccional de ventas de KOAJ y desarrolla un proceso reproducible mediante Python y Jupyter Notebooks. Los datos son limpiados y estructurados en un modelo dimensional, almacenados en PostgreSQL y finalmente utilizados en un dashboard interactivo de Power BI.

## Objetivo

Construir una solución de datos que permita transformar información de ventas en una estructura preparada para análisis comercial y responder preguntas de negocio relacionadas con:

- evolución de las ventas;
- categorías con mayor participación;
- distribución geográfica de las ventas;
- comparación interanual y mensual;
- relación entre precio y cantidad vendida.

## Tecnologías

| Tecnología | Uso |
|---|---|
| Python 3.10+ | Lenguaje principal del procesamiento |
| Jupyter Notebook | Ejecución documentada del proceso ETL |
| Pandas | Manipulación, limpieza y transformación de datos |
| NumPy | Operaciones y transformaciones numéricas |
| python-dotenv | Lectura de variables de entorno |
| SQLAlchemy | Conexión y operaciones con PostgreSQL |
| PostgreSQL 14+ | Base de datos relacional |
| Power BI Desktop | Modelado analítico y visualización |
| DAX | Cálculo de indicadores y análisis temporal |

## Estructura del proyecto

```text
KOAJ/
│
├── .env.example                  # Plantilla de configuración de PostgreSQL
├── .gitignore                    # Archivos excluidos de control de versiones
├── requirements.txt              # Dependencias de Python
├── README.md                     # Documentación del proyecto
│
├── data/
│   ├── raw/
│   │   └── ventas_raw.csv        # Dataset original
│   │
│   └── processed/
│       ├── ventas_limpio.csv     # Dataset después de la limpieza
│       ├── fact_ventas.csv       # Tabla de hechos
│       ├── dim_cliente.csv       # Dimensión de clientes
│       ├── dim_producto.csv      # Dimensión de productos
│       └── dim_tienda.csv        # Dimensión de tiendas
│
├── notebooks/
│   ├── 01_Exploracion.ipynb      # Exploración y perfilado inicial
│   ├── 02_Limpieza.ipynb         # Limpieza y validación
│   ├── 03_Normalizacion.ipynb    # Normalización y modelo dimensional
│   └── 04_CargarSQL.ipynb        # Creación y carga de PostgreSQL
│
└── doc/
    ├── Documento_Diseño_Software_KOAJ.docx
    ├── Esquema Pipeline.pdf
    └── KOAJ.pbix                  # Dashboard de Power BI
```

> Los archivos de credenciales reales (`.env`) no deben publicarse en repositorios. Se debe utilizar `.env.example` como plantilla.

## Requisitos previos

Antes de ejecutar el proyecto se necesita tener instalado:

1. **Python 3.10 o superior**
2. **PostgreSQL 14 o superior**
3. **Power BI Desktop** para abrir el dashboard
4. **Git** si el proyecto se obtiene desde un repositorio

La versión de PostgreSQL indicada corresponde al entorno utilizado para el proyecto. Versiones posteriores pueden funcionar, pero no forman parte del entorno originalmente documentado.

## Instalación

### 1. Clonar el repositorio

```bash
git clone <URL_DEL_REPOSITORIO>
cd KOAJ
```

Si el proyecto ya se encuentra descargado localmente, este paso no es necesario.

### 2. Crear el entorno virtual

#### Windows

```powershell
python -m venv env
env\Scripts\activate
```

#### Linux / macOS

```bash
python3 -m venv env
source env/bin/activate
```

Cuando el entorno esté activo, el terminal normalmente mostrará `(env)` al inicio de la línea.

### 3. Actualizar pip

```bash
python -m pip install --upgrade pip
```

### 4. Instalar las dependencias

```bash
pip install -r requirements.txt
```

El archivo `requirements.txt` contiene las versiones de las dependencias del entorno utilizado para ejecutar los notebooks.

### 5. Verificar las instalaciones principales

```bash
python --version
pip --version
```

También puede comprobarse que las librerías principales estén disponibles:

```bash
python -c "import pandas, numpy, dotenv, sqlalchemy; print('Dependencias principales instaladas correctamente')"
```

## Configuración de PostgreSQL

El notebook `04_CargarSQL.ipynb` utiliza variables de entorno para establecer la conexión con PostgreSQL.

### 1. Crear la base de datos

Desde **pgAdmin** o desde `psql`, crear la base de datos que se utilizará para el proyecto.

Ejemplo:

```sql
CREATE DATABASE koaj_ventas;
```

### 2. Crear el archivo `.env`

Copiar `.env.example` y renombrarlo como `.env`.

#### Windows PowerShell

```powershell
Copy-Item .env.example .env
```

#### Linux / macOS

```bash
cp .env.example .env
```

Después, editar `.env` con los valores reales de la instalación local:

```env
MI_CONTRA=tu_contraseña
DB_USER=postgres
DB_HOST=localhost
DB_PORT=5432
DB_NAME=koaj_ventas
```

El archivo `.env` es utilizado por `04_CargarSQL.ipynb` mediante `python-dotenv`.

## Ejecución del pipeline

Los notebooks deben ejecutarse **en el orden indicado**, ya que cada etapa utiliza los resultados generados por la anterior.

### 01 — Exploración

`notebooks/01_Exploracion.ipynb`

Objetivo:

- conocer la estructura del dataset;
- revisar tipos de datos;
- identificar valores nulos;
- detectar duplicados;
- revisar valores únicos y rangos;
- identificar posibles problemas de calidad.

### 02 — Limpieza

`notebooks/02_Limpieza.ipynb`

Objetivo:

- corregir tipos de datos;
- tratar valores faltantes;
- detectar y eliminar duplicados;
- validar rangos y valores;
- generar el dataset limpio.

Salida principal:

```text
data/processed/ventas_limpio.csv
```

### 03 — Normalización

`notebooks/03_Normalizacion.ipynb`

Objetivo:

- separar atributos descriptivos y transaccionales;
- construir las dimensiones;
- construir la tabla de hechos;
- preparar el modelo dimensional para PostgreSQL y Power BI.

Salidas principales:

```text
data/processed/dim_cliente.csv
data/processed/dim_producto.csv
data/processed/dim_tienda.csv
data/processed/fact_ventas.csv
```

### 04 — Carga a PostgreSQL

`notebooks/04_CargarSQL.ipynb`

Objetivo:

- conectarse a PostgreSQL;
- crear las tablas;
- definir claves primarias y foráneas;
- cargar las dimensiones;
- cargar la tabla de hechos;
- comprobar que la carga se realizó correctamente.

El notebook crea las siguientes tablas:

```text
dim_producto
dim_cliente
dim_tienda
fact_ventas
```

Las dimensiones se cargan antes que la tabla de hechos para respetar las claves foráneas.

## Modelo de datos

La base de datos utiliza un esquema dimensional tipo **estrella**.

```text
                         dim_producto
                              │
                              │
                              ▼
                        fact_ventas
                         ▲      ▲
                         │      │
                         │      │
                dim_cliente   dim_tienda
```

### `fact_ventas`

Tabla central del modelo. Contiene la información transaccional de las líneas de venta.

Una fila representa **un producto dentro de una compra/transacción**.

Clave primaria compuesta:

```text
(id_venta, numero_linea)
```

Principales campos:

- `id_venta`
- `numero_linea`
- `fecha_venta`
- `id_cliente`
- `id_producto`
- `id_tienda`
- `cantidad`
- `precio_unitario`
- `descuento`
- `metodo_pago`
- `canal`
- `temporada`
- `talla`

### `dim_producto`

Contiene información descriptiva de los productos.

Clave primaria:

```text
id_producto
```

### `dim_cliente`

Contiene información descriptiva asociada al cliente.

Clave primaria:

```text
id_cliente
```

### `dim_tienda`

Contiene información relacionada con las tiendas y su ubicación.

Clave primaria:

```text
id_tienda
```

### Relaciones

```text
dim_producto.id_producto  ───► fact_ventas.id_producto
dim_cliente.id_cliente    ───► fact_ventas.id_cliente
dim_tienda.id_tienda      ───► fact_ventas.id_tienda
```

La dimensión de fechas utilizada para el análisis temporal se encuentra implementada en **Power BI** (`DimFecha`).

## Power BI

El archivo del dashboard se encuentra en:

```text
doc/KOAJ.pbix
```

El dashboard permite analizar el comportamiento comercial mediante filtros y medidas DAX.

### Principales indicadores

- **Ventas Netas**
- **Número de Ventas**
- **Unidades Vendidas**
- **Ticket Promedio**
- **Ventas Año Anterior**
- **Variación Interanual %**
- **Variación Mensual %**

### Medidas principales

```DAX
Numero de Ventas =
DISTINCTCOUNT(fact_ventas[id_venta])
```

```DAX
Unidades Vendidas =
SUM(fact_ventas[cantidad])
```

```DAX
Ventas Netas =
SUMX(
    fact_ventas,
    fact_ventas[cantidad]
        * fact_ventas[precio_unitario]
        * (1 - fact_ventas[descuento])
)
```

```DAX
Ticket Promedio =
DIVIDE([Ventas Netas], [Numero de Ventas], 0)
```

```DAX
Ventas Año Anterior =
CALCULATE(
    [Ventas Netas],
    SAMEPERIODLASTYEAR(DimFecha[Date])
)
```

```DAX
Ventas Mes Anterior =
CALCULATE(
    [Ventas Netas],
    DATEADD(DimFecha[Date], -1, MONTH)
)
```

```DAX
Variacion Interanual % =
DIVIDE(
    [Ventas Netas] - [Ventas Año Anterior],
    [Ventas Año Anterior],
    0
)
```

```DAX
Variacion Mensual % =
DIVIDE(
    [Ventas Netas] - [Ventas Mes Anterior],
    [Ventas Mes Anterior],
    0
)
```

## Preguntas de negocio

El dashboard fue construido para responder las siguientes preguntas:

### 1. Evolución de ventas

**¿Cómo han evolucionado las ventas netas entre 2023 y 2025?**

Permite identificar tendencias y cambios a lo largo del tiempo.

### 2. Categorías principales

**¿Cuáles son las categorías que generan mayores ventas netas?**

Permite identificar las categorías con mayor contribución al ingreso.

### 3. Distribución geográfica

**¿Cómo se distribuyen las ventas entre las ciudades?**

Permite identificar mercados con mayor participación y concentración geográfica.

### 4. Variación interanual

**¿Cuál es la variación porcentual de las ventas frente al año anterior?**

Permite evaluar crecimiento o disminución de las ventas en períodos comparables.

### 5. Relación precio-cantidad

**¿Existe una relación observable entre el precio unitario y la cantidad vendida?**

Permite explorar el comportamiento de la demanda frente a diferentes niveles de precio.

## Flujo completo de la solución

```text
                 DATOS RAW
                    │
                    ▼
          Exploración y perfilado
                    │
                    ▼
             Limpieza y validación
                    │
                    ▼
          Normalización / Modelo estrella
                    │
                    ▼
               PostgreSQL
                    │
                    ▼
                 Power BI
                    │
                    ▼
          Indicadores + visualizaciones
                    │
                    ▼
             Análisis de negocio
```

## Recomendaciones para ejecutar correctamente el proyecto

- Ejecutar los cuatro notebooks en orden.
- Mantener activa la carpeta del proyecto como directorio de trabajo.
- Verificar que PostgreSQL esté iniciado antes de ejecutar `04_CargarSQL.ipynb`.
- Confirmar que `.env` contenga las credenciales correctas.
- No modificar las rutas de `data/raw` y `data/processed` sin actualizar los notebooks.
- No subir `.env` a GitHub o GitLab.
- Abrir `KOAJ.pbix` después de completar la preparación de los datos.

## Solución de problemas frecuentes

### Error de conexión a PostgreSQL

Verificar:

```text
DB_USER
MI_CONTRA
DB_HOST
DB_PORT
DB_NAME
```

y confirmar que el servicio de PostgreSQL esté activo.

### Error `ModuleNotFoundError`

Activar el entorno virtual y reinstalar las dependencias:

```bash
# Windows
.\env\Scripts\activate

# Linux / macOS
source env/bin/activate

pip install -r requirements.txt
```

### El notebook no encuentra el CSV

Comprobar que exista:

```text
data/raw/ventas_raw.csv
```

y que el notebook se esté ejecutando desde el directorio correcto del proyecto.

### Power BI no puede actualizar los datos

Verificar que:

- PostgreSQL esté disponible;
- la base de datos `koaj_ventas` exista;
- las tablas hayan sido cargadas por `04_CargarSQL.ipynb`;
- la conexión configurada en Power BI corresponda al entorno local.

## Archivos de documentación

La carpeta `doc/` contiene los documentos asociados al proyecto:

| Archivo | Descripción |
|---|---|
| `Documento_Diseño_Software_KOAJ.docx` | Documento de diseño del sistema |
| `Esquema Pipeline.pdf` | Representación del flujo del proceso |
| `KOAJ.pbix` | Dashboard interactivo de Power BI |

## Resultado esperado

Al finalizar la ejecución, el proyecto debe contar con:

1. Un dataset limpio.
2. Tablas dimensionales preparadas.
3. Una tabla de hechos con las transacciones.
4. La base de datos cargada en PostgreSQL.
5. El modelo analítico disponible en Power BI.
6. Un dashboard funcional para el análisis comercial.

## Contexto académico

Proyecto académico de análisis y procesamiento de datos, desarrollado para practicar un flujo completo de ingeniería y analítica de datos: **preparación, transformación, modelado, almacenamiento y visualización de información para la toma de decisiones**.
