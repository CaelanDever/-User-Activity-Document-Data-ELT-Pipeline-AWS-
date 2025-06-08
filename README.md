# 🧠 User Activity Document Data ELT Pipeline (AWS)

## ⚡️ 30-Second Summary

This project showcases a realistic **cloud-based ELT pipeline** for document-style (JSON) data using AWS. It simulates **user activity logs**, transforms them using AWS Glue (PySpark), stores them in S3, and makes them queryable with AWS Athena. These are core tasks in **Data Engineer**, **ETL Developer**, and **Cloud Engineer** roles.

**Why it matters:** Companies collect large volumes of unstructured or semi-structured data (e.g., logs, JSON, NoSQL exports). This project gives you hands-on experience in turning that data into insights using scalable, serverless cloud tools.

### 🔍 Key Skills Practiced:

* 📦 Ingesting JSON log data into S3
* 🔥 Cataloging and transforming data using AWS Glue
* 🔎 Querying nested JSON with Athena
* 🛠 Automating ingestion via Python & AWS CLI
* 📊 Building a mini data lake pipeline (raw → cleaned → analytics)

### 🧱 Part of a larger stack:

This project represents the "middle layer" in the modern data stack: ingest → transform → serve. It builds cloud skills for serverless processing and is foundational before building dashboards or deploying ML pipelines.

---

## 📁 Project Structure

```
user-activity-pipeline/
├── data_generator.py        # Simulates document ingestion
├── upload_to_s3.py          # Push raw data to S3
├── glue_script.py           # Spark-based transformation
├── README.md                # Project documentation
```

---

## 🛠 Project Details

### ✅ Step 1: Generate Simulated JSON Activity Data

**What:** Generate fake activity logs in JSON format.

```python
# data_generator.py
import json, random, time
from datetime import datetime
import uuid

def generate_log():
    return {
        "event_id": str(uuid.uuid4()),
        "user_id": random.randint(1, 100),
        "action": random.choice(["login", "logout", "purchase", "view"]),
        "timestamp": datetime.utcnow().isoformat(),
        "metadata": {
            "device": random.choice(["mobile", "desktop", "tablet"]),
            "location": random.choice(["US", "EU", "ASIA"])
        }
    }

with open("user_activity.json", "w") as f:
    for _ in range(1000):
        f.write(json.dumps(generate_log()) + "\n")
```

**Why:** Simulates real user events to mimic production log pipelines.

**📸 Screenshot Placeholder:** Example snippet of `user_activity.json`

<img width="551" alt="datagen" src="https://github.com/user-attachments/assets/6f0649ca-4e0c-4943-87ea-e8ec56eda035" />

<img width="534" alt="verify data gen" src="https://github.com/user-attachments/assets/9b1395e7-60f7-45c8-bf74-a8ca0bc836eb" />


---

### ✅ Step 2: Upload Raw Data to AWS S3

**What:** Push the generated file to S3.

```bash
aws s3 mb s3://user-activity-raw-data
aws s3 cp user_activity.json s3://user-activity-raw-data/logs/user_activity.json
```

**Why:** S3 acts as your raw data zone in a cloud data lake architecture.

**📸 Screenshot Placeholder:** S3 bucket view in AWS Console

<img width="488" alt="jsontoaws" src="https://github.com/user-attachments/assets/7143a9bf-990c-4fdd-9612-6c2aaa28fa55" />

<img width="769" alt="awss3log" src="https://github.com/user-attachments/assets/4cc0c613-e308-4bcd-9d9f-953047d6603a" />

---

### ✅ Step 3: Catalog Raw Data with AWS Glue Crawler

**What:** Use AWS Glue Crawler to infer schema from JSON logs.

```bash
aws glue create-crawler \
  --name raw-user-activity-crawler \
  --role AWSGlueServiceRoleDefault \
  --database-name user_activity_db \
  --targets "{\"S3Targets\": [{\"Path\": \"s3://user-activity-raw-data/logs/\"}]}"

aws glue start-crawler --name raw-user-activity-crawler
```

