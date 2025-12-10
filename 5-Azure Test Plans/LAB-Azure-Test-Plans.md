# LAB 06 : GESTION COMPLÈTE DES TESTS DANS AZURE DEVOPS

## Informations Générales

| Élément | Description |
|---------|-------------|
| **Durée** | 4-5 heures (modulable par composantes) |
| **Niveau** | Intermédiaire à Avancé |
| **Prérequis** | Accès à Azure DevOps, connaissance basique des cycles de développement |
| **Objectifs** | Maîtriser la planification, exécution et rapportage des tests manuels dans Azure Test Plans |
| **Plateforme** | Azure DevOps Services / Azure DevOps Server 2022+ |

---

## OBJECTIFS D'APPRENTISSAGE

À la fin de ce LAB, l'apprenant sera capable de :

1. **Créer et structurer des plans de test** avec suites et cas de test
2. **Planifier les tests** en définissant les configurations et les matrices de test
3. **Exécuter les tests manuels** et enregistrer les résultats
4. **Finaliser les tests manuels** avec traçabilité et documentation
5. **Générer des rapports d'avancement** et analyser les métriques
6. **Lier les résultats de test** aux éléments de travail (User Stories, Bugs)
7. **Intégrer le test management** dans le cycle DevOps complet

---

## COMPOSANTE 1 : FONDAMENTAUX DE LA PLANIFICATION DES TESTS

### 1.1 Introduction aux Test Plans

**Objectif pédagogique** : Comprendre l'architecture des plans de test dans Azure DevOps

#### Concepts clés

Un **Plan de Test** dans Azure DevOps est composé de :

- **Test Plan** : Conteneur principal qui regroupe tous les éléments de test pour une version ou un sprint
- **Test Suite** : Groupement logique de cas de test (statique, dynamique ou basé sur les exigences)
- **Test Case** : Cas de test individuel avec étapes définies et résultats attendus
- **Configuration** : Ensemble de paramètres (navigateur, OS, résolution) pour exécuter les tests
- **Test Run** : Exécution d'une série de cas de test avec enregistrement des résultats

#### Types de suites de test

| Type | Description | Usage |
|------|-------------|-------|
| **Statique** | Groupement manuel de cas de test | Regroupement personnalisé |
| **Dynamique (Query-based)** | Suites basées sur des requêtes WIQL | Mise à jour automatique |
| **Basé sur exigences** | Suites associées aux User Stories | Traçabilité complète |

### 1.2 Création d'un Plan de Test

#### Étape 1 : Accéder à Test Plans

```
Navigation : Projet Azure DevOps
→ Test Plans (dans le menu principal)
→ Test plans
→ Bouton "+ New Test Plan"
```

#### Étape 2 : Configurer le plan

**Informations requises** :

- **Nom** : Exemple : "Planification des tests - Sprint 24"
- **Zone d'itération** : Sélectionner le sprint ou milestone cible
- **Description** : Contexte et objectifs du plan
- **Build** : Associer un build spécifique (optionnel)

#### Étape 3 : Ajouter des configurations

**Objectif** : Définir les environnements de test

1. Accéder à l'onglet "Configurations"
2. Créer des configurations par :
   - **Système d'exploitation** (Windows 10, Windows 11, macOS)
   - **Navigateur** (Chrome, Firefox, Edge)
   - **Résolution d'écran** (1024x768, 1920x1080)
   - **Langue** (FR, EN, AR)

**Exemple** :
```
Configuration 1 : Windows 10 + Chrome + 1920x1080 + FR
Configuration 2 : Windows 11 + Edge + 1920x1080 + FR
Configuration 3 : macOS + Safari + 1440x900 + FR
```

### 1.3 Créer des Cas de Test

#### Structure d'un cas de test

```
Titre : Vérifier l'authentification avec email valide
ID : TC-001
Zone : Authentification
Priorité : 1 (Critique)

Étapes :
  1. Naviguer vers la page de connexion
     ↓ Résultat attendu : La page charge correctement
  
  2. Entrer email : test@exemple.com
     ↓ Résultat attendu : L'email s'affiche dans le champ
  
  3. Entrer mot de passe : Test123!
     ↓ Résultat attendu : Le mot de passe est masqué
  
  4. Cliquer sur le bouton "Connexion"
     ↓ Résultat attendu : Redirection vers le dashboard utilisateur
```

