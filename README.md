# site-heure

Site minimal affichant l'heure en direct, conteneurisé, avec pipeline CI/CD prêt pour Oracle Cloud.

## Structure

```
site-heure/
├── .github/workflows/deploy.yml   # pipeline CI/CD
├── app/
│   ├── Dockerfile
│   └── index.html
├── docker-compose.yml             # pour tester EN LOCAL (build direct)
├── docker-compose.prod.yml        # pour la VM (utilise l'image déjà buildée)
└── Caddyfile                      # reverse proxy + HTTPS auto
```

## Mise en place du repo (à faire une fois)

```bash
cd site-heure
git init
git add .
git commit -m "Initial commit"
```

Crée un repo vide sur GitHub (sans README ni .gitignore auto-généré), puis :

```bash
git remote add origin https://github.com/TON_USER/site-heure.git
git branch -M main
git push -u origin main
```

## Avant le premier déploiement automatique

1. **Remplace `OWNER/REPO`** dans `docker-compose.prod.yml` par ton vrai `github-user/site-heure`.
2. **Ajoute les secrets** du repo (Settings → Secrets and variables → Actions → New repository secret) :
   - `SSH_HOST` : IP publique de ta VM Oracle
   - `SSH_USER` : utilisateur SSH (`ubuntu` ou `opc` selon l'image)
   - `SSH_PRIVATE_KEY` : le contenu de ta clé privée SSH (celle générée à la création de la VM)
3. Sur la VM, crée le dossier de destination une seule fois :
   ```bash
   mkdir -p ~/site-heure
   ```

## Fonctionnement du pipeline

À chaque `git push` sur `main` :
1. L'image Docker est construite depuis `app/`
2. Elle est poussée sur `ghcr.io/TON_USER/site-heure`
3. `docker-compose.prod.yml` et `Caddyfile` sont copiés sur la VM
4. La VM tire la nouvelle image et redémarre les conteneurs

`GITHUB_TOKEN` est généré automatiquement par GitHub Actions, pas besoin de le créer.

## Passer en dynamique plus tard

Seul le contenu de `app/` change (Dockerfile + code applicatif). Le pipeline, le registre, le
reverse proxy et le HTTPS restent identiques.
