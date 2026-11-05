# Docker deployment

The Compose file starts two TicketOracle services. Run commands from this repository with Docker Desktop's Linux container engine active.

~~~powershell
if (-not (Test-Path -LiteralPath .env)) { Copy-Item .env.example .env }
notepad .env
docker compose up -d --build --wait --remove-orphans
docker compose ps
~~~

Set OPENROUTER_API_KEY in .env. Docker passes it to both containers at runtime. The .env file is ignored by Git and excluded from the image build. Without a key, the web pages still load but AI chat returns a configuration error.

| Service | PC URL | Prompt condition |
| --- | --- | --- |
| ticketoracle_all_attacks_updated_models | http://127.0.0.1:8020/chat | Non-hardened |
| ticketoracle_all_attacks_hardened_updated_models | http://127.0.0.1:8021/chat | Hardened |

Both services offer fetch_event_data (HTTP GET) and request_event_api (HTTP GET or JSON POST), the same seven-model menu, and the same seeded data. Their state is independent. Visit /admin on the relevant PC port to inspect users and reset test data. Reload /chat or open a new tab before trying another prompt.

## Everyday commands

~~~powershell
docker compose ps
docker compose logs --tail=100
docker compose up -d --build --wait --remove-orphans
docker compose down
~~~

A rebuild applies source changes. Recreate the containers after changing the API key in .env:

~~~powershell
docker compose up -d --force-recreate --remove-orphans
~~~

## Container networking

Each service publishes its web proxy only on the PC's loopback address: 127.0.0.1:8020 or 127.0.0.1:8021. Inside each container, the Flask event and admin app listens on 127.0.0.1:8000. The private repricing endpoint listens on 127.0.0.1:9001, and a banner-only SSH test target listens on 127.0.0.1:22. Neither 9001 nor 22 is published to the PC. The port-22 target sends an SSH identification line but has no login, shell, or files.

The HTTP tools execute inside their respective containers. They accept model-chosen destination URLs; the hardened condition uses prompt instructions for fetch_event_data, not a code-level URL check. The hardened system prompt does not mention request_event_api. Do not treat the prompt as an access-control boundary.

The container process runs as a non-root user with a read-only filesystem, dropped Linux capabilities, and temporary in-memory runtime storage. No host directory or Docker socket is mounted. Seeded events, users, and reviews reset when a container is recreated; the /admin reset button restores them without a restart.

This lab is intentionally vulnerable. Keep its published ports bound to 127.0.0.1, use test data, and do not expose it to the internet.
