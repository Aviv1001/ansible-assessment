# webstack

Preps a host for a containerized web service: packages, app user + dir, config, git clone, cron, nginx container + health check.

Needs ansible-core 2.14+, `ansible-galaxy collection install community.docker`, docker on the control node (container tasks delegate to localhost). Target needs python3 + python3-apt. Tested on Debian 12, dnf path untested.

## Variables (defaults)

- app_user: appsvc — owns the app files
- app_dir: /opt/webapp
- web_port: 8088 — nginx host port
- web_image: nginx:alpine
- web_container: webstack-web
- web_packages: git,cron
- app_secret — required, vaulted in inventory/group_vars/web/vault.yml

## Example

```yaml
- hosts: web
  roles:
    - webstack
```

## Idempotency & limitations

- 2nd run is changed=0 (apt cache_valid_time, changed_when: false, stat guard on the marker)
- host_key_checking off: fine for a local lab container with no ssh, not for prod (MITM)
- target container has no systemd/docker → cron via sysv fallback, nginx + uri delegated to localhost. nginx serves its default page
- --check on a fresh host dies at the git clone (apt only pretended to install git); clean once converged
