# prom-docker

Simple Docker Compose configuration for running Prometheus as a metrics database
for Grafana. Prometheus starts with a basic configuration that scrapes its own
metrics every 15 seconds. Add scrape jobs in `prometheus.yml` to collect
metrics from other services.

## Requirements

- Docker Engine
- Docker Compose v2 (`docker compose`)

## Start Prometheus

Run these commands from the repository root:

```sh
docker compose up -d
```

Check that the service is running and view its logs:

```sh
docker compose ps
docker compose logs -f prometheus
```

## Access Prometheus and connect Grafana

Open the Prometheus web UI at <http://localhost:9090>. Prometheus's HTTP API is
available at the same address.

To add Prometheus in Grafana, create a Prometheus data source and set its URL to
`http://localhost:9090` when Grafana runs directly on your host. If Grafana runs
in a container, `localhost` refers to that container, not your host. On Docker
Desktop, use `http://host.docker.internal:9090`; on other setups, configure a
network route that lets the Grafana container reach the Prometheus service.

## Configure scraping

Edit `prometheus.yml` to add scrape jobs and targets. The current
configuration only scrapes Prometheus itself. Restart the service to apply
configuration changes:

```sh
docker compose restart prometheus
```

## Stop and remove

Stop and remove the container and network while keeping the stored metrics:

```sh
docker compose down
```

To also delete the named `prometheus_data` volume and all stored metrics:

```sh
docker compose down -v
```
