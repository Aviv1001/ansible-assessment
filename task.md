# Practical Assessment — Junior Ansible Automation Engineer

**Time limit:** ~45 minutes
**Format:** Hands-on. You will configure Ansible, build a role, document it, and prove it works against a target host.

---

## Scenario

Your team needs a reusable Ansible role called **`webstack`** that prepares a Linux host to run a small containerized web service. You will set up Ansible locally, define an inventory, build the role, document it, and run it against a working environment to confirm it converges.

You may target a local VM, a container, or `localhost` (with `connection: local`). Pick whatever working environment you have available — what matters is that the playbook **runs and reports success**.

---

## Environment & Rules

- Use Ansible (core 2.14+). Install it if it is not already present.
- Keep everything under a single project directory, e.g. `~/ansible-assessment/`.
- Your play must be **idempotent**: a second run should report `changed=0`.
- Prefer fully-qualified collection names (e.g. `ansible.builtin.copy`, `community.docker.docker_container`) where appropriate.
- If a module can't run in your environment (e.g. no Docker), note it in your docs and use `check_mode`/a guard — but explain your reasoning.

---

## Part 1 — Setup & Inventory (~8 min)

1. Install Ansible and confirm the version (`ansible --version`).
2. Create the project layout:
   ```
   ansible-assessment/
   ├── ansible.cfg
   ├── inventory/
   │   └── hosts.ini      (or hosts.yml)
   ├── site.yml
   └── roles/
       └── webstack/
   ```
