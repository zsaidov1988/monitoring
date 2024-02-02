# Monitoring

## Install Prometheus & Node Exporter

[Download Prometheus](https://prometheus.io/download/)

```
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus
sudo cp prometheus.yml /etc/prometheus/prometheus.yml
sudo mkdir /var/lib/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
sudo cp prometheus /usr/local/bin
sudo cp promtool /usr/local/bin
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo cp -r consoles /etc/prometheus
sudo cp -r console_libraries /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus
```

![Unit file](images/image_unit_prometheus.png)
/etc/systemd/system/prometheus.service
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=promethes
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config-file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
![Unit file of node exporter](images/image_node_exporter.png)

## Prometheus configuration

![Alt text](images/image_prometheus_config.png)

## How to make secure connection between Node-exporter and Prometheus

### TLS

```
sudo openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout node_exporter.key -out node_exporter.crt -subj "/C=Tashkent/ST=Uacademy/L=Udevs/O=Proxima/CN=localhost" -addext "subjectAltName = DNS:localhost"
vi config.yml

tls_server_config:
  cert_file: node_exporter.crt
  key_file: node_exporter.key
```

Move certificate and config file to /etc/node_exporter
```
sudo mkdir /etc/node_exporter
mv node_exporter.* /etc/node_exporter
sudo cp config.yaml /etc/node_exporter
chown -R node_exporter:node_exporter /etc/node_exporter

./node_exporter --web.config=config.yml
```

#### Check connection

```
curl https://localhost:9100/metrics # Error beradi. Self-signed sert bo'gani uchun
curl -k https://localhost:9100/metrics # Allow insecure connection
```

#### Copy certificate from node_exporter to Prometheus server
```
scp node_exporter:/node_exporter.crt prometheus:/etc/prometheus
chown -R prometheus:prometheus /etc/prometheus
vim /etc/prometheus/prometheus.yml
```
/etc/prometheus/prometheus.yml
```
scrape_configs:
  - job_name: "node"
    scheme: https
    tls_config:
      ca_file: /etc/prometheus/prometheus.yml
      insecure_skip_verify: true # self-signed bo'lgani uchun
```
### Authentication
Generate encrypted password
```
sudo apt install apache2-utils
htpasswd -nBC 12 "" | tr -d ':\n'
```
/etc/node_exporter/config.yml
```
basic_auth_users:
  prometheus: $12fdfgdfg...
```
Prometheus configuration
/etc/prometheus/prometheus.yml
```
- job_name: "node"
   basic_auth:
     username: prometheus
     password: nimadir # Encrypt qilinmagan password
```
## Metric types
![Metrics](images/image_metric_types.png)
## Counter
![Counter](images/image_counter.png)
## Gauge
![Gauge](images/image_gauge.png)
## Histogram
![Histogram](images/image_histogram.png)
## Summary
![Summary](images/image_summary.png)
## Labels
![Labels](images/image_labels.png)