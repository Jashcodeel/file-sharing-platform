# Production-Ready Self-Hosted File Sharing Platform

This project deploys a self-hosted file sharing platform using Docker Compose on an AWS EC2 instance.

## Current Components

- PostgreSQL database with persistent Docker volume
- Redis cache with password authentication
- Nextcloud application container
- Docker Compose networking
- Health checks for PostgreSQL and Redis
- Environment-based configuration using `.env`

## Current Access

Nextcloud is temporarily exposed on port 8080 for testing:

http://EC2_PUBLIC_IP:8080

Later, Nginx reverse proxy and HTTPS will be added.
