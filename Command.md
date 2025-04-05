# Odoo en mode container

## Disable hybernation, screen down
```
sudoedit /etc/systemctl/logind
```
Retirer le # et remplacer par ces valeurs.
```
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

## Déploiement

🧱 Prérequis
- Ubuntu Server
- Accès root ou sudo
- Docker et Docker Compose installés
- Un domaine pointé vers ton serveur (ex: prod.idsec.ca et preprod.idsec.ca)


🔧 Étape 1 — Installer Docker et Docker Compose
```
sudo apt update
sudo apt install docker.io docker-compose -y
sudo systemctl enable docker
sudo usermod -aG docker $USER => permet de bypasser sudo donc je ne l'ai pas fait
```
Déconnecte/reconnecte ta session pour appliquer le group docker.



📁 Étape 2 — Cloner le projet Odoocker
```
git clone https://github.com/odoocker/odoocker.git
cd odoocker
```


🏗️ Étape 3 — Créer deux dossiers : prod et preprod
```
mkdir www pp
cp -r examples/full/* www/
cp -r examples/full/* pp/
```


📝 Étape 4 — Configurer docker-compose.yml pour chaque environnement

Dans prod/docker-compose.yml, modifie le port (par exemple PostgreSQL ou Odoo) pour ne pas qu’il entre en conflit avec preprod.
```
services:
  odoo:
    ports:
      - "8070:8069" # Prod Odoo sur port 8070
```
Fais pareil pour preprod/docker-compose.yml, en laissant par exemple 8069.


📝 Étape 5 — Modifier les noms de containers pour éviter les conflits

Dans chaque docker-compose.yml, ajoute un container_name unique à chaque service.
```
  odoo:
    container_name: odoo18_prod
  db:
    container_name: odoo18_prod_db
```
Et même chose pour preprod (odoo18_preprod, etc).


🚀 Étape 6 — Lancer les containers
Dans chaque dossier :
```
docker-compose up -d
```
Teste localement :
	•	http://localhost:8069 (preprod)
	•	http://localhost:8070 (prod)


🌐 Étape 7 — Configurer Nginx comme reverse proxy

Installe Nginx :
```
sudo apt install nginx -y
```
Crée un fichier de conf pour chaque domaine dans /etc/nginx/sites-available/
```
server {
    server_name prod.idsec.ca;

    location / {
        proxy_pass http://127.0.0.1:8070;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    listen 80;
}
```

Active les sites
```
sudo ln -s /etc/nginx/sites-available/prod.idsec.ca /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/preprod.idsec.ca /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```


🔐 Étape 8 — Obtenir un certificat SSL avec Certbot

Installe Cerbot:
```
sudo apt install certbot python3-certbot-nginx -y
```

Execute:
```
sudo certbot --nginx -d www.idsec.ca -d www.idsec.ca
sudo certbot --nginx -d pp.idsec.ca -d pp.idsec.ca
```