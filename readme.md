# 🚀 Script de déploiement LuuxCraft

Ce script automatise la mise en place d’un environnement Docker avec **Caddy** comme reverse proxy et la connexion à **GitHub Container Registry (GHCR)** pour récupérer vos images Docker.

Il configure automatiquement :

* L’installation de **Docker** (si absent)
* La connexion à **ghcr.io** avec votre compte GitHub (`luuxis`) et une clé API
* La génération d’un **Caddyfile** avec votre domaine et son sous-domaine `staging.`
* Le lancement de **Caddy en container Docker** avec HTTPS (Let's Encrypt)
* La création d’un **réseau Docker partagé** (`luuxcraft`) pour connecter vos services applicatifs

---

## 📦 Prérequis

* Un serveur Linux (Ubuntu/Debian recommandé)
* Accès **root** ou via `sudo`
* Une clé API GitHub (Personal Access Token) avec accès au **GitHub Container Registry**
* Un domaine configuré avec deux enregistrements DNS pointant sur le serveur :

  * `example.com`
  * `staging.example.com`

---

## 🚀 Utilisation

Lancer le script en root (avec `sudo`) et passer votre clé API GHCR en argument :

```bash
curl -fsSL https://raw.githubusercontent.com/LuuxCraft/deploy-luuxcraft/refs/heads/master/deploy | sudo bash -s -- <VOTRE_USER_GITHUB> <VOTRE_CLE_API_GHCR>
```

Exemple :

```bash
curl -fsSL https://raw.githubusercontent.com/LuuxCraft/deploy-luuxcraft/refs/heads/master/deploy | sudo bash -s -- luuxis ghp_xxxxxxxxxxxxxxxxxxxxx
```

---

## 📝 Fonctionnement du script

1. **Installation de Docker**
   Si Docker n’est pas installé, le script l’installe automatiquement via le script officiel `get.docker.com`.

2. **Connexion à GHCR**
   Le script exécute :

   ```bash
   echo "API_KEY" | docker login ghcr.io -u luuxis --password-stdin
   ```

   → Cela permet à Docker de télécharger vos images privées.

3. **Demande du domaine**
   Le script demande à l’utilisateur de saisir son domaine (ex: `example.com`).

4. **Génération du Caddyfile**
   Un fichier est créé dans :

   ```
   /home/<user>/deploy-luuxcraft/caddy/Caddyfile
   ```

   Avec les entrées suivantes :

   * `example.com` → proxy vers `production:8080`
   * `staging.example.com` → proxy vers `staging:8080`

5. **Mise en place de Caddy**

   * Création du réseau Docker `luuxcraft` (si non existant)
   * Lancement d’un container Caddy relié à ce réseau
   * Montage des volumes pour les logs, config et certificats SSL

6. **Connexion des services applicatifs**
   Pour que Caddy fonctionne, vos containers applicatifs doivent être sur le réseau `luuxcraft`.

   Exemple :

   ```bash
   docker network connect luuxcraft production
   docker network connect luuxcraft staging
   ```

---

## 📂 Emplacements importants

* **Caddyfile** :
  `/home/<user>/deploy-luuxcraft/caddy/Caddyfile`

* **Logs Caddy** :
  `/home/<user>/deploy-luuxcraft/caddy/logs/`

---

## ✅ Exemple complet

```bash
curl -fsSL https://raw.githubusercontent.com/LuuxCraft/deploy-luuxcraft/refs/heads/master/deploy | sudo bash -s -- luuxis ghp_xxxxxxxxxxxxxxxxxxxxx
```

Le script demandera :

```
🌐 Domaine principal (ex: example.com) :
```

Tu saisis :

```
luuxcraft.fr
```

👉 Résultat :

* `https://luuxcraft.fr` redirige vers `production:8080`
* `https://staging.luuxcraft.fr` redirige vers `staging:8080`

---

## ⚠️ Notes importantes

* Si vous exécutez le script pour la première fois, **déconnectez-vous / reconnectez-vous** pour appliquer les droits Docker à votre utilisateur.
* Les clés API passées en paramètre apparaissent dans l’historique shell → **privilégiez l’utilisation de tokens temporaires**.
* Vérifiez vos DNS avant de lancer Caddy, sinon la génération automatique des certificats **Let’s Encrypt** échouera.
