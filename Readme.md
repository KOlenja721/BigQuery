Navigate to the project directory
````
cd ~/w205/project-2-anand-eyunni
````

Download json data for assessments
````
curl -L -o assessment-attempts-20180128-121051-nested.json https://goo.gl/ME6hjp
````

Copy docker configuration yml file from the course directory
````
cp ~/w205/course-content/08-Querying-Data/docker-compose.yml .
````


Spin up the cluster
````
docker-compose up -d
````

Create a kafka topic assessments
````
docker-compose exec kafka kafka-topics --create --topic assessments --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
````


Display the kafka topic
````
docker-compose exec kafka kafka-topics --describe --topic assessments --zookeeper zookeeper:32181
````


Use kafkacat to produce test messages to the assessments topic
````
docker-compose exec mids bash -c "cat /w205/project-2-anand-eyunni/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t assessments"


docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t assessments -o beginning -e"
````

spin up pyspark process using the spark container
````
docker-compose exec spark pyspark
````

Import json libraries
````
import json
````

At the prompt, read from kafka
````
raw_assessments = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","assessments").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 
````

Suppress warnings
````
raw_assessments.cache()
````

Print schema of assessments
````
raw_assessments.printSchema()
````

Cast raw data into string
````
assessments = raw_assessments.selectExpr("CAST(value AS STRING)")
````

Write the assessments data to hdfs
````
assessments.write.parquet("/tmp/assessments")
````


Check the result in hadoop
````
docker-compose exec cloudera hadoop fs -ls /tmp/


docker-compose exec cloudera hadoop fs -ls /tmp/assessments/

````

See the written results
````
assessments.show()

````


Default to unicode from ascii
````
import sys
sys.stdout = open(sys.stdout.fileno(), mode='w', encoding='utf8', buffering=1)
````


Extract an assessments data frame that looks like the content of our json
````
import json
from pyspark.sql import Row
extracted_assessments = assessments.rdd.map(lambda x: Row(**json.loads(x.value))).toDF()
extracted_assessments.show()
````

Save the assessments data to parquet file for efficient querying. This one is a flat file
````
extracted_assessments.write.parquet("/tmp/extracted_assessments")
````

Show the assessments data
````
assessments.show()
extracted_assessments.show()
````

Create a spark TempTable/view
````
extracted_assessments.registerTempTable('assessments')
````

Create data frames and answer our four questions:

1. What are all the different types of assessments?
2. How many distinct assessments are available?
3. How many test takers have taken each assessment?
4. How many assessments are related to Python?

````
spark.sql("select distinct(exam_name) from assessments").show(100)
````

````
spark.sql("select count(distinct(exam_name)) from assessments").show(100)
````

````
spark.sql("select exam_name, count(*) as count  from assessments group by exam_name order by count desc").show(100)
````


````
spark.sql("select exam_name, count(*) as count  from assessments where exam_name like '%Python%' group by exam_name order by count desc").show(100)
````

````
exit()

docker-compose down
````

For commands,
````
history | grep -i assessments
````