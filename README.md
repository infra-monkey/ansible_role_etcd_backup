# Overview
This role does configures daily backups of etcd data.
This role requires root permissions. It must be called as root. This needs to be managed at the ansible or playbook level.

>Backups are stored in {{ backup_dir }}/{{ ansible_fqdn }}/etcd/date

>Use `etcdctl` to restore those backups. Make sure to set the appropriate options.

>Webhook notifications only support rocketchat for the moment.

# Variables

| Name  | Type | Required | Default Value | Description |
| ----- | ---- | -------- | ------------- | ----------- |
| etcdctl_path | string | no | `/usr/local/bin/etcdctl` | The path to etcdctl. |
| etcd_backup_options | string | no | `""` | Extra options to pass to `etcdctl`. |
| etcd_backup_compress | boolean | no | `True` | Compress the snapshot as a tar.gz. |
| etcd_backup_secure | boolean | no | `True` | Use a tls to connect to etcd endpoint. |
| etcd_backup_cacert | string | no | `""` | Path to the certificate authority trusted for the etcd certificates. |
| etcd_backup_cert | string | no | `""` | Path to the certificate used to connect to etcd endpoints. |
| etcd_backup_key | string | no | `""` | Path to the certificate key used to connect to etcd endpoints. |
| etcd_backup_user | string | no | `root` | User that runs the backup script. Need read permissions on the certificates. The user must exist. |
| etcd_backup_group | string | no | `root` | Group that runs the backup script. Need read permissions on the certificates. The user must exist. |
| etcd_backup_mount_type | string | no | `local` | Type of storage that will hold the backup files. Supported types: local, nfs |
| etcd_backup_dir | string | no | `/tmp/etcd_backup` | Path where the backups are sent. Is the mount point in case of network storage. |
| etcd_backup_remote_mount_path | string | no | `nfsserver:/path/to/mount` | The remote path of the mount command. Depends on the protocol. |
| etcd_backup_retention | string | no | `14` | The default number of backups to keep. |
| etcd_backup_periodicity | string | no | `OnCalendar=*-*-* 22:00:00` | The default periodicity of backups (every night at 10pm). Systemd timer format. |
| etcd_backup_webhook_notification | boolean | no | `false` | Send the result of the backup at the end of execution |
| etcd_backup_webhook_url | string | no | n.a. | The url to send the payload to |
| etcd_backup_webhook_script | string | no | `/usr/local/bin/webhook-notify.sh`| The path of the webhook script to call (the default value is set for infra_monkey.webhook_notify galaxy role) |


# Example of inventory variables

    etcd_backup_mount_type: "nfs"
    etcd_backup_dir: "/mnt/backup"
    etcd_backup_remote_mount_path: "192.168.25.2:/exports/backups"
    etcd_backup_periodicity: "OnCalendar=*-*-* 1:00:00"


# Reqirements

This role uses the collection based ansible modules which requires:
- ansible >= 2.10
This role depends on the ansible collections:
- ansible.builtin
- ansible.posix

Default webhook script from ansible galaxy (you can use your own script):
- infra_monkey.webhook_notify

The role configures **systemd timers**, if your host doesn't use systemd, it will fail.

# Automatique Testing

This role is tested using Molecule against:
- Python 3.7 and 3.8
- CentOS 7/8
- Debian 9/10