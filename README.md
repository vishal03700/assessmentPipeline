# PostgreSQL to Apache Iceberg Sync via OLake + Spark SQL

**Author:** Vishal Kumar Yadav  
**Date:** 23-04-2025

---

## 📌 Objective

Build a pipeline to:
- Extract data from PostgreSQL.
- Sync into Apache Iceberg format using OLake + REST Catalog.
- Query data via Spark SQL.

---

🎥 Watch the walkthrough video here:
👉 Loom Video Walkthrough : https://www.loom.com/share/dcf744c8039a407f921330af2f7f6908

## 🧱 System Components

| Component         | Description                                 |
|------------------|---------------------------------------------|
| **PostgreSQL**    | Local DB (via Docker/native)                |
| **OLake**         | Data integration CLI with JDBC              |
| **Apache Iceberg**| Data lake table format                      |
| **Rest Catalog**  | Metadata catalog for Iceberg tables         |
| **Spark SQL**     | SQL CLI engine for querying Iceberg tables  |

---

## ⚙️ Pipeline Steps

### 1️⃣ Start OLake Stack

```bash
git clone https://github.com/datazip-inc/olake.git
cd olake

/home/xender/datazip/olake/drivers/postgres/config/docker-compose.yml
Run : docker compose -f docker-compose.yml up -d

config/ writers/iceberg/local-test/
Run : docker compose -f docker-compose.yml up -d
```

---

### 2️⃣ PostgreSQL Setup

**Table: `sample_data`**

```sql
CREATE TABLE sample_data (
    id SERIAL PRIMARY KEY,
    str_col VARCHAR(100),
    num_col INT
);

INSERT INTO sample_data (str_col, num_col)
VALUES
('Hello world', 123),
('Hello world', 123),
('Sample text', 456),
('Sample Vishal', 786),
('Sample Radha', 124),
('Sample Vikash', 125),
('Hello datazip', 123),
('Hello Ram', 123),
('Hello Ram', 123),
('Hello Ram', 123),
('Hello Ram', 123),
('Hello Ram', 123);
```

---

### 3️⃣ Discover Schema (OLake)

```bash
Initial schema discovery using Olake config before syncing data.

OLAKE_BASE_PATH="$HOME/datazip/olake/drivers/postgres/config" && ./build.sh driver-postgres discover   --config "$OLAKE_BASE_PATH/config.json"

📝 **Expected Schema Output:**
```json
{
  "fields": [
    { "name": "id", "type": "int" },
    { "name": "str_col", "type": "string" },
    { "name": "num_col", "type": "int" }
  ]
}
```
![Screenshot from 2025-04-23 11-27-16](https://github.com/user-attachments/assets/3859dade-a0bd-47de-a449-75afcf8a5773)

---

### 4️⃣ Sync Data to Iceberg

```bash
./build.sh driver-postgres sync --config /home/xender/datazip/olake/drivers/postgres/config/config.json --catalog /home/xender/datazip/olake/drivers/postgres/config/catalog.json --destination /home/xender/datazip/olake/drivers/postgres/config/writer-jdbc.json
```

#### Optional: Sync with State Tracking

```bash
./build.sh driver-postgres sync \
  --config "$OLAKE_BASE_PATH/config.json" \
  --catalog "$OLAKE_BASE_PATH/catalog.json" \
  --destination "$OLAKE_BASE_PATH/writer-jdbc.json" \
  --state "$OLAKE_BASE_PATH/state.json"
```
![Screenshot from 2025-04-23 11-27-16](https://github.com/user-attachments/assets/eed2fc64-f2b0-420d-8bb5-13b08ef74917)

---

### 5️⃣ Query Iceberg Table with Spark SQL

```bash
docker exec -it spark-iceberg bash
rm -rf /opt/spark/metastore_db
spark-sql
```

```sql
SHOW TABLES FROM olake_iceberg.ICEBERG_DATABASE_NAME;

SELECT * FROM olake_iceberg.ICEBERG_DATABASE_NAME.sample_data;
```
![Screenshot from 2025-04-23 11-26-41](https://github.com/user-attachments/assets/21822624-02ad-436a-b8b5-c835465a2922)

---

## ✅ Results


![Screenshot from 2025-04-23 11-26-56](https://github.com/user-attachments/assets/72719629-cd0f-46f7-b7fa-ce928defa909)

---

## 🔁 Next Steps

- Add support for **incremental syncs** (CDC-based).
- Visualize results via **Spark UI** or BI tools.
- Integrate in a **CI/CD workflow** using GitHub Actions or Jenkins.
- Use **MinIO** or **S3** as Iceberg storage backend in production.

---

## 📁 Key Files & Paths

- **OLake Config**: `drivers/postgres/config/config.json`
- **Catalog**: `drivers/postgres/config/catalog.json`
- **Writer Config**: `drivers/postgres/config/writer-jdbc.json`
- **State File**: `drivers/postgres/config/state.json`

