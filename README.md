# crowdsec-web-ui

Déploiement de [CrowdSec Web UI](https://github.com/TheDuffman85/crowdsec-web-ui) en Docker — compatible CrowdSec natif et CrowdSec Docker.

> Guide complet disponible sur [thadji.fr](https://thadji.fr/crowdsec-web-ui-interface-gestion)

---

## Ce que ça fait

- Interface web pour visualiser les alertes et décisions CrowdSec
- Carte géographique des attaquants et statistiques par scénario
- Gestion manuelle des décisions (ban/unban) sans terminal
- Notifications par email, ntfy, MQTT ou webhook
- Partage d'accès avec un tiers de confiance sans accès SSH

## Prérequis

- CrowdSec installé et fonctionnel (natif ou Docker)
- Docker et Docker Compose
- Nginx comme reverse proxy
- Authentik ou équivalent pour protéger l'accès WAN

---

## Cas 1 — CrowdSec en paquet natif

La LAPI écoute par défaut sur `127.0.0.1:8080`, inaccessible depuis Docker.
Modifiez `/etc/crowdsec/config.yaml` :

```yaml
api:
  server:
    listen_uri: 192.168.1.30:8080
```

Remplacez `192.168.1.30` par l'IP locale de votre serveur. Puis :

```bash
sudo systemctl restart crowdsec
sudo cscli machines add crowdsec-web-ui --password <mot-de-passe> -f /dev/null
```

Dans le fichier `.env`, utilisez :

```
CROWDSEC_URL=http://192.168.1.30:8080
```

Déployez ensuite avec le `docker-compose.yml` fourni :

```yaml
services:
  crowdsec-web-ui:
    image: ghcr.io/theduffman85/crowdsec-web-ui:latest
    container_name: crowdsec_web_ui
    env_file:
      - .env
    volumes:
      - ./data:/app/data
    restart: unless-stopped
```

Pas besoin de réseau Docker externe ici — le conteneur joint CrowdSec via l'IP locale de l'hôte.

---

## Cas 2 — CrowdSec en Docker

Mettez les deux conteneurs sur le même réseau Docker.
CrowdSec Web UI peut alors joindre CrowdSec par son nom de service, sans passer par l'IP de l'hôte.

```bash
docker exec crowdsec cscli machines add crowdsec-web-ui --password <mot-de-passe> -f /dev/null
```

Dans `.env`, utilisez :

```
CROWDSEC_URL=http://crowdsec:8080
```

Le `docker-compose.yml` doit attacher le conteneur au même réseau que CrowdSec :

```yaml
services:
  crowdsec-web-ui:
    image: ghcr.io/theduffman85/crowdsec-web-ui:latest
    container_name: crowdsec_web_ui
    env_file:
      - .env
    volumes:
      - ./data:/app/data
    restart: unless-stopped
    networks:
      - crowdsec-net

networks:
  crowdsec-net:
    external: true
```

Vérifiez le nom exact du réseau CrowdSec avec :

```bash
docker inspect crowdsec | grep -A 5 Networks
```

---

## Installation

```bash
# 1. Générer un mot de passe
openssl rand -hex 32

# 2. Créer le compte machine (voir cas 1 ou 2 ci-dessus)

# 3. Copier et remplir le fichier .env
cp .env.example .env |

# 4. Démarrer
docker compose up -d

# 5. Vérifier
curl http://localhost:3000/api/health
```

---

## Sécurité

CrowdSec Web UI n'a pas d'authentification intégrée.
**Ne jamais exposer le port 3000 directement sur Internet.**
Utilisez un reverse proxy avec authentification (Authentik, Authelia, Keycloak).

---

## Structure

```
crowdsec-web-ui/
├── docker-compose.yml
├── .env.example
└── README.md
```

---

## Source

Projet original : [TheDuffman85/crowdsec-web-ui](https://github.com/TheDuffman85/crowdsec-web-ui)
Licence originale : AGPL-3.0

## Licence

MIT — libre de réutilisation, lien vers [thadji.fr](https://thadji.fr) apprécié.
