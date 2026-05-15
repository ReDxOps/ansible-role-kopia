# Ansible Role: Kopia Backup

An automated, robust, and secure backup solution using **Kopia** with dynamic multi-database discovery (MariaDB, MySQL, PostgreSQL) and S3 replication.

## Features

- **Hybrid Execution**: One-shot backup jobs with a persistent UI server for management.
- **Dynamic DB Discovery**: Automatically detects and dumps containers based on labels.
- **Systemd Orchestration**: Controlled by `systemd` timers with clean logging and status.
- **S3 Connectivity**: Native compression (zstd) and encryption sent to any S3-compatible storage.
- **Traefik Integration**: Optional native support for Traefik with automatic TLS.
- **Secure by Design**: Runs with `no-new-privileges`, resource limits, and healthchecks.
- **FUSE Restore**: Mount your backups as a local filesystem for easy recovery.

## Role Variables

All variables are defined in [defaults/main.yml](defaults/main.yml).

### General Configuration
| Variable | Default | Description |
|----------|---------|-------------|
| `kopia_service_name` | `kopia` | Base name for containers and folders |
| `kopia_root_dir` | `/opt/kopia` | Root directory for config and scripts |
| `kopia_image_tag` | `0.22.3` | Kopia docker image version |
| `kopia_timezone` | `Europe/Paris` | Container timezone |

### S3 Backend & Auth
| Variable | Default | Description |
|----------|---------|-------------|
| `kopia_password` | `required` | Master password for the repository |
| `kopia_repository_username` | `required` | Kopia client owner name |
| `kopia_s3_bucket_name` | `required` | Your S3 bucket name |
| `kopia_s3_endpoint_url` | `required` | Your S3 endpoint API |

### Database Discovery
| Variable | Default | Description |
|----------|---------|-------------|
| `kopia_container_label` | `kopia.backup.target=true` | Label used to identify DB containers |
| `kopia_cleanup_sqldumps_after_backup` | `true` | Delete local SQL dumps after upload |

### UI & Ingress (Traefik)
| Variable | Default | Description |
|----------|---------|-------------|
| `kopia_traefik_enabled` | `false` | Enable Traefik labels & network |
| `kopia_traefik_domain` | `kopia.example.com` | Domain for the Kopia UI |
| `kopia_web_port_expose` | `51515` | Direct port if Traefik is disabled |

### Backup Targets & Retention
| Variable | Default | Description |
|----------|---------|-------------|
| `kopia_folders_to_save` | `['/opt/app']` | List of host directories to backup |
| `kopia_policy_keep_daily` | `7` | Number of daily snapshots to keep |
| `kopia_policy_keep_weekly` | `4` | Number of weekly snapshots to keep |
| `kopia_policy_keep_monthly` | `6` | Number of monthly snapshots to keep |
| `kopia_policy_compression` | `zstd` | Compression algorithm (zstd, s2-default, none) |

### Scheduling & Exclusions
| Variable | Default | Description |
|----------|---------|-------------|
| `kopia_backup_execution_timer` | `*-*-* 03:00:00` | Backup frequency (Systemd timer syntax) |
| `kopia_ignore_custom` | `[]` | Custom patterns to exclude from backups |

### Notifications (SMTP)
| Variable | Default | Description |
|----------|---------|-------------|
| `kopia_email_report_enabled` | `false` | Send email report after backup |
| `kopia_email_report_to` | `admin@example.com` | Destination email address |
| `kopia_email_from` | `kopia@example.com` | Source email address |
| `kopia_smtp_host` | `smtp.example.com` | SMTP server address |
| `kopia_smtp_port` | `587` | SMTP server port |

## Database Auto-Discovery

The role automatically detects and dumps databases from containers labeled with `kopia_container_label`.

**Example for a PostgreSQL container:**
```yaml
services:
  db:
    image: postgres:18
    labels:
      - "kopia.backup.target=true"
```
*Note: The internal backup script supports PostgreSQL, MySQL, and MariaDB.*

## Kopia UI

If `kopia_traefik_enabled` is `true`, you can access the UI at `https://{{ kopia_traefik_domain }}`.
Otherwise, use `http://<server-ip>:{{ kopia_web_port_expose }}`.

## Restoration (FUSE Mount)

Kopia allows you to mount your snapshots as a local filesystem for granular recovery. To use this, start the `restore` profile:

```bash
# Navigate to your kopia root
cd /opt/kopia

# Start the restore service
docker-compose --profile restore up -d

# Browse your backups
ls -lh /mnt/kopia-restore
```

## Example Playbook

```yaml
- hosts: servers
  become: true
  roles:
    - role: redxops.kopia
      vars:
        kopia_s3_bucket_name: "my-backup-bucket"
        kopia_password: "vaulted_password"
        kopia_folders_to_save:
          - "/opt/my-app"
```

## License

MIT

## Author Information

Created in 2026 by **ReDxOps**.
