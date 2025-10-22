# Rapport explicatif — Projet DSMS (MySQL)

## Objectif
Ce projet met en œuvre une application Spring Boot (Java 21) qui simule une base de données distribuée sur une seule machine à l'aide de **trois bases MySQL** (Dakar, Thiès, Saint‑Louis). La synchronisation entre bases est automatique et configurable, basée sur une stratégie **last-write-wins**.

---

## Résumé de l'architecture
- Application Spring Boot (package `com.example.dsms`)
- Entité unique : `Vente`
- Trois DataSources et trois `JpaRepository` séparés :
  - `VenteRepositoryDakar` → base `ventes_dakar`
  - `VenteRepositoryThies` → base `ventes_thies`
  - `VenteRepositoryStl`  → base `ventes_stlouis`
- `MultiVenteService` : opérations CRUD par région et vue consolidée
- `SyncService` : scheduler qui exécute la synchronisation (configurable via `sync.interval`)
- UI : Thymeleaf, page principale avec formulaire d'ajout et bouton de synchronisation manuelle

---

## Critères d'évaluation — conformité

### 1. Fonctionnement correct des 3 bases et synchronisation
- Les trois DataSources sont configurées séparément dans `application.yml`.
- Le `SyncService` lit les enregistrements de chaque base, choisit la version la plus récente (sur `updatedAt`) et réplique/écrase les versions plus anciennes dans les autres bases.
- `sql/init_mysql.sql` permet de créer rapidement les 3 bases et l'utilisateur `dsms_user` avant d'exécuter l'application.
- La propriété `spring.jpa.hibernate.ddl-auto: update` crée automatiquement la table `vente` dans chaque base au premier démarrage.

**Vérification manuelle proposée :**
1. Lancer MySQL et exécuter `sql/init_mysql.sql`.
2. Démarrer l'application.
3. Ajouter une vente dans la région Dakar via l'UI → vérifier qu'elle apparaît dans la table `vente` de `ventes_dakar`.
4. Cliquer sur "Synchronisation manuelle" → vérifier que la même ligne est ajoutée dans `ventes_thies` et `ventes_stlouis`.

---

### 2. Code structuré et clair
- Code organisé par package : `model`, `dakar.repository`, `thies.repository`, `stl.repository`, `service`, `config`, `controller`.
- `DataSource` configurations séparées : classes `DakarDataSourceConfig`, `ThiesDataSourceConfig`, `StlDataSourceConfig`.
- Services découpés en responsabilités : `MultiVenteService` pour opérations, `SyncService` pour réplication.
- README et commentaires inclus pour faciliter la lecture.

---

### 3. Interface web opérationnelle
- Template Thymeleaf `index.html` : formulaire d'ajout (produit, montant, région), bouton de synchronisation manuelle, et table listant les ventes consolidées.
- Accessible par défaut à `http://localhost:8080`.

---

### 4. Rapport explicatif et démonstration des scénarios
- Ce fichier `REPORT.md` décrit le fonctionnement, la procédure de tests et les scénarios de démonstration.
- Scénarios de démonstration fournis ci-dessous.

---

## Scénarios de test et démonstration

### Scénario A — Ajout et propagation
1. Via l'UI, ajouter une vente region=`dakar`.
2. Vérifier présence immédiate dans `ventes_dakar.vente`.
3. Appuyer sur "Synchronisation manuelle".
4. Vérifier présence de l'enregistrement dans `ventes_thies.vente` et `ventes_stlouis.vente`.

### Scénario B — Conflit et last-write-wins
1. Insérer le même `id` dans `ventes_dakar` et `ventes_thies` (ou créer dans une DB, copier l'UUID et modifier champ `montant` dans l'autre).
2. Modifier manuellement la colonne `updated_at` (ou mettre à jour la ligne via l'UI) pour simuler des horodatages différents.
3. Lancer la synchronisation manuelle.
4. La version avec le `updatedAt` le plus récent est répliquée partout (last-write-wins).

### Scénario C — Panne et récupération
1. Arrêter temporairement une instance MySQL (ex : rendre inaccessible `ventes_thies` en stoppant MySQL ou via réseau).
2. Ajouter des ventes dans les autres régions.
3. Redémarrer MySQL.
4. Lancer la synchronisation manuelle — les enregistrements manquants doivent être répliqués.

---

## Limitations & améliorations possibles
- Pas de gestion des transactions distribuées (XA) — risque de partial commits en cas de pannes au milieu d'une sync : acceptable pour le but pédagogique.
- Pas de journal de synchronisation ni d'ID de changement incrémental (CDC) — peut être ajouté en bonus.
- Améliorations : réplication incrémentale, gestion avancée des conflits (merge fields), dashboard de monitoring, tests automatisés.

---

## Fichiers utiles dans l'archive
- `sql/init_mysql.sql` : script d'initialisation MySQL
- `src/main/resources/application.yml` : configuration MySQL
- `README.md` : instructions d'exécution
- `REPORT.md` : ce rapport

---

## Commandes rapides
```bash
# lancer MySQL init (en tant que root)
mysql -u root -p < sql/init_mysql.sql

# compiler et lancer
mvn clean package
java -jar target/dsms-mysql-0.0.1-SNAPSHOT.jar
```

Merci — le projet est prêt pour démonstration. Si tu veux, je peux :
- générer un **script de test automatisé** (JUnit + tests d'intégration) ; ou
- ajouter un **dashboard de logs** et endpoints `actuator` pour monitorer la sync.
