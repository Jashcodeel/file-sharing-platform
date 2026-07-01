
Production-Ready Self-Hosted File Sharing Platform

This project is a production-style self-hosted file sharing platform deployed on an AWS EC2 instance using Docker Compose.

The application is built using Nextcloud and includes PostgreSQL, Redis, Nginx reverse proxy, self-signed HTTPS, backup and restore scripts, monitoring with Prometheus and Grafana, and Docker log rotation.

Project Objective

The goal of this project is to gain hands-on experience with Docker and production-style infrastructure operations by deploying a real-world multi-container application.

This project focuses on:

Docker Compose
Container networking
Persistent storage
Reverse proxy configuration
HTTPS setup
Backup and restore
Monitoring
Logging
Git-based infrastructure documentation


Features Implemented
Nextcloud file sharing platform
PostgreSQL database container
Redis cache and file-locking container
Nginx reverse proxy
Self-signed HTTPS certificate
Docker named volumes for persistent storage
Separate Docker networks for frontend, backend, and monitoring
Backup script for PostgreSQL and Nextcloud data
Restore testing performed
Prometheus and Grafana monitoring stack
Host metrics using Node Exporter
Container metrics using cAdvisor
Docker log rotation
Environment-based configuration using .env
Git-tracked infrastructure files

How to Start the Stack

Run:

docker compose up -d

Check running containers:

docker ps

Expected containers:

nginx
nextcloud
postgres
redis
prometheus
grafana
cadvisor
node-exporter

Access URLs

Nextcloud:

https://EC2_PUBLIC_IP

Grafana:

http://EC2_PUBLIC_IP:3000

Because this project currently uses a self-signed certificate, the browser will show a certificate warning when accessing Nextcloud over HTTPS.


Useful Commands

Check containers:

docker ps

Check logs:

docker logs nginx --tail 100
docker logs nextcloud --tail 100
docker logs postgres --tail 100
docker logs redis --tail 100

Follow logs live:

docker logs -f nginx
docker logs -f nextcloud

Check Docker disk usage:

docker system df

Check host disk usage:

df -h

Check container resource usage:

docker stats

Validate Docker Compose file:

docker compose config

Stop the stack:

docker compose down

Start the stack:

docker compose up -d


Project Directory Structure
file-sharing-platform/
|
├── compose.yaml
├── README.md
├── .env.example
├── .gitignore
|
├── nginx/
│   ├── default.conf
│   └── certs/              # ignored from Git
|
├── monitoring/
│   └── prometheus.yml
|
├── scripts/
│   └── backup.sh
|
├── backups/                # backup files ignored from Git
│   └── .gitkeep
|
└── docs/
    ├── architecture.md
    ├── backup-restore.md
    └── operations-runbook.md

Learning Outcomes

This project helped me understand:

How Docker Compose manages multi-container applications
How Docker networks isolate services
How Docker volumes provide persistent storage
How reverse proxies work with containerized applications
How PostgreSQL and Redis support real-world applications
How to configure self-signed HTTPS
How to create and test backups
How restore operations work
How to monitor containers and host resources
How to document infrastructure projects professionally


Final Summary

This project demonstrates a production-style containerized file sharing platform deployed on AWS EC2.

It includes application deployment, persistent storage, reverse proxy, HTTPS, monitoring, backup, restore testing, log rotation, and documentation.

The project is designed as a hands-on Docker learning project and can be extended further into a more production-ready architecture using AWS managed services such as RDS, ElastiCache, EFS, S3, and Application Load Balancer.


***END***
