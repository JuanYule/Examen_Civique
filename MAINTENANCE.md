# MAINTENANCE.md — instructions à suivre à chaque modification

Ce fichier est la **procédure obligatoire** pour toute modification de `index.html`. Il s'adresse à toute personne ou assistant qui reprend le projet.

**Il doit être mis à jour à chaque amélioration.** La section [Journal](#9-journal-des-modifications) en bas est à compléter systématiquement — c'est la dernière étape de toute intervention.

---

## 1. Comprendre le projet en trois minutes

Application web de préparation à l'examen civique français. **Un seul fichier**, `index.html`, contenant le HTML, le CSS, le JavaScript et les 442 questions. Aucune dépendance, aucun build, aucun framework.

- **Hébergement** : GitHub Pages, `github.com/JuanYule/Examen_Civique`
- **Cible** : iPhone, ajoutée à l'écran d'accueil depuis Safari
- **Contrainte principale** : la fluidité tactile. Plusieurs versions ont été consacrées à corriger des saccades. Ne pas les réintroduire.

### Pourquoi un fichier unique

Ce n'est pas de la négligence, c'est un choix. Pas de build à casser, pas de dépendance qui expire, fonctionnement hors connexion garanti, mise à jour par simple copier-coller sur GitHub. **Ne pas découper le fichier** sans raison impérieuse.

### Se repérer dans le fichier

L'en-tête de `index.html` contient une **carte complète**. Chaque section porte un code entre crochets : rechercher `[J11]` mène directement au code du glissement.

| Famille | Contenu |
|---|---|
| `[C1]` à `[C19]` | Styles CSS |
| `[H1]` à `[H7]` | Structure HTML des cinq écrans |
| `[J1]` à `[J17]` | Logique JavaScript |

---

## 2. Les cinq règles à ne jamais enfreindre

Chacune vient d'un bug réel, diagnostiqué et corrigé. Les enfreindre fait revenir les saccades.

### Règle 1 — N'animer que `transform` et `opacity`

Le navigateur traite une animation en trois étapes, de la moins chère à la plus chère :

| Étape | Propriétés | Coût | Verdict |
|---|---|---|---|
| Composition | `transform`, `opacity` | GPU, quasi gratuit | ✅ |
| Peinture | `box-shadow`, `filter`, `background`, `color` | Repeinture par image | ⚠️ jamais en animation |
| Mise en page | `width`, `height`, `top`, `left`, `display` | Recalcul complet | ❌ jamais |

*Origine* : en v13, le halo animait `box-shadow` sur 620 ms. Une ombre floue repeinte à chaque image, déclenchée juste avant le glissement — le téléphone était saturé quand le doigt bougeait.

**Pour un effet d'ombre dynamique** : placer l'ombre sur un calque séparé (`.card-shadow`, section `[C8]`) et n'animer que sa position et son opacité. C'est exactement ce que fait le code actuel.

### Règle 2 — Un seul propriétaire du `transform`

Le `transform` de la fiche appartient **exclusivement** au geste de glissement (`[J11]`). Si une animation CSS écrit aussi dedans, les deux se combattent et la carte tremble.

*Origine* : v13, l'animation `tap-pulse` s'appliquait à la fiche. La règle CSS exclut désormais explicitement la carte : `.tap-pulse:not(.flashcard)`.

### Règle 3 — Une écriture par image

Les évènements de pointeur arrivent plus vite que le rafraîchissement de l'écran. Toute mise à jour visuelle pendant un geste passe par `requestAnimationFrame` (voir `scheduleCardFrame()` dans `[J11]`).

*Vérification* : 120 évènements doivent produire **0** écriture immédiate.

### Règle 4 — Ne pas toucher à `display` pendant une interaction

Passer de `display:none` à `display:flex` change la hauteur du document et force un recalcul de toute la page. Utiliser `visibility` et `opacity`, qui réservent la place.

*Origine* : v14, les boutons apparaissaient au retournement de la fiche, provoquant un recalcul au moment précis où l'utilisateur allait glisser.

### Règle 5 — Le geste ne doit pas bloquer le défilement

La carte est en `touch-action: pan-y`. Le geste n'est capté comme horizontal que si le déplacement latéral dépasse le vertical d'un facteur 1,4 (`SWIPE_HORIZONTAL_RATIO`). En cas de doute, le défilement gagne.

*Origine* : v14 utilisait `touch-action: none`, ce qui empêchait de faire défiler en posant le doigt sur la carte — bloquant en usage à une main.

---

## 3. Procédure pour chaque modification

### Étape 1 — Situer

Trouver la ou les sections concernées grâce à la carte de l'en-tête. Lire le code existant **et ses commentaires** : beaucoup expliquent pourquoi une solution évidente a été écartée.

### Étape 2 — Vérifier les règles

La modification enfreint-elle l'une des cinq règles ? Si oui, chercher une autre voie. Il en existe presque toujours une : le halo coloré des fiches imite un flou sans en payer le coût, grâce à des dégradés radiaux.

### Étape 3 — Modifier

- Modifier **le minimum**. Pas de réécriture opportuniste.
- Conserver le style du code existant.
- Commenter le **pourquoi**, pas le comment. `// évite le conflit avec le geste` vaut mieux que `// met à jour la variable`.
- Si une constante est introduite, la nommer en majuscules et la placer près des autres.

### Étape 4 — Tester

Voir la [section 4](#4-tests). **Aucune modification ne se livre sans tests.**

### Étape 5 — Documenter

1. Compléter le [journal](#9-journal-des-modifications) de ce fichier.
2. Ajouter une entrée dans `CHANGELOG.md`.
3. Si une règle nouvelle a été découverte, l'ajouter à la [section 2](#2-les-cinq-règles-à-ne-jamais-enfreindre).
4. Si une section a été ajoutée, mettre à jour la carte de l'en-tête de `index.html`.

### Étape 6 — Publier

Sur `github.com/JuanYule/Examen_Civique` : ouvrir `index.html`, icône crayon, tout remplacer, **Commit changes**. La mise en ligne prend une à deux minutes.

---

## 4. Tests

### Mise en place

```bash
npm install jsdom
```

Extraire le script et vérifier la syntaxe avant tout :

```bash
node --check extracted.js
```

### Le point aveugle à connaître

Les tests tournent sous **jsdom**, un DOM simulé qui **n'affiche rien**.

| Détecté | Non détecté |
|---|---|
| Logique, seuils, état | Saccades, tremblements |
| Structure, classes appliquées | Coût de peinture |
| Régressions fonctionnelles | Images par seconde |
| Propriétés CSS déclarées | Rendu réel de Safari |

**Conséquence** : un test vert ne garantit pas la fluidité. Toujours finir par un essai sur iPhone.

### Batterie minimale

Toute modification doit passer ces contrôles :

| # | Contrôle | Attendu |
|---|---|---|
| 1 | Base de questions | 442 au total, 362 connaissance, 80 situations, 36 bancs |
| 2 | Animations | Aucune propriété coûteuse animée |
| 3 | Groupage par image | 120 évènements → 0 écriture immédiate |
| 4 | Défilement vertical | La carte reste immobile, `preventDefault` non appelé |
| 5 | Glissement horizontal | Valide et enregistre |
| 6 | QCM | 4 options, 1 correcte, correction affichée |
| 7 | Examens (20 sujets) | 40 questions, composition 28/12, aucun doublon |
| 8 | Chronomètre | 15 min restantes après 30 min hors application |
| 9 | Seuil de réussite | 31/40 refusé, 32/40 accepté |
| 10 | Copie d'examen | Détail conservé, erreurs listées |
| 11 | Écran Progrès | Anneau, 5 barres, ligne de seuil |
| 12 | Reprise de session | Même carte, même paquet après fermeture |

### Tests spécifiques par type de modification

**Modification du geste** → ajouter : appui long immobile avec bruit simulé, multi-touch, verrouillage d'axe aux bornes du ratio, geste rapide et court, geste hésitant, interruption.

**Modification visuelle** → ajouter : inventaire des propriétés animées, plans (`z-index`) entre halo, contenu et indications, dimensions et opacités.

**Modification des données** → ajouter : structure de chaque question (4 options, 1 bonne réponse, 3 distracteurs distincts), pas de doublon entre banc de solutions et distracteurs, effectifs suffisants par niveau pour générer un examen.

### Écrire un test

Modèle de départ :

```js
const {makeApp,wait}=require('./lib');
(async()=>{
  const w=makeApp(); const ev=c=>w.eval(c); const errs=[]; let n=0;
  const CP=(c,l)=>{n++;console.log(`  ${c?'✓':'✗'} T${n} — ${l}`); if(!c)errs.push('T'+n+': '+l);};
  await wait(700);                       // laisser l'application démarrer

  CP(ev('QUESTIONS.length')===442, 'base intacte');

  console.log(errs.length?'❌ '+errs.join('\n'):'✅ VALIDÉ');
  process.exit(errs.length?1:0);
})();
```

Points d'attention :
- Les `const` du script ne sont pas exposés sur `window` : passer par `w.eval('MaConstante')`.
- jsdom n'implémente pas `PointerEvent` : utiliser l'assistant `PE()` de `lib.js`.
- L'envol d'une carte dure 260 ms : attendre au moins 300 ms entre deux gestes simulés, sinon le verrou les bloque.
- jsdom retire les propriétés `-webkit-` inconnues du CSSOM : les vérifier dans le texte source.

---

## 5. Déboguer la fluidité sur iPhone

Connecter l'iPhone au Mac, puis **Safari → Développement → iPhone → la page**.

```js
// Images par seconde pendant un geste — en dessous de 50, il y a un problème
(function(){ let n=0,t=performance.now();
  (function l(){ n++; const d=performance.now()-t;
    if(d>=1000){ console.log('FPS:',Math.round(n*1000/d)); n=0; t=performance.now(); }
    requestAnimationFrame(l); })(); })();

// Détecter une propriété coûteuse animée
[...document.styleSheets].flatMap(s=>[...s.cssRules]).forEach(function w(r){
  if(r.cssRules) [...r.cssRules].forEach(w);
  if(r.constructor.name==='CSSKeyframesRule'){
    const p=new Set(); for(const k of r.cssRules) for(let i=0;i<k.style.length;i++) p.add(k.style[i]);
    const cher=[...p].filter(x=>/box-shadow|filter|width|height|top|left/.test(x));
    console.log(r.name,[...p].join(','),cher.length?'⚠️ COÛTEUX':'✓'); }});

// Suivre l'état du geste
setInterval(()=>console.log('dx',Math.round(swipe.dx),'| actif',swipe.active,
  '| axe',swipe.horizontal?'H':'-','| verrou',swipe.committing),500);
```

**Safari → Développement → Afficher les couches composées** : la carte doit avoir son propre calque pendant le geste.

---

## 6. Réglages courants

### Geste de glissement — section `[J11]`

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

⚠️ Le délai de 260 ms doit rester **inférieur** à l'envol de 300 ms, sinon la carte suivante apparaît avant la fin de l'animation.

### Examen — section `[J7]`

```js
const EXAM_TOTAL = 40;               // questions par épreuve
const EXAM_CONNAISSANCE = 28;        // dont questions de connaissance
const EXAM_SITUATION = 12;           // dont mises en situation
const EXAM_PASS = 32;                // seuil de réussite (80 %)
const EXAM_DURATION_MS = 45*60*1000; // durée maximale
```

Ces valeurs reproduisent l'examen officiel. **Ne les modifier que si le ministère change les règles.**

### Couleurs — section `[C1]`

Deux familles distinctes, à ne pas confondre :

- **Thématiques** (`--t-*`) : identifient un thème. Fixes, associées à une icône. Servent le double codage mémoriel — ne pas les changer sans raison.
- **Vigilance** (`--alerte`, `--attention`, `--consolide`, `--acquis`) : évaluent une performance. Le seuil du vert est calé sur les 80 % de l'examen.

La fonction `perfColor(p)` dans `[J8]` est le point unique où l'échelle est définie.

---

## 7. Ajouter des questions

Structure d'une question (section `[J1]`) :

```js
{
  id: 443,                              // unique, incrémental
  level: "CR",                          // "CR", "CSP" ou "both"
  type: "connaissance",                 // ou "situation"
  theme: "Droits et devoirs",           // exactement l'un des 5 libellés
  q: "Énoncé de la question ?",
  a: "La bonne réponse",
  d: ["Distracteur 1","Distracteur 2","Distracteur 3"],   // exactement 3
  exp: "Explication courte, ~75 caractères.",
  alts: []                              // facultatif : autres réponses valables
}
```

Contrôles obligatoires :
- Exactement 3 distracteurs, tous distincts, aucun identique à la bonne réponse.
- Le libellé de `theme` doit correspondre **exactement** à l'un des cinq existants.
- Une explication qui **ajoute** une information : une date, un chiffre, une nuance. Ne pas reformuler la réponse.
- Si `alts` est renseigné, aucune de ses entrées ne doit figurer dans `d` — sinon l'application se contredirait entre le QCM et la fiche.

### Rédiger une mise en situation

Méthode appliquée aux 80 existantes :
1. Un scénario court, en une ou deux phrases, à la deuxième personne.
2. La bonne réponse est une **action**, pas une définition. La question demande « que faites-vous ? ».
3. Priorité au **dialogue** avant la sanction, hors urgence ou danger.
4. La bonne réponse identifie l'**institution compétente** : mairie, préfecture, Défenseur des droits, inspection du travail, CPAM, France Travail, secours.
5. Les distracteurs couvrent : ignorer, contourner la procédure, se faire justice soi-même.

---

## 8. Erreurs déjà commises

À lire avant de proposer une solution : elle a peut-être déjà été essayée et écartée.

| Idée | Pourquoi elle a échoué |
|---|---|
| Animer `box-shadow` pour un halo | Repeinture d'une zone floue à chaque image, saccades |
| Fausses cartes empilées derrière la fiche | Rendu figé, illisible sur fond sombre |
| Calque d'ombre restant centré pendant le glissement | Apparaît à découvert, ressemble à une seconde carte |
| `touch-action: none` sur la carte | Empêche le défilement à une main |
| Indications de glissement au même plan que le texte | Apparaissent derrière le contenu |
| Animer via `requestAnimationFrame` seul, sans transition CSS | Ne se déclenche pas si l'onglet est en arrière-plan : graphiques vides |
| Vitesse mesurée entre deux points consécutifs | Intervalles d'une milliseconde, valeurs aberrantes, validations fantômes |
| Aucun filtrage du pointeur | Une paume ou un second doigt projette la carte |
| Aucun verrou pendant l'envol | Double validation, cartes sautées |
| Restaurer une session sans vérifier le vivier | Paquet figé à 362 cartes après ajout de questions |
| Une barre de position ressemblant à une barre de progression | L'utilisateur croit perdre sa progression à chaque ouverture |

---

## 9. Journal des modifications

**À compléter à chaque intervention.** Une ligne par modification, la plus récente en haut.

| Date | Sections | Modification | Tests | Règle nouvelle |
|---|---|---|---|---|
| v20 | En-tête, tous bandeaux | Carte du fichier, 43 sections codées, création de ce document | Cohérence des codes + non-régression complète | — |
| v19 | `[C8]` `[C9]` `[J11]` `[J3]` | Indications au premier plan, ombre suivant la fiche, halo renforcé | 10 tests + non-régression | — |
| v18 | `[C8]` `[J3]` `[J11]` | Profondeur dynamique par calque séparé, lavis de couleur | 19 tests + non-régression | Ombre dynamique = calque séparé |
| v17 | `[J7]` `[J14]` `[J15]` `[C8]` `[C9]` | Copie d'examen consultable, indications centrées, relief | 30 points de contrôle | — |
| v16 | `[C1]` `[J8]` `[J9]` `[J13]` | Palette de vigilance à quatre paliers | Échelle + non-régression | — |
| v15 | `[C8]` `[J11]` | Défilement rendu depuis la carte | 10 tests | Règle 5 |
| v14 | `[C8]` `[C19]` `[J11]` | Quatre causes de saccades corrigées | 10 tests | Règles 1 et 4 |
| v13 | `[J11]` | Tremblement du glissement | 4 campagnes | Règles 2 et 3 |

### Modèle d'entrée

```
| v21 | [J11] | Description courte de la modification | Tests exécutés | Règle ajoutée ou — |
```

---

## 10. Avant de livrer — liste de contrôle

- [ ] `node --check` passe sur le script extrait
- [ ] Les 12 contrôles de la batterie minimale passent
- [ ] Les tests spécifiques au type de modification passent
- [ ] Aucune propriété coûteuse n'est animée
- [ ] Essai réel sur iPhone : glissement lent, rapide, appui long, défilement à une main
- [ ] Le journal de ce fichier est complété
- [ ] `CHANGELOG.md` est complété
- [ ] La carte de l'en-tête de `index.html` reflète les sections existantes
