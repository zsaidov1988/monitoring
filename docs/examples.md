[Main menu](../README.md)

## Monitoring example files

### Docker container

#### Unit files

/etc/systemd/system/alertmanager.service

```
[Unit]
Description=Docker Service for alertmanager
After=network.service docker.service
Requires=docker.service

[Service]
ExecStartPre=-/usr/bin/docker stop -t 60 alertmanager
ExecStartPre=-/usr/bin/docker rm alertmanager
ExecStart=/usr/bin/docker run \
--rm \
--name alertmanager \
--publish 9093:9093 \
-v /etc/victoriametrics/alertmanager.yml:/config/alertmanager.yml \
-v /etc/victoriametrics/default.tmpl:/config/default.tmpl \
-v /usr/share/zoneinfo/Asia/Tashkent:/etc/localtime \
--network docker_network \
prom/alertmanager:v0.25.0 \
--config.file=/config/alertmanager.yml 
ExecStop=-/usr/bin/docker stop -t 60 alertmanager

ExecReload=/usr/bin/docker restart 'alertmanager'

Restart=always
RestartSec=20s

SuccessExitStatus=SIGKILL SIGTERM 143 137

[Install]
WantedBy=multi-user.target
WantedBy=docker.service
```

/etc/systemd/system/grafana.service

```
[Unit]
Description=Docker Service for grafana
After=network.service docker.service
Requires=docker.service

[Service]
ExecStartPre=-/usr/bin/docker stop -t 60 grafana
ExecStartPre=-/usr/bin/docker rm grafana
ExecStart=/usr/bin/docker run \
--rm \
--name grafana \
--publish 3000:3000 \
-v grafanadata:/var/lib/grafana \
-v /etc/victoriametrics/provisioning/:/etc/grafana/provisioning/ \
--network docker_network \
grafana/grafana:9.2.7 
ExecStop=-/usr/bin/docker stop -t 60 grafana

ExecReload=/usr/bin/docker restart 'grafana'

Restart=always
RestartSec=20s

SuccessExitStatus=SIGKILL SIGTERM 143 137

[Install]
WantedBy=multi-user.target
WantedBy=docker.service
```

/etc/systemd/system/kuma.service

```
[Unit]
Description=Docker Service for kuma
After=network.service docker.service
Requires=docker.service

[Service]
ExecStartPre=-/usr/bin/docker stop -t 60 kuma
ExecStartPre=-/usr/bin/docker rm kuma
ExecStart=/usr/bin/docker run \
--rm \
--name kuma \
--publish 3003:3001 \
-v uptime-kuma:/app/data \
--network docker_network \
louislam/uptime-kuma:1

ExecReload=/usr/bin/docker restart 'kuma'

Restart=always
RestartSec=20s

SuccessExitStatus=SIGKILL SIGTERM 143 137

[Install]
WantedBy=multi-user.target
WantedBy=docker.service
```

/etc/systemd/system/loki.service

```
[Unit]
Description=Docker Service for loki
After=network.service docker.service
Requires=docker.service

[Service]
ExecStartPre=-/usr/bin/docker stop -t 60 loki
ExecStartPre=-/usr/bin/docker rm loki
ExecStart=/usr/bin/docker run \
--rm \
--name loki \
--publish 3100:3100 \
-v loki-data:/loki \
-v /etc/victoriametrics/local-config.yml:/etc/loki/local-config.yml \
--network docker_network \
grafana/loki:latest \
-config.file=/etc/loki/local-config.yml
ExecStop=-/usr/bin/docker stop -t 60 loki

ExecReload=/usr/bin/docker restart 'loki'

Restart=always
RestartSec=20s

SuccessExitStatus=SIGKILL SIGTERM 143 137

[Install]
WantedBy=multi-user.target
WantedBy=docker.service
```

/etc/systemd/system/node-exporter.service

