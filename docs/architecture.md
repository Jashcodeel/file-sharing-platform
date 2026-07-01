# Architecture

This document explains the architecture of the Production-Ready Self-Hosted File Sharing Platform built using Docker Compose on an AWS EC2 instance.

The platform is based on Nextcloud and includes PostgreSQL, Redis, Nginx, Prometheus, Grafana, cAdvisor, and Node Exporter.

---

## 1. High-Level Architecture


User Browser
    |
    | HTTPS
    v
AWS Security Group
    |
    v
Nginx Reverse Proxy
    |
    v
Nextcloud Application
    |
    |---------------- PostgreSQL Database
    |
    |---------------- Redis Cache / File Locking

Monitoring Stack:
Prometheus + Grafana + cAdvisor + Node Exporter
```

The application is hosted on a single AWS EC2 instance. All services run as Docker containers and are managed using Docker Compose.

Nginx acts as the public entry point. Nextcloud is not directly exposed to the internet. PostgreSQL and Redis are also not exposed publicly and are only accessible through Docker internal networking.

---

## 2. Component Overview

| Component      | Purpose                                            |
| -------------- | -------------------------------------------------- |
| AWS EC2        | Virtual machine hosting the Docker environment     |
| Docker Engine  | Container runtime used to run application services |
| Docker Compose | Defines and manages the multi-container stack      |
| Nginx          | Reverse proxy and HTTPS endpoint                   |
| Nextcloud      | File sharing and collaboration application         |
| PostgreSQL     | Database used by Nextcloud for metadata            |
| Redis          | Cache and file-locking backend                     |
| Prometheus     | Metrics collection system                          |
| Grafana        | Monitoring dashboard and visualization tool        |
| cAdvisor       | Container-level metrics collection                 |
| Node Exporter  | Host-level EC2/Linux metrics collection            |

---

## 3. Request Flow

When a user accesses the application, the request follows this path:


User
 |
 | HTTPS Request
 v
Nginx Reverse Proxy
 |
 | Internal Docker Network
 v
Nextcloud
 |
 | Database Queries
 v
PostgreSQL

Nextcloud also communicates with Redis for caching and file locking.
```

Nginx receives the HTTPS request on port `443` and forwards it internally to the Nextcloud container on port `80`.

---

## 4. Docker Network Design

The stack uses separate Docker networks to isolate traffic between services.


frontend network:
- nginx
- nextcloud

backend network:
- nextcloud
- postgres
- redis

monitoring network:
- prometheus
- grafana
- cadvisor
- node-exporter
```

### frontend network

The frontend network is used for web traffic between Nginx and Nextcloud.


nginx <----> nextcloud
```

### backend network

The backend network is used for internal application dependencies.


nextcloud <----> postgres
nextcloud <----> redis
```

PostgreSQL and Redis are not connected to the frontend network and are not exposed to the internet.

### monitoring network

The monitoring network is used for metrics collection and dashboard communication.


prometheus <----> cadvisor
prometheus <----> node-exporter
grafana <----> prometheus
```

This separation improves security and follows the principle of least privilege.

---

## 5. Storage Architecture

The project uses Docker named volumes for persistent storage.


Docker Volumes
 |
 |-- file-sharing-platform_nextcloud_data
 |-- file-sharing-platform_postgres_data
 |-- file-sharing-platform_redis_data
 |-- file-sharing-platform_prometheus_data
 |-- file-sharing-platform_grafana_data
```

### Nextcloud data

Uploaded files are stored in the Nextcloud volume:


file-sharing-platform_nextcloud_data
```

Inside the container, uploaded files are stored under:


/var/www/html/data/<username>/files/
```

### PostgreSQL data

PostgreSQL database files are stored in:


file-sharing-platform_postgres_data
```

Inside the container, PostgreSQL stores data under:


/var/lib/postgresql/data
```

### Redis data

Redis data is stored in:


file-sharing-platform_redis_data
```

Redis is mainly used for cache and file locking.

---

## 6. Data Flow

When a user uploads a file:


User uploads file
    |
    v
Nginx receives HTTPS request
    |
    v
Nginx forwards request to Nextcloud
    |
    v
Nextcloud stores file content in nextcloud_data volume
    |
    v
Nextcloud stores file metadata in PostgreSQL
    |
    v
Redis supports caching and file locking
```

The file content and metadata are stored separately:

| Data Type             | Stored In             |
| --------------------- | --------------------- |
| Uploaded file content | Nextcloud data volume |
| File metadata         | PostgreSQL database   |
| Cache / file locking  | Redis                 |

---

## 7. Security Design

The architecture follows basic security practices:

* Only Nginx is exposed as the public application entry point.
* PostgreSQL is not exposed publicly.
* Redis is not exposed publicly.
* Application secrets are stored in `.env`, which is excluded from Git.
* SSL private keys are excluded from Git.
* Backup files are excluded from Git.
* Docker networks are separated by function.
* Grafana access should be restricted using AWS Security Group rules.

Current HTTPS is configured using a self-signed certificate for lab testing. For production use, this should be replaced with a trusted certificate from Let's Encrypt or another certificate authority.

---

## 8. Backup Architecture

The backup process includes:


PostgreSQL dump
Nextcloud data volume archive
Nginx certificate backup
Configuration reference files
```

Backup script location:


scripts/backup.sh
```

Backup output location:


backups/<timestamp>/
```

The backup script places Nextcloud into maintenance mode before taking the backup. This helps keep the database and file data consistent during the backup process.

---

## 9. Monitoring Architecture

The monitoring stack contains:


Grafana
   |
   v
Prometheus
   |
   |-- cAdvisor
   |-- Node Exporter
```

### Prometheus

Prometheus collects metrics from configured targets.

### Grafana

Grafana is used to visualize metrics collected by Prometheus.

### cAdvisor

cAdvisor collects container-level metrics such as CPU, memory, and network usage.

### Node Exporter

Node Exporter collects EC2/Linux host-level metrics such as CPU, memory, disk, and filesystem usage.

---

## 10. Current Limitations

This is a single-node lab-grade production-style deployment.

Current limitations:

* No high availability
* No real domain name
* Self-signed SSL certificate
* PostgreSQL runs as a local container
* Redis runs as a local container
* Docker volumes are local to one EC2 instance
* No centralized logging
* No automated alerting
* No CI/CD pipeline

---

## 11. Recommended Production Improvements

For a real production deployment, the following improvements are recommended:

| Area              | Recommended Improvement                                            |
| ----------------- | ------------------------------------------------------------------ |
| Domain            | Use a real DNS domain                                              |
| HTTPS             | Replace self-signed SSL with Let's Encrypt                         |
| Database          | Move PostgreSQL to Amazon RDS                                      |
| Redis             | Move Redis to Amazon ElastiCache                                   |
| File Storage      | Use Amazon EFS or external shared storage                          |
| High Availability | Use multiple application nodes behind an Application Load Balancer |
| Backup            | Store backups in Amazon S3                                         |
| Monitoring        | Add Grafana alerts                                                 |
| Logging           | Add centralized logging                                            |
| Security          | Use AWS Secrets Manager or a secure vault                          |
| Deployment        | Add CI/CD pipeline                                                 |

---

## 12. Final Architecture Summary

This project demonstrates a production-style containerized application deployment on AWS EC2.

It includes:

* Multi-container Docker Compose architecture
* Reverse proxy using Nginx
* Self-signed HTTPS
* Persistent storage using Docker volumes
* PostgreSQL database backend
* Redis cache and file-locking backend
* Backup and restore process
* Monitoring using Prometheus and Grafana
* Docker network segmentation
* Git-based infrastructure documentation


***END***