3. Write an `ansible.cfg` that:
   - Points at your inventory by default.
   - Sets `roles_path = ./roles`.
   - Disables host key checking for the lab (and say *why* that's acceptable here but not in production, in your docs).
4. Define an inventory with at least one group (e.g. `[web]`) containing your target host, plus a group variable (e.g. `app_user: appsvc`).
5. Prove connectivity: `ansible web -m ansible.builtin.ping` (or `setup`) succeeds.

**Deliverable:** working `ansible.cfg`, inventory, and a successful connectivity check.

---

## Part 2 — Build the `webstack` Role (~22 min)

Create the role with `ansible-galaxy role init roles/webstack` and implement the tasks below. **You must use every module in the checklist at least once**, choosing a sensible task for each.

### Required behavior

1. **Detect the OS family** and install packages accordingly:
   - Use `ansible.builtin.apt` on Debian/Ubuntu and `ansible.builtin.dnf` on RHEL/Fedora, guarded with `when:` on `ansible_os_family`.
   - Use `ansible.builtin.package` for at least one OS-agnostic package (e.g. `git`, `curl`).
   - Use `ansible.builtin.pip` to install at least one Python package (e.g. `docker` or `requests`) needed by your automation.
2. **Create a service account** with `ansible.builtin.user` (use the `app_user` group var, create a home dir, set a shell).
3. **Lay down files and directories:**
   - `ansible.builtin.file` to create an app directory (owned by `app_user`, mode `0750`).
   - `ansible.builtin.copy` to drop a config file or static `index.html` into that directory.
   - `ansible.builtin.stat` to check whether a marker/config file already exists, and register the result.
4. **Conditional logic with stat:** use the registered `stat` result so a follow-up task only runs when the file is (or isn't) present.
5. **Pull application content** with `ansible.builtin.git` (clone a small public repo, or a local bare repo, into a work directory).
6. **Run commands the right way:**
   - `ansible.builtin.command` for a task that does **not** need a shell (e.g. checking a binary version) — capture output with `register`.
   - `ansible.builtin.shell` for a task that genuinely needs shell features (pipe/redirect/glob) — and add a `creates:` or `when:` guard so it stays idempotent.
7. **Containerized service:**
   - Use `community.docker.docker_container` (the `docker` module) to run a simple web container (e.g. `nginx`) exposing a port, OR — if Docker isn't available — install it via the package modules and document the limitation.
8. **Manage a service** with `ansible.builtin.service` (enable + start something real on the host, e.g. `docker`, `ssh`, or `cron`).
9. **Health check:** use `ansible.builtin.uri` to GET the web endpoint (the container or local service) and assert a `200` status.

### Module checklist (tick each as you use it)

- [ ] `user`
- [ ] `shell`
- [ ] `command`
- [ ] `service`
- [ ] `docker` (`community.docker.docker_container`)
- [ ] `copy`
- [ ] `file`
- [ ] `apt`
- [ ] `package`
- [ ] `dnf`
- [ ] `uri`
- [ ] `pip`
- [ ] `git`
- [ ] `stat`

### Role hygiene

- Put tunables (package list, app dir, port, repo URL) in `roles/webstack/defaults/main.yml`.
- Use at least one **handler** (e.g. restart the container/service when config changes).
- Use **tags** on logical groups of tasks (e.g. `packages`, `deploy`, `healthcheck`).

**Deliverable:** a complete `roles/webstack/` and a `site.yml` that applies it to the `web` group.

---

## Part 3 — Documentation (~7 min)

Write `roles/webstack/README.md` covering:

1. **What the role does** (one paragraph).
2. **Requirements** (Ansible version, collections — list them and the `ansible-galaxy collection install` command; target OS support).
3. **Role variables** — a table of every default in `defaults/main.yml` with description and default value.
4. **Example playbook** showing how to use the role.
5. **Idempotency & limitations** — note anything that won't run everywhere (e.g. Docker) and the host-key-checking decision from Part 1.

**Deliverable:** a clear, accurate `README.md` someone else could follow.

---

## Part 4 — Test on a Working Environment (~8 min)

1. **Syntax check:** `ansible-playbook site.yml --syntax-check`.
2. **Dry run:** `ansible-playbook site.yml --check` (note where check mode can't fully evaluate, e.g. registered command output).
3. **Apply:** `ansible-playbook site.yml` — capture the recap (`ok=`, `changed=`, `failed=0`).
4. **Prove idempotency:** run it a **second time** and confirm `changed=0`.
5. **Prove the service works:** show the `uri` health check passing, or `curl` the endpoint manually and paste the result.

**Deliverable:** terminal output (or a short log) of the syntax check, both runs, and the passing health check.

---

## What to Submit

- The full project directory tree (`tree` output is fine).
- `roles/webstack/README.md`.
- Console output proving: connectivity, first apply, idempotent second run, and a passing health check.

---

## Scoring (100 pts)

| Area | Points | What we look for |
|------|-------:|------------------|
| Setup & inventory | 15 | Clean `ansible.cfg`, grouped inventory, group vars, successful ping |
| Role correctness | 35 | All 14 modules used appropriately; correct `when:` OS guards |
| Idempotency | 15 | Second run is `changed=0`; proper `creates:`/`stat` guards |
| Role hygiene | 10 | Defaults, handlers, tags, FQCN usage |
| Documentation | 15 | Accurate README with variable table and examples |
| Testing & evidence | 10 | Syntax check, dry run, two applies, passing health check shown |

### Bonus (up to +10)

- Use `ansible-vault` for one sensitive variable.
- Add a `molecule` scenario or a `--check` CI-style guard.
- Use `block`/`rescue`/`always` for error handling around the Docker tasks.

---

### Hints

- `command` vs `shell`: reach for `command` first; only use `shell` when you need pipes, redirects, globs, or environment expansion.
- `package` is the generic wrapper; `apt`/`dnf` give you OS-specific options — the `when: ansible_os_family == "Debian"` / `"RedHat"` pattern keeps the role portable.
- `stat` + `register` + `when: not result.stat.exists` is the idiomatic "only do this once" guard.
- The `community.docker` collection is required for the docker module: `ansible-galaxy collection install community.docker`.
- A `uri` task with `status_code: 200` doubles as both a health check and an assertion.