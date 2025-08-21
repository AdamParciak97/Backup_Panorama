## Backup_Panorama
Create Backup Panorama on script schedule


### Architecture:
* PANORAMA
* SERVER LINUX (for BACKUP)

### Part 1. ** **Create ACCOUNT on PANORAMA

<img width="1607" height="267" alt="image" src="https://github.com/user-attachments/assets/7982842e-0520-49cd-9110-e8c9c6955e40" />

### Part 2. Create API KEY for this Account The key is valid as long as the account exists and you do not change its password.

Run CMD and paste this command:

```bash
curl -sk "https://192.168.10.44/api/?type=keygen&user=super_admin&password=PASSWORD_FOR_THIS_ACCOUNT"
```

### Part 3. Create directory on Linux server

```bash
mkdir -p /home/Backup/panorama/archive
mkdir -p /home/Backup/panorama/log
```

### Part 4. Create file to login and API-KEY

```bash
touch /home/Backup/panorama/.env
chmod 600 /home/Backup/panorama/.env
```

### Part 5. Modifying .env file

```bash
# === Panorama API ===
PANORAMA_HOST="IP_OR_FQDN_PANORAMY"
PANORAMA_API_KEY="PASTE_API_KEY"

# === Local settings ===
LOCAL_BACKUP_DIR="/home/Backup/panorama/archive"
RETENTION_DAYS=30
TLS_INSECURE=true          # true = curl -k (ignore cert). Set false if you have correct certificate.
```

### Part 6. Add script to backup

```bash
#!/usr/bin/env bash
set -euo pipefail

ENV_FILE="/home/Backup/panorama/.env"
LOG_DIR="/home/Backup/panorama/log"
LOG_FILE="${LOG_DIR}/backup_config.log"

[[ -f "$ENV_FILE" ]] || { echo "Brak $ENV_FILE"; exit 1; }
# shellcheck disable=SC1090
source "$ENV_FILE"

mkdir -p "$LOCAL_BACKUP_DIR" "$LOG_DIR"
: "${PANORAMA_HOST:?}" "${PANORAMA_API_KEY:?}" "${LOCAL_BACKUP_DIR:?}" "${RETENTION_DAYS:?}" "${TLS_INSECURE:?}"

CURL_INSECURE_FLAG=""
[[ "${TLS_INSECURE,,}" == "true" ]] && CURL_INSECURE_FLAG="-k"

TS="$(date +%F_%H%M%S)"
TMP="$(mktemp "/tmp/panorama_configuration_${TS}.XXXXXX")"
OUT="${LOCAL_BACKUP_DIR}/panorama_configuration_${TS}.xml"

log(){ echo "[$(date '+%F %T')] $*" | tee -a "$LOG_FILE"; }

URL="https://${PANORAMA_HOST}/api/?type=export&category=configuration&key=${PANORAMA_API_KEY}"
log "Pobieram configuration z ${URL}"
if ! curl -s ${CURL_INSECURE_FLAG} --fail "${URL}" -o "${TMP}"; then
  log "BŁĄD: pobieranie configuration nie powiodło się"; rm -f "${TMP}"; exit 2
fi
[[ -s "$TMP" ]] || { log "BŁĄD: pusty plik configuration"; rm -f "$TMP"; exit 3; }
if grep -aq '<response status="error"' "$TMP" 2>/dev/null; then
  log "BŁĄD API (configuration):"; head -n 30 "$TMP" | sed 's/^/[API]/' | tee -a "$LOG_FILE"; rm -f "$TMP"; exit 4
fi

mv "$TMP" "$OUT"
gzip -9 "$OUT"
OUT="${OUT}.gz"
log "Zapisano: $OUT"

find "$LOCAL_BACKUP_DIR" -type f -mtime +"$RETENTION_DAYS" -name 'panorama_*' -print -delete | tee -a "$LOG_FILE" || true
log "CONFIG backup OK."
```

### Part 7. Add file permissions

```bash
chmod 600 /home/Backup/panorama/.env
chown super_admin:super_admin /home/Backup/panorama/.env

chmod 700 /home/Backup/panorama/backup_panorama.sh
chown super_admin:super_admin /home/Backup/panorama/backup_panorama.sh

chmod 700 /home/Backup/panorama/archive
chown -R super_admin:super_admin /home/Backup/panorama/archive

chmod 700 /home/Backup/panorama/log
chown -R super_admin:super_admin /home/Backup/panorama/log
```

### Part 8. Add modyfing permission to script

```bash
chmod +x /home/Backup/panorama/backup_panorama.sh
```

### Part 9. Testing manual

```bash
./backup_panorama.sh
```

### Part 10. Add to crontab

```bash
(crontab -l 2>/dev/null; echo "15 1 * * * /home/Backup/panorama/backup_panorama.sh >/dev/null 2>&1") | crontab -
```

