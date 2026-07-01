Backup

A backup script is available at:

scripts/backup.sh

Run backup:

./scripts/backup.sh

The backup includes:

PostgreSQL database dump
Nextcloud data volume archive
Nginx certificate backup
Configuration reference files

Backups are stored under:

backups/<timestamp>/

Backup files are excluded from Git.

Restore

Restore testing was performed as part of this project.

The restore process includes:

1. Stop Docker Compose stack
2. Remove/recreate required Docker volumes
3. Start PostgreSQL and Redis
4. Restore PostgreSQL database
5. Restore Nextcloud data volume
6. Restore Nginx certificates
7. Start full stack
8. Disable maintenance mode
9. Validate login and uploaded files
