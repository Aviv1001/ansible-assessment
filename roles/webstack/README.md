# webstack

Preps a host for a containerized web service: packages, app user + dir, config, cron, nginx container + health check.

Needs ansible-core 2.14+, `ansible-galaxy collection install community.docker`, docker on the control node (container tasks delegate to localhost). Target needs python3 + python3-apt.

## Variables

- app_dir: /opt/webapp
- web_port: 8088 nginx host port
- web_container: webstack-web
- web_packages: git,cron
- app_user, app_secret is set in inventory (group var + vault in inventory/group_vars/web/vault.yml)

## Example

```yaml
- hosts: web
  roles: [webstack]
```

## Idempotency & limitations

- 2nd run is changed=0 (changed_when: false on the version check, stat guard on the marker)
- no systemd/docker in the target container -> cron via sysv fallback; nginx serves its default page
