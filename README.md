# Accès à l'infrastructure pour un projet en mode produit (Lean Startup/BetaGouv)
Document visant à expliquer les besoins d'acces a l'infra dans un produit (Mode produit, LeanStartup) ainsi que les justifications qui explique c'est besoins


***

## 1. Introduction : Comprendre les enjeux des deux côtés

### 1.1. Reconnaître les contraintes structurelles

Nous comprenons que les équipes SI ont des **obligations historiques** :

* Sécurité des données et conformité (RGPD, normes internes).

* Stabilité et fiabilité des systèmes y compris critiques.

* Processus de validation et de contrôle hérités.

**Notre objectif n’est pas de remettre en cause ces contraintes**, mais de montrer comment un **accès ciblé et encadré** à certaines couches de l’infra peut **améliorer l’agilité sans compromettre la sécurité**.

Notre objectif est de **trouver ensemble** comment fonctionner en **agile** grâce à un accès **accès ciblé et encadré** à certaines couches de l’infra **sans compromettre la sécurité et la stabilité des systèmes et en s'intégrant dans les processus DSI**.

### 1.2. Pourquoi ce document ?

Ce document vise à :

* **Lister clairement** les besoins d’accès des développeurs (critiques, optionnels, non nécessaires).

* **Expliquer** pourquoi ces accès sont pertinents pour une approche produit itérative (ex. : SAMI).

* **Proposer un cadre** pour rassurer les équipes SI : traçabilité, principe de moindre privilège, audit.

* **Trouver un équilibre** entre le "mode produit idéal" (accès total) et les réalités structurelles.

***

## 2. Le paradoxe : Un produit agile avec une infra non agile

### 2.1. L’inadéquation des modèles

Un projet en mode produit agile (Lean Startup, DevOps) **nécessite une infra tout aussi agile** pour éviter des **goulots d’étranglement** qui annulent les bénéfices de l’agilité.

* **Problème** : Avoir un produit en mode agile et une infra en cycle en V (ou non adaptée) crée **plus de ralentissements** que d’avoir tout en cycle en V.

* **Exemple** : Si les développeurs itèrent rapidement mais doivent attendre des semaines pour un déploiement ou un accès aux logs, **l’agilité perd tout son sens**.

### 2.2. L’agilité, c’est aussi (surtout) une question de déploiement

* **Principe Lean Startup** : "Construire, Mesurer, Apprendre" (Eric Ries). **Sans accès aux logs, au monitoring et à un déploiement rapide, impossible de "mesurer" et d’apprendre"** en temps réel.

* **Risque** : Les freins infrastructurels transforment l’agilité en **poudre aux yeux** : on développe en agile, mais on déploie en cycle en V.

* **Conséquence** : Perte de réactivité, frustration des équipes, et **annulation des gains d’efficacité** attendus.

### 2.3. L’hybridation comme solution

* **Objectif** : Trouver un **modèle hybride** où l’infra s’adapte aux besoins du produit agile, **sans sacrifier la sécurité**.

* **Exemple** : Donner aux devs un accès **ciblé et contrôlé** aux couches applicatives (logs, monitoring, déploiement en staging), tout en gardant les couches basses (hardware, VM) sous contrôle SI.

***

## 3. Besoins d’accès : Ce qui est critique, optionnel, non nécessaire

### 3.1. Couches de l’infra et besoins associés