#### Procédure de création

1. **Dans le Test Plan** : Cliquer sur "+ New Test Case"
2. **Remplir les informations** :
   - Titre descriptif
   - Zone de test (Feature, Epic)
   - Étapes précises et vérifiables
3. **Associer des configurations**
4. **Sauvegarder**

**Bonnes pratiques** :

- ✅ Utiliser des verbes d'action clairs ("Vérifier", "Cliquer", "Valider")
- ✅ Une action et un résultat attendu par étape
- ✅ Éviter les cas de test trop longs (max 10 étapes)
- ✅ Inclure des données de test spécifiques
- ❌ Éviter : "faire un test", "vérifier si ça marche"

#### Lier les cas de test aux User Stories

1. **Depuis le cas de test** : Cliquer sur "Link work items"
2. **Sélectionner la User Story associée**
3. **Confirmer** : La traçabilité s'établit automatiquement

**Avantage** : Chaque User Story est tracée par ses cas de test. Les rapports montrent le taux de couverture.

---

## COMPOSANTE 2 : EXÉCUTION DES TESTS MANUELS

### 2.1 Préparer l'environnement de test

#### Checklist de préparation

- [ ] Vérifier que toutes les configurations sont définies
- [ ] Assigner les cas de test aux testeurs
- [ ] Préparer les données de test (comptes, données fictives)
- [ ] Vérifier l'accès à l'environnement de test
- [ ] Préparer Microsoft Test Runner ou Web-based Test Runner
- [ ] Définir les critères d'acceptation

### 2.2 Lancer une exécution de test

#### Étape 1 : Créer une Test Run

```
Test Plans → [Votre Plan] → Onglet Execute
→ Bouton "Run for web application" ou "Run for desktop application"
```

#### Étape 2 : Sélectionner les configurations

**Fenêtre de configuration** :

```
Plan de test : Planification Sprint 24
Suite de test : Authentification
Configuration sélectionnée : Windows 10 + Chrome + 1920x1080
Testeur assigné : [Vous]
Build : Build-2024-12-02
```

#### Étape 3 : Exécuter les cas de test

**Web Runner (Interface)** :

```
┌─────────────────────────────────────────────┐
│ Test Runner                                 │
│─────────────────────────────────────────────│
│ Cas 1/5 : Vérifier l'authentification      │
│─────────────────────────────────────────────│
│ ✓ Étape 1 : Page de connexion affichée    │
│ ✓ Étape 2 : Email saisi correctement      │
│ ✓ Étape 3 : Mot de passe masqué           │
│ ✓ Étape 4 : Redirection réussie           │
│─────────────────────────────────────────────│
│ [Passer] [Échec] [Bloquer] [Reprendre]    │
└─────────────────────────────────────────────┘
```

### 2.3 Enregistrer les résultats de test

#### Résultats possibles

| Résultat | Icône | Signification | Action |
|----------|-------|---------------|--------|
| **Réussi** | ✓ | Tous les résultats attendus confirmés | Continuer |
| **Échoué** | ✗ | Au moins une étape ne correspond pas | Créer un bug |
| **Bloqué** | ⊗ | Impossible de continuer | Eskalader |
| **Non exécuté** | — | Cas non testé | Reporter |

#### Créer un bug lors de l'exécution

**Procédure intégrée** :

1. Cliquer sur "Create bug" dans Test Runner
2. Le bug capture automatiquement :
   - Configuration de test
   - Cas de test associé
   - Screenshots (si capturés)
   - Historique des étapes exécutées
3. Remplir les champs additionnels :
   - Titre descriptif du bug
   - Étapes de reproduction
   - Résultat attendu vs. réel
   - Sévérité
4. Sauvegarder : Le bug est lié au test run

**Exemple de bug** :

```
Titre : Bouton "Connexion" n'est pas responsive
Sévérité : Critique
Étapes :
  1. Ouvrir la page de connexion
  2. Remplir les champs email et mot de passe
  3. Cliquer sur le bouton "Connexion"
Résultat attendu : Page charge et affiche le dashboard
Résultat réel : Bouton ne répond pas, page reste bloquée
Environnement : Windows 11 + Edge 2024
```

### 2.4 Finaliser les tests manuels

#### Vérification avant clôture

**Checklist de finalisation** :

