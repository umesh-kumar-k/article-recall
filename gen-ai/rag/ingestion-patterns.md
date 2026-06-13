---
aliases:
  - Ingestion Patterns
highlights: |-
  Batch Ingestion: Scheduled full/incremental loads; suitable for stable datasets; simpler orchestration (Airflow, Prefect)

  Streaming Ingestion: Real-time indexing via CDC(Debezium, Kafka); required for time-sensitive use cases; operational complexity 

  Lazy Indexing: Index on-demand when documents accessed; reduces upfront costs; adds latency to first query

  Hierarchical Chunking: Parent-child relationships (summarizes -> sections -> paragraphs); retrieve at appropriate granularity
tags:
  - rag
  - ingestion
  - ingestion-pipeline
Source 1: https://docs.databricks.com/aws/en/generative-ai/tutorials/ai-cookbook/quality-data-pipeline-rag
Source 2: https://www.integrate.io/blog/enterprise-data-pipelines/
---
