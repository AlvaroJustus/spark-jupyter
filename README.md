# Spark Cluster with Jupyter Notebook

[![Docker Compose](https://img.shields.io/badge/Docker%20Compose-1.29.2-blue)](https://docs.docker.com/compose/)
[![Apache Spark](https://img.shields.io/badge/Apache%20Spark-3.5.0-orange)](https://spark.apache.org/)
[![Jupyter Notebook](https://img.shields.io/badge/Jupyter%20Notebook-6.4.3-yellow)](https://jupyter.org/)

This project sets up an Apache Spark cluster with a Jupyter Notebook interface using Docker Compose. The setup includes:

- Spark Master
- Two Spark Workers
- Spark History Server
- Jupyter Notebook for interactive data analysis

## Prerequisites

- Docker
- Docker Compose

## File structure

- `docker-compose.yaml`: Defines Docker services for Spark Master, Spark Workers, and Jupyter Notebook.

## Getting Started

### Clone the Repository

```bash
git clone https://github.com/your-repo/spark-cluster-jupyter.git
cd spark-cluster-jupyter
```

### Start the Cluster
To start the Spark cluster with Jupyter Notebook, run:
```bash
docker-compose up -d
```
## Components

### Overview

### Spark Master
- The Spark Master coordinates the cluster and schedules the jobs.

### Spark Workers
- The Spark Workers perform the actual data processing. You can adjust the number of workers and their resources (memory and cores) in the `docker-compose.yml` file.

### Spark History Server

- The Spark History Server provides a web interface to view completed Spark applications.

### Jupyter Notebook

- The Jupyter Notebook provides an interactive interface for data analysis using PySpark.

### Accessing the Components
- **Spark Master**: 
    - URL: http://localhost:8080
    - Spark Master web interface for cluster monitoring.

- **Spark History Server**: 
    - URL: http://localhost:18080
    - Storing logs in the Host's `./logs` directory.

- **Jupyter Notebook**: 
    - URL: http://localhost:8888
    - Jupyter Notebook web interface to run interactive notebooks.


## Configuration

### Adjusting Number of Workers, Memory, and Cores

You can adjust the number of Spark workers, memory, and cores by modifying the `docker-compose.yml` file.

**Example: Adding Another Worker**\
To add another worker, copy the spark-worker-2 service and update the service name and environment variables as needed.
```yaml
  spark-worker-3:
    image: bitnami/spark:3.5.0
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://spark-master:7077
      - SPARK_WORKER_MEMORY=4G
      - SPARK_WORKER_CORES=4
      - SPARK_EXTRA_CLASSPATH=/opt/bitnami/spark/jars/spark-measure_2.13-0.24.jar
    depends_on:
      - spark-master
    networks:
      - spark-network
    volumes:
      - ./jars:/opt/bitnami/spark/jars
```
## Changing Worker Memory and Cores
Adjust the environment variables `SPARK_WORKER_MEMORY` and `SPARK_WORKER_CORES` for each worker service in the docker-compose.yml file.

**Example Spark Application**\
You can run Spark applications using the Jupyter Notebook. Here is an example of a simple Spark job:
```python
from pyspark.sql import SparkSession
import findspark
findspark.init()

# Initialize Spark session
spark = SparkSession.builder \
    .appName("JupyterSparkExample") \
    .master("spark://spark-master:7077") \
    .config("spark.eventLog.enabled", "true") \
    .config("spark.jars", "/home/jovyan/jars/spark-measure_2.12-0.24.jar") \
    .config("spark.eventLog.dir", "file:///home/jovyan/work/logs") \
    .getOrCreate()

# Sample DataFrame
data = [("John Doe", 30), ("Jane Doe", 25), ("Sam Brown", 35)]
columns = ["Name", "Age"]
df = spark.createDataFrame(data, columns)

# Show DataFrame
df.show()

df_grouped = df.groupBy("Age").count()
df_grouped.show()
```
## Using Spark Measure

Spark Measure is a tool for measuring and profiling Spark jobs. It can be used to collect metrics and provide insights into the performance of your Spark applications.

### Example Usage

To use Spark Measure in your Jupyter Notebook, you can follow these steps:

1. **Import Required Libraries:**

    ```python
    from sparkmeasure import StageMetrics
    from pyspark.sql import SparkSession
    import findspark
    findspark.init()
    ```

2. **Initialize Spark Session:**

    ```python
    spark = SparkSession.builder \
    .appName("JupyterSparkExample") \
    .master("spark://spark-master:7077") \
    .config("spark.eventLog.enabled", "true") \
    .config("spark.jars", "/home/jovyan/jars/spark-measure_2.12-0.24.jar") \
    .config("spark.eventLog.dir", "file:///home/jovyan/work/logs") \
    .getOrCreate()
    ```

3. **Initialize StageMetrics:**

    ```python
    stagemetrics = StageMetrics(spark)
    ```

4. **Run and Measure Spark Job:**

    ```python
    query = """spark.sql("SELECT SUM(value) FROM (SELECT EXPLODE(SEQUENCE(1, 1000000)) AS value)").show()"""

    stagemetrics.runandmeasure(globals(), query)
    ```

5. **Collect and Display Metrics:**

    ```python
    # Collect and display metrics
    stagemetrics.begin()
    spark.sql("SELECT SUM(value) FROM (SELECT EXPLODE(SEQUENCE(1, 1000000)) AS value)").show()
    stagemetrics.end()

    stagemetrics.print_memory_report()
    ```

### Further Examples

You can also collect metrics for different types of Spark operations. Here is an example of collecting metrics for a DataFrame operation that performs aggregation:

```python
# Initialize StageMetrics
stagemetrics = StageMetrics(spark)

# Begin collecting metrics
stagemetrics.begin()

# Example DataFrame operation: Aggregation
data = [("Alice", "HR", 1000), ("Bob", "HR", 1500), ("Cathy", "IT", 2000), ("David", "IT", 2500)]
columns = ["Name", "Department", "Salary"]
df = spark.createDataFrame(data, columns)

# Group by department and calculate average salary
df.groupBy("Department").avg("Salary").show()

# End collecting metrics
stagemetrics.end()

# Display the collected metrics
metrics_df = stagemetrics.create_spark_dataframe()
metrics_df.show(truncate=False)
```

## Contributions

We welcome contributions to enhance the functionality and usability of this project. Here are some ways you can contribute:

### Reporting Bugs

If you find a bug, please report it by opening an issue on the [GitHub Issues](https://github.com/AlvaroJustus/spark-jupyter/issues) page. Be sure to include details about the bug, how to reproduce it, and any relevant logs or screenshots.

### Suggesting Features

Have an idea for a new feature? We'd love to hear from you! Open an issue on the [GitHub Issues](https://github.com/AlvaroJustus/spark-jupyter/issues) page and describe the feature in detail, including its potential benefits and use cases.

### Submitting Pull Requests

1. **Fork the Repository**: Click on the "Fork" button at the top right of the repository page.
2. **Clone Your Fork**: Clone your forked repository to your local machine.
   ```bash
   git clone https://github.com/AlvaroJustus/spark-jupyter.git
    ```
3. **Create a Branch**: Create a new branch for your feature or bugfix.
    ```bash
    git checkout -b feature/your-feature-name
    ```
4. **Make Changes**: Make your changes to the codebase.
5. **Commit Changes**: Commit your changes with a descriptive commit message.
    ```bash
    git add .
    git commit -m "Description of the changes made"
    ```
6. **Push Changes**: Push your changes to your forked repository.
    ```bash
    git push origin feature/your-feature-name
    ```
7. **Open a Pull Request**: Go to the original repository and open a pull request from your forked branch.