```
[Unit]
Description=Docker Service for node-exporter
After=network.service docker.service
Requires=docker.service

[Service]
ExecStartPre=-/usr/bin/docker stop -t 60 node-exporter
ExecStartPre=-/usr/bin/docker rm node-exporter
ExecStart=/usr/bin/docker run \
--rm \
--name node-exporter \
--publish 9100:9100 \
-v /proc:/host/proc:ro \
-v /sys:/host/sys:ro \
-v /:/rootfs:ro \
--network docker_network \
prom/node-exporter:latest \
--path.procfs=/host/proc \
--path.sysfs=/host/sys \
--collector.filesystem.ignored-mount-points \
"^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)"
ExecStop=-/usr/bin/docker stop -t 60 node-exporter

ExecReload=/usr/bin/docker restart 'node-exporter'

Restart=always
RestartSec=20s

SuccessExitStatus=SIGKILL SIGTERM 143 137

[Install]
WantedBy=multi-user.target
WantedBy=docker.service
```

/etc/systemd/system/victoriametrics.service
```
[Unit]
Description=Docker Service for victoriametrics
After=network.service docker.service
Requires=docker.service

[Service]
ExecStartPre=-/usr/bin/docker stop -t 60 victoriametrics
ExecStartPre=-/usr/bin/docker rm victoriametrics
ExecStart=/usr/bin/docker run \
--rm \
--name victoriametrics \
--publish 8428:8428 \
--publish 8089:8089 \
--publish 8089:8089/udp \
--publish 2003:2003 \
--publish 2003:2003/udp \
--publish 4242:4242 \
-v vmdata:/storage \
--network docker_network \
victoriametrics/victoria-metrics:v1.89.1 \
--storageDataPath=/storage \
--graphiteListenAddr=:2003 \
--opentsdbListenAddr=:4242 \
--httpListenAddr=:8428 \
--influxListenAddr=:8089 \
--vmalert.proxyURL=http://vmalert:8880
ExecStop=-/usr/bin/docker stop -t 60 victoriametrics

ExecReload=/usr/bin/docker restart 'victoriametrics'

Restart=always
RestartSec=20s

SuccessExitStatus=SIGKILL SIGTERM 143 137

[Install]
WantedBy=multi-user.target
WantedBy=docker.service
```

/etc/systemd/system/vmagent.service

```
[Unit]
Description=Docker Service for vmagent
After=network.service docker.service
Requires=docker.service

[Service]
ExecStartPre=-/usr/bin/docker stop -t 60 vmagent
ExecStartPre=-/usr/bin/docker rm vmagent
ExecStart=/usr/bin/docker run \
--rm \
--name vmagent \
--publish 8429:8429 \
-v vmagentdata:/vmagentdata \
-v /etc/victoriametrics/prometheus.yml:/etc/prometheus/prometheus.yml \
--network docker_network \
victoriametrics/vmagent:v1.89.1 \
--promscrape.config=/etc/prometheus/prometheus.yml \
--promscrape.config.strictParse=false \
--remoteWrite.url=http://victoriametrics:8428/api/v1/write 
ExecStop=-/usr/bin/docker stop -t 60 vmagent

ExecReload=/usr/bin/docker restart 'vmagent'

Restart=always
RestartSec=20s

SuccessExitStatus=SIGKILL SIGTERM 143 137

[Install]
WantedBy=multi-user.target
WantedBy=docker.service
```

/etc/systemd/system/vmalert.service

```
[Unit]
Description=Docker Service for vmalert
After=network.service docker.service
Requires=docker.service

[Service]
ExecStartPre=-/usr/bin/docker stop -t 60 vmalert
ExecStartPre=-/usr/bin/docker rm vmalert
ExecStart=/usr/bin/docker run \
--rm \
--name vmalert \
--publish 8880:8880 \
-v /etc/victoriametrics/vmalert/alerts.yml:/etc/alerts/alerts.yml \
-v /usr/share/zoneinfo/Asia/Tashkent:/etc/localtime \
--network docker_network \
victoriametrics/vmalert:v1.89.1 \
--datasource.url=http://victoriametrics:8428/ \
--remoteRead.url=http://victoriametrics:8428/ \
--remoteWrite.url=http://victoriametrics:8428/ \
--notifier.url=http://alertmanager:9093/ \
--rule=/etc/alerts/*.yml \
--external.url=http://127.0.0.1:3000 \
--external.alert.source=explore?orgId=1&left=["now-1h","now","VictoriaMetrics",{"expr":{{$$expr|jsonEscape|queryEscape}} },{"mode":"Metrics"},{"ui":[true,true,true,"none"]}]
ExecStop=-/usr/bin/docker stop -t 60 vmalert

ExecReload=/usr/bin/docker restart 'vmalert'

Restart=always
RestartSec=20s

SuccessExitStatus=SIGKILL SIGTERM 143 137

[Install]
WantedBy=multi-user.target
WantedBy=docker.service
```

