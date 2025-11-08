# ☁️ 1. Création et configuration du VPS Oracle Cloud

## 1.1. Créer un compte Oracle Cloud Free Tier

1. Rendez-vous sur [https://www.oracle.com/cloud/free](https://www.oracle.com/cloud/free)
2. Créez un compte et connectez-vous à la **console Oracle Cloud**.
3. Accédez à **Compute → Instances → Create Instance** pour créer votre machine virtuelle gratuite.

---

## 1.2. Configuration de la machine

| Paramètre | Valeur recommandée |
|------------|--------------------|
| Nom de l’instance | `instance-vpn` |
| Image | Ubuntu 22.04 LTS |
| Type | Always Free VM.Standard.E2.1.Micro |
| Stockage | 50 Go (minimum) |
| Réseau | Créez un **VCN** + **Subnet Public** |
| Adresse IP publique | Oui |
| Clé SSH | Générée via `ssh-keygen` |

💡 **Astuce :** Les instances “Always Free” sont limitées en ressources, mais idéales pour héberger un petit site WordPress, un VPN, ou un environnement de test.

---

## 1.3. Connexion SSH à la machine

Il est **vivement recommandé** de se connecter via une **clé SSH**, car cette méthode est bien plus sécurisée qu’un mot de passe.

Le principe est simple :  
- Votre **machine locale** possède la clé privée (`id_rsa`)  
- Le **serveur Oracle Cloud** détient la clé publique correspondante (`id_rsa.pub`)  

Lors de la connexion, votre machine prouve son identité via l’empreinte de la clé.

```bash
ssh -i ~/.ssh/id_rsa ubuntu@<IP_PUBLIQUE>
```

## 1.4. Mise à jour du système

Avant toute configuration, il est essentiel de mettre à jour la machine :
```bash
sudo apt update && sudo apt upgrade -y
```

💡 Cela garantit que le système est protégé contre les vulnérabilités connues.

## 1.5. Configuration du pare-feu sur Oracle Cloud

Pour renforcer la sécurité du VPS, il est recommandé d’adopter une approche “deny all” (refuser tout par défaut) et d’ouvrir uniquement les ports nécessaires.

Étapes :

* Allez dans Networking → Virtual Cloud Network (VCN)  
* Sélectionnez votre VCN  
* Ouvrez Security Lists → Default Security List  
* Ajoutez les règles suivantes : 

<br> 

| Protocole	| Port |	Source	| Action|
|------------|-----|------------|-------|
|TCP|	22	|0.0.0.0/0	|SSH
|TCP|	80  |0.0.0.0/0	|HTTP
|TCP|	443	|0.0.0.0/0 |HTTPS
<br> 

💡 Pour un environnement de production, il est recommandé de restreindre les adresses IP sources (par exemple, votre propre IP publique pour SSH).

## 1.6. Sécurisation de base d’un serveur Linux

Lorsqu’un serveur vient d’être déployé, il est crucial de le sécuriser avant toute mise en production.

**🔐 Bonnes pratiques essentielles :**

Changer le mot de passe root :
```bash
sudo passwd root
```

Désactiver la connexion SSH directe en root :

Éditez le fichier SSH :
```bash
sudo nano /etc/ssh/sshd_config
```

Modifiez ou ajoutez la ligne suivante :
```bash
PermitRootLogin no
```

Puis redémarrez le service :
```bash
sudo systemctl restart ssh
```

Créer un nouvel utilisateur avec droits sudo :
```bash
sudo adduser adminuser
sudo usermod -aG sudo adminuser
```

Configurer un pare-feu local avec UFW (Uncomplicated Firewall) :
```bash
sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
sudo ufw status
```

Activer les mises à jour automatiques de sécurité :
```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Installer Fail2Ban (protection contre les attaques bruteforce) :
```bash
sudo apt install fail2ban -y
```

Surveiller les connexions SSH :
```bash
sudo journalctl -u ssh
```
<br>

**🧠 En résumé :**

Avant de déployer un service web, il faut d’abord sécuriser la base système.
Une machine non protégée est une cible facile, même si elle n’héberge qu’un petit site.

## 🔥 1.7 Configuration et sécurisation avec iptables

`iptables` est un pare-feu intégré à Linux permettant de filtrer, rediriger ou bloquer le trafic réseau selon des règles personnalisées.  
Lors de la configuration du serveur, il a été nécessaire d’ajouter certaines règles afin d’autoriser les flux essentiels (SSH, HTTP, HTTPS), tout en conservant une **politique par défaut restrictive** (`deny any`).

### 🧱 Bonnes pratiques de sécurité

- Toujours **bloquer par défaut** tous les flux entrants.
- N’autoriser que les ports nécessaires **(ex. : 22, 80, 443).**
- Journaliser les paquets refusés pour détecter les tentatives d’intrusion.
- Sauvegarder la configuration pour qu’elle soit persistante après redémarrage.

### ⚙️ Exemple de configuration minimale

```bash
# Réinitialiser les règles existantes
sudo iptables -F

# Définir la politique par défaut sur DROP (deny all)
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Autoriser les connexions établies et internes
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -i lo -j ACCEPT

# Autoriser les ports nécessaires : SSH (22), HTTP (80), HTTPS (443)
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# (Optionnel) Journaliser les paquets bloqués
sudo iptables -A INPUT -j LOG --log-prefix "iptables denied: " --log-level 7
```

**💾 Sauvegarde et persistance des règles**
```bash
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
sudo netfilter-persistent reload
```

Grâce à cette configuration, le serveur est protégé contre les connexions non autorisées, tout en permettant uniquement le trafic nécessaire à son bon fonctionnement.
Cette approche s’inscrit dans une logique `"deny by default"`, pilier fondamental de la sécurité réseau.
