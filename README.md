# KOAJ Sales Data Pipeline

## Project Overview

End-to-end ETL pipeline for KOAJ (Colombian fashion retailer) sales data. The project extracts raw transactional data, transforms it using Python/pandas, loads it into PostgreSQL following a star schema, and delivers an interactive Power BI dashboard answering 5 key business questions.

**Dataset:** 6,000 sales transactions from KOAJ stores across Colombian cities (2023-2025).

---

## Tech Stack

| Layer | Technology |
|---|---|
| Extraction | Python 3.10+, pandas |
| Transformation | pandas, numpy |
| Storage | PostgreSQL 14+ |
| Business Intelligence | Power BI Desktop |
| Environment | python-dotenv, virtualenv |

---

## Project Structure

```text
KOAJ/
├── .env                          # Database credentials (not tracked)
├── .gitignore                    # Git ignore rules
├── requirements.txt              # Python dependencies
├── data/
│   ├── raw/                      # Original raw CSV
│   └── processed/                # Cleaned & transformed CSVs
├── notebooks/
│   ├── 01_Exploracion.ipynb      # Data exploration & profiling
│   ├── 02_Limpieza.ipynb         # Data cleaning & validation
│   ├── 03_Normalizacion.ipynb    # Star schema modeling
│   └── 04_CargarSQL.ipynb        # PostgreSQL load & verification
├── docs/
│   ├── GUIA_DASHBOARD.md         # Step-by-step Power BI guide
│   └── Simulacro_Prueba_Desempeno_RIWI.md
└── powerbi/
    └── dashboard_koaj.pbix       # Interactive dashboard
```

---

## Setup & Installation

### Prerequisites

- Python 3.10+
- PostgreSQL 14+ (running on localhost:5432)
- Power BI Desktop

### Steps

```bash
# 1. Clone the repository
git clone <repository-url>
cd KOAJ

# 2. Create virtual environment
python -m venv env

# Windows
env\Scripts\activate

# Linux/Mac
source env/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure database credentials
# Edit .env with your PostgreSQL credentials
```

### Database Configuration

Edit `.env`:

```text
MI_CONTRA=your_password_here
DB_USER=postgres
DB_HOST=localhost
DB_PORT=5432
DB_NAME=koaj_ventas
```

---

## Pipeline Execution

Run the notebooks in this order:

| Step | Notebook | Purpose |
|---|---|---|
| 1 | `01_Exploracion.ipynb` | Profile data and identify quality issues |
| 2 | `02_Limpieza.ipynb` | Clean nulls, fix types, remove duplicates and validate |
| 3 | `03_Normalizacion.ipynb` | Build the star schema |
| 4 | `04_CargarSQL.ipynb` | Load tables into PostgreSQL and verify the load |

---

## Data Model

The project uses a simple star schema:

```text
                    ┌───────────────┐
                    │ dim_producto  │
                    │ PK id_producto│
                    └───────┬───────┘
                            │
┌───────────────┐     ┌─────▼─────┐     ┌───────────────┐
│ dim_cliente   ├────►│fact_ventas│◄────┤  dim_tienda   │
│ PK id_cliente │     │            │     │ PK id_tienda  │
└───────────────┘     └────────────┘     └───────────────┘
```

The fact table contains the transactional information, while the dimensions provide descriptive attributes for analysis.

| Table | Description |
|---|---|
| `fact_ventas` | Sales lines, transaction keys and numeric measures; one row = one product line within a purchase |
| `dim_producto` | Product, category and garment information |
| `dim_cliente` | Customer profile information |
| `dim_tienda` | Store, city and department information |

---


---

## Granularidad de los datos

El dataset RAW y la tabla `fact_ventas` trabajan a nivel de **línea de venta**.

> **Una fila representa un producto dentro de una compra/transacción.**

La lógica de los identificadores es:

```text
id_venta        → identifica la compra completa
numero_linea    → identifica la línea del producto dentro de esa compra
id_producto     → identifica el producto vendido
cantidad        → unidades vendidas en esa línea
precio_unitario → precio por unidad
descuento       → descuento aplicado a esa línea
```

