# Examen Civique — application de préparation

Application web de préparation à l'**examen civique français**, aux niveaux **CR** (carte de résident) et **CSP** (carte de séjour pluriannuelle).

🔗 **[Ouvrir l'application](https://juanyule.github.io/Examen_Civique/)**

Fichier unique, aucune dépendance, fonctionne hors connexion une fois ajoutée à l'écran d'accueil.

**589 questions** · fiches, QCM et examen blanc en conditions réelles · suivi de progression

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
- **Validation par glissement** : à droite pour « je savais », à gauche pour « à revoir ». Boutons équivalents conservés pour l'accessibilité
- **Filtres** par niveau (CR / CSP), par thématique et par difficulté (facile / moyen)
- **Banc de solutions** : les 36 questions à réponses multiples listent jusqu'à 10 autres réponses acceptées
- **Reprise de session** : l'application rouvre exactement sur la carte où vous vous étiez arrêté, avec les mêmes filtres
- Halo coloré par thématique et profondeur dynamique de la fiche pendant le geste

### 📝 QCM d'entraînement
- Sessions de 10 questions, 4 réponses proposées, 1 seule correcte
- Correction immédiate, avec **justification de la réponse cochée** lorsqu'elle est fausse
- Filtrable par niveau, thématique et difficulté

### 🎯 Examen blanc
Reproduit les conditions officielles :
- **40 questions** — 28 de connaissance + 12 mises en situation
- **45 minutes**, chronomètre à horloge absolue : le temps continue de courir même si vous quittez l'application
- **Seuil de réussite : 32/40** (80 %)
- Aucune correction pendant l'épreuve, navigation libre entre les questions
- Un examen interrompu se reprend là où il s'est arrêté
- **Tirage sans remise** : toutes les questions d'un vivier sortent avant qu'aucune ne revienne

### 📊 Suivi de progression
- Anneau de progression globale, avec détail CR / CSP
- Barres par thématique, dépliables pour le détail
- **Palette de vigilance** : rouge sous 40 %, ambre de 40 à 69 %, doré de 70 à 79 %, vert à partir de 80 % — le seuil du vert correspond exactement à celui de l'examen
- **Historique des examens** : score, tendance par rapport à la tentative précédente, meilleur score, graphique d'évolution avec la ligne de seuil
- **Copie d'examen consultable** : chaque tentative conserve ses 40 questions et vos réponses. Une fenêtre s'ouvre au toucher et montre les quatre propositions côte à côte, la bonne en vert, la vôtre en rouge, avec la justification de l'erreur
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

**589 questions** réparties sur les cinq thématiques officielles :

| Thématique | Repère visuel | Questions |
|---|---|---|
| Principes et valeurs de la République | 🏛️ bleu | 96 |
| Système institutionnel et politique | ⚖️ violet | 121 |
| Droits et devoirs | 📜 vert | 105 |
| Histoire, géographie et culture | 🗺️ terracotta | 154 |
| Vivre dans la société française | 🏘️ magenta | 113 |

Le code couleur et les icônes sont fixes dans toute l'application, d'après la théorie du double codage (Paivio) et l'effet Von Restorff : associer une information à un repère visuel distinct et constant améliore la rétention.

### Par type

| Type | Nombre | Source |
|---|---|---|
| Questions de connaissance | 479 | Listes officielles du ministère, plus des compléments rédigés |
| dont **questions pièges** | 60 | Trois affirmations exactes, une fausse |
| Mises en situation | 110 | Rédigées pour ce projet |

### Par difficulté

229 faciles, 360 moyennes. La difficulté est attribuée selon des critères objectifs cumulés : niveau CR, mise en situation, énoncé long, réponse nuancée, procédure ou institution précise à identifier.

Le filtre de difficulté est disponible en Fiches et en QCM, **volontairement absent de l'examen blanc** : l'épreuve officielle ne propose aucun choix de difficulté.

### Qualité des questions

- **Biais de longueur neutralisé** : la bonne réponse est la plus longue dans 27,8 % des cas, contre 25 % attendus au hasard. Avant correction, ce taux était de 69 % — cocher systématiquement la réponse la plus longue rapportait 69 % sans aucune connaissance
- **Indices syntaxiques éliminés** : parenthèses, chiffres ou nuances exclusifs à la bonne réponse, absolus concentrés dans les distracteurs. 0,4 % de questions résiduelles, contre 10,1 % avant audit
- **384 justifications rédigées** pour expliquer pourquoi une proposition écartée est fausse, complétées par un dictionnaire de 209 notions et des règles de conduite. Couverture mesurée : 61,7 % des réponses fausses reçoivent un texte explicite

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
├── index.html                    # Application complète (~400 Ko)
├── README.md                     # Ce fichier
├── CHANGELOG.md                  # Historique des versions
├── MAINTENANCE.md                # Procédure obligatoire avant toute modification
├── DEBUG-RENDU-ET-TESTS.md       # Débogage de la fluidité et cas de test
└── GUIDE-APPLICATION-MOBILE.md   # Guide pour en faire une application Android
```

**Choix techniques :**

- **Fichier unique, aucune dépendance externe.** Pas de framework, pas de CDN, pas de build. L'application fonctionne hors connexion et ne peut pas casser à cause d'une dépendance tierce
- **Code navigable** : un en-tête donne la carte du fichier, et **44 sections** portent un code repérable par simple recherche — `[C1]` à `[C19]` pour les styles, `[H1]` à `[H8]` pour la structure, `[J1]` à `[J17]` pour la logique
- **Données intégrées.** Les 589 questions sont dans un tableau JavaScript au sein du fichier. Aucune requête réseau au démarrage
- **Stockage local.** `localStorage` conserve la progression, la session en cours, l'historique des examens et les sacs de tirage
- **Chronomètre à horloge absolue.** Le temps restant est calculé depuis l'horodatage de départ, jamais décrémenté par un minuteur : il reste juste même application fermée
- **Animations composées par le GPU.** Seuls `transform` et `opacity` sont animés

**Tirage des examens — le sac de tirage :**

Un sac par combinaison (niveau, type, thématique), soit 21 sacs, chacun étant une permutation Fisher-Yates du vivier parcourue par un curseur. Toutes les questions sortent avant qu'aucune ne revienne. À l'épuisement, le sac est remélangé en écartant la collision de jointure. Les sacs sont persistés, la rotation se poursuit d'une session à l'autre.

Mesuré sur 100 examens au même niveau : **0,44 question en commun** entre deux examens consécutifs (contre 6,2 avant), 69 paires sur 99 totalement disjointes, couverture de 100 % du vivier, écart maximal de 1 à l'intérieur d'un sac.

**Gestes tactiles — règles appliquées :**

| Règle | Raison |
|---|---|
| Un seul propriétaire du `transform` | Deux écritures concurrentes font trembler l'élément |
| Filtrage du `pointerId` | Une paume ou un second doigt ne doit pas déplacer la carte |
| Écritures groupées dans `requestAnimationFrame` | Les évènements arrivent plus vite que le rafraîchissement |
| `touch-action: pan-y` et verrouillage d'axe à 1,4 | Le défilement reste possible depuis la carte |
| `visibility`/`opacity` au lieu de `display` | `display` force un recalcul de la mise en page |
| Verrou pendant l'animation de sortie | Évite une double validation |
| Vitesse lissée sur 100 ms | Mesurée sur deux points, elle donne des valeurs aberrantes |
| Aucune ombre animée | Animer `box-shadow` provoque des saccades sur iPhone |

---

## Tests

Les tests s'exécutent avec Node et jsdom :

```bash
npm install jsdom
node t100.js     # 100 examens, couverture des explications
node sim100b.js  # 100 examens, analyse statistique du mélange
```

**Limite connue :** jsdom n'affiche rien à l'écran. Ces tests valident la logique et la structure, **pas la fluidité réelle**. Une validation sur iPhone reste nécessaire avant chaque mise en ligne. Le détail figure dans [DEBUG-RENDU-ET-TESTS.md](DEBUG-RENDU-ET-TESTS.md).

**Validations passées :**

| Domaine | Résultat |
|---|---|
| Génération d'examens | 100 sujets : composition 28/12 respectée 100 fois, aucun doublon, 5 thématiques par sujet |
| Mélange | 0,44 question commune entre examens consécutifs, couverture 100 % du vivier |
| Qualité des questions | Biais de longueur 27,8 %, indices syntaxiques 0,4 % |
| Explications | 61,7 % des réponses fausses justifiées, sur 982 erreurs analysées |
| Rendu | Aucune propriété coûteuse animée, 0 écriture immédiate sur 120 évènements |
| Gestes | Appui long, multi-touch, défilement, seuils, vitesse, interruption |

---

## Avertissements

⚠️ **Ce projet n'est ni officiel ni affilié au ministère de l'Intérieur.** C'est un outil personnel de révision.

**Sur les questions de connaissance :** les énoncés des listes officielles proviennent des deux listes publiées par le ministère. **Les réponses ne sont pas publiées** par le ministère — elles ont été rédigées à partir de connaissances générales vérifiées. Des erreurs restent possibles.

**Sur les mises en situation et les questions pièges :** le ministère ne les publie pas. Elles ont été rédigées pour ce projet, d'après le format décrit officiellement. Les mises en situation suivent une méthode constante : privilégier le dialogue avant la sanction hors urgence, identifier l'institution compétente, écarter les réponses inciviques ou extrêmes.

**Sur les compléments de contenu :** certains sujets ont été ajoutés après consultation de plateformes de préparation externes. Seuls les **sujets** ont été relevés, jamais les questions, qui sont protégées.

**Réussir les examens blancs de cette application ne garantit pas de réussir l'épreuve officielle.**

**Source officielle :** [formation-civique.interieur.gouv.fr](https://formation-civique.interieur.gouv.fr/)

---

## Documentation

- **[CHANGELOG.md](CHANGELOG.md)** — historique complet des 26 versions, des problèmes rencontrés et des corrections
- **[MAINTENANCE.md](MAINTENANCE.md)** — procédure obligatoire avant toute modification, règles à ne pas enfreindre, batterie de tests
- **[DEBUG-RENDU-ET-TESTS.md](DEBUG-RENDU-ET-TESTS.md)** — réglages, mode diagnostic pour la console Safari, cas de test détaillés
- **[GUIDE-APPLICATION-MOBILE.md](GUIDE-APPLICATION-MOBILE.md)** — transformer l'application web en application Android installable

---

## Licence

Projet personnel. Le contenu des questions de connaissance provient de listes publiques du ministère de l'Intérieur.
