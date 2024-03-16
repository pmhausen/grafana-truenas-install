# grafana-truenas-install

Manual installation of Grafana and InfluxDB in a jail on TrueNAS CORE. This is aiming to provide a complete production ready setup.

## Prerequisites

- create a VNET jail on TrueNAS CORE
- make sure network settings are correct and the jail can connect to the Internet
- Log in to the jail as root either directly via SSH or via `iocage console <jail>` from the TrueNAS host

## Installation

### Install required packages

#### Configure `pkg` to use "latest" instead of "quarterly" repository

```sh
mkdir -p /usr/local/etc/pkg/repos
echo 'FreeBSD: { url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest" }' >/usr/local/etc/pkg/repos/FreeBSD.conf
pkg upgrade -y
pkg autoremove -y
```

#### Install packages

```sh
pkg install influxdb grafana
```

### Configure, enable and start InfluxDB

#### Create influxd configuration file

*[Complete separate config file](influxd.conf)*

Create `/usr/local/etc/influxd.conf` with this content:

```ini
[meta]
  dir = "/var/db/influxdb/meta"

[data]
  dir = "/var/db/influxdb/data"
  wal-dir = "/var/db/influxdb/wal"

[[graphite]]
  enabled = true
  database = "graphite"
  retention-policy = ""
  bind-address = ":2003"
  protocol = "tcp"
  consistency-level = "one"

  separator = "."

  templates = [
    "servers.* .hostname.resource.instance.measurement*",
  ]
```

This configures influxd with a single connector using the "Graphite plaintext protocol". TrueNAS CORE and e.g. OPNsense can readily send metrics to influxd using this protocol.

#### Enable and start the influxd server

```sh
sysrc influxd_enable=YES
service influxd start
```

---
EOF
