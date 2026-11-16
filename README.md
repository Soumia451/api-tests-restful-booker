# Tests API automatisés — Postman / Newman

![Tests API](https://github.com/Soumia451/api-tests-restful-booker/actions/workflows/api-tests.yml/badge.svg)

Tests automatisés de l'API REST **[Restful Booker](https://restful-booker.herokuapp.com/apidoc/index.html)** (gestion de réservations d'hôtel), écrits dans Postman et exécutés avec Newman à chaque push grâce à GitHub Actions.

## Objectifs

- Vérifier le parcours complet d'une réservation : création, consultation, recherche, modification, suppression
- Couvrir les cas d'erreur : identifiants invalides, accès sans token, données incomplètes, ressource inexistante
- Contrôler les données renvoyées (valeurs et schéma JSON), pas seulement les codes de statut
- Signaler les écarts entre le comportement observé et le comportement attendu d'une API REST

## Scénarios couverts (14 requêtes)

| # | Domaine | Scénario | Résultat attendu |
|---|---|---|---|
| 1 | Disponibilité | Healthcheck | L'API répond en moins de 5 s |
| 2 | Authentification | Connexion valide | Un token est renvoyé |
| 3 | Authentification | Mot de passe invalide | Pas de token, message « Bad credentials » |
| 4 | Réservation | Créer une réservation | Identifiant renvoyé, données et schéma conformes |
| 5 | Réservation | Consulter la réservation créée | Les données correspondent |
| 6 | Réservation | Rechercher par nom | La réservation apparaît dans les résultats |
| 7 | Réservation | Modifier (PUT, avec token) | Le prix est mis à jour |
| 8 | Réservation | Modifier partiellement (PATCH) | Seul le champ envoyé change |
| 9 | Réservation | Supprimer | Suppression acceptée |
| 10 | Réservation | Consulter après suppression | 404 |
| 11 | Sécurité | Modifier sans token | 403 |
| 12 | Sécurité | Supprimer avec un token invalide | 403 |
| 13 | Erreur | Créer une réservation incomplète | Refusée |
| 14 | Erreur | Consulter une réservation inexistante | 404 |

Les requêtes sont chaînées : le token et l'identifiant de réservation sont enregistrés dans des variables et réutilisés par les requêtes suivantes.

## Anomalies relevées

| ID | Requête | Attendu | Observé | Gravité |
|---|---|---|---|---|
| AN-01 | `POST /auth` avec mot de passe invalide | 401 Unauthorized | 200 OK + `{"reason": "Bad credentials"}` | Moyenne : un client ne peut pas détecter l'échec par le code HTTP |
| AN-02 | `POST /booking` avec champs obligatoires manquants | 400 Bad Request | 500 Internal Server Error | Majeure : la validation des données n'est pas gérée côté serveur |
| AN-03 | `DELETE /booking/{id}` | 200 OK ou 204 No Content | 201 Created | Mineure : code incohérent avec l'action |

Les tests marqués `[AN-xx]` vérifient le comportement actuel pour que la campagne reste stable : si l'anomalie est corrigée, le test échoue et signale le changement.

## Stack technique

| Outil | Usage |
|---|---|
| Postman | Écriture des requêtes et des assertions |
| Newman | Exécution de la collection en ligne de commande |
| newman-reporter-htmlextra | Rapport HTML détaillé |
| GitHub Actions | Exécution automatique à chaque push et chaque lundi |

## Structure du projet

```
collections/
└── restful-booker.postman_collection.json   # Requêtes, assertions et variables
.github/workflows/
└── api-tests.yml                            # Pipeline d'intégration continue
```

## Lancer les tests

**Dans Postman :** File → Import → choisir le fichier de la collection, puis « Run collection ».

**En ligne de commande :**

```bash
npm install -g newman newman-reporter-htmlextra
newman run collections/restful-booker.postman_collection.json -r cli,htmlextra
```

Le rapport HTML est généré dans le dossier `newman/`.

**Sur GitHub :** onglet *Actions* → dernière exécution → artefact `rapport-newman`.

## Autrice

Soumia Sadki — Testeuse QA junior · [LinkedIn](https://www.linkedin.com/in/soumia-sadki/)
