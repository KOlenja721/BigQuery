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
|Example Exam For ...|
|Building Web Serv...|
|Cloud Native Arch...|
|Introduction to A...|
|      Arduino Inputs|
|Learning Linux Se...|
|Introduction to M...|
|Arduino Prototypi...|
|  Learning Java EE 7|
|Nulls, Three-valu...|
| Mastering Web Views|
|An Introduction t...|
|              TCP/IP|
|       Mastering Git|
|Reproducible Rese...|
|Introduction to D...|
|Learning to Progr...|
|Using Storytellin...|
|Learning Apache C...|
|Design Patterns i...|
|The Principles of...|
|Hibernate and JPA...|
|          Great Bash|
|Architectural Con...|
|Learning Data Str...|
|Getting Ready for...|
|Beginning C# Prog...|
|Software Architec...|
|The Closed World ...|
|Hadoop Fundamenta...|
|Web & Native Work...|
|        Learning SQL|
|Practical Java Pr...|
|Using R for Big D...|
|SQL: Beyond the B...|
|JavaScript: The G...|
|Intermediate Pyth...|
|Collaborating wit...|
|Introduction to T...|
|Introduction to M...|
|    Learning Eclipse|
|Working with Algo...|
|Using Web Components|
|Git Fundamentals ...|
|Relational Theory...|
|Normal Forms and ...|
|What's New in Jav...|
|Introduction to M...|
|Software Architec...|
|Learning SQL for ...|
|Data Science with...|
|Native Web Apps f...|
|   Python Epiphanies|
|Introduction to S...|
|Client-Side Data ...|
|Introduction to A...|
|Advanced Machine ...|
|Cloud Computing W...|
|Operating Red Hat...|
|    HTML5 The Basics|
|Expert Data Wrang...|
|Beginning Program...|
+--------------------+
````
````
spark.sql("select count(distinct(exam_name)) from assessments").show(100)
````

+-------------------------+                                                     
|count(DISTINCT exam_name)|
+-------------------------+
|                      103|
+-------------------------+


````
spark.sql("select exam_name, count(*) as count  from assessments group by exam_name order by count desc").show(100)
````

+--------------------+-----+                                                    
|           exam_name|count|
+--------------------+-----+
|        Learning Git|  394|
|Introduction to P...|  162|
|Intermediate Pyth...|  158|
|Introduction to J...|  158|
|Learning to Progr...|  128|
|Introduction to M...|  119|
|Software Architec...|  109|
|Beginning C# Prog...|   95|
|    Learning Eclipse|   85|
|Learning Apache M...|   80|
|Beginning Program...|   79|
|       Mastering Git|   77|
|Introduction to B...|   75|
|Advanced Machine ...|   67|
|Learning Linux Sy...|   59|
|JavaScript: The G...|   58|
|        Learning SQL|   57|
|Practical Java Pr...|   53|
|    HTML5 The Basics|   52|
|   Python Epiphanies|   51|
|Software Architec...|   48|
|Intermediate C# P...|   43|
|Introduction to D...|   43|
|        Learning DNS|   40|
|Learning C# Best ...|   35|
|Expert Data Wrang...|   35|
|Mastering Advance...|   34|
|An Introduction t...|   33|
|Data Visualizatio...|   31|
|Python Data Struc...|   29|
|Cloud Native Arch...|   29|
|Introduction to T...|   28|
|Git Fundamentals ...|   28|
|Introduction to S...|   27|
|Learning Linux Se...|   27|
|  Learning Java EE 7|   25|
|Mastering Python ...|   25|
|Using R for Big D...|   24|
|Learning C# Desig...|   23|
|Reproducible Rese...|   21|
|              TCP/IP|   21|
|JavaScript Templa...|   21|
|Refactor a Monoli...|   17|
|Cloud Computing W...|   17|
|Learning iPython ...|   17|
|Learning Apache H...|   16|
|Networking for Pe...|   15|
|I'm a Software Ar...|   15|
|Design Patterns i...|   15|
|Relational Theory...|   15|
|Working with Algo...|   14|
|          Great Bash|   14|
|Introduction to A...|   14|
|         Offline Web|   13|
|Introduction to M...|   13|
|Learning Data Str...|   13|
|Introduction to A...|   13|
|Learning Apache C...|   12|
|Amazon Web Servic...|   12|

````
spark.sql("select exam_name, count(*) as count  from assessments where exam_name like '%Python%' group by exam_name order by count desc").show(100)
````

+--------------------+-----+                                                    
|           exam_name|count|
+--------------------+-----+
|Introduction to P...|  162|
|Intermediate Pyth...|  158|
|   Python Epiphanies|   51|
|Python Data Struc...|   29|
|Mastering Python ...|   25|
|Learning iPython ...|   17|
|Working with Algo...|   14|
+--------------------+-----+


````
exit()

docker-compose down
````

For commands,
````
history | grep -i assessments
````
