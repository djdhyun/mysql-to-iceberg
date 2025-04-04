ICEBERG_VERSION       := 1.4.3
SPARK_VERSION         := 3.4
SCALA_VERSION         := 2.12
# NOTE: Hive 4.0.1 doesn't work with Spark 3.5 and Iceberg >= 1.3
HIVE_VERSION          := 3.1.3 
POSTGRESQL_VERSION    := 42.7.2

MAVEN_REPO            := https://repo1.maven.org/maven2

ICEBERG_JAR           := config/spark/jars/iceberg-spark-runtime-$(SPARK_VERSION)_$(SCALA_VERSION)-$(ICEBERG_VERSION).jar
POSTGRES_JAR          := config/hive/jars/postgresql-$(POSTGRESQL_VERSION).jar

.PHONY: up
up: jars
	mkdir -p ./data/postgresql
	mkdir -p ./data/warehouse

	SPARK_VERSION=$(SPARK_VERSION) \
		HIVE_VERSION=$(HIVE_VERSION) \
		IS_RESUME=$(IS_RESUME) \
		docker-compose -f docker-compose.yaml up -d

.PHONY: down
down:
	SPARK_VERSION=$(SPARK_VERSION) \
		HIVE_VERSION=$(HIVE_VERSION) \
		docker-compose -f docker-compose.yaml down --remove-orphans

.PHONY: clean
clean: down
	rm -rf data

.PHONY: log
log:
	docker-compose -f docker-compose.yaml logs -f


.PHONY: jars
jars: $(ICEBERG_JAR) $(POSTGRES_JAR)

$(ICEBERG_JAR):
	curl -fLo $@ $(MAVEN_REPO)/org/apache/iceberg/iceberg-spark-runtime-$(SPARK_VERSION)_$(SCALA_VERSION)/$(ICEBERG_VERSION)/iceberg-spark-runtime-$(SPARK_VERSION)_$(SCALA_VERSION)-$(ICEBERG_VERSION).jar

$(POSTGRES_JAR):
	curl -fLo $@ $(MAVEN_REPO)/org/postgresql/postgresql/$(POSTGRESQL_VERSION)/postgresql-$(POSTGRESQL_VERSION).jar


