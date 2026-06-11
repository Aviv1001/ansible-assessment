# evidence

Target = docker container (no passwordless sudo here):

```
docker run -d --name webstack-target python:3.11-slim sleep infinity
docker exec webstack-target sh -c 'apt-get update && apt-get install -y python3-apt'
```

ansible [core 2.16.3]

```
$ ansible web -m ping
webstack-target | SUCCESS => { "ping": "pong" }

$ ansible-playbook site.yml --syntax-check
playbook: site.yml

$ ansible-playbook site.yml --check     # fresh host: fails at the git clone,
                                        # apt only simulated installing git

$ ansible-playbook site.yml
webstack-target : ok=15  changed=11  failed=0  skipped=1

$ ansible-playbook site.yml
webstack-target : ok=13  changed=0   failed=0  skipped=2    # idempotent

$ curl -s -o /dev/null -w '%{http_code}' http://127.0.0.1:8088
200
```

skipped = dnf (RedHat guard) + deploy marker (stat guard).

tree: ansible.cfg, site.yml, inventory/(hosts.ini, group_vars/web/vault.yml), roles/webstack/(defaults, tasks, handlers, meta, README.md)
