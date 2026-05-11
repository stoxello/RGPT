
# RGPT Docker Install Guide

RGPT is an AI-assisted Robinhood cryptocurrency trading dashboard and bot. The Docker image runs the `Rgpt.Web` application, stores runtime data in a Docker volume, and exposes the web UI on port `8080`.

This guide explains how to install and start RGPT with Docker Compose, complete the first-run setup page, activate the demo license, and access the application.

## Requirements

- Docker Desktop or Docker Engine with Docker Compose
- Internet access so Docker can pull the RGPT image
- OpenAI API key
- Robinhood Crypto API key and base64 private key
- RGPT license key

Demo license key:

```text
RGPT-NBV8-5RJN-TRIAL
```

## Docker Compose File

The repo includes a `docker-compose.yml` file that runs the published RGPT image:

```yaml
services:
  rgpt-web:
    image: stoxellosupport/rgpt:latest
    container_name: rgpt-web
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      ASPNETCORE_URLS: http://+:8080
      APP_URL: http://0.0.0.0:8080/
      RGPT_DATA_DIR: /app/data
      RGPT_BOOTSTRAP_SETTINGS_PATH: /app/data/bootstrap_settings.json
      ConnectionStrings__Identity: Data Source=/app/data/rgpt_identity.db
      LIVE_TRADE: "false"
    volumes:
      - rgpt-data:/app/data

volumes:
  rgpt-data:
```

The `rgpt-data` volume keeps the setup file, SQLite identity database, bot settings, AI settings, and trading runtime data between container restarts.

## Install and Start

From the repo folder, run:

```powershell
docker compose up -d
```

Docker Compose will pull `stoxellosupport/rgpt:latest`, create the `rgpt-data` volume, and start the `rgpt-web` container.

Check that the container is running:

```powershell
docker compose ps
```

View logs if startup fails:

```powershell
docker compose logs --tail 100 rgpt-web
```

## Open the Setup Page

After the container starts, open:

```text
http://localhost:8080/Setup
```

If this is the first run and no `bootstrap_settings.json` exists in the Docker volume, RGPT will allow anonymous access to the setup page. After setup is saved, `/Setup` is no longer public. Future visits redirect to login unless you are signed in as an admin.

Complete the setup form:

- **Configured App URL:** use `http://localhost:8080/` when running locally
- **OpenAI API Key:** your OpenAI API key
- **OpenAI Model:** default is `gpt-5.4`
- **Robinhood API Key:** your Robinhood Crypto API key
- **Robinhood Private Key (Base64):** your Robinhood API private key as base64 text
- **Admin Email:** the email address for the first admin user
- **Admin Password:** password for the first admin user, minimum 8 characters
- **Enable Live Trading:** leave unchecked for demo or paper review; enable only when you are ready for real orders

Click **Save setup**. RGPT writes the setup settings to `/app/data/bootstrap_settings.json` inside the container volume.

Restart the container after saving setup:

```powershell
docker compose restart rgpt-web
```

## Log In

Open the application:

```text
http://localhost:8080
```

Log in with the admin email and password you created on the setup page.

## Activate the Demo License

After logging in, go to:

```text
http://localhost:8080/License
```

Paste this demo key into the license activation box:

```text
RGPT-NBV8-5RJN-TRIAL
```

Click **Activate**. The License page shows the current license status, product code, plan, expiration date, days remaining, and machine ID.

## Common Commands

Stop RGPT:

```powershell
docker compose down
```

Start RGPT again:

```powershell
docker compose up -d
```

Pull the latest image and restart:

```powershell
docker compose pull
docker compose up -d
```

Reset all setup and runtime data:

```powershell
docker compose down --volumes
docker compose up -d
```

Only reset the data volume if you want to remove the setup file, users, license state, and stored bot data.

## Notes

- The container listens on port `8080`; change the left side of the port mapping, for example `"8090:8080"`, if port `8080` is already in use on your host.
- Keep `APP_URL` as `http://0.0.0.0:8080/` in Docker so ASP.NET binds inside the container correctly.
- Use `http://localhost:8080/` as the configured app URL on the setup page for local browser access.
- Live trading can place real orders. Keep `LIVE_TRADE` set to `false` and leave live trading unchecked until your keys, risk settings, and license are confirmed.

## Disclaimer

RGPT is provided for educational and research purposes. Cryptocurrency trading involves risk, including the possible loss of capital. You are responsible for your trading decisions, API credentials, account permissions, and financial outcomes.