**Why:** Automatically detects schema and registers it to Glue Data Catalog for querying.

**📸 Screenshot Placeholder:** Crawler results showing detected schema

<img width="489" alt="gluerole" src="https://github.com/user-attachments/assets/231f9b8e-5b2e-4edc-b90e-713d76a9061c" />


---

### ✅ Step 4: Transform Data with Glue PySpark Job

**What:** Clean and flatten data using AWS Glue script.

```python
# glue_script.py
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

datasource = glueContext.create_dynamic_frame.from_catalog(
    database="user_activity_db",
    table_name="logs"
)

flattened = datasource.relationalize("root", "s3://temp-bucket/tmp/")
main = flattened.select("root")

cleaned = Filter.apply(frame=main, f=lambda x: x["action"] != "logout")

glueContext.write_dynamic_frame.from_options(
    frame=cleaned,
    connection_type="s3",
    connection_options={"path": "s3://user-activity-cleaned-data/"},
    format="json"
)

job.commit()
```

**Why:** This step represents the "T" in ELT: transforming for analysis.

**📸 Screenshot Placeholder:** Glue job success message

<img width="927" alt="gluejob" src="https://github.com/user-attachments/assets/720f4d35-abde-459c-85b9-b40ed295a436" />

---

### ✅ Step 5: Query Data in Athena

**What:** Create an Athena table for the cleaned dataset.

```sql
CREATE EXTERNAL TABLE user_activity_cleaned (
  event_id string,
  user_id int,
  action string,
  timestamp string,
  metadata struct<device:string,location:string>
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
LOCATION 's3://user-activity-cleaned-data/';
```

**Why:** Use SQL to analyze structured JSON data in S3.

**📸 Screenshot Placeholder:** Athena query result with grouped actions by location
<img width="919" alt="atherests" src="https://github.com/user-attachments/assets/76e9987c-7242-4d3c-8f03-42baca274245" />



---

### ✅ Step 6: Analyze Data

```sql
SELECT metadata.location, action, COUNT(*)
FROM user_activity_cleaned
GROUP BY metadata.location, action;
```

**Why:** Real-world scenario of creating business-level metrics on top of cleaned data.

<img width="706" alt="athenaq2" src="https://github.com/user-attachments/assets/65bcfe0d-0d07-4434-a4a3-e96f59ad46a7" />


<img width="671" alt="quesr2" src="https://github.com/user-attachments/assets/b47e2b03-4a1e-47b6-abfb-9c61a20fc7d2" />


---

## 🧩 Optional Bonus: Airflow Automation

Use Apache Airflow to schedule:

* Data generation
* Upload to S3
* Glue job trigger
* Athena query execution for reporting

**📸 Screenshot Placeholder:** Airflow DAG view (if implemented)

---

## 🚧 Challenges & Solutions

* **Schema errors in Glue:** Resolved by ensuring all JSON records had identical keys.
* **Nested fields in Athena:** Solved using Glue's `relationalize()` and `struct` in DDL.
* **Slow transformations:** Optimized Glue job with partitioning.

---

## ✅ Summary: What I Built

* 🎯 Simulated realistic JSON logs using Python
* ☁️ Built cloud-native ELT pipeline using AWS Glue + Athena
* 🗃 Created S3-based data lake zones (raw and cleaned)
* 🧠 Learned to flatten, clean, and analyze nested JSON at scale
* 📈 Queried complex documents using serverless SQL

---

## 🧹 Cleanup Steps

```bash
aws s3 rm s3://user-activity-raw-data --recursive
aws s3 rm s3://user-activity-cleaned-data --recursive
aws glue delete-job --job-name <your-glue-job>
aws glue delete-crawler --name raw-user-activity-crawler
```

---

## 📚 References

* [AWS Glue Docs](https://docs.aws.amazon.com/glue/)
* [Athena JSON Docs](https://docs.aws.amazon.com/athena/latest/ug/json-serde.html)
* [Boto3 S3](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html)
* [Open Data JSON SerDe](https://github.com/rcongiu/Hive-JSON-Serde)

---

**Congratulations, you've completed the project! 🎉**
