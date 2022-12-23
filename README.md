# Prometheus:

## REFERENCE:
https://prometheus.io/
https://hub.docker.com/r/prom/prometheus


## Examples:

```shell
docker-compose up -d --remove-orphans
```

#### At this point, navigate to Prometheus at [](http://localhost:9090), and Grafana at [](http://localhost:3000)

enter the following into http://localhost:9090/graph
```shell
prometheus_target_interval_length_seconds
prometheus_target_interval_length_seconds{quantile="0.99"}
count(prometheus_target_interval_length_seconds)
```

## Backup:
1. enable web admin api with `--web.enable-admin-api`
```shell
docker image inspect prom/prometheus
# output:
# ...
# "Cmd": [
#     "--config.file=/etc/prometheus/prometheus.yml",
#     "--storage.tsdb.path=/prometheus",
#     "--web.console.libraries=/usr/share/prometheus/console_libraries",
#     "--web.console.templates=/usr/share/prometheus/consoles"
# ],
```

### This is an example that adds the enable admin api flag with some minor adjustments elsewhere
```shell
docker run -it --rm \
    -p 9090:9090 \
    -v "$(pwd)/shared_volume_dir/prometheus:/etc/prometheus" \
    prom/prometheus \
    --config.file=/etc/prometheus/prometheus.yaml \
    --storage.tsdb.path=/etc/prometheus/tsdb \
    --web.console.libraries=/usr/share/prometheus/console_libraries \
    --web.console.templates=/usr/share/prometheus/consoles \
    --web.enable-admin-api
```

### Take a snapshot
```shell
curl -X POST http://localhost:9090/api/v1/admin/tsdb/snapshot
```

### Restore Prometheus
Just clear the existing tsdb directory and drop everything from any snapshot into that directory
```shell
rm -rf shared_volume_dir/prometheus/tsdb
mkdir shared_volume_dir/prometheus/tsdb          
cp -rf prometheus_snapshots/20221223T170048Z-3ee3de9ec6c58d78/* shared_volume_dir/prometheus/tsdb
```


## Pushing Metrics
https://github.com/prometheus/pushgateway/blob/master/README.md
```shell
docker pull prom/pushgateway
docker run -d -p 9091:9091 prom/pushgateway
```

### Push Examples
https://github.com/prometheus/pushgateway
```shell
echo "some_metric 3.14" | \
    curl \
        --data-binary @- \
        http://localhost:9091/metrics/job/pushed_metrics

echo "some_metric 1" | curl --data-binary @- http://localhost:9091/metrics/job/pushed_metrics && \
```