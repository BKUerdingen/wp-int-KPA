# KI-Prompts – Entwurf der docker-compose.yml

Verwendete KI: Claude (Anthropic). Die wichtigsten Prompts und Antworten in Stichpunkten.

## 1. Imageauswahl und Versionen
**Prompt:** „Welche Images und Versionen sollen wir für WordPress, MariaDB und phpMyAdmin in einer lokalen Entwicklungsumgebung nehmen?“

**Antwort (Auszug):**
- Nur offizielle Images von Docker Hub verwenden.
- Feste Versionen statt `latest`, damit alle im Team dieselbe Umgebung haben:
  - `wordpress:6-php8.3-apache` (WordPress 6.x, PHP 8.3, beim Test 6.9.4)
  - `mariadb:11.4` (LTS-Version)
  - `phpmyadmin:5.2`

## 2. Healthcheck und depends_on
**Prompt:** „Wie sorgen wir dafür, dass WordPress erst startet, wenn die Datenbank wirklich bereit ist?“

**Antwort (Auszug):**
- Ein einfaches `depends_on` wartet nur auf den *Start* des Containers, nicht darauf, dass die DB bereit ist.
- Das MariaDB-Image enthält `healthcheck.sh`:
  `test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]`
- Bei `wordpress` und `phpmyadmin`: `depends_on: db: condition: service_healthy`.
- `start_period: 30s`, weil die DB beim ersten Start die Datenbank anlegt.

## 3. Volumes
**Prompt:** „Named Volume oder Bind-Mount – was nehmen wir für die Datenbank und was für WordPress?“

**Antwort (Auszug):**
- Datenbank: **Named Volume** (`db_data`). Docker verwaltet es, es ist schnell und bleibt bei `docker compose down` erhalten.
- WordPress: **Bind-Mount** nur für `wp-content`, weil dort Plugins, Themes und Uploads liegen. Der WordPress-Kern bleibt im Image.
- Die Standard-Themes und Plugins, die WordPress automatisch in `wp-content` kopiert, kommen in die `.gitignore`, damit nur eigene Entwicklungen versioniert werden.

## 4. Sicherheit
**Prompt:** „Wie verhindern wir, dass Passwörter im Git-Repository landen?“

**Antwort (Auszug):**
- Alle Zugangsdaten als Variablen `${...}` in der Compose-Datei, die Werte stehen in der `.env`.
- `.env` in die `.gitignore`. Ins Repo kommt nur `.env.example` mit Platzhaltern.
- Jedes Teammitglied erzeugt eigene, sichere Passwörter.
- Die Datenbank bekommt keinen Port nach außen. WordPress und phpMyAdmin erreichen sie über das interne Compose-Netzwerk.
- Nur für die lokale Entwicklung gedacht, nicht so ins Internet stellen.

## 5. Port-Konflikte
**Prompt:** „Was tun, wenn 8080 oder 8081 auf einem Rechner schon belegt ist?“

**Antwort (Auszug):**
- Prüfen unter Windows: `netstat -ano | findstr :8080`
- Die Ports sind über `WP_PORT` und `PMA_PORT` in der `.env` änderbar (Standard 8080/8081), die Compose-Datei bleibt gleich.
- Port 3306 wird gar nicht nach außen gegeben, also gibt es keinen Konflikt mit einer lokal installierten MySQL/MariaDB.

## 6. Fehlersuche Docker Desktop
**Prompt:** „Docker Desktop meldet ‚Virtualization support not detected‘ – was fehlt?“

**Antwort (Auszug):**
- Die Virtualisierung war im BIOS aktiv, aber WSL war nicht installiert.
- Lösung: `wsl --install --no-distribution` als Administrator, danach neu starten.
