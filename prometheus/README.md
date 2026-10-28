# Prometheus and static metrics

Docker Compose configuration for running Prometheus with an Nginx server that exposes a static metrics file. Prometheus stores time-series data in a persistent Docker volume and scrapes the static endpoint every 15 seconds.

## Requirements

- Docker Engine
- Docker Compose v2 (`docker compose`)

## Start the services

From the `prometheus` directory, start Prometheus and the Nginx metrics server:

```sh
cd prometheus
docker compose up -d
```

Check the containers and view logs:

```sh
docker compose ps
docker compose logs -f
```

## Access the services

- Prometheus web UI and HTTP API: <http://localhost:9090>

Nginx serves `metrics.txt` internally on port 80. Because this Compose file does not publish an Nginx host port, access the file from inside the container with:

```sh
docker compose exec static-metrics wget -qO- http://localhost/metrics.txt
```

## Static scrape configuration

`prometheus.yml` contains a `static_metrics` scrape job:

```yaml
- job_name: 'static_metrics'
  metrics_path: /metrics.txt
  static_configs:
    - targets: ['static-metrics:80']
```

Prometheus scrapes the `metrics` file served by Nginx at `http://static-metrics:80/metrics.txt` every 15 seconds.

Edit `metrics` and Prometheus will serve the updated file after Nginx detects the bind-mounted file change. Restart Prometheus only when changing `prometheus.yml`:

```sh
docker compose restart promdb
```

The file uses Prometheus text exposition format. For example:

```text
temperature_f{name="Dry Bulb",location="Outdoor"} 75
```

Each unique label combination creates a separate time series. Labels such as `name` and `location` must always use double quotes.

## Connect Grafana

For Grafana running in a separate Docker Compose network, use the Prometheus service address `http://promdb:9090`. When Grafana runs on the host, use `http://localhost:9090`; use `http://host.docker.internal:9090` for Docker Desktop only when host networking is required.

## Stop and remove

Stop and remove the containers and network while keeping stored metrics:

```sh
docker compose down
```

To delete stored Prometheus metrics as well:

```sh
docker compose down -v
```
