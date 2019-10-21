````
cd ~/w205/project-2-anand-eyunni
````

curl -L -o assessment-attempts-20180128-121051-nested.json https://goo.gl/ME6hjp

cp ~/w205/course-content/08-Querying-Data/docker-compose.yml .

docker-compose up -d

docker-compose exec kafka kafka-topics --create --topic assessments --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181

docker-compose exec kafka kafka-topics --describe --topic assessments --zookeeper zookeeper:32181

docker-compose exec mids bash -c "cat /w205/project-2-anand-eyunni/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t assessments"

docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t assessments -o beginning -e"
	
docker-compose exec spark pyspark

import json

raw_assessments = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","assessments").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 

raw_assessments.cache()

raw_assessments.printSchema()

assessments = raw_assessments.selectExpr("CAST(value AS STRING)")

assessments.write.parquet("/tmp/assessments")

docker-compose exec cloudera hadoop fs -ls /tmp/

docker-compose exec cloudera hadoop fs -ls /tmp/assessments/

assessments.show()

import sys
sys.stdout = open(sys.stdout.fileno(), mode='w', encoding='utf8', buffering=1)

import json

from pyspark.sql import Row
extracted_assessments = assessments.rdd.map(lambda x: Row(**json.loads(x.value))).toDF()
extracted_assessments.show()

extracted_assessments.write.parquet("/tmp/extracted_assessments")

assessments.show()
extracted_assessments.show()

extracted_assessments.registerTempTable('assessments')

spark.sql("select distinct(exam_name) from assessments").show(100)

spark.sql("select count(distinct(exam_name)) from assessments").show(100)

spark.sql("select exam_name, count(*) as count  from assessments group by exam_name order by count desc").show(100)

spark.sql("select exam_name, count(*) as count  from assessments group by exam_name order by count desc").show(100)
	
spark.sql("select exam_name, count(*) as count  from assessments where exam_name like '%Python%' group by exam_name order by count desc").show(100)



history | grep -i assessments
