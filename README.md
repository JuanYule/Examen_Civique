# Examen Civique — application de préparation

Application web de préparation à l'**examen civique français**, aux niveaux **CR** (carte de résident) et **CSP** (carte de séjour pluriannuelle).

🔗 **[Ouvrir l'application](https://juanyule.github.io/Examen_Civique/)**

Fichier unique, aucune dépendance, fonctionne hors connexion une fois ajoutée à l'écran d'accueil.

---

## Sommaire

- [Fonctionnalités](#fonctionnalités)
- [Installation sur iPhone](#installation-sur-iphone)
- [Contenu](#contenu)
- [Mettre à jour l'application](#mettre-à-jour-lapplication)
- [Architecture technique](#architecture-technique)
- [Tests](#tests)
- [Avertissements](#avertissements)
- [Documentation](#documentation)

---

## Fonctionnalités

### 📚 Fiches de révision
- Flashcards à retourner, avec explication contextuelle sur chaque réponse
- **Validation par glissement** : à droite pour « je savais », à gauche pour « à revoir »
- Boutons équivalents conservés pour l'accessibilité
- Filtres par niveau (CR / CSP) et par thématique
- **Reprise de session** : l'application rouvre exactement sur la carte où vous vous étiez arrêté, avec les mêmes filtres

### 📝 QCM d'entraînement
- Sessions de 10 questions, 4 réponses proposées, 1 seule correcte
- Correction immédiate avec explication
- Filtrables par niveau et par thématique

### 🎯 Examen blanc
Reproduit les conditions officielles :
- **40 questions** — 28 de connaissance + 12 mises en situation
- **45 minutes**, chronomètre à horloge absolue : le temps continue de courir même si vous quittez l'application
- **Seuil de réussite : 32/40** (80 %)
- Aucune correction pendant l'épreuve, navigation libre entre les questions
- Un examen interrompu se reprend là où il s'est arrêté

### 📊 Suivi de progression
- Anneau de progression globale, avec détail CR / CSP
- Barres par thématique, dépliables pour le détail
- **Historique des examens** : score, tendance par rapport à la tentative précédente, meilleur score, graphique d'évolution avec la ligne de seuil à 32/40
- Analyse post-examen : performance par type de question, thématiques classées de la plus faible à la plus solide, corrigé complet des erreurs
- **Export / import** de la progression, par fichier `.json` ou par code texte

---

## Installation sur iPhone

1. Ouvrir **[l'application](https://juanyule.github.io/Examen_Civique/)** dans **Safari** (obligatoire — Chrome ne propose pas l'ajout à l'écran d'accueil sur iOS)
2. Appuyer sur l'icône de partage (carré avec une flèche vers le haut)
3. Choisir **« Sur l'écran d'accueil »**
4. Appuyer sur **« Ajouter »**

L'application s'ouvre alors en plein écran, avec sa propre icône.

> **Note :** l'ouverture d'un fichier HTML local ne fonctionne pas sur iOS — Safari la bloque systématiquement. L'hébergement sur GitHub Pages est la solution retenue pour cette raison.

**Sauvegarde :** la progression est stockée localement sur l'appareil. Pour la transférer ou la conserver, utiliser l'export dans l'onglet **Progrès**.

---

## Contenu

**442 questions** réparties sur les cinq thématiques officielles :

| Thématique | Repère visuel |
|---|---|
| Principes et valeurs de la République | 🏛️ bleu |
| Système institutionnel et politique | ⚖️ violet |
| Droits et devoirs | 📜 vert |
| Histoire, géographie et culture | 🗺️ terracotta |
| Vivre dans la société française | 🏘️ magenta |

Le code couleur et les icônes sont fixes dans toute l'application, d'après la théorie du double codage (Paivio) et l'effet Von Restorff : associer une information à un repère visuel distinct et constant améliore la rétention.

| Type | Nombre | Source |
|---|---|---|
| Questions de connaissance | 362 | Listes officielles du ministère de l'Intérieur |
| Mises en situation | 80 | Rédigées pour ce projet (voir [Avertissements](#avertissements)) |

**36 questions à réponses multiples** disposent d'un **banc de solutions** — jusqu'à 10 autres réponses valables, pour les questions du type « Quel écrivain est français ? » où l'ensemble des réponses acceptables est ouvert.

---

## Mettre à jour l'application

L'application est un fichier unique. Pour publier une nouvelle version :

1. Ouvrir `index.html` dans ce dépôt
2. Cliquer sur l'icône crayon (**Edit this file**)
3. Tout sélectionner, supprimer, puis coller le contenu de la nouvelle version
4. **Commit changes**
5. Attendre 1 à 2 minutes — GitHub Pages se met à jour automatiquement

---

## Architecture technique

```
Examen_Civique/
├── index.html                  # Application complète (HTML + CSS + JS + données)
├── README.md                   # Ce fichier
├── CHANGELOG.md                # Historique des versions
└── DEBUG-RENDU-ET-TESTS.md     # Guide de débogage et cas de test
```

**Choix techniques :**

- **Fichier unique, aucune dépendance externe.** Pas de framework, pas de CDN, pas de build. L'application fonctionne hors connexion et ne peut pas casser à cause d'une dépendance tierce.
- **Données intégrées.** Les 442 questions sont dans un tableau JavaScript au sein du fichier. Pas de requête réseau au démarrage.
- **Stockage local.** `localStorage` conserve la progression, la session en cours et l'historique des examens.
- **Chronomètre à horloge absolue.** Le temps restant est calculé depuis l'horodatage de départ, jamais décrémenté par un minuteur : il reste juste même si l'application est fermée ou le téléphone verrouillé.
- **Animations composées par le GPU.** Seuls `transform` et `opacity` sont animés. Les propriétés déclenchant une repeinture ou un recalcul de mise en page sont proscrites.

**Gestes tactiles** — règles appliquées :

| Règle | Raison |
|---|---|
| Un seul propriétaire du `transform` | Deux écritures concurrentes font trembler l'élément |
| Filtrage du `pointerId` | Une paume ou un second doigt ne doit pas déplacer la carte |
| Écritures groupées dans `requestAnimationFrame` | Les évènements arrivent plus vite que le rafraîchissement |
| `touch-action: none` sur la carte | Empêche Safari de reprendre la main en plein geste |
| `visibility`/`opacity` au lieu de `display` | `display` force un recalcul de la mise en page |
| Verrou pendant l'animation de sortie | Évite une double validation |
| Vitesse lissée sur 100 ms | Mesurée sur deux points, elle donne des valeurs aberrantes |

---

## Tests

Les tests s'exécutent avec Node et jsdom :

```bash
npm install jsdom
node v01.js   # ... jusqu'à v10.js
```

**44 cas répartis en cinq groupes :**

| Groupe | Cas | Objet |
|---|---|---|
| Rendu | 10 | Propriétés animées, `touch-action`, écritures groupées, absence de recalcul |
| Geste | 10 | Suivi du doigt, appui long, multi-touch, verrouillage d'axe |
| Validation | 10 | Seuils, vitesse, annulation, double validation, endurance |
| Fonctionnel | 11 | Banc de solutions, QCM, génération d'examens, chronomètre, progression |
| Accessibilité | 3 | Boutons, mouvements réduits, navigation manuelle |

**Limite connue :** jsdom n'affiche rien à l'écran. Ces tests valident la logique et la structure, **pas la fluidité réelle**. Une validation sur iPhone reste nécessaire avant chaque mise en ligne. Le détail figure dans [DEBUG-RENDU-ET-TESTS.md](DEBUG-RENDU-ET-TESTS.md).

La génération d'examens est validée par **simulation de 100 sujets** : composition 28/12 respectée, aucun doublon, aucune question hors niveau, cinq thématiques couvertes par sujet.

---

## Avertissements

⚠️ **Ce projet n'est ni officiel ni affilié au ministère de l'Intérieur.** C'est un outil personnel de révision.

**Sur les questions de connaissance :** les énoncés proviennent des deux listes officielles publiées par le ministère. **Les réponses ne sont pas publiées** par le ministère — elles ont été rédigées à partir de connaissances générales vérifiées. Des erreurs restent possibles.

**Sur les mises en situation :** le ministère ne les publie pas. Les 80 mises en situation de cette application ont été rédigées pour ce projet, d'après le format décrit officiellement et les principes de la formation civique. Elles suivent une méthode constante : privilégier le dialogue avant la sanction hors urgence, identifier l'institution compétente, et écarter les réponses inciviques ou extrêmes.

**Source officielle :** [formation-civique.interieur.gouv.fr](https://formation-civique.interieur.gouv.fr/)

---

## Documentation

- **[CHANGELOG.md](CHANGELOG.md)** — historique complet des versions, des problèmes rencontrés et des corrections
- **[DEBUG-RENDU-ET-TESTS.md](DEBUG-RENDU-ET-TESTS.md)** — réglages de débogage, mode diagnostic pour la console Safari, et les 44 cas de test détaillés

---

## Licence

Projet personnel. Le contenu des questions de connaissance provient de listes publiques du ministère de l'Intérieur.

