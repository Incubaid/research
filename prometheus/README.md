# Prometheus research

## Comparison with influxdb:
See https://prometheus.io/docs/introduction/comparison/#prometheus-vs-influxdb

## Comparison with our aggregator architecture
Our current aggregator system works by application pushing data to a redis instances.
Inside redis we do some aggregation, then another process need to pull the data out of redis and push then into influxdb.

Prometheus works differently, It has a pulling model where application needs to expose some metrics on an HTTP endpoint. Prometheus then collect these metrics by querying the application periodically.

It also have some alerting mechanism. You can configure it so it send alert
when a condition is met. It has support for a lot's of different communication means, telegram, mail, ...


## POCs
- [Redis aggregator](pocs/redis_aggregator)
- [Instrumenting a python application](pocs/instrument_python_app)
- [Node system monitoring](pocs/node_system)