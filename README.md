# ETL Pair Programming — Retail (Italian Products)

## 1) Project overview
The project applies **ETL pipelines** for data exploration, cleaning, transformation, and integration of sales, product, and customer datasets. The objective is to consolidate these sources into a single analytics-ready table for business analysis.  
These pipelines are implemented in **Python scripts (.py)** and can be reused or extended to automate other data analysis processes.

## 2) Repository structure
```
RETAIL-ETL-PIPELINE-PROJECT
├─ data/
│  ├─ clientes.csv
│  ├─ productos.csv
│  ├─ ventas.csv
│  └─ (output) sales_merged.csv
├─ src/
│  ├─ support.py
│  ├─ queries.py
│─ data-cleaning.py
│─ main.py
└─ README.md
```

## 3) Tasks & acceptance criteria
### 3.1 Read & explore
- Read all CSVs.
- Print shape, columns, dtypes, nulls, duplicates, and key descriptive statistics for each dataset.

### 3.2 Cleaning
Minimum rules (extend as needed):
- **Nulls:** fill non-critical text fields with `"unknown"` (e.g., `email`, `gender`, `Address`); consider dropping rows only if key identifiers are missing.
- **Duplicates:** remove exact duplicates on full row; for customers/products, dedupe on business keys (`id`, `ID_Producto`).
- **Types:** cast dates to `datetime`, IDs to `string` (or categorical), numeric columns to numeric.
- **Standardization:** trim spaces, normalize case where appropriate, and align category labels.
- **Integrity checks:** no orphan sales—every `ventas.ID_Producto` must exist in `productos`, every `ventas.id` must exist in `clientes`.

### 3.3 Integration (joins)
- Join **ventas ↔ productos** on `ID_Producto` (left join from sales).
- Join the result ↔ **clientes** on `id` (left join from sales).
- Persist as **`data/sales_enriched.csv`** (or `.parquet` if preferred).

## 4) How to run (CLI + config)
You can run with sensible defaults:
```bash
python src/main.py
```

## 5) Example Code Structure
```python
def exploracion(df1, df2, df3):
    for df, name in zip([df1, df2, df3], ["productos", "clientes", "ventas"]):
        print(f"INFO — {name.upper()}")
        print("Shape:", df.shape, "\nColumns:", df.columns, "\nDtypes:\n", df.dtypes)
        print("\nNulls:\n", df.isnull().sum())
        print("\nDuplicates:", df.duplicated().sum())
        print("\nDescribe:\n", df.describe(include="all").T, "\n")

def limpieza(df_customers, df_products, df_sales):
    print("CLEAN — clientes:")
    df_customers[["email", "gender", "Address"]] = (
        df_customers[["email", "gender", "Address"]].fillna("unknown")
    )
    # Add more cleaning rules as needed
    return df_customers, df_products, df_sales

def union(df_products, df_customers, df_sales):
    df_vp = pd.merge(df_sales, df_products, on="ID_Producto", how="left")
    merged = pd.merge(df_vp, df_customers, on="id", how="left")
    return merged
```

## 6) Data quality checks
Run explicit validations and log PASS/FAIL:
- **Referential integrity:** no orphan `ID_Producto` or `id` after joins.
- **Schema:** expected columns present; dtypes match config.
- **Value rules:** `Cantidad >= 0`, `PrecioUnitario >= 0`.
- **Null thresholds:** fields remain below configured `%` of nulls.
- **Duplicates:** unique constraints on product and customer keys.
- **Date validity:** `Fecha` within business range; parsable `datetime`.

## 7) Deliverables
- Clean, commented Python scripts in `src/` (`funciones.py`, `queries.py`, `main.py`).
- Output file: `data/sales_enriched.csv` (or `.parquet`).
- Logs demonstrating exploration, cleaning actions, and validation results.
- Configurable CLI execution (`config.yaml`, `Makefile`).
- This README.md.

## 8) Next steps 
- Persist to a database (e.g., MySQL/PostgreSQL).
- Build simple KPIs (GMV, AOV, units/order) and cohort/RFM baselines.
- Add unit tests for transforms and joins.

## Example Code Structure
```python
def insertar_datos(query, contraseña, nombre_bdd, lista_tuplas):
    
    cnx = mysql.connector.connect(
        user="root", 
        password=contraseña, 
        host="127.0.0.1", database=nombre_bdd)
    
    mycursor = cnx.cursor()
    
    try:
        mycursor.executemany(query, lista_tuplas)
        cnx.commit()
        print(mycursor.rowcount, "registro/s insertado/s.")
        cnx.close()
        
    except mysql.connector.Error as err:
        print(err)
        print("Error Code:", err.errno)
        print("SQLSTATE", err.sqlstate)
        print("Message", err.msg)
        cnx.close()
```

---