### Configuration files

/etc/victoriametrics/alertmanager.yml

```
route:
  group_wait: 1s
  group_interval: 5s
  repeat_interval: 5h
  group_by: []
  receiver: "test"
  routes:
    - match:
        job: "prod_app"
      receiver: "prod"
    - match:
        job: "prod"
      receiver: "prod"
receivers:
  - name: prod
    telegram_configs:
      - bot_token: hfghfgh
        send_resolved: true
        chat_id: hfgh
        api_url: https://api.telegram.org
        parse_mode: "HTML"
        message: '{{ template "telegram" .}}'
        
  - name: test
    telegram_configs:
      - bot_token: iuyittuyi
        send_resolved: true
        chat_id: uyity
        api_url: https://api.telegram.org
        parse_mode: "HTML"
        message: '{{ template "telegram" .}}'
        

templates:
  - "/config/default.tmpl"
```

/etc/victoriametrics/default.tmpl

```
{{ define "telegram" }}
    {{ if gt (len .Alerts.Firing) 0 }}
{{- len .Alerts.Firing }} alert(s)
        {{- range .Alerts.Firing }} 
❌ <u><b>{{ .Labels.alertname }}</b></u>
{{ .Annotations.message }}
{{ .Labels.job }}
{{ .Labels.instance }}
        {{ end }}
    {{ end }}
    {{ if gt (len .Alerts.Resolved) 0 }}
{{- len .Alerts.Resolved }} alert(s)
        {{- range .Alerts.Resolved }} 
✅<u><b>{{ .Labels.alertname }}</b></u> 
{{ .Labels.job }}
{{ .Labels.instance }}
{{ .StartsAt.Format "2006-01-02" }} - {{(.EndsAt.Sub .StartsAt).Truncate 1000000000}}
{{ .StartsAt.Format "15:04:05" }} - {{ .EndsAt.Format "15:04:05" }}
        {{ end }}
    {{ end }}
{{ end }}
```

/etc/victoriametrics/local-config.yml 

```
auth_enabled: false

server:
  http_listen_port: 3100

ingester:
  wal:
    enabled: false
  lifecycler:
    address: 127.0.0.1
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
    final_sleep: 0s
  chunk_idle_period: 5m
  chunk_retain_period: 30s
  max_transfer_retries: 0

schema_config:
  configs:
  - from: 2018-04-15
    store: boltdb
    object_store: filesystem
    schema: v11
    index:
      prefix: index_
      period: 168h

storage_config:
  boltdb:
    directory: /tmp/loki/index

  filesystem:
    directory: /tmp/loki/chunks

limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h
  ingestion_rate_mb: 16
  ingestion_burst_size_mb: 24


chunk_store_config:
  max_look_back_period: 0

table_manager:
  chunk_tables_provisioning:
    inactive_read_throughput: 0
    inactive_write_throughput: 0
    provisioned_read_throughput: 0
    provisioned_write_throughput: 0
  index_tables_provisioning:
    inactive_read_throughput: 0
    inactive_write_throughput: 0
    provisioned_read_throughput: 0
    provisioned_write_throughput: 0
  retention_deletes_enabled: true
  retention_period: 5d
```

/etc/victoriametrics/prometheus.yml 

