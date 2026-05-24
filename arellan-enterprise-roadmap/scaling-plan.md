# Architectural Cloud Scaling Plan

## 1. Base Compute Strategy
The system transitions from an initial single-node Docker Compose setup into an elastic, container-managed cloud tier using AWS ECS (Elastic Container Service) or Azure Kubernetes Service (AKS).

## 2. Database Read/Write Segregation
To support intense analytics queries from `arellan-data-pipeline` without blocking workshop operations, PostgreSQL will scale utilizing a primary cluster for transaction ingestion and an asynchronous read-replica array dedicated to dashboards and reporting engines.