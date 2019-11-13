# Prometheus (Server) Setup

## Prepare Environment

1. Create prometheus user, without login priviledges.

```
sudo useradd --no-create-home --shell /usr/sbin/nologin prometheus
```

2. Create the folders required to store binaries of Prometheus and configuration files

```
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

3. Set ownership of these directories to `prometheus` user.

```
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

## Prometheus (Server) Setup

1. Download and unpack prometheus

```
wget https://github.com/prometheus/prometheus/releases/download/v2.14.0/prometheus-2.14.0.linux-amd64.tar.gz
tar xfz prometheus-*.tar.gz
cd prometheus-*
```

2. Copy binries to `/usr/local/bin` directory

```
sudo cp ./prometheus /usr/local/bin/
sudo cp ./promtool /usr/local/bin/
```

3. Set ownership of these files to the `prometheus` user

```
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

4. Copy the `consoles` and `console_libraries` directories to `/etc/prometheus`

```
sudo cp -r ./consoles /etc/prometheus
sudo cp -r ./console_libraries /etc/prometheus
```

5. Set ownership of these directories, recursively to the `prometheus` user

```
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

6. Cleanup install files

```
cd .. && rm -rf prometheus-*
```

## Configuring Prometheus Server

1. Open the `prometheus.yml` in a text editor

```
sudo vi /etc/prometheus/prometheus.yml
```

2. Add a basic config, including the `node_exporter` to the scraping rules

```
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100']
```

3. Set ownership of file to prometheus

```
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

## Prometheus Server - Running

1. Run prometheus directly from command line to ensure setup was successful

```
sudo -u prometheus /usr/local/bin/prometheus --config.file /etc/prometheus/prometheus.yml --storage.tsdb.path /var/lib/prometheus/ --web.console.templates=/etc/prometheus/consoles --web.console.libraries=/etc/prometheus/console_libraries
```

The server starts displaying multiple status messages and the information that the server has started:

```
$ sudo -u prometheus /usr/local/bin/prometheus --config.file /etc/prometheus/prometheus.yml --storage.tsdb.path /var/lib/prometheus/ --web.console.templates=/etc/prometheus/consoles --web.console.libraries=/etc/prometheus/console_libraries
level=info ts=2019-11-11T22:59:30.889Z caller=main.go:296 msg="no time or size retention was set so using the default time retention" duration=15d
level=info ts=2019-11-11T22:59:30.889Z caller=main.go:332 msg="Starting Prometheus" version="(version=2.14.0, branch=HEAD, revision=edeb7a44cbf745f1d8be4ea6f215e79e651bfe19)"
level=info ts=2019-11-11T22:59:30.889Z caller=main.go:333 build_context="(go=go1.13.4, user=root@df2327081015, date=20191111-14:27:12)"
level=info ts=2019-11-11T22:59:30.890Z caller=main.go:334 host_details="(Linux 3.10.0-862.3.3.el7.x86_64 #1 SMP Wed Jun 13 05:44:23 EDT 2018 x86_64 RTVAURCMESERV02 (none))"
level=info ts=2019-11-11T22:59:30.890Z caller=main.go:335 fd_limits="(soft=1024, hard=4096)"
level=info ts=2019-11-11T22:59:30.890Z caller=main.go:336 vm_limits="(soft=unlimited, hard=unlimited)"
level=info ts=2019-11-11T22:59:30.893Z caller=web.go:496 component=web msg="Start listening for connections" address=0.0.0.0:9090
level=info ts=2019-11-11T22:59:30.893Z caller=main.go:657 msg="Starting TSDB ..."
level=info ts=2019-11-11T22:59:30.899Z caller=head.go:535 component=tsdb msg="replaying WAL, this may take awhile"
level=info ts=2019-11-11T22:59:30.899Z caller=head.go:583 component=tsdb msg="WAL segment loaded" segment=0 maxSegment=0
level=info ts=2019-11-11T22:59:30.902Z caller=main.go:672 fs_type=XFS_SUPER_MAGIC
level=info ts=2019-11-11T22:59:30.902Z caller=main.go:673 msg="TSDB started"
level=info ts=2019-11-11T22:59:30.902Z caller=main.go:743 msg="Loading configuration file" filename=/etc/prometheus/prometheus.yml
level=info ts=2019-11-11T22:59:30.918Z caller=main.go:771 msg="Completed loading of configuration file" filename=/etc/prometheus/prometheus.yml
level=info ts=2019-11-11T22:59:30.918Z caller=main.go:626 msg="Server is ready to receive web requests."
```

2. Create prometheus as a service

```
sudo vi /etc/systemd/system/prometheus.service
```

3. Resart `systemctl` to use new service

```
sudo systemctl daemon-reload
```

4. Enable service so it automatically starts during boot

```
sudo systemctl enable prometheus
```

5. Start prometheus service

```
sudo systemctl start prometheus
```

## Installing Graphana (Server)

1. Add repository information

```
cat <<EOF | sudo tee /etc/yum.repos.d/grafana.repo
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
EOF
```

2. Install Graphana

```
sudo yum -y install grafana
```

3. View package information

```
 rpm -qi grafana 
Name        : grafana
Version     : 6.4.4
Release     : 1
Architecture: x86_64
Install Date: Mon 11 Nov 2019 23:14:21 UTC
Group       : default
Size        : 172474341
License     : "Apache 2.0"
Signature   : RSA/SHA1, Wed 06 Nov 2019 14:02:47 UTC, Key ID 8c8c34c524098cb6
Source RPM  : grafana-6.4.4-1.src.rpm
Build Date  : Wed 06 Nov 2019 14:00:33 UTC
Build Host  : 38c564474d0d
Relocations : / 
Packager    : contact@grafana.com
Vendor      : Grafana
URL         : https://grafana.com
Summary     : Grafana
Description :
Grafana
```

4. Start graphana service, configure to be enabled from boot

```
sudo systemctl enable --now grafana-server.service 
```

5. Verify `status`

```
systemctl status grafana-server.service
```

6. Access dashboard

```
HOSTNAME:3000 admin/!S0ft....@g!
```

## Prometheus (Node) Setup

1. Add `node_exporter` user with limited priviledges

```
sudo useradd --no-create-home --shell /bin/false node_exporter
```

2. Download and install node explorer from go-enabled environment

```
wget --no-check-certificate https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
```

3. Unpack downloaded archive

```
tar xvf node_exporter-*
```

4. Copy the binary file to the directory `/usr/local/bin` and set ownership to `node_exporter` user 

```
sudo cp node_exporter-*/node_exporter /usr/local/bin
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

5. Remove leftover file(s)

```
rm -rf node_exporter-*
```

6. Setup node_explorer to run automatically on each boot

```
sudo vi /etc/systemd/system/node_exporter.service
```

7. Copy the following to the `*.service` file

```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

8. Reload systemd

```
sudo systemctl daemon-reload
```

9. Start node explorter service

```
sudo systemctl start node_exporter
```

10. Verify software has been started successfully

```
sudo systemctl status node_exporter
```

11. If everything looks okay, `enable` service to run on each bootup

```
sudo systemctl enable node_exporter
```


# References:

- https://prometheus.io/docs/prometheus/latest/getting_started/
- https://www.scaleway.com/en/docs/configure-prometheus-monitoring-with-grafana/