- [ ] Tous les cas de test exécutés ou documentés
- [ ] Résultats enregistrés (Pass/Fail/Blocked)
- [ ] Bugs créés avec description complète
- [ ] Screenshots et logs attachés
- [ ] Données de test nettoyées
- [ ] Environnement de test restauré

#### Clôturer un Test Run

```
Test Plans → Test Runs
Sélectionner le run → Bouton "Close"
Vérifier le résumé :
  - X cas réussis
  - Y cas échoués
  - Z cas bloqués
```

---

## COMPOSANTE 3 : RAPPORTS D'AVANCEMENT

### 3.1 Générer des rapports

#### Types de rapports disponibles

##### A. Rapport d'exécution des tests

**Accès** : Test Plans → [Votre Plan] → Onglet Reports

Éléments affichés :
```
├── Nombre total de tests
├── Tests réussis (%)
├── Tests échoués (%)
├── Tests non exécutés (%)
├── Tendance dans le temps
└── Tests par configuration
```

##### B. Rapport de traçabilité

**Objectif** : Lier User Stories → Test Cases → Test Results

```
User Story US-101 : Authentification utilisateur
  ├── Test Case TC-001 ✓ Réussi
  ├── Test Case TC-002 ✗ Échoué (Bug B-043)
  └── Test Case TC-003 ✓ Réussi
  
Couverture : 2/3 = 67%
Statut : PARTIELLEMENT VALIDÉ
```

##### C. Dashboard de test

**Configuration** :

1. Accéder à Dashboards du projet
2. Ajouter des widgets de test
3. Widgets disponibles :
   - Test Results Trend
   - Requirement Coverage
   - Tests Failed
   - Test Execution Rate

**Exemple de dashboard** :

```
┌─────────────────┬──────────────────┐
│ Tests Réussis   │ Tests Échoués     │
│ 45/50 (90%)     │ 5/50 (10%)        │
└─────────────────┴──────────────────┘
┌──────────────────────────────────────┐
│ Taux d'exécution par jour            │
│ ▁▂▃▄▅▆▇█ (Tendance croissante)      │
└──────────────────────────────────────┘
┌──────────────────────────────────────┐
│ Couverture par User Story            │
│ US-101 : 100% ✓                      │
│ US-102 : 67%  ⚠                      │
│ US-103 : 0%   ✗                      │
└──────────────────────────────────────┘
```

### 3.2 Analyser les métriques

#### Métriques clés

| Métrique | Calcul | Interprétation |
|----------|--------|----------------|
| **Taux de couverture** | (Tests exécutés / Total tests) × 100 | % de tests complétés |
| **Taux de réussite** | (Tests réussis / Tests exécutés) × 100 | Qualité du produit |
| **Temps moyen par test** | Temps total / Nombre de tests | Efficacité de test |
| **Découverte de bugs** | Nombre de bugs / Cas de test | Densité de défauts |
| **Déviation de calendrier** | (Tests prévus - Tests réels) | Retard de test |

#### Interprétation des résultats

**Scénario 1** : Taux de réussite 95%, 1 bug critique

→ **Recommandation** : Reporter la release, corriger le bug critique, ré-tester

**Scénario 2** : Taux de couverture 70%, délai restant 2 jours

→ **Recommandation** : Prioriser les cas de test critiques, accepter les risques moins importants

**Scénario 3** : 30 bugs découverts en 100 tests

→ **Recommandation** : Revoir les critères d'acceptation, augmenter la qualité de développement

---

## COMPOSANTE 4 : INTÉGRATION DANS AZURE DEVOPS

### 4.1 Lier les tests à la pipeline CI/CD

#### Approche manuelle + Automatisée

```
Development Cycle
│
├─ Développement (Repos)
│  └─ Commit → Build Pipeline
│
├─ Build Pipeline
│  ├─ Compiler
│  ├─ Tests unitaires (automatisés)
│  └─ Déployer vers Test
│
├─ Manual Testing (Test Plans)
│  ├─ Exécuter les cas de test
│  ├─ Enregistrer les résultats
│  └─ Créer les bugs
│
└─ Release Pipeline
   ├─ Approver Release
   └─ Déployer en Production
```

### 4.2 Traçabilité complète

#### Architecture de traçabilité

