# MAINTENANCE.md — instructions à suivre à chaque modification

Procédure **obligatoire** pour toute modification de `index.html`. S'adresse à toute personne ou assistant qui reprend le projet.

**Ce fichier doit être mis à jour à chaque amélioration.** Compléter le [journal](#9-journal-des-modifications) est la dernière étape de toute intervention.

---

## 1. Comprendre le projet en trois minutes

Application web de préparation à l'examen civique français. **Un seul fichier**, `index.html`, environ 400 Ko, contenant le HTML, le CSS, le JavaScript et les 589 questions. Aucune dépendance, aucun build, aucun framework.

- **Hébergement** : GitHub Pages, `github.com/JuanYule/Examen_Civique`
- **Cible** : iPhone, ajoutée à l'écran d'accueil depuis Safari
- **Contrainte principale** : la fluidité tactile. Plusieurs versions ont été consacrées à corriger des saccades. Ne pas les réintroduire

### Pourquoi un fichier unique

Ce n'est pas de la négligence, c'est un choix. Pas de build à casser, pas de dépendance qui expire, fonctionnement hors connexion garanti, mise à jour par simple copier-coller sur GitHub. **Ne pas découper le fichier** sans raison impérieuse.

### Se repérer dans le fichier

L'en-tête de `index.html` contient une **carte complète**. Chaque section porte un code entre crochets : rechercher `[J11]` mène directement au code du glissement.

| Famille | Contenu |
|---|---|
| `[C1]` à `[C19]` | Styles CSS |
| `[H1]` à `[H8]` | Structure HTML des cinq écrans et de la fenêtre de détail |
| `[J1]` à `[J17]` | Logique JavaScript |

Sections les plus consultées : `[J1]` données, `[J7]` examen et sacs de tirage, `[J11]` glissement, `[J15]` copie d'examen et fenêtre de détail, `[C8]` fiche.

---

## 2. Les six règles à ne jamais enfreindre

Chacune vient d'un bug réel, diagnostiqué et corrigé.

### Règle 1 — N'animer que `transform` et `opacity`

| Étape | Propriétés | Coût | Verdict |
|---|---|---|---|
| Composition | `transform`, `opacity` | GPU, quasi gratuit | ✅ |
| Peinture | `box-shadow`, `filter`, `background`, `color` | Repeinture par image | ⚠️ jamais en animation |
| Mise en page | `width`, `height`, `top`, `left`, `display` | Recalcul complet | ❌ jamais |

*Origine* : en v13, le halo animait `box-shadow` sur 620 ms — une ombre floue repeinte à chaque image, déclenchée juste avant le glissement.

**Pour un effet d'ombre dynamique** : placer l'ombre sur un calque séparé (`.card-shadow`, section `[C8]`) et n'animer que sa position et son opacité.

### Règle 2 — Un seul propriétaire du `transform`

Le `transform` de la fiche appartient **exclusivement** au geste de glissement (`[J11]`). Si une animation CSS écrit aussi dedans, les deux se combattent et la carte tremble. La règle CSS exclut explicitement la carte : `.tap-pulse:not(.flashcard)`.

### Règle 3 — Une écriture par image

Toute mise à jour visuelle pendant un geste passe par `requestAnimationFrame` (`scheduleCardFrame()` dans `[J11]`).
*Vérification* : 120 évènements doivent produire **0** écriture immédiate.

### Règle 4 — Ne pas toucher à `display` pendant une interaction

Utiliser `visibility` et `opacity`, qui réservent la place. *Origine* : v14, les boutons apparaissaient au retournement, provoquant un recalcul au moment précis du glissement.

### Règle 5 — Le geste ne doit pas bloquer le défilement

La carte est en `touch-action: pan-y`. Le geste n'est capté comme horizontal que si le déplacement latéral dépasse le vertical d'un facteur 1,4 (`SWIPE_HORIZONTAL_RATIO`). En cas de doute, le défilement gagne.

### Règle 6 — Un calque d'ombre suit exactement l'élément qu'il ombre

Même translation, même rotation ; seul le décalage vertical varie. *Origine* : v19, le calque suivait à 88 % et pivotait moins que la fiche — son bord apparaissait et se lisait comme une seconde carte.

---

## 3. Procédure pour chaque modification

1. **Situer** — trouver les sections concernées grâce à la carte. Lire le code **et ses commentaires** : beaucoup expliquent pourquoi une solution évidente a été écartée
2. **Vérifier les règles** — la modification enfreint-elle l'une des six ? Il existe presque toujours une autre voie : le halo coloré imite un flou sans en payer le coût, grâce à des dégradés radiaux
3. **Modifier** — le minimum. Pas de réécriture opportuniste. Commenter le **pourquoi**, pas le comment
4. **Tester** — voir la [section 4](#4-tests). Aucune modification ne se livre sans tests
5. **Documenter** — journal de ce fichier, `CHANGELOG.md`, et la carte de l'en-tête si une section est ajoutée
6. **Publier** — `index.html` sur GitHub, icône crayon, tout remplacer, **Commit changes**

---

## 4. Tests

```bash
npm install jsdom
node --check extracted.js    # syntaxe, avant tout
```

### Le point aveugle

Les tests tournent sous **jsdom**, un DOM simulé qui **n'affiche rien**.

| Détecté | Non détecté |
|---|---|
| Logique, seuils, état | Saccades, tremblements |
| Structure, classes appliquées | Coût de peinture |
| Régressions fonctionnelles | Images par seconde |
| Propriétés CSS déclarées | Rendu réel de Safari |

**Un test vert ne garantit pas la fluidité.** Toujours finir par un essai sur iPhone.

### Batterie minimale

| # | Contrôle | Attendu |
|---|---|---|
| 1 | Base de questions | 589 au total, 479 connaissance dont 60 pièges, 110 situations |
| 2 | Structure | 3 distracteurs distincts, explication et difficulté sur chaque question, aucun doublon d'énoncé |
| 3 | Animations | Aucune propriété coûteuse animée |
| 4 | Groupage par image | 120 évènements → 0 écriture immédiate |
| 5 | Défilement vertical | La carte reste immobile, `preventDefault` non appelé |
| 6 | Glissement horizontal | Valide et enregistre |
| 7 | Banc de solutions | Affiché sur les 36 questions concernées, masqué ailleurs |
| 8 | Filtres de difficulté | Fiches et QCM, homogènes, comptes cohérents |
| 9 | QCM | 4 options, 1 correcte, correction affichée |
| 10 | Examens (20 sujets) | 40 questions, composition 28/12, aucun doublon, 5 thématiques |
| 11 | Mélange | Recouvrement consécutif ≤ 1,5 au même niveau |
| 12 | Chronomètre | 15 min restantes après 30 min hors application |
| 13 | Seuil | 31/40 refusé, 32/40 accepté |
| 14 | Copie d'examen | Détail des 40 questions conservé, lignes cliquables |
| 15 | Fenêtre de détail | 4 propositions, bonne et choisie marquées, défilement de fond bloqué |
| 16 | Explications | Aucun texte sur une bonne réponse ni sans réponse |
| 17 | Écran Progrès | Anneau, 5 barres, ligne de seuil, 3 indicateurs |
| 18 | Reprise de session | Même carte, même paquet, mêmes filtres après fermeture |
| 19 | Sacs de tirage | Persistés dans `localStorage` |
| 20 | Export / import | Fichier et code texte fonctionnels |

### Tests spécifiques

**Geste** → appui long immobile avec bruit simulé, multi-touch, verrouillage d'axe aux bornes du ratio, geste rapide et court, geste hésitant, interruption.

**Visuel** → inventaire des propriétés animées, plans (`z-index`) entre halo, contenu et indications, suivi du calque d'ombre.

**Données** → structure de chaque question, aucun doublon entre banc de solutions et distracteurs, effectifs suffisants par niveau et par sac, biais de longueur, indices syntaxiques.

**Mélange** → 100 examens au **même niveau** : recouvrement consécutif, couverture du vivier, uniformité **à l'intérieur de chaque sac** (écart max-min ≤ 1), délai de retour.

### Pièges d'écriture de tests

- Les `const` du script ne sont pas exposés sur `window` : passer par `w.eval('MaConstante')`
- jsdom n'implémente pas `PointerEvent` : utiliser l'assistant `PE()` de `lib.js`
- L'envol d'une carte dure 260 ms : attendre au moins 300 ms entre deux gestes simulés
- jsdom retire les propriétés `-webkit-` inconnues du CSSOM : les vérifier dans le texte source
- **Mesurer le mélange à niveau constant** : en alternant CR et CSP, seules les questions « CR + CSP » peuvent être communes, ce qui fausse la lecture
- **Mesurer l'uniformité par sac**, pas globalement : une question d'un petit vivier doit légitimement sortir plus souvent

---

## 5. Déboguer la fluidité sur iPhone

Connecter l'iPhone au Mac, puis **Safari → Développement → iPhone → la page**. Le détail des scripts figure dans `DEBUG-RENDU-ET-TESTS.md`, section 3.3.

---

## 6. Réglages courants

### Geste de glissement — `[J11]`

```js
const SWIPE_COMMIT_PX = 88;            // distance de validation
const SWIPE_COMMIT_VELOCITY = 0.45;    // vitesse de validation (px/ms)
const SWIPE_AXIS_LOCK_PX = 7;          // seuil de décision d'axe
const SWIPE_HORIZONTAL_RATIO = 1.4;    // exigence d'intention horizontale
const SWIPE_VELOCITY_WINDOW_MS = 100;  // lissage de la vitesse
const SWIPE_VELOCITY_MIN_DT = 12;      // intervalle minimum mesurable
```

| Symptôme | Réglage |
|---|---|
| Valide trop facilement | `SWIPE_COMMIT_PX` → 110, `SWIPE_COMMIT_VELOCITY` → 0.6 |
| Il faut trop glisser | `SWIPE_COMMIT_PX` → 70 |
| La carte part alors qu'on voulait faire défiler | `SWIPE_HORIZONTAL_RATIO` → 1.8 |
| Difficile de faire glisser | `SWIPE_HORIZONTAL_RATIO` → 1.2 |
| Un geste lent valide par accident | `SWIPE_VELOCITY_MIN_DT` → 20 |

### Durées d'animation

| Élément | Emplacement | Valeur |
|---|---|---|
| Retour élastique | `.flashcard.snapping` `[C8]` | 380 ms |
| Envol de la carte | `.flashcard.flying` `[C8]` | 300 ms |
| Délai avant carte suivante | `setTimeout(…, 260)` `[J11]` | 260 ms |
| Retour de l'ombre | `.card-shadow` `[C8]` | 320 ms |
| Ouverture de la fenêtre | `.modal` `[C8]` | 320 ms |

⚠️ Le délai de 260 ms doit rester **inférieur** à l'envol de 300 ms.

### Examen — `[J7]`

```js
const EXAM_TOTAL = 40;               // questions par épreuve
const EXAM_CONNAISSANCE = 28;        // dont questions de connaissance
const EXAM_SITUATION = 12;           // dont mises en situation
const EXAM_PASS = 32;                // seuil de réussite (80 %)
const EXAM_DURATION_MS = 45*60*1000; // durée maximale
```

Ces valeurs reproduisent l'examen officiel. **Ne les modifier que si le ministère change les règles.**

### Sacs de tirage — `[J7]`

`drawFromBag(niveau, type, thématique, n, dejaPris)` puise dans un sac persisté. Un sac est reconstruit automatiquement si le vivier change (ajout ou retrait de questions) — **aucune action manuelle n'est requise après un ajout de questions**.

`quotasThemes(total, rotation)` répartit un effectif entre les cinq thématiques, le reste tournant d'un examen à l'autre.

### Couleurs — `[C1]`

Deux familles distinctes :
- **Thématiques** (`--t-*`) : identifient un thème, fixes, associées à une icône. Ne pas les changer sans raison
- **Vigilance** (`--alerte`, `--attention`, `--consolide`, `--acquis`) : évaluent une performance. Le seuil du vert est calé sur les 80 % de l'examen

`perfColor(p)` dans `[J8]` est le point unique où l'échelle est définie.

---

## 7. Ajouter ou modifier des questions

Structure d'une question (section `[J1]`) :

```js
{
  id: 590,                              // unique, incrémental
  level: "CR",                          // "CR", "CSP" ou "both"
  type: "connaissance",                 // ou "situation"
  theme: "Droits et devoirs",           // exactement l'un des 5 libellés
  diff: "moyen",                        // "facile" ou "moyen"
  q: "Énoncé de la question ?",
  a: "La bonne réponse",
  d: ["Distracteur 1","Distracteur 2","Distracteur 3"],   // exactement 3
  exp: "Explication courte, ~75 caractères.",
  de: ["Pourquoi le 1 est faux","Pourquoi le 2 est faux","Pourquoi le 3 est faux"],  // facultatif
  alts: [],                             // facultatif : autres réponses valables
  trap: false                           // true pour une question piège
}
```

### Contrôles obligatoires

- Exactement 3 distracteurs, tous distincts, aucun identique à la bonne réponse
- Le libellé de `theme` doit correspondre **exactement** à l'un des cinq existants
- Une explication qui **ajoute** une information : une date, un chiffre, une nuance. Ne pas reformuler la réponse
- Si `alts` est renseigné, aucune de ses entrées ne doit figurer dans `d`
- Si `de` est renseigné, il doit contenir autant d'entrées que `d`, dans le même ordre

### Éviter les indices qui trahissent la réponse

Vérifier après chaque ajout, par test automatisé :

| Indice | Règle |
|---|---|
| Longueur | La bonne réponse ne doit pas être systématiquement la plus longue. Cible : ≤ 32 % |
| Parenthèse | Jamais exclusive à la bonne réponse |
| Chiffre | Si la réponse contient une date ou un nombre, au moins un distracteur doit en contenir aussi |
| Absolus | Ne pas concentrer « jamais », « uniquement », « aucun » dans les distracteurs |
| Nuance | « sauf », « y compris » ne doivent pas être exclusifs à la bonne réponse |
| Casse et ponctuation | Toutes les options au même format |

### Rédiger une mise en situation

1. Un scénario court, en une ou deux phrases, à la deuxième personne
2. La bonne réponse est une **action**, pas une définition
3. Priorité au **dialogue** avant la sanction, hors urgence ou danger
4. La bonne réponse identifie l'**institution compétente**
5. Les distracteurs couvrent : ignorer, contourner la procédure, se faire justice soi-même

### Rédiger une question piège

Trois affirmations exactes, une fausse. L'énoncé doit annoncer explicitement le mot **FAUSSE**. Vérifier que l'affirmation fausse n'est pas repérable par sa longueur. `trap: true` et `diff: "moyen"`.

### Rédiger une justification de distracteur (`de`)

Nommer l'erreur, ne pas la constater. « L'excès de vitesse est une contravention » vaut mieux que « ce n'est pas la bonne réponse ». Si la proposition désigne une notion réelle, rappeler ce qu'elle est.

Trois mécanismes automatiques complètent les textes écrits, dans `[J15]` :
1. Question piège → règle systématique
2. Dictionnaire de 209 notions → ce que la proposition désigne réellement
3. Règles de conduite pour les mises en situation → ignorer, contourner, riposter

Le champ `de` prime toujours sur ces mécanismes.

---

## 8. Erreurs déjà commises

| Idée | Pourquoi elle a échoué |
|---|---|
| Animer `box-shadow` pour un halo | Repeinture d'une zone floue à chaque image, saccades |
| Flou de calque (`filter: blur`) pour le halo coloré | Coût majeur sur iPhone ; les dégradés radiaux donnent le même rendu gratuitement |
| Fausses cartes empilées derrière la fiche | Rendu figé, illisible sur fond sombre |
| Calque d'ombre restant centré ou suivant partiellement | Apparaît à découvert, ressemble à une seconde carte |
| `touch-action: none` sur la carte | Empêche le défilement à une main |
| Indications de glissement au même plan que le texte | Apparaissent derrière le contenu |
| `requestAnimationFrame` seul, sans transition CSS | Ne se déclenche pas en arrière-plan : graphiques vides |
| Vitesse mesurée entre deux points consécutifs | Intervalles d'une milliseconde, validations fantômes |
| Aucun filtrage du pointeur | Une paume ou un second doigt projette la carte |
| Aucun verrou pendant l'envol | Double validation, cartes sautées |
| Restaurer une session sans vérifier le vivier | Paquet figé après ajout de questions |
| Une barre de position ressemblant à une barre de progression | L'utilisateur croit perdre sa progression |
| Tirage indépendant à chaque examen | 6,2 questions communes entre examens consécutifs |
| Mesurer le mélange en alternant CR et CSP | Fausse le résultat : seules les questions « both » peuvent être communes |
| Mesurer l'uniformité globalement plutôt que par sac | Fausse le résultat : les petits viviers sortent légitimement plus souvent |
| Écrire des justifications génériques pour couvrir 100 % | Texte creux, sans valeur pédagogique |

---

## 9. Journal des modifications

| Version | Sections | Modification | Tests |
|---|---|---|---|
| v26 | `[J1]` `[J15]` | 384 justifications écrites, champ `de[]` prioritaire | 100 examens, couverture 61,7 % |
| v25 | `[C8]` `[H8]` `[J15]` | Fenêtre de détail, dictionnaire de notions, règles de conduite | 100 examens, 16 contrôles |
| v24 | `[J7]` `[J1]` | Sacs de tirage, 24 situations ajoutées | 10 puis 100 examens, statistiques |
| v23 | `[J1]` `[C8]` `[J11]` | Audit syntaxique, 60 questions pièges, calque corrigé | 30 contrôles + 20 fonctionnels |
| v22 | `[J1]` | 65 questions sur les sujets manquants, doublons fusionnés | 20 simulations |
| v21 | `[J1]` `[C7]` `[J10]` `[J12]` | Biais de longueur corrigé, niveaux de difficulté | 10 tests |
| v20 | En-tête, tous bandeaux | Carte du fichier, 44 sections codées | Cohérence + non-régression |
| v19 | `[C8]` `[C9]` `[J11]` | Indications au premier plan, ombre suivant la fiche, halo renforcé | 10 tests |
| v18 | `[C8]` `[J3]` `[J11]` | Profondeur dynamique par calque séparé, lavis de couleur | 19 tests |
| v17 | `[J7]` `[J14]` `[J15]` | Copie d'examen consultable, indications centrées, relief | 30 points de contrôle |
| v16 | `[C1]` `[J8]` | Palette de vigilance à quatre paliers | Échelle + non-régression |
| v15 | `[C8]` `[J11]` | Défilement rendu depuis la carte | 10 tests |
| v14 | `[C8]` `[C19]` `[J11]` | Quatre causes de saccades corrigées | 10 tests |
| v13 | `[J11]` | Tremblement du glissement | 4 campagnes |

### Modèle d'entrée

```
| v27 | [J11] | Description courte | Tests exécutés |
```

---

## 10. Avant de livrer — liste de contrôle

- [ ] `node --check` passe sur le script extrait
- [ ] Les 20 contrôles de la batterie minimale passent
- [ ] Les tests spécifiques au type de modification passent
- [ ] Aucune propriété coûteuse n'est animée
- [ ] Si des questions ont été ajoutées : biais de longueur et indices syntaxiques revérifiés
- [ ] Essai réel sur iPhone : glissement lent, rapide, appui long, défilement à une main
- [ ] Le journal de ce fichier est complété
- [ ] `CHANGELOG.md` est complété
- [ ] `README.md` est à jour si les chiffres ont changé
- [ ] La carte de l'en-tête de `index.html` reflète les sections existantes