Una compra puede tener varias filas. Para todas las líneas de una misma `id_venta`, `fecha_venta`, `id_cliente` e `id_tienda` se mantienen consistentes.

Por lo tanto:
- Filas de `fact_ventas` = líneas de venta.
- `id_venta` = compra/transacción.
- `cantidad` = unidades vendidas.
- `id_venta + numero_linea` = clave de una línea de venta después de limpiar duplicados.

Para contar compras no se deben contar filas. Se utiliza:

```DAX
Numero de Ventas =
DISTINCTCOUNT(fact_ventas[id_venta])
```

## Main Metrics

The dashboard uses measures based on the sales fact table.

| Metric | Purpose |
|---|---|
| **Ventas Netas** | Total revenue after applying discounts |
| **Unidades Vendidas** | Total units sold |
| **Número de Ventas** | Distinct sales/orders |
| **Ticket Promedio** | Average value per sale |
| **Ventas Año Anterior** | Previous-year comparison |
| **Variación Ventas %** | Percentage change versus the previous year |

---

# Business Questions

The Power BI dashboard is designed to answer the five business questions required by the performance test.

## Q1. Tendencia temporal

**¿Cómo han evolucionado las ventas netas de KOAJ entre 2023 y 2025?**

The objective is to identify the monthly or quarterly evolution of the main business metric and detect periods of growth, decline or seasonality.

**Recommended visual:** line chart.

---

## Q2. Top 5 categorías

**¿Cuáles son las cinco categorías de productos que generan mayores ventas netas?**

The objective is to identify the categories that contribute the most to total revenue and evaluate whether sales are concentrated in a small number of categories.

**Recommended visual:** horizontal Top 5 bar chart.

---

## Q3. Distribución y concentración geográfica

**¿Cómo se distribuyen las ventas netas entre las ciudades y qué nivel de concentración existe?**

The objective is to identify the cities with the highest sales contribution and determine whether the business is highly concentrated in a few geographic markets.

**Recommended visual:** horizontal bar chart by city, optionally supported by a Pareto-style analysis.

---

## Q4. Comparación entre periodos

**¿Cuál fue la variación porcentual de las ventas netas frente al año anterior?**

The objective is to compare equivalent periods and determine whether KOAJ is growing or decreasing over time.

**Recommended visual:** KPI/card with percentage variation, supported by a year or monthly comparison.

---

## Q5. Relación entre variables

**¿Existe una relación observable entre el precio unitario y la cantidad vendida?**

The objective is to explore whether higher-priced products tend to sell fewer units and whether the relationship provides a useful commercial insight.

**Recommended visual:** scatter plot.

---

## Dashboard Requirements

The dashboard should include at least:

- 1 summary KPI/card
- 1 temporal trend chart
- 1 Top-N/ranking chart
- 1 distribution chart
- 1 interactive slicer/filter

These elements directly correspond to the requirements of the performance test.

---

## Expected Insights

The analysis should not invent conclusions before looking at the final data.

For each business question, use this structure:

```text
DATA
  ↓
INTERPRETATION
  ↓
BUSINESS DECISION
```

Example:

> A city represents a high percentage of total sales.

This is the **data**.

> The business has a high geographic concentration in that market.

This is the **interpretation**.

> Inventory and commercial campaigns could prioritize that market while testing growth strategies in secondary cities.

This is the **possible decision**.

The final conclusions must be based on the actual results obtained in Power BI.

---

## Academic Deliverables

The project is organized around the following deliverables:

- Dataset source
- Dataset description
- Pipeline diagram
- Python/pandas ETL notebooks
- PostgreSQL tables and load evidence
- Power BI dashboard
- Business questions and answers
- Final conclusions and insights

---

## Academic Context

Academic project for practice of the RIWI Data Engineering performance test.

The objective is to demonstrate the complete flow:

```text
RAW DATA
   ↓
EXTRACT
   ↓
TRANSFORM
   ↓
VALIDATE
   ↓
NORMALIZE
   ↓
POSTGRESQL
   ↓
POWER BI
   ↓
BUSINESS INSIGHTS
```
