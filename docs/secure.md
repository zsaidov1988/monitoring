[Main menu](../README.md)

## How to make secure connection between Node-exporter and Prometheus

### TLS

```
openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout /etc/node_exporter/node_exporter.key -out /etc/node_exporter/node_exporter.crt -subj "/C=BE/ST=Antwerp/L=Brasschaat/O=Inuits/CN=localhost" -addext "subjectAltName = DNS:localhost"

/etc/node_exporter/config.yml

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

./node_exporter --web.config.file=/etc/node_exporter/config.yml
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
      ca_file: /etc/prometheus/node_exporter.crt
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

[Main menu](../README.md)