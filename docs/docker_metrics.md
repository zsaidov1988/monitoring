[Main menu](../README.md)

## Collect docker metrics

/etc/docker/daemon.json
```
{
  "metrics-addr": "127.0.0.1:9323",
  "experimental": true
}
```
```
systemctl restart docker
curl localhost:9323/metrics
```

[Main menu](../README.md)