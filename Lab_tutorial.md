# Lab Tutorial: Wildlife Data Pipeline on HDFS & Hive

---

## 1. Upload Dataset to Linux from Local PC

```bash
scp "YOURDATASETSPATH\metadata_mammalia_global.csv" USERNAME@ipaddress:/home/nshah37/

scp "YOURDATASETSPATH\observations_mammalia_global.csv" USERNAME@ipaddress:/home/nshah37/
```

---

## 2. Set Up HDFS Directory & Load Data

```bash
# Create a directory in HDFS for your project
hdfs dfs -mkdir -p /user/nshah37/wildlife

# Move your CSV files from Linux to HDFS
hdfs dfs -put observations.csv /user/nshah37/wildlife/
hdfs dfs -put metadata.csv /user/nshah37/wildlife/

# Give your team Read and Execute permissions
hdfs dfs -setfacl -m user:rmehra:r-x /user/nshah37/wildlife
hdfs dfs -setfacl -m user:rperez46:r-x /user/nshah37/wildlife
hdfs dfs -setfacl -m user:ffigue14:r-x /user/nshah37/wildlife
```

---

## 3. Create & Select Database

```sql
-- Create your database if you haven't yet
CREATE DATABASE IF NOT EXISTS nshah37;

-- Switch to your database
USE nshah37;
```

---

## 4. Create External Tables

### 4.1 Observations Table

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS observations_raw (
    id                        BIGINT,
    observed_on               STRING,
    local_time_observed_at    STRING,
    latitude                  DOUBLE,
    longitude                 DOUBLE,
    positional_accuracy       INT,
    public_positional_accuracy INT,
    image_url                 STRING,
    license                   STRING,
    geoprivacy                STRING,
    taxon_geoprivacy          STRING,
    scientific_name           STRING,
    common_name               STRING,
    taxon_id                  INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION '/user/nshah37/wildlife/'
TBLPROPERTIES ("skip.header.line.count"="1");

-- Inspect table structure
DESCRIBE FORMATTED observations_raw;
```

---

### 4.2 Metadata Table

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS metadata_raw (
    id                                BIGINT,
    lat                               DOUBLE,
    long                              DOUBLE,
    observed_on                       STRING,
    time_zone                         STRING,
    elevation                         DOUBLE,
    time_stamp                        STRING,
    temperature_2m                    DOUBLE,
    relativehumidity_2m               INT,
    dewpoint_2m                       DOUBLE,
    apparent_temperature              DOUBLE,
    surface_pressure                  DOUBLE,
    precipitation                     DOUBLE,
    rain                              DOUBLE,
    snowfall                          DOUBLE,
    cloudcover                        INT,
    cloudcover_low                    INT,
    cloudcover_mid                    INT,
    cloudcover_high                   INT,
    shortwave_radiation               DOUBLE,
    direct_radiation                  DOUBLE,
    diffuse_radiation                 DOUBLE,
    windspeed_10m                     DOUBLE,
    windspeed_100m                    DOUBLE,
    winddirection_10m                 INT,
    winddirection_100m                INT,
    windgusts_10m                     DOUBLE,
    et0_fao_evapotranspiration_hourly DOUBLE,
    weathercode_hourly                INT,
    vapor_pressure_deficit            DOUBLE,
    soil_temperature_0_to_7cm        DOUBLE,
    soil_temperature_7_to_28cm       DOUBLE,
    soil_temperature_28_to_100cm     DOUBLE,
    soil_moisture_0_to_7cm           DOUBLE,
    soil_moisture_7_to_28cm          DOUBLE,
    soil_moisture_28_to_100cm        DOUBLE,
    weathercode_daily                 INT,
    temperature_2m_max                DOUBLE,
    temperature_2m_min                DOUBLE,
    apparent_temperature_max          DOUBLE,
    apparent_temperature_min          DOUBLE,
    precipitation_sum                 DOUBLE,
    rain_sum                          DOUBLE,
    snowfall_sum                      DOUBLE,
    precipitation_hours               DOUBLE,
    sunrise                           STRING,
    sunset                            STRING,
    windspeed_10m_max                 DOUBLE,
    windgusts_10m_max                 DOUBLE,
    winddirection_10m_dominant        INT,
    shortwave_radiation_sum           DOUBLE,
    et0_fao_evapotranspiration_daily  DOUBLE
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION '/user/nshah37/wildlife/'
TBLPROPERTIES ("skip.header.line.count"="1");

-- Inspect table structure
DESCRIBE FORMATTED metadata_raw;

-- Show all tables
SHOW TABLES;
```

---

## 5. Row Count Verification

```sql
-- Count rows in metadata_raw
SELECT COUNT(*) FROM metadata_raw;

-- Count rows in observations_raw
SELECT COUNT(*) FROM observations_raw;
```

---

## 6. Create the Master Table (Golden Dataset)

This master table acts as the **"Golden Dataset"** for the group. It merges the specific columns required by every team member's research question. By joining on the `id` column, each row contains both wildlife details and the corresponding weather/location context.

| Column | Purpose |
|---|---|
| `common_name`, `scientific_name` | Species identification |
| `latitude`, `longitude` | Location (Niyati, Perez, Radhika) |
| `observed_on` | Date of observation (Niyati, Perez, Radhika) |
| `temperature_2m` | Weather context (Radhika) |
| `obs_hour` | Hourly timestamp (Fatima) |
| `elevation` | Extra spatial context (all members) |

```sql
CREATE TABLE fab_four_master_table AS
SELECT
    o.id,
    o.common_name,
    o.scientific_name,
    o.latitude,
    o.longitude,
    o.observed_on,           -- For Niyati, Perez, and Radhika
    m.temperature_2m,        -- For Radhika
    m.time_stamp AS obs_hour, -- For Fatima
    m.elevation              -- Extra context for any member
FROM observations_raw o
JOIN metadata_raw m ON (o.id = m.id);
```

---

## 7. Verify the Master Table

```sql
-- Preview first 5 rows
SELECT * FROM fab_four_master_table LIMIT 5;

-- Verify table structure
DESCRIBE FORMATTED fab_four_master_table;
```