| Couche                                           | Besoin d’accès                                            | Justification                                                                                                              | Niveau de criticité                            |
| ------------------------------------------------ | --------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| **Hardware** (machines physiques)                | ❌ Non                                                     | Géré par les équipes SI, pas de valeur ajoutée pour les devs.                                                              | Non nécessaire                                 |
| **Infra bas niveau** (VM, Kubernetes)            | ❌ Non                                                     | Orchestration et gestion des clusters restent du ressort SI.                                                               | Non nécessaire                                 |
| **Conteneurs applicatifs** (Docker, Pods)        | ✅ **Oui** (déploiement, redémarrage, monitoring basique)  | Permet de déployer des corrections ou nouvelles versions **sans dépendre d’un tiers**. Ex. : Rollback urgent après un bug. | **Critique**                                   |
| **Bases de données**                             | ✅ **Oui** (lecture en prod, lecture/écriture en non-prod) | Diagnostiquer des incohérences, valider des migrations. Ex. : Vérifier pourquoi une requête est lente.                     | **Critique**                                   |
| **Logs & Monitoring** (Grafana, ELK, Prometheus) | ✅ **Oui** (lecture temps réel)                            | Comprendre les bugs, surveiller les performances. Ex. : Identifier une erreur 500 en prod.                                 | **Critique**                                   |
| **Services tiers** (FTP, APIs externes)          | ✅ **Oui** (visibilité, redémarrage)                       | Diagnostiquer des pannes (ex. : fichier non transféré) et relancer un service.                                             | Optionnel (peut être contourné temporairement) |
| **Déploiement en production**                    | ✅ **Oui** (via pipeline automatisé + validation)          | Autonomie pour livrer des corrections **sans attendre un processus manuel**.                                               | **Critique**                                   |

**→ Résumé** :

* **Critique** : Conteneurs, logs, monitoring, bases de données (non-prod), déploiement en staging/prod via pipeline.

* **Optionnel** : Services tiers (peut être géré par SI en cas de blocage).

* **Non nécessaire** : Hardware, VM, orchestration bas niveau.

***

## 4. Pourquoi ces accès sont-ils pertinents ?

### 4.1. Itération rapide et autonomie

* **Problème actuel** : Chaque déploiement ou diagnostic nécessite une **demande externe** → délais, blocages, frustration.

* **Solution** : Accès direct aux couches applicatives → **réduction des cycles de feedback** (ex. : corriger un bug en 1h au lieu de 1 jour).

