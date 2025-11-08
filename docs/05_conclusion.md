
# 🏁 Conclusion du projet

Ce projet a permis de **migrer un site WordPress complet et sécurisé** sur une instance **Oracle Cloud Free Tier**, en suivant une approche proche d’une mise en production réelle.

---

## 🚀 Ce qui a été accompli

- Installation et configuration d’un **serveur Ubuntu** sur Oracle Cloud (solution gratuite).
- Mise en place d’un environnement **Nginx + PHP-FPM + MariaDB**.
- Déploiement complet de **WordPress**.
- Configuration d’un **nom de domaine personnalisé** et gestion DNS (enregistrement A).
- Mise en place d’un **certificat SSL** via Let’s Encrypt.
- Redirection automatique **HTTP → HTTPS**.
- Correction des **erreurs CORS et mixed content**.
- Optimisation de la **sécurité et des performances** (cache, uploads, accès).

---

## 🔐 Enjeux principaux

- Sécuriser les échanges avec SSL/TLS.
- Assurer une redirection fiable entre IP, domaine et sous-domaines.
- Maintenir une configuration Nginx propre et maintenable.

---

## 🧩 Améliorations possibles

- Ajouter **un reverse proxy** ou **un CDN** (Cloudflare) pour renforcer la sécurité.  
- Mettre en place des **sauvegardes automatisées**.  
- Configurer un **monitoring** de disponibilité du site (Uptime Kuma, StatusCake…).  
- Déployer via **GitHub Actions** pour un workflow CI/CD simple.

<br>

Cette expérience démontre la maîtrise du cycle complet :  
**Cloud → Serveur → Web → Sécurité → Optimisation.**