```
Epic E-01 : Gestion d'authentification
  │
  ├─ Feature F-01 : Authentification par email
  │   │
  │   ├─ User Story US-101 : "En tant qu'utilisateur..."
  │   │   │
  │   │   ├─ Task T-101 : Développement
  │   │   ├─ Task T-102 : Révision code
  │   │   │
  │   │   └─ Test Cases
  │   │       ├─ TC-001 ✓
  │   │       ├─ TC-002 ✗ → Bug B-043
  │   │       └─ TC-003 ✓
  │   │
  │   └─ Bug B-043 : Fix et re-test
  │
  └─ Rapport : Couverture = 100%, Réussite = 100% ✓
```

### 4.3 Gestion des configurations dans Azure DevOps

#### Matrice de configurations

Configurations × Cas de test = Test Matrix

```
Navigateurs : Chrome, Firefox, Edge (3)
OS : Windows 10, 11, macOS (3)
Résolutions : 1024x768, 1920x1080 (2)
Langues : FR, EN (2)

Matrice complète = 3 × 3 × 2 × 2 = 36 configurations
Cas de test : 50
Tests totaux requis : 50 × 36 = 1800 exécutions

Stratégie : Sélectionner les configurations critiques
→ Combinaison la plus fréquente chez les utilisateurs
```

---

## EXERCICES PRATIQUES

### Exercice 1 : Créer un plan de test complet

**Durée** : 30 minutes

**Scénario** : Vous testez une nouvelle fonctionnalité "Panier d'achat"

**Tâches** :

1. Créer un Plan de Test nommé "Test Panier - Sprint 24"
2. Ajouter 3 configurations (navigateurs/OS)
3. Créer 5 cas de test :
   - TC-001 : Ajouter un produit au panier
   - TC-002 : Modifier la quantité
   - TC-003 : Supprimer un produit
   - TC-004 : Vérifier le total
   - TC-005 : Procéder au paiement
4. Lier les cas de test à la User Story "US-105 : Panier d'achat"

**Critères d'évaluation** :

- ✓ Plan créé avec bonnes pratiques
- ✓ Tous les cas ont des étapes claires
- ✓ Traçabilité établie

---

### Exercice 2 : Exécuter et rapporter les tests

**Durée** : 45 minutes

**Conditions** : Utiliser le plan créé à l'Exercice 1

**Tâches** :

1. Créer une Test Run pour tous les cas
2. Exécuter chaque cas de test dans Chrome + Windows
3. Simuler les résultats :
   - TC-001 et TC-002 : Réussi ✓
   - TC-003 : Échoué ✗ (bug : bouton supprimer désactivé)
   - TC-004 et TC-005 : Réussi ✓
4. Créer un bug pour TC-003 avec description complète
5. Générer le rapport d'exécution

**Résultat attendu** :

```
Taux de réussite : 4/5 = 80%
Bugs découverts : 1
Couverture : 100%
```

---

### Exercice 3 : Analyser les métriques et proposer des actions

**Durée** : 30 minutes

**Données fournies** :

Test Run du projet fictif :
```
- Total tests : 50
- Réussis : 35 (70%)
- Échoués : 10 (20%)
- Bloqués : 5 (10%)
- Bugs découverts : 15
- Temps écoulé : 8 jours
- Temps restant : 2 jours
```

**Tâches** :

1. Analyser les métriques
2. Identifier les points d'inquiétude
3. Proposer 3 actions correctives
4. Recommander la décision (Continuer le test / Reporter / Relâcher)

**Justifier votre décision** avec données et contexte.

---

## BONNES PRATIQUES ET ANTIPATTERNS

### ✅ Bonnes pratiques

| Domaine | Pratique |
|---------|----------|
| **Structuration** | Organiser les cas en suites logiques par feature |
| **Nomenclature** | Utiliser des ID et noms standardisés (TC-001, TC-002) |
| **Documentation** | Chaque cas a étapes claires et résultats attendus |
| **Traçabilité** | Lier chaque test aux User Stories |
| **Configurations** | Tester sur les navigateurs/OS réels des utilisateurs |
| **Résultats** | Enregistrer systématiquement tous les résultats |
| **Bugs** | Créer des bugs immédiatement avec reproduction |
| **Rapports** | Générer des rapports réguliers (fin de sprint) |
| **Automatisation** | Automatiser les tests répétitifs et critiques |

### ❌ Antipatterns (à éviter)

