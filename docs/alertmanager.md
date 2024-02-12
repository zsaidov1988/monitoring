[Main menu](../README.md)

## Alertmanager

[Download](https://prometheus.io/download/)

[Developer Portal](https://developer.couchbase.com/tutorial-configure-alertmanager)

### Install

```
sudo useradd --no-create-home --shell /bin/false alertmanager
sudo mkdir /etc/alertmanager
sudo cp alertmanager.yml /etc/alertmanager/
sudo mkdir /var/lib/alertmanager
sudo chown -R alertmanager:alertmanager /etc/alertmanager
sudo chown alertmanager:alertmanager /var/lib/alertmanager
sudo cp alertmanager /usr/local/bin
sudo cp amtool /usr/local/bin
sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager
sudo chown alertmanager:alertmanager /usr/local/bin/amtool
```
### Unit file

/etc/systemd/system/alertmanager.service

```
[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=alertmanager
Group=alertmanager
ExecStart=/usr/local/bin/alertmanager \
  --config.file=/etc/alertmanager/alertmanager.yml
  --storage.path=/var/lib/alertmanager

Restart=always

[Install]
WantedBy=multi-user.target
```


[Main menu](../README.md)