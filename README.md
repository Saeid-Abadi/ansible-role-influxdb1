# InfluxDB1
Initial installation and configuration of InfluxDB v1. Currently, only Debian-based systems are supported.

# InfluxDB Ansible Role — Variables Reference

This document provides an overview of all variables used to configure an InfluxDB 1.x instance with this Ansible role.

---

## 📂 Directory Paths

| Variable | Default | Description |
|---------|---------|-------------|
| `influxdb_db_dir` | `/var/lib/influxdb` | Base directory for InfluxDB data, WAL, and metadata. |
| `influxdb_conf_dir` | `/etc/influxdb` | Directory where `influxdb.conf` will be written. |
| `influxdb_log_dir` | `/var/log/influxdb` | Directory for InfluxDB logs. |

---

## 🌐 Network & Binding

| Variable | Default | Description |
|---------|---------|-------------|
| `influxdb_bind_address` | `0.0.0.0` | Address for HTTP and internal services. |
| `influxdb_port` | `8086` | InfluxDB HTTP port. |

---

## 🔐 Authentication

| Variable | Default | Description |
|---------|---------|-------------|
| `influxdb_admin_user` | `admin` | Admin username. |
| `influxdb_admin_pw` | `admin` | Admin password. |
| `influxdb_basic_auth` | `true` | Enables HTTP Basic Auth. |

---

## 🔒 TLS / HTTPS Configuration

| Variable | Default | Description |
|---------|---------|-------------|
| `tls_cert_path` | `""` | Path to TLS certificate. |
| `tls_key_path` | `""` | Path to private key. |
| `influxdb_https` | `false` | Enables HTTPS listener. |

---

## ⚙️ InfluxDB Configuration Template (`influxdb_config`)

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

