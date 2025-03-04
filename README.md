# Docker Compose - Media Services Stack

Ce projet Docker Compose contient une configuration pour déployer une infrastructure permettant de gérer un environnement de médiathèque, y compris un proxy, un serveur Plex et plusieurs outils d'automatisation pour le téléchargement et la gestion des médias.

## Services Inclus

### 1. **Nginx Proxy Manager**
- **Image** : `jc21/nginx-proxy-manager:latest`
- **Rôles** :
    - Sert de proxy inversé pour les différents services.
    - Gestion des certificats SSL (Let's Encrypt).
- **Ports exposés** :
    - `80`: HTTP
    - `443`: HTTPS
    - `81`: Interface d'administration
- **Volumes** :
    - `./nginx-proxy-manager/data:/data` : Données de configuration.
    - `./nginx-proxy-manager/letsencrypt:/etc/letsencrypt` : Certificats SSL.
- **Redémarrage** : `unless-stopped`

### 2. **Plex**
- **Image** : `lscr.io/linuxserver/plex:latest`
- **Container Name** : `plex`
- **Rôles** :
    - Streaming et organisation de la médiathèque.
- **Network Mode** : `host`
- **Variables d'environnement** :
    - `PUID=1000`, `PGID=1000`: Identifiants utilisateur/groupe pour les permissions.
    - `TZ`: Fuseau horaire (ex.: `Europe/Paris`).
    - `VERSION`: Version à utiliser (ex.: `docker`).
- **Volumes** :
    - `./plex/library:/config` : Configuration du serveur Plex.
    - `./plex/tvseries:/tv` : Séries.
    - `./plex/movies:/movies` : Films.
    - `${MEDIA_STORAGE_PATH}:/mnt/medias` : Stockage des médias.
- **Redémarrage** : `unless-stopped`

### 3. **Jellyseerr**
- **Image** : `fallenbagel/jellyseerr:latest`
- **Container Name** : `jellyseerr`
- **Rôles** :
    - Application de gestion des requêtes pour la médiathèque.
- **Ports exposés** :
    - `5055`: Interface web.
- **Variables d'environnement** :
    - `LOG_LEVEL=debug`: Niveau de log.
    - `TZ`: Fuseau horaire.
- **Volumes** :
    - `./jellyseerr:/app/config` : Configuration de Jellyseerr.
- **Redémarrage** : `unless-stopped`

### 4. **Prowlarr**
- **Image** : `lscr.io/linuxserver/prowlarr:latest`
- **Container Name** : `prowlarr`
- **Rôles** :
    - Gestion centralisée des indexeurs/torrents.
- **Ports exposés** :
    - `9696`: Interface web.
- **Variables d'environnement** :
    - `PUID=1000`, `PGID=1000`: Identifiants utilisateur/groupe.
    - `TZ`: Fuseau horaire.
- **Volumes** :
    - `./prowlarr:/config` : Configuration de Prowlarr.
    - `${MEDIA_STORAGE_PATH}:/mnt/medias` : Accès aux médias.
- **Réseaux** :
    - `medias_network`
- **Redémarrage** : `unless-stopped`

### 5. **Sonarr**
- **Image** : `lscr.io/linuxserver/sonarr:latest`
- **Container Name** : `sonarr`
- **Rôles** :
    - Gestion des séries TV.
    - Automatisation des téléchargements.
- **Ports exposés** :
    - `8989`: Interface web.
- **Variables d'environnement** :
    - `PUID=1000`, `PGID=1000`: Identifiants utilisateur/groupe.
    - `TZ`: Fuseau horaire.
- **Volumes** :
    - `./sonarr:/config` : Configuration de Sonarr.
    - `./sonarr/tv:/tv`: Dossier pour les séries (optionnel).
    - `./sonarr/downloads:/downloads`: Téléchargements (optionnel).
    - `${MEDIA_STORAGE_PATH}:/mnt/medias` : Accès aux médias.
- **Réseaux** :
    - `medias_network`
- **Redémarrage** : `unless-stopped`

### 6. **Radarr**
- **Image** : `lscr.io/linuxserver/radarr:latest`
- **Container Name** : `radarr`
- **Rôles** :
    - Gestion des films.
    - Automatisation des téléchargements.
- **Ports exposés** :
    - `7878`: Interface web.
- **Variables d'environnement** :
    - `PUID=1000`, `PGID=1000`: Identifiants utilisateur/groupe.
    - `TZ`: Fuseau horaire.
- **Volumes** :
    - `./radarr:/config` : Configuration de Radarr.
    - `./radarr/movies:/movies`: Dossier pour les films (optionnel).
    - `./radarr/downloads:/downloads`: Téléchargements (optionnel).
    - `${MEDIA_STORAGE_PATH}:/mnt/medias` : Accès aux médias.
- **Réseaux** :
    - `medias_network`
- **Redémarrage** : `unless-stopped`

### 7. **Nzbget**
- **Image** : `lscr.io/linuxserver/nzbget:latest`
- **Container Name** : `nzbget`
- **Rôles** :
    - Gestion de téléchargements NZB.
- **Ports exposés** :
    - `6789`: Interface web.
- **Variables d'environnement** :
    - `PUID=1000`, `PGID=1000`: Identifiants utilisateur/groupe.
    - `TZ`: Fuseau horaire.
- **Volumes** :
    - `./nzbget:/config` : Configuration de Nzbget.
    - `./nzbget/downloads:/downloads`: Téléchargements (optionnel).
    - `${MEDIA_STORAGE_PATH}:/mnt/medias` : Accès aux médias.
- **Réseaux** :
    - `medias_network`
- **Redémarrage** : `unless-stopped`

---

---

## Réseaux

- **`medias_network`** : Réseau `bridge` utilisé pour isoler les services multimédias et assurer leur interconnexion.

---

## Commandes Utiles

Pour démarrer le stack :
```bash
docker-compose up -d
```

Pour arrêter le stack :
```bash
docker-compose down
```

Pour voir les journaux d’un service :
```bash
docker logs [nom_du_service]
```

Pour reconstruire les conteneurs (après modification du fichier `docker-compose.yml`) :
```bash
docker-compose up -d --build
```

---

## Notes

1. **Permissions**
    - Assurez-vous que les utilisateurs et groupes (`PUID`, `PGID`) ont les permissions nécessaires sur les fichiers et dossiers montés.

2. **Personnalisation**
    - Modifiez les chemins de volumes et les ports si nécessaire pour éviter des conflits avec d'autres services sur la machine hôte.

3. **Variables d’environnement**
    - N’oubliez pas de définir la variable d'environnement `MEDIA_STORAGE_PATH` pour le volume partagé dans vos fichiers `.env` ou système.