| Antipattern | Risque |
|-------------|--------|
| Cas de test sans étapes précises | Tests incomplets, résultats inconsistants |
| Tests non tracés aux exigences | Couverture inconnue, tests redondants |
| Pas de documentations de bugs | Bugs mal reproduits, corrections lentes |
| Tester sans configurations définies | Résultats non fiables |
| Tests exécutés hors pipeline | Manque de traçabilité CI/CD |
| Rapports générés trop tard | Décisions tardives, délais impactés |
| Pas d'automatisation des tests | Temps perdu, tests peu fiables |

---

## RESSOURCES ET RÉFÉRENCES

### Documentation Azure DevOps

- [Créer des plans de test et suites](https://learn.microsoft.com/en-us/azure/devops/test/create-a-test-plan) (Microsoft Learn)
- [Exécuter des tests manuels](https://learn.microsoft.com/en-us/azure/devops/test/run-manual-tests) (Microsoft Learn)
- [Gestion des résultats de test](https://learn.microsoft.com/en-us/azure/devops/test/test-runs) (Microsoft Learn)
- [Rapportage des tests](https://learn.microsoft.com/en-us/azure/devops/boards/plans/safe-review-roadmaps-progress) (Microsoft Learn)

### Frameworks de test

- **Manual Testing** : Microsoft Test Runner
- **Automated Testing** : Selenium, JUnit, NUnit, TestNG
- **Integration** : Azure Pipelines (YAML/Classic)

### Métriques et KPIs

Référence SAFe et Agile :
- Velocity (Vélocité)
- Cycle Time (Temps de cycle)
- Lead Time (Délai de mise en marché)
- Cumulative Flow Diagram (CFD)

---

## ÉVALUATION FINALE

### Critères d'évaluation

| Critère | Points |
|---------|--------|
| Création de plans et cas de test | /20 |
| Exécution et documentation des tests | /20 |
| Création et gestion des bugs | /15 |
| Générations de rapports et analyse | /20 |
| Traçabilité et intégration DevOps | /15 |
| Respect des bonnes pratiques | /10 |
| **Total** | **/100** |

### Notes de passage

- **80-100** : Excellent - Prêt pour la production
- **70-79** : Bon - Quelques améliorations nécessaires
- **60-69** : Acceptable - Nécessite consolidation
- **< 60** : À reprendre

---

## CONCLUSION

Ce LAB couvre l'intégralité du cycle de gestion des tests manuels dans Azure DevOps, de la planification à la rapportage. Les compétences acquises permettent :

- ✓ Structurer les tests de manière professionnelle
- ✓ Documenter et tracer les résultats
- ✓ Collaborer efficacement en équipe
- ✓ Intégrer les tests dans un pipeline DevOps
- ✓ Prendre des décisions de release basées sur les données

**Étape suivante** : Appliquer ces pratiques en environnement réel et explorer l'automatisation des tests.

---

## ANNEXES

### Annexe A : Modèle de cas de test

```
**Test Case ID** : TC-XXX
**Titre** : [Description concise]
**Zone** : [Feature/Epic]
**Priorité** : [1-Critique / 2-Haut / 3-Normal / 4-Bas]
**Pré-conditions** : [État initial requis]

| Étape | Action | Résultat attendu |
|-------|--------|------------------|
| 1 | [Action concrète] | [Résultat observable] |
| 2 | [Action concrète] | [Résultat observable] |
| N | [Action concrète] | [Résultat observable] |

**Post-conditions** : [État final]
**Données de test** : [Données spécifiques requises]
**Configurations** : [Navigateur, OS, Résolution]
```

### Annexe B : Modèle de rapport de bug

```
**Bug ID** : B-XXX
**Titre** : [Description du problème]
**Sévérité** : [1-Critique / 2-Majeure / 3-Mineure / 4-Triviale]
**Statut** : Ouvert / En investigation / Corrigé / Vérifié

**Étapes de reproduction** :
1. [Étape 1]
2. [Étape 2]
3. [Étape N]

**Résultat attendu** : [Ce qui devrait se passer]
**Résultat réel** : [Ce qui s'est passé]

**Environnement** :
- OS : [Système d'exploitation]
- Navigateur : [Navigateur et version]
- Version App : [Numéro de version]

**Attachements** : Screenshot, logs, vidéo
```

---

**Auteur** : Formation Azure DevOps  
**Version** : 1.0  
**Date** : Décembre 2024  
**Licence** : Libre d'usage pédagogique
