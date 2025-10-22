# DSMS - Version MySQL

Projet de base de données distribuées sur une seule machine avec trois régions (Dakar, Thiès, Saint-Louis).

## 🧩 Technologies
- Java 21
- Spring Boot 3.2
- MySQL 8+
- Thymeleaf

## ⚙️ Pré-requis
Installer :
- Java 21
- Maven
- MySQL

Créer les bases et utilisateur :
```sql
CREATE DATABASE ventes_dakar;
CREATE DATABASE ventes_thies;
CREATE DATABASE ventes_stlouis;

CREATE USER 'dsms_user'@'%' IDENTIFIED BY 'dsms_pass';
GRANT ALL PRIVILEGES ON ventes_dakar.* TO 'dsms_user'@'%';
GRANT ALL PRIVILEGES ON ventes_thies.* TO 'dsms_user'@'%';
GRANT ALL PRIVILEGES ON ventes_stlouis.* TO 'dsms_user'@'%';
FLUSH PRIVILEGES;
```

## ▶️ Exécution
```bash
mvn clean package
java -jar target/dsms-mysql-0.0.1-SNAPSHOT.jar
```
Accéder à [http://localhost:8080](http://localhost:8080)

## 🧠 Fonctionnalités
- Trois DataSources : Dakar, Thiès, Saint-Louis
- Synchronisation automatique (last-write-wins)
- Interface web (Thymeleaf)
- Synchronisation manuelle

## 🧰 Tests conseillés
1. Ajouter une vente dans chaque région
2. Lancer une synchronisation manuelle
3. Vérifier la propagation dans les trois bases
4. Tester les conflits (mise à jour + timestamp)


Files added:
- sql/init_mysql.sql (create DBs & user)
- REPORT.md (explanatory report and test scenarios)
