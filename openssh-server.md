
1. Update
```
sudo apt update && sudo apt upgrade
```

2. Install
```
sudo apt install openssh-server
```

3. Vérification du démarrage automatique du service
```
sudo systemctl status ssh
```

Si le service nn'est pas running
```
sudo systemctl enable --now ssh
```

4. Firewall
Autoriser ssh
```
sudo ufw allow ssh
```

Vérification du statut
```
sudo ufw status
```