* **Source** : ["Les développeurs, devenus Opérateurs applicatifs dans une organisation DevOps, consomment les ressources sans se soucier de la technologie sous-jacente."](https://blog.octo.com/linfrastructure-agile/) (OCTO, 2022)

### 4.2. Meilleure qualité et résilience

* **Exemple** : Si un conteneur plante en production, pouvoir le **redémarrer immédiatement** évite une indisponibilité prolongée.

* **Source** : ["La livraison continue permet aux développeurs de toujours disposer d’un artéfact prêt au déploiement après des tests normalisés."](https://iac.goffinet.org/infrastructure-as-code/introduction-devops/) (DevOps/IaC)

### 4.3. Visibilité transverse pour éviter les silos

* **Risque** : Séparer code et infra crée des **zones d’ombre** où personne n’a une vue globale.

* **Bénéfice** : Une équipe produit avec une **visibilité sur les logs et le monitoring** prend de meilleures décisions techniques.

* **Source** : ["Les équipes techniques peuvent échanger sur leurs contraintes et l’état d’avancement du projet, ce qui rapproche toutes les parties prenantes."](https://www.projexion.com/carrefour-apprentissage/rex/travailler-mode-agile/) (Projexion, 2025)

### 4.4. Réduire les contournements (Shadow IT)

* **Risque** : Sans accès, les devs peuvent utiliser des **solutions non approuvées** (ex. : logs locaux, outils externes).

* **Solution** : Donner un accès **cadre et sécurisé** limite ce risque.

* **Source** : ["Si les développeurs n’ont pas accès aux logs ou au monitoring, ils risquent de contourner les processus officiels."](https://www.reddit.com/r/devops/comments/u2xz7e/should_devs_have_access_to_production/) (r/DevOps, 2022)

***

## 5. Comment rassurer les équipes SI ?

### 5.1. Principe de moindre privilège

* **Accorder uniquement les droits nécessaires** :

  * Lecture seule en production (sauf pour les déploiements via pipeline).

  * Lecture/écriture en non-production.

  * Traçabilité de toutes les actions (qui a redémarré un conteneur ?).

### 5.2. Audit et traçabilité

* **Logger toutes les actions** dans un outil centralisé (ex. : Splunk, ELK).

* **Revues régulières** des accès avec les équipes SI.

### 5.3. Formation et accompagnement

* **Former les devs** aux bonnes pratiques de sécurité (ex. : ne jamais modifier la prod directement).

* **Désigner un "ambassadeur DevOps"** dans l’équipe pour faciliter le dialogue avec la SI.

### 5.4. Pipeline de déploiement sécurisé

* **Automatiser les déploiements** en production via CI/CD (ex. : GitHub Actions, GitLab CI).

* **Validation manuelle** requise pour les déploiements critiques.

***

## 6. Exemples concrets et retours d’expérience

### 6.1. Cas d’une entreprise ayant ouvert les accès

* **Contexte** : Une équipe produit bloquée par des processus SI lents.

* **Solution** : Accès en lecture aux logs et monitoring, + déploiement en staging autonome.

* **Résultat** : Réduction de 70% des temps de résolution de bugs, meilleure collaboration SI/devs.

* **Source** : ["Les équipes techniques et métiers collaborent mieux grâce à une visibilité partagée."](https://www.projexion.com/carrefour-apprentissage/rex/travailler-mode-agile/) (Projexion, 2025)

### 6.2. Lean Startup et besoin d’itération

* **Principe** : "Construire, Mesurer, Apprendre" (Eric Ries).

* **Application** : Sans accès aux logs et au monitoring, impossible de **mesurer** l’impact des changements.

* **Source** : ["Chaque produit est une expérimentation dont le but est de tirer des enseignements."](https://www.leanenligne.com/blog/lean-startup) (Lean Startup, 2025)

***

## 7. Proposition de mise en œuvre progressive

| Étape | Action                                                                   | Responsable         |
| ----- | ------------------------------------------------------------------------ | ------------------- |
| 1     | Audit des besoins réels avec les devs et la SI.                          | SI + Équipe produit |
| 2     | Mise en place d’un accès en lecture aux logs/monitoring (env. non-prod). | SI                  |
| 3     | Formation des devs aux bonnes pratiques de sécurité.                     | SI                  |
| 4     | Déploiement autonome en staging via pipeline CI/CD.                      | SI + Devs           |
| 5     | Évaluation et ajustement (feedback, audit).                              | SI + Équipe produit |

***

## 8. Conclusion : Un compromis gagnant-gagnant

### 8.1. Pour les équipes SI

* **Maîtrise des risques** (traçabilité, principe de moindre privilège).

* **Réduction du shadow IT** (moins de contournements).

* **Meilleure collaboration** avec les équipes produit.

### 8.2. Pour les développeurs

* **Autonomie et réactivité** (moins de blocages).

* **Meilleure qualité** (diagnostics rapides, déploiements fiables).

* **Alignement avec les méthodes agiles/lean** (itération, feedback).

### 8.3. Prochaine étape

* **Valider ce cadre** avec les équipes SI pour lancer un **pilote** sur le projet SAMI.

* **Ajuster** en fonction des retours terrain.

***

### Références utiles :

* [L’Infrastructure agile – OCTO Talks](https://blog.octo.com/linfrastructure-agile/)

* [Introduction à DevOps et Infrastructure as Code](https://iac.goffinet.org/infrastructure-as-code/introduction-devops/)

* [Retours d’expérience sur l’agilité en infrastructure](https://www.projexion.com/carrefour-apprentissage/rex/travailler-mode-agile/)

* [Pourquoi les devs devraient-ils avoir accès à la production ? (r/DevOps)](https://www.reddit.com/r/devops/comments/u2xz7e/should_devs_have_access_to_production/)

***

### Points clés à retenir :

* **L’agilité ne se limite pas au développement** : sans infra adaptée, les gains sont annulés.

* **L’hybridation est possible** : accès ciblé aux couches applicatives + contrôle SI sur les couches basses.

* **L’objectif est commun** : livrer de la valeur rapidement, en toute sécurité.
