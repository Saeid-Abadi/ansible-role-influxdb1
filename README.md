# InfluxDB1

Initial installation and configuration of InfluxDB v1.

## Supported Platforms

| OS family | Distribution | Package source |
|-----------|--------------|----------------|
| Debian    | Debian, Ubuntu | Distribution repository (`apt`) |
| RedHat    | RHEL, CentOS, Rocky Linux, AlmaLinux (8 / 9) | Official InfluxData YUM repository |

The role auto-detects `ansible_os_family` and runs the matching install path. On RHEL-based systems the official InfluxData repository is added automatically (controlled by `influxdb_manage_repo`), because InfluxDB v1 is not available in the default RHEL/CentOS repositories.

---

## Role Layout

```
tasks/
  main.yml              # Orchestrator: asserts OS, includes install/configure/database
  install-Debian.yml    # apt-based install
  install-RedHat.yml    # yum_repository + package install
  configure.yml         # Directories, config template, service enable/start
  database.yml          # Admin user, database, db-user creation
vars/
  Debian.yml            # Debian/Ubuntu package + service names, prereqs
  RedHat.yml            # RHEL/CentOS repo URLs, package + service names, prereqs
```

---

## Directory Paths

| Variable | Default | Description |
|---------|---------|-------------|
| `influxdb_db_dir` | `/var/lib/influxdb` | Base directory for InfluxDB data, WAL, and metadata. |
| `influxdb_conf_dir` | `/etc/influxdb` | Directory where `influxdb.conf` will be written. |
| `influxdb_log_dir` | `/var/log/influxdb` | Directory for InfluxDB logs. |
| `influxdb_user` | `influxdb` | Owner of config and data files. |
| `influxdb_group` | `influxdb` | Group of config and data files. |

---

## Network & Binding

| Variable | Default | Description |
|---------|---------|-------------|
| `influxdb_bind_address` | `0.0.0.0` | Address for HTTP and internal services. |
| `influxdb_port` | `8086` | InfluxDB HTTP port. |
| `influxdb_api_host` | `127.0.0.1` when bind is `0.0.0.0`, else `influxdb_bind_address` | Endpoint used by the role for local API calls (auth check, user/db creation). |

---

## Authentication

| Variable | Default | Description |
|---------|---------|-------------|
| `influxdb_admin_user` | `admin` | Admin username. |
| `influxdb_admin_pw` | `admin` | Admin password. |
| `influxdb_basic_auth` | `true` | Enables HTTP Basic Auth. |

---

## TLS / HTTPS Configuration

| Variable | Default | Description |
|---------|---------|-------------|
| `tls_cert_path` | `""` | Path to TLS certificate. |
| `tls_key_path` | `""` | Path to private key. |
| `influxdb_https` | `false` | Enables HTTPS listener. |

---

## Repository & Client

| Variable | Default | Description |
|---------|---------|-------------|
| `influxdb_manage_repo` | `true` | (RHEL/CentOS only) Add the official InfluxData YUM repository. Set to `false` if you ship the repo via your own channel. |
| `influxdb_python_client` | `influxdb` | PyPI package name for the Python client used by `community.general.influxdb_*` modules. Installed via `pip`. |

---

## InfluxDB Configuration Template (`influxdb_config`)

```yaml
influxdb_config: |
  reporting-disabled = true
  bind-address = "{{ influxdb_bind_address }}:8088"

  [meta]
    dir = "{{ influxdb_db_dir }}/meta"

  [data]
    dir = "{{ influxdb_db_dir }}/data"
    wal-dir = "{{ influxdb_db_dir }}/wal"

  [coordinator]
  [retention]
  [shard-precreation]
  [monitor]

  [http]
    auth-enabled = {{ influxdb_basic_auth | string | lower }}
    bind-address = "{{ influxdb_bind_address }}:8086"
    https-enabled = {{ influxdb_https | string | lower }}
    https-certificate = "{{ tls_cert_path }}"
    https-private-key = "{{ tls_key_path }}"
    access-log-path = "{{ influxdb_log_dir }}/access.log"

  [ifql]
  [logging]
  [subscriber]

  [[graphite]]
  [[collectd]]
  [[opentsdb]]
  [[udp]]

  [tls]
    min-version = "tls1.2"
    max-version = "tls1.2"
    ciphers = [
      "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
      "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384",
      "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256",
      "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384"
    ]
```

---

## Database Creation Variables

| Variable | Default | Description |
|---------|---------|-------------|
| `influxdb_db_user` | `icinga` | Username for the database. |
| `influxdb_db_pw` | `icinga` | Password for the database user. |
| `influxdb_db_name` | `icinga` | Database name to be created. |
| `influxdb_db_priviliges` | `ALL` on `influxdb_db_name` | Privileges assigned to the user for the database. |

---

## Example Playbook

```yaml
- hosts: influxdb
  become: true
  roles:
    - role: influxdb1
      vars:
        influxdb_admin_pw: "{{ vault_influxdb_admin_pw }}"
        influxdb_db_name: metrics
        influxdb_db_user: metrics_rw
        influxdb_db_pw: "{{ vault_influxdb_db_pw }}"
```

---

## Testing

The role ships with a [Molecule](https://ansible.readthedocs.io/projects/molecule/) scenario that runs the role against Docker containers of every supported OS and verifies that the service is running, the HTTP API answers, and the configured database and user were created.

### Tested platforms

| Platform | Image |
|----------|-------|
| Debian 12 | `geerlingguy/docker-debian12-ansible` |
| Ubuntu 22.04 | `geerlingguy/docker-ubuntu2204-ansible` |
| Rocky Linux 9 (RHEL 9 / AlmaLinux 9) | `geerlingguy/docker-rockylinux9-ansible` |
| Rocky Linux 8 (RHEL 8 / AlmaLinux 8) | `geerlingguy/docker-rockylinux8-ansible` |

### Run locally

```bash
# Install test dependencies (once)
pip install 'ansible-core>=2.15' 'molecule>=6' 'molecule-plugins[docker]' docker
ansible-galaxy collection install community.general community.docker ansible.posix

# Full test cycle across all platforms (create → converge → idempotence → verify → destroy)
molecule test

# Run against a single OS only
molecule test --platform-name rockylinux9

# Iterative workflow — keeps containers alive between runs
molecule converge                # apply the role
molecule verify                  # run assertions
molecule login --host debian12   # shell into a container
molecule destroy                 # tear down
```

Docker (or Podman with the `DOCKER_HOST` shim) must be running locally. On macOS use Docker Desktop, Colima, or OrbStack.

### CI

`.github/workflows/molecule.yml` runs on every push to `main` and on pull requests:
- `lint` job — `yamllint` and `ansible-lint` (production profile)
- `molecule` job — matrix over the four distros, each one goes through the full `molecule test` cycle

