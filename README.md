# Storingsmelder

Cloud computing — Jorne

Een webapplicatie waarmee je storingen meldt, bekijkt en afsluit. Docker Compose start de applicatie en PostgreSQL samen. De databasegegevens blijven bewaard in een named volume.

## Vereisten

- Git
- Docker Desktop, gestart en ingesteld op Linux-containers
- Een vrije lokale poort 8000

De onderstaande commando’s zijn bedoeld voor PowerShell.

## Vanaf nul starten

### 1. Repository klonen

```powershell
git clone https://github.com/Jornevw/cloud-computing.git
cd cloud-computing
```

### 2. Lokale instellingen invullen

Maak een lokaal configuratiebestand:

```powershell
Copy-Item .env.example .env
notepad .env
```

Vul voor deze drie variabelen een eigen waarde in:

```text
POSTGRES_USER=
POSTGRES_PASSWORD=
POSTGRES_DB=
```

Laat geen waarden leeg. Gebruik voor de gebruikersnaam en databasenaam letters, cijfers en underscores. Kies een lang, uniek wachtwoord met letters en cijfers. Omdat het wachtwoord rechtstreeks in een verbindings-URL wordt gebruikt, vermijden we hier speciale tekens die URL-encoding vereisen.

Sla het bestand op. Het echte `.env`-bestand blijft lokaal en wordt door `.gitignore` en `.dockerignore` uitgesloten. `.env.example` bevat alleen de variabelenamen met lege waarden.

### 3. Applicatie en database starten

```powershell
docker compose up -d --build
```

Compose bouwt de applicatie vanuit de Dockerfile en start beide containers. Het databasevolume wordt automatisch aangemaakt. Bij de eerste start kan het even duren voordat PostgreSQL klaar is.

Open de applicatie op http://localhost:8000.

### 4. Werking controleren

```powershell
docker compose ps
curl.exe http://localhost:8000/health
```

De healthcheck hoort dit te tonen:

```json
{"opslag":"PostgreSQL","status":"ok"}
```

Is de applicatie nog niet bereikbaar, wacht dan enkele seconden en probeer opnieuw. Bekijk bij problemen de logs:

```powershell
docker compose logs web db
```

## Werking van de containers

De webcontainer luistert op poort 8000. Compose maakt deze bereikbaar op poort 8000 van de computer.

De applicatie verbindt met PostgreSQL via de servicenaam `db` op poort 5432. Deze databasepoort wordt niet gepubliceerd op de computer.

`depends_on` bepaalt de startvolgorde, maar wacht in deze configuratie niet totdat PostgreSQL verbindingen accepteert. `restart: unless-stopped` laat de webcontainer opnieuw starten als de applicatie door een opstartfout stopt.

## Gegevens bewaren

PostgreSQL 18 bewaart zijn gegevens in het named volume `postgres-data`, gekoppeld aan `/var/lib/postgresql`.

Containers stoppen en verwijderen:

```powershell
docker compose down
```

Opnieuw starten:

```powershell
docker compose up -d
```

Het databasevolume blijft hierbij behouden. Dit is getest met de melding `Bewaarproef opdracht 1`: na het verwijderen en opnieuw aanmaken van de containers stond de melding nog in de lijst.

Let op: `docker compose down -v` verwijdert ook het databasevolume en de opgeslagen meldingen.

De PostgreSQL-instellingen uit `.env` worden gebruikt bij het initialiseren van een lege database. Alleen `.env` wijzigen verandert geen gebruikersnaam of wachtwoord in een bestaande database.

## Publiek containerimage

Het applicatie-image is publiek beschikbaar op GitHub Container Registry:

```text
ghcr.io/jornevw/storingsmelder:1.1
```

Ophalen kan zonder aanmelden:

```powershell
docker pull ghcr.io/jornevw/storingsmelder:1.1
```

Versie `1.0` is eveneens gepubliceerd. We gebruiken expliciete versienummers in plaats van `latest`.

De Compose-configuratie bouwt de applicatie lokaal vanuit de Dockerfile.

## Bestanden in de repository

- `Dockerfile`: bouwt het applicatie-image.
- `compose.yaml`: start de applicatie en PostgreSQL samen.
- `.dockerignore`: sluit onder andere `.env`, Git-bestanden en lokale data uit van de build.
- `.gitignore`: sluit onder andere `.env` uit van Git.
- `.env.example`: toont welke lokale instellingen nodig zijn.
- `app.py`, `requirements.txt` en `templates/`: broncode en afhankelijkheden.
- `k8s/`: manifests van het afzonderlijke labo van lesweek 4; niet nodig om deze Compose-opstelling te starten.