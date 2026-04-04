# Ansible Role: Kopia Backup

An automated, robust, and secure backup solution using **Kopia** with dynamic multi-database discovery (MariaDB, MySQL, PostgreSQL) and S3 replication.

## Features

- **One-Shot Docker Execution**: Low overhead, no daemon required for backup jobs.
- **Dynamic DB Discovery**: Automatically detects and dumps containers based on suffixes.
- **Systemd Orchestration**: Controlled by `systemd` timers with clean logging and status.
- **S3 Connectivity**: Native compression (zstd) and encryption sent to any S3-compatible storage (Cloudflare R2, AWS S3, etc.).
- **Notifications**: Optional automated email reports via SMTP.
- **Community Ready**: Fully portable and independent of specific infrastructure names.

## Requirements

- Ansible 2.10 or higher.
- Docker installed on the target host (recommended: `geerlingguy.docker`).

## Role Variables

All variables are defined in [defaults/main.yml](defaults/main.yml).

### General Configuration
| Variable | Default | Description |
|----------|---------|-------------|
| `kopia_service_name` | `kopia` | Base name for containers and folders |
| `kopia_root_dir` | `/opt/talos/kopia` | Root directory for config and scripts |
| `kopia_image_tag` | `0.15.0` | Kopia docker image version |

### S3 Backend
| Variable | Type | Description |
|----------|------|-------------|
| `kopia_s3_bucket_name` | `string` | Your S3 bucket name |
| `kopia_s3_endpoint_url` | `string` | Your S3 endpoint API |
| `kopia_s3_access_key_id` | `vault` | S3 credentials |
| `kopia_s3_access_key_secret` | `vault` | S3 credentials |

### Retention Policy
| Variable | Default | Description |
|----------|---------|-------------|
| `kopia_policy_keep_daily` | `7` | Keep 7 daily snapshots |
| `kopia_policy_keep_weekly` | `4` | Keep 4 weekly snapshots |
| `kopia_policy_keep_monthly` | `6` | Keep 6 monthly snapshots |

## Dependencies

- `geerlingguy.docker` (optional, for docker setup)

## Example Playbook

```yaml
- hosts: servers
  become: true
  roles:
    - role: ReDxOps.kopia
      vars:
        kopia_s3_bucket_name: "my-backup-bucket"
        kopia_folders_to_save:
          - "/opt/my-app"
          - "/home/user/data"
```

## License

MIT

## Author Information

This role was created in 2026 by **ReDxOps**.