```
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'external'
    static_configs:
      - targets: ['165.34.23.23:9100']
      - targets: ['165.34.23.23:9100']
      - targets: ['165.34.23.23:9100']

  - job_name: prod_app
    relabel_configs:
      - source_labels: [__meta_ec2_public_ip]
        target_label: instance
      - source_labels: [__meta_ec2_public_ip]
        replacement: ${1}:39010
        target_label: __address__
      - source_labels: [__meta_ec2_public_ip]
        target_label: pub_ip_addr
      - source_labels: [__meta_ec2_instance_state]
        target_label: instance_state
    ec2_sd_configs:
      - region: ap-south-1
        port: 39010
        profile: "dddddd"
        access_key: "fdgdfsgh"
        secret_key: "hfgdh"
        filters:
          - name: instance-state-name
            values:
              - running
          - name: tag:NE
            values:
              - App
          - name: tag:Env
            values:
              - Prod
          - name: tag:Payer
            values:
              - Shipox
```

/etc/victoriametrics/vmalert/alerts.yml 

```
groups:
  - name: node
    interval: 60s
    rules:
      - alert: HostOutOfDiskSpace
        expr: (100 - (100 * node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) > 80
        for: 1m
        labels:
          severity: warning
          instance: "{{ $labels.instance }}"
        annotations:
          message: "Mountpoint: {{ $labels.mountpoint }}\n< 20% = {{ humanize $value }}%"

      - alert: Volume Of Logs is Almost Full
        expr: (100 - (100 * node_filesystem_avail_bytes{mountpoint="/var/log"} / node_filesystem_size_bytes{mountpoint="/var/log"})) > 80
        for: 1m
        labels:
          severity: warning
          instance: "{{ $labels.instance }}"
        annotations:
          message: "Mountpoint: {{ $labels.mountpoint }}\n< 20% = {{ humanize $value }}%"

      - alert: Volume Of DB is Almost Full
        expr: (100 - (100 * node_filesystem_avail_bytes{mountpoint="/home/ebs"} / node_filesystem_size_bytes{mountpoint="/home/ebs"})) > 80
        for: 1m
        labels:
          severity: warning
          instance: "{{ $labels.instance }}"
        annotations:
          message: "Mountpoint: {{ $labels.mountpoint }}\n< 20% = {{ humanize $value }}%"

      - alert: Custom Volume is Almost Full
        expr: (100 - (100 * node_filesystem_avail_bytes{mountpoint!~"/|/var/log|/home/ebs|/tmp/.*|/snap/.*|/boot/.*|/var/.*|/run/.*"} / node_filesystem_size_bytes{mountpoint!~"/|/var/log|/home/ebs|/tmp/.*|/snap/.*|/boot/.*|/var/.*|/run/.*"})) > 80
        for: 1m
        labels:
          severity: warning
          instance: "{{ $labels.instance }}"
        annotations:
          message: "Mountpoint: {{ $labels.mountpoint }}\n< 20% = {{ humanize $value }}%"

      - alert: HostHighCpuLoad
        expr: 100 - (avg by(instance, job) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 90
        for: 1m
        labels:
          severity: warning
          instance: "{{ $labels.instance }}"
        annotations:
          message: "> 90% = {{ humanize $value }}%"

      - alert: Node is down
        expr: up{job="shipox_nodes"} == 0
        for: 1m
        labels:
          instance: "{{ $labels.instance }}"
          severity: critical
        annotations:
          message: "One of host is down"

      - alert: Node is down
        expr: up{job="sa_nodes"} == 0
        for: 1m
        labels:
          instance: "{{ $labels.instance }}"
          severity: critical
        annotations:
          message: "One of host is down"

      - alert: HostOutOfMemory
        expr: node_memory_MemAvailable_bytes{instance!="Shipox-ORS-Prod"} / node_memory_MemTotal_bytes{instance!="Shipox-ORS-Prod"} * 100 < 10
        for: 1m
        labels:
          instance: "{{ $labels.instance }}"
          severity: warning
        annotations:
          message: "< 10% = {{ humanize $value }}%"

      - alert: HostOutOfMemory
        expr: node_memory_MemAvailable_bytes{job="shipox_prod_app"} / node_memory_MemTotal_bytes{job="shipox_prod_app"} * 100 < 30
        for: 1m
        labels:
          instance: "{{ $labels.instance }}"
          severity: warning
        annotations:
          message: "< 30% = {{ humanize $value }}%"
```

