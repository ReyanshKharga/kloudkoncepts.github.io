---
description: 
---

# Set Up Logging Using Fluent Bit and CloudWatch

To send logs from your containers to Amazon CloudWatch Logs, you can use `Fluent Bit` or `Fluentd`.

`Fluentd` is a versatile log collector that gathers logs from different places and sends them to databases or other tools like Kafka, Elasticsearch, CloudWatch, InfluxDB, etc. It's highly adaptable and works well in complex setups.

Meanwhile, `Fluent Bit` is a lightweight counterpart to `Fluentd` ideal for places where resources are limited, like in edge computing or kubernetes environments. It efficiently processes and sends logs while using very little memory.

