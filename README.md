# Backup_Panorama
Create Backup Panorama on script schedule


**Architecture:**
PANORAMA
SERVER LINUX (for BACKUP)

**Part 1. ** **Create ACCOUNT on PANORAMA**

<img width="1607" height="267" alt="image" src="https://github.com/user-attachments/assets/7982842e-0520-49cd-9110-e8c9c6955e40" />

**Part 2. ** **Create API KEY for this Account** The key is valid as long as the account exists and you do not change its password.

Run CMD and paste this command:

curl -sk "https://192.168.10.44/api/?type=keygen&user=super_admin&password=PASSWORD_FOR_THIS_ACCOUNT"

**Part 3. ** **Create directory on Linux server**

mkdir -p /home/Backup/panorama/archive
mkdir -p /home/Backup/panorama/log

**Part 4. ** **Create file to login and API-KEY**

touch /home/Backup/panorama/.env
chmod 600 /home/Backup/panorama/.env

**Part 5. ** **Modifying .env file**

# === Panorama API ===
PANORAMA_HOST="IP_OR_FQDN_PANORAMY"
PANORAMA_API_KEY="PASTE_API_KEY"

# === Local settings ===
LOCAL_BACKUP_DIR="/home/Backup/panorama/archive"
RETENTION_DAYS=30
TLS_INSECURE=true          # true = curl -k (ignore cert). Set false if you have correct certificate.


**Part 6. ** **Add script to backup**



**Part 7. ** **Add file permissions**

chmod 600 /home/Backup/panorama/.env
chown super_admin:super_admin /home/Backup/panorama/.env

chmod 700 /home/Backup/panorama/backup_panorama.sh
chown super_admin:super_admin /home/Backup/panorama/backup_panorama.sh

chmod 700 /home/Backup/panorama/archive
chown -R super_admin:super_admin /home/Backup/panorama/archive

chmod 700 /home/Backup/panorama/log
chown -R super_admin:super_admin /home/Backup/panorama/log

**Part 8. ** **Add modyfing permission to script.**

chmod +x /home/Backup/panorama/backup_panorama.sh

**Part 9. ** **Testing manual**


