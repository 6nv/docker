# Grafana with Prometheus

This Docker Compose stack runs Grafana, Prometheus, and Prometheus Node Exporter's text file collector.

## Services

- **Grafana:** <http://localhost:3000>, with port `3000` exposed on the host.
- **Prometheus:** available to Compose services at `http://prometheus:9090`; it is not published on the host.
- **Node Exporter:** available at `node-exporter:9100` and serves files from the `metrics/` directory.

Startup order is Node Exporter → Prometheus → Grafana.

## Start and stop

From the repository root:

```sh
cd grafana
docker compose up -d
docker compose ps
```

View logs or stop the stack:

```sh
docker compose logs -f
docker compose down
```

To also remove Grafana and Prometheus data:

```sh
docker compose down -v
```

## Grafana login

Open <http://localhost:3000> and log in with username `admin`.

The password is read from [`admin_password.txt`](./admin_password.txt). Before the first startup, change it with:

```sh
cd grafana
printf '%s\n' 'your-secure-password' > admin_password.txt
docker compose up -d
```

Changing the file does not update an existing Grafana installation's password. Grafana stores that password in the `grafana_data` volume; change it through the web interface or delete the volume to reset the installation.

Prometheus is automatically provisioned as Grafana's default datasource at `http://prometheus:9090`.

## Add text file metrics

Prometheus scrapes Node Exporter, which reads every `.prom` file directly inside [`metrics/`](./metrics). The directory is mounted read-only at `/textfile`.

Create a metric file from the Grafana directory:

```sh
printf '%s\n' 'building_temperature_celsius{building="main"} 21.4' > metrics/building_temperature.prom
```

Wait up to the 15-second scrape interval, then query Grafana **Explore**:

```promql
building_temperature_celsius
```

No additional Prometheus scrape job is needed; `prometheus.yml` already scrapes `node-exporter:9100`.

## Text file mistakes to avoid

- Use double quotes for labels: `building="main"`, not `building='main'`.
- End every `.prom` file with a newline.
- Use the `.prom` extension only.
- Put one metric sample on each line.
- Use numeric values: `21.4`, not `warm`.
- Do not include Markdown, JSON, shell output, or timestamps in metric files.
- Avoid duplicate metrics or labels within the same label set.
- Write files atomically to prevent Node Exporter from reading partial content.

`node_textfile_scrape_error` should be `0` when files parse successfully.

## Verify metrics

Use these queries in Grafana **Explore**:

```promql
up
node_textfile_scrape_error
building_temperature_celsius
```

Check Node Exporter logs with:

```sh
cd grafana
docker compose logs -f node-exporter
```

## Common commands

```sh
cd grafana
docker compose ps
docker compose logs -f prometheus
docker compose restart node-exporter
docker compose restart prometheus
docker compose restart grafana
docker compose exec grafana sh
```
