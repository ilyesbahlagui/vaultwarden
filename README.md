# 🔐 Vaultwarden - Self-Hosted Password Manager

> **Si Vaultwarden disparaît demain :**
> Aucun problème — Bitwarden (la société officielle) propose un import direct du format Vaultwarden/Bitwarden.
> Tu exportes ton coffre en `.json` depuis l'interface, et tu l'importes sur [bitwarden.com](https://bitwarden.com) en 2 clics.
> Les deux sont 100% compatibles puisque c'est le même format.
>
> **Bonne pratique :**
> Fais un export régulier de ton coffre :
>
> * Dans l'app Bitwarden → Paramètres → Exporter le coffre
> * Garde le fichier `.json` chiffré dans un endroit sûr (pas sur le même VPS)
>
> Tu n'es donc jamais prisonnier de Vaultwarden. ✅

---

Déploiement de Vaultwarden avec Docker, derrière Nginx Proxy Manager.

> Vaultwarden est une réécriture non officielle du serveur Bitwarden en Rust, ultra légère.
> Les applications clientes officielles Bitwarden (mobile, navigateur, desktop) fonctionnent avec Vaultwarden.

---

## 🧱 Stack

* **Vaultwarden** : serveur de mots de passe
* **Base de données** : SQLite intégrée (aucun container supplémentaire nécessaire)
* **Reverse proxy** : Nginx Proxy Manager (réseau externe `nginx-proxy-manager-default`)

---

## 📁 Structure

```text id="bc9jqy"
vaultwarden/
├── docker-compose.yml
└── data/   ← Volume SQLite + données (généré automatiquement)
```

---

## 🚀 Installation

### 1. Créer le dossier

```bash id="qxzt7w"
mkdir -p /root/prod/vaultwarden/data
cd /root/prod/vaultwarden
```

---

### 2. Créer le `docker-compose.yml`

```yaml id="ek4oka"
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      DOMAIN: "https://vaultwarden.ton-domaine.fr"
      SIGNUPS_ALLOWED: "true"   # passer à "false" après création de ton compte
    volumes:
      - ./data:/data
    networks:
      - nginx-proxy-manager_default

networks:
  nginx-proxy-manager_default:
    external: true
```

---

### 3. Lancer le container

```bash id="3vcatm"
docker compose up -d
docker logs vaultwarden
```

---

### 4. Configurer Nginx Proxy Manager

Créer un **Proxy Host** dans NPM :

* **Domain** : `vaultwarden.ton-domaine.fr`
* **Forward Hostname** : `vaultwarden`
* **Forward Port** : `80`
* **SSL** : Let's Encrypt ✅

---

### 5. Créer ton compte

Accède à :
👉 https://vaultwarden.ton-domaine.fr

---

### 6. Désactiver les inscriptions

Modifier le `docker-compose.yml` :

```yaml id="21tz6u"
SIGNUPS_ALLOWED: "false"
```

Puis redémarrer :

```bash id="9i2q06"
docker compose up -d
```

---

## 🛠️ Commandes utiles

```bash id="u9bmwg"
# Voir les logs
docker logs vaultwarden

# Redémarrer
docker compose restart

# Mettre à jour l'image
docker compose pull
docker compose up -d

# Arrêter
docker compose down
```

---

## 💾 Sauvegarde

Les données sont dans `./data/` — sauvegarde ce dossier régulièrement :

```bash id="e4mslp"
cp -r /root/prod/vaultwarden/data /root/backups/vaultwarden-$(date +%Y%m%d)
```

---

## ⚠️ Points clés

* SQLite intégré → aucun container DB nécessaire
* `SIGNUPS_ALLOWED: "false"` → à activer après création du compte
* HTTPS obligatoire → Vaultwarden ne fonctionne pas sans SSL
* Réseau Docker → utiliser `nginx-proxy-manager-default`

---
