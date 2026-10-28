# Grafana with Prometheus

Docker Compose configuration for running Grafana and Prometheus together. Grafana publishes port `3000`; Prometheus is kept on the internal Compose network and can only be reached by other services as `http://prometheus:9090`. Prometheus is automatically provisioned as Grafana's default datasource.

## Requirements

- Docker Engine
- Docker Compose v2 (`docker compose`)

## Configure the administrator password

The default username is `admin`, and the temporary password is stored in [`admin_password.txt`](./admin_password.txt).

Change the password before starting Grafana for the first time by replacing its contents:

```sh
printf '%s\n' 'your-secure-password' > admin_password.txt
```

The value is mounted as a Docker Compose secret rather than passed directly through a Compose environment variable.

> Grafana only uses the configured password when creating a new installation. If Grafana already has persistent data in the `grafana_data` volume, changing this file does not change the password stored in Grafana. Change it from the Grafana web interface or remove the `grafana_data` volume to reset the installation and password.

## Start and stop the stack

From the `grafana` project folder, first change into the Grafana project directory:

```sh
cd grafana
docker compose up -d
```

Check container status and logs:

```sh
docker compose ps
docker compose logs -f
```

Stop the stack while preserving Grafana and Prometheus data:

```sh
docker compose down
```

Stop the stack and delete its persistent data volumes:

```sh
docker compose down -v
```

## Access and log in

Open <http://localhost:3000> and sign in with:

- Username: `admin`
- Password: the temporary password in `admin_password.txt`, or the password currently configured in Grafana

After the first login, Grafana may ask you to change the password. The provisioned **Prometheus** datasource is available under **Connections > Data sources** and is selected as the default datasource.

## Useful Docker commands

Recreate the containers after changing `admin_password.txt` or Compose configuration:

```sh
docker compose up -d --force-recreate
```

Restart Grafana after provisioning or configuration changes:

```sh
docker compose restart grafana
```

Open a shell in Grafana:

```sh
docker compose exec grafana sh
```
