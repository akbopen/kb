# Knowdge Base - To learn, to share

# Services - Frameworks

## Pub/Sub - Distributed Message System
- Kafka

## Relational DB Managment System
- MySQL

## Object-Relational DB management System
- PostgreSQL

## Distributed Time Series Database
- InfluxDB
- ClickHouse
- VictoriaMetrics
https://medium.com/@valyala/insert-benchmarks-with-inch-influxdb-vs-victoriametrics-e31a41ae2893

Conclusions
VictoriaMetrics has better insert performance than InfluxDB in all the tests. The performance gap between VictoriaMetrics and InfluxDB increases with higher cardinality.
VictoriaMetrics uses less RAM than InfluxDB on high cardinality time series.
It is easy to reproduce benchmark results— just run inch tool against docker containers with VictoriaMetrics and InfluxDB on your hardware. Post your results in comments.
Raw benchmark results from this post are available in this spreadsheet. As for the select performance, see this spreadsheet. In short, VictoriaMetrics outperforms InfluxDB in all the queries, especially on heavy queries touching millions of data points and thousands of time series.
Though VictoriaMetrics’ main purpose is the best long-term remote storage for Prometheus, its’ single-server version still can substitute InfluxDB for collecting data from Influx-compatible agents such as Telegraf. VictoriaMetrics supports native PromQL, so simpler yet powerful queries could be used for building graphs from influx data comparing to InfluxQL or Flux.

- Other
https://devconnected.com/4-best-time-series-databases-to-watch-in-2019/


## In-Memory Data structure store, NoSQL DB
- Redis

## Distributed NoSQL DB
- Cassandra

## Metrics Dashboard
- Grafana

## Search & Analyze Data in Real Time
- Elasticsearch


# Clound

- Ali

- AWS, Start from AWS?

- Google

- Azure

- DigitalOcean

- UpClound

# Service Region Comparation
- per Continent/Country/Market

# Service Plan Comparation
- Grasp? Compare after discount?

# Payment functionality?
# Web GUI functionality
# Prototype: Web GUI -> restful setup services on different cloud?
# Use cases
- [Streaming data pipeline using Kafka, KSQL, InfluxDB and Grafana](https://medium.com/@ketulsheth2/streaming-data-pipeline-using-kafka-ksql-influxdb-and-grafana-8a934569fcb9)

# Startup Operation
- How is a startup running/organizing?

- How to market?

## Security?

## Law/IP issues

