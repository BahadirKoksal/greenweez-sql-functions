# ⚙️ Greenweez SQL Functions

BigQuery UDF (User Defined Function) practice project — creating reusable SQL functions to transform raw Greenweez data into analytical dimensions.

---

## 🎯 Objective

Replace repetitive `IF` and `CASE WHEN` blocks with reusable functions. The goal is cleaner queries, easier maintenance, and turning raw values into meaningful analytical categories.

---

## 🗃️ Dataset

Two tables from BigQuery (`course17` dataset):

| Table | Rows | Description |
|-------|------|-------------|
| `gwz_mail_17` | 146 | Email campaign data — journey names, send volumes, KPIs |
| `gwz_nps_17` | 6,294 | Order-level delivery satisfaction scores |

Key columns in `gwz_mail_17`:

| Column | Description |
|--------|-------------|
| `journey_id` | Campaign ID |
| `journey_name` | Campaign name (encodes market and type) |
| `sent_nb` | Number of emails sent |
| `opening_nb` | Number of opens |
| `click_nb` | Number of clicks |
| `turnover` | Revenue generated (€) |

Key columns in `gwz_nps_17`:

| Column | Description |
|--------|-------------|
| `date_date` | Order date |
| `orders_id` | Order identifier |
| `transporter` | Full transporter name (e.g. "Chrono Pickup", "DPD Home") |
| `global_note` | Customer satisfaction score (0–10) |

---

## 📋 Project Steps

### 1. IF-based Segmentation
Used `IF(journey_name LIKE "%_nlbe_%", 1, 0)` to label Belgium campaigns as binary flags directly in a query.

### 2. First UDF — `is_mail_be(journey_name)`
Saved the same `IF()` logic as a reusable function. Instead of repeating the condition in every query, now just call `course17.is_mail_be(journey_name)`.

### 3. Second UDF — `mail_type(journey_name)`
Used `CASE WHEN + LIKE` to classify campaigns into types:

| Output | Condition |
|--------|-----------|
| `abandoned_basket` | name contains `panier_abandonne` |
| `back_in_stock` | name contains `back_in_stock` |
| `newsletter` | name contains `nl` |
| `NULL` | none of the above |

### 4. NPS Transformation
Used `CASE WHEN` to convert raw `global_note` scores (0–10) into standard NPS categories — turning a number into an actionable signal.

### 5. Third UDF — `nps(global_note)`
Saved the NPS logic as a function. Input: INT64. Output: -1, 0, or 1.

| Output | Condition | Label |
|--------|-----------|-------|
| `1` | score IN (9, 10) | Promoter |
| `0` | score IN (7, 8) | Passive |
| `-1` | score BETWEEN 0 AND 6 | Detractor |

### 6. Fourth UDF — `transporter_brand(transporter)`
Extracts the carrier brand from the full transporter name:

| Output | Condition |
|--------|-----------|
| `Chrono` | name contains `Chrono` |
| `Dpd` | name contains `DPD` |
| `NULL` | none of the above |

### 7. Fifth UDF — `delivery_mode(transporter)`
Extracts the delivery mode from the same transporter column:

| Output | Condition |
|--------|-----------|
| `Pickup` | name contains `Pickup` |
| `Home` | name contains `Home` |
| `NULL` | none of the above |

### 8. Final Query — All Functions Combined
Applied all three NPS/transporter functions in a single query to enrich `gwz_nps_17` with derived analytical columns.

---

## 📊 Key Findings

- High-volume campaigns are not from the Belgium segment — main NL campaigns reach much larger audiences
- `Chrono Pickup` is the most common transporter combination in the dataset
- A single raw `transporter` column yields two analytical dimensions: brand and delivery mode
- Without UDFs, the final query would require 3 separate `CASE WHEN` blocks inline — functions reduce it to 3 clean function calls

---

## ⚙️ Functions Created

| Function | Input | Output Type | Logic |
|----------|-------|-------------|-------|
| `course17.is_mail_be(journey_name)` | STRING | INT64 (0/1) | `IF + LIKE` |
| `course17.mail_type(journey_name)` | STRING | STRING | `CASE WHEN + LIKE` |
| `course17.nps(global_note)` | INT64 | INT64 (-1/0/1) | `CASE WHEN + IN / BETWEEN` |
| `course17.transporter_brand(transporter)` | STRING | STRING | `CASE WHEN + LIKE` |
| `course17.delivery_mode(transporter)` | STRING | STRING | `CASE WHEN + LIKE` |

---

## 🛠️ SQL Techniques Used

| Technique | Purpose |
|-----------|---------|
| `IF(condition, true, false)` | Simple binary labeling |
| `CASE WHEN ... THEN ... ELSE ... END` | Multi-condition categorization |
| `CREATE OR REPLACE FUNCTION` | Creating and updating UDFs |
| `LIKE "%...%"` | Pattern matching within strings |
| `IN (a, b)` | Checking for specific values |
| `BETWEEN a AND b` | Numeric range check |

---

## 🔧 Tools

- **Google BigQuery** — SQL engine and data warehouse
- **SQL** — All analysis and functions written in standard SQL with BigQuery-specific UDF syntax
