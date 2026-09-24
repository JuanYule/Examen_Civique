# Débogage du rendu et cas de test — App Examen Civique

Fichier de référence pour diagnostiquer les problèmes de **fluidité** (saccades, tremblements) et valider le code avant chaque mise en ligne.

**Dépôt :** `github.com/JuanYule/Examen_Civique` · **En ligne :** `https://juanyule.github.io/Examen_Civique/`
**État :** 589 questions, fichier d'environ 400 Ko, 44 sections codées.

---

## 1. Le point aveugle à connaître

Les tests automatisés de ce projet tournent sous **jsdom**, un DOM simulé qui **n'affiche rien à l'écran**.

| Ce que jsdom détecte | Ce que jsdom ne détecte pas |
|---|---|
| Logique du geste, seuils, état | Saccades, tremblements |
| Structure HTML, classes appliquées | Coût de peinture d'une image |
| Valeurs calculées, persistance | Nombre d'images par seconde |
| Propriétés CSS déclarées | Comportement réel de Safari iOS |

**Conséquence pratique :** un test vert ne garantit pas la fluidité. Il faut tester séparément **les propriétés animées**, puis valider sur un vrai iPhone.

---

## 2. Règles de rendu (à ne jamais enfreindre)

### 2.1 Propriétés animables sans coût

| Étape | Propriétés concernées | Coût | Verdict |
|---|---|---|---|
| **Composition** | `transform`, `opacity` | GPU, quasi gratuit | ✅ à utiliser |
| **Peinture** | `box-shadow`, `filter`, `background`, `border-radius`, `color` | Repeinture à chaque image | ⚠️ à éviter en animation |
| **Mise en page** | `width`, `height`, `top`, `left`, `margin`, `padding`, `display` | Recalcul de toute la page | ❌ jamais en animation |

**Erreur commise en v13 :** le halo de la carte animait `box-shadow` sur 620 ms. Une ombre floue de 16 px repeinte à chaque image, déclenchée juste avant le glissement — le téléphone était déjà saturé quand le doigt bougeait.
**Correction :** bordure sur un calque `::after` animée en `opacity`.

**Erreur évitée en v18 :** les références fournies utilisaient un flou de calque (`Layer blur 200`). Un flou est parmi les propriétés les plus coûteuses sur iPhone. Le halo coloré des fiches est obtenu par **deux dégradés radiaux superposés**, pour le même rendu diffus à coût nul.

### 2.2 Configuration des gestes tactiles

| Réglage | Valeur | Pourquoi |
|---|---|---|
| `touch-action` | `pan-y` sur la carte | Laisse le défilement vertical au système, l'horizontal au geste |
| `will-change` | `transform` | Réserve un calque GPU dédié |
| `-webkit-touch-callout` | `none` | Pas de menu contextuel sur appui long |
| `-webkit-user-drag` | `none` | Pas de glisser-déposer natif |
| `user-select` | `none` | Pas de sélection de texte pendant le geste |
| Affichage conditionnel | `visibility` + `opacity`, jamais `display` | `display` change la hauteur du document |

### 2.3 Discipline JavaScript pendant un geste

1. **Un seul propriétaire du `transform`.** Deux écritures concurrentes font trembler l'élément
2. **Filtrer le pointeur.** Mémoriser le `pointerId` du premier contact, ignorer tous les autres
3. **Une écriture par image.** Grouper dans `requestAnimationFrame`
4. **`translate3d` plutôt que `translateX`.** Force la composition matérielle
5. **Aucune lecture de géométrie** (`offsetWidth`, `getBoundingClientRect`) dans le chemin du geste
6. **Verrouiller pendant l'animation de sortie**, sinon double validation
7. **Vitesse lissée sur une fenêtre** (~100 ms, minimum 12 ms)
8. **Un calque d'ombre suit exactement l'élément qu'il ombre** : même translation, même rotation

---

## 3. Réglages de débogage

### 3.1 Constantes du geste — `[J11]`

```js
const SWIPE_COMMIT_PX = 88;            // distance de validation, en pixels
const SWIPE_COMMIT_VELOCITY = 0.45;    // vitesse de validation, en px/ms
const SWIPE_AXIS_LOCK_PX = 7;          // seuil de décision horizontal/vertical
const SWIPE_HORIZONTAL_RATIO = 1.4;    // exigence d'intention horizontale
const SWIPE_VELOCITY_WINDOW_MS = 100;  // fenêtre de lissage de la vitesse
const SWIPE_VELOCITY_MIN_DT = 12;      // intervalle minimum pour mesurer une vitesse
```

| Symptôme | Réglage à modifier |
|---|---|
| Le geste valide trop facilement | Augmenter `SWIPE_COMMIT_PX` (→ 110) et `SWIPE_COMMIT_VELOCITY` (→ 0.6) |
| Il faut trop glisser pour valider | Diminuer `SWIPE_COMMIT_PX` (→ 70) |
| La carte part alors qu'on voulait faire défiler | Augmenter `SWIPE_HORIZONTAL_RATIO` (→ 1.8) |
| Difficile de déclencher le glissement | Diminuer `SWIPE_HORIZONTAL_RATIO` (→ 1.2) |
| Un geste lent valide par accident | Augmenter `SWIPE_VELOCITY_MIN_DT` (→ 20) |
| Rotation trop marquée | Dans `applyCardTransform`, baisser le facteur `0.035` (→ 0.02) |
| Ombre trop ou pas assez marquée | Dans `applyCardTransform`, ajuster `0.42 + lift * 0.30` |

### 3.2 Durées d'animation

| Constante | Emplacement | Valeur | Rôle |
|---|---|---|---|
| Retour élastique | CSS `.flashcard.snapping` `[C8]` | 380 ms | Retour quand le seuil n'est pas atteint |
| Envol de la carte | CSS `.flashcard.flying` `[C8]` | 300 ms | Sortie d'écran après validation |
| Délai avant carte suivante | JS `setTimeout(…, 260)` `[J11]` | 260 ms | Doit rester **inférieur** à l'envol |
| Retour de l'ombre | CSS `.card-shadow` `[C8]` | 320 ms | Repos du calque de profondeur |
| Halo thématique | CSS `glowFade` `[C19]` | 560 ms | Confirmation au retournement |
| Ouverture de la fenêtre | CSS `.modal` `[C8]` | 320 ms | Montée depuis le bas |

⚠️ Le délai de 260 ms et l'envol de 300 ms sont liés. Si l'un change, vérifier que le délai reste inférieur.

### 3.3 Mode diagnostic à coller dans la console Safari

Connecter l'iPhone au Mac, puis **Safari → Développement → iPhone → la page**.

```js
// A. Images par seconde pendant un geste — en dessous de 50, il y a un problème
(function(){ let n=0, t=performance.now();
  (function loop(){ n++; const d=performance.now()-t;
    if(d>=1000){ console.log('FPS:', Math.round(n*1000/d)); n=0; t=performance.now(); }
    requestAnimationFrame(loop); })();
})();

// B. Détecter une propriété coûteuse animée
[...document.styleSheets].flatMap(s=>[...s.cssRules]).forEach(function walk(r){
  if(r.cssRules) [...r.cssRules].forEach(walk);
  if(r.constructor.name==='CSSKeyframesRule'){
    const p=new Set(); for(const k of r.cssRules) for(let i=0;i<k.style.length;i++) p.add(k.style[i]);
    const cher=[...p].filter(x=>/box-shadow|filter|width|height|top|left|margin|padding/.test(x));
    console.log(r.name, [...p].join(','), cher.length?'⚠️ COÛTEUX':'✓');
  }});

// C. Suivre l'état du geste en direct
setInterval(()=>console.log('dx',Math.round(swipe.dx),'| actif',swipe.active,
  '| axe',swipe.horizontal?'H':'-','| verrou',swipe.committing), 500);

// D. Compter les écritures de transform (doit rester à 1 par image)
(function(){ const el=document.getElementById('flashcard'); let n=0, v='';
  Object.defineProperty(el.style,'transform',{configurable:true,get:()=>v,set(x){n++;v=x;}});
  setInterval(()=>{ if(n) console.log('écritures/s:',n); n=0; },1000);
})();

// E. Inspecter l'état des sacs de tirage
console.log(Object.entries(progress.bags||{}).map(([k,v])=>
  k==='_rot' ? `rotation:${v}` : `${k}: ${v.i}/${v.order.length}`).join('\n'));

// F. Mesurer la couverture des explications sur la base
(function(){ let ok=0, total=0;
  for(const q of QUESTIONS) for(const d of q.d){ total++; if(whyWrong(q,d)) ok++; }
  console.log(`justifications : ${ok}/${total} = ${(ok/total*100).toFixed(1)} %`);
})();
```

### 3.4 Réglages visuels de Safari

- **Développement → Afficher les couches composées** : la carte doit avoir son propre calque pendant le geste
- **Timeline → Rendering** : chercher les barres vertes (peinture). Pendant un glissement, il ne devrait presque rien y avoir

---

## 4. Cas de test

Les tests s'exécutent avec Node et jsdom (`npm install jsdom`). Chaque fichier est autonome et retourne un code d'erreur si un cas échoue.

### 4.1 Tests de rendu (le point aveugle)

| # | Cas | Vérification |
|---|---|---|
| R1 | Inventaire des animations | Aucune règle `@keyframes` n'anime une propriété coûteuse |
| R2 | Halo composité | `glowFade` n'anime que `opacity`, sur un calque `::after` dédié |
| R3 | Lavis coloré | Obtenu par dégradés radiaux, sans `filter: blur` |
| R4 | `touch-action` | Vaut `pan-y` sur `.flashcard` |
| R5 | Protections iOS | `-webkit-touch-callout` et `-webkit-user-drag` présents |
| R6 | Calque GPU | `will-change: transform` sur la carte et sur le calque d'ombre |
| R7 | Pas de `display` animé | Les contrôles basculent par classe |
| R8 | Place réservée | `.deck-controls` et `.swipe-tip` en `visibility:hidden; opacity:0` |
| R9 | Écritures groupées | 120 évènements → **0** écriture de `transform` avant l'image suivante |
| R10 | Pas de lecture de géométrie | Aucun `offsetWidth` dans le chemin du geste |
| R11 | `translate3d` | Le transform appliqué contient `translate3d` |
| R12 | Calque d'ombre | Suit exactement la fiche : même translation, même rotation, échelle ≤ 0,94 |

### 4.2 Tests du geste

| # | Cas | Attendu |
|---|---|---|
| G1 | Appui long immobile, 180 évènements avec bruit ±2 px | La carte ne bouge pas, aucun enregistrement |
| G2 | Appui maintenu 3 s à 60 px avec bruit ±1,5 px | Amplitude suivie ≤ 3,5 px |
| G3 | Glissement horizontal | La carte suit au pixel près, écart maximal 0 |
| G4 | Rotation | Strictement proportionnelle, incréments constants |
| G5 | Opacité des indications | Progression en courbe, plafonnée à 0,92 |
| G6 | Geste vertical | La carte ne bouge pas, `preventDefault` non appelé |
| G7 | Micro-mouvement (3 px) | Aucun glissement déclenché |
| G8 | Carte non retournée | Geste inactif |
| G9 | Multi-touch | Un second doigt ne déplace pas la carte et ne clôt pas le geste |
| G10 | Frontière d'axe | Testée de part et d'autre du ratio 1,4, dans les deux directions |
| G11 | Geste du pouce en arc | Le défilement reste possible |
| G12 | Interruption (`pointercancel`) | Traitée comme une fin de geste, pas un retour brutal |

### 4.3 Tests de validation

| # | Cas | Attendu |
|---|---|---|
| V1 | Glissement droite ≥ 88 px | « Je savais », carte suivante |
| V2 | Glissement gauche ≥ 88 px | « À revoir », carte suivante |
| V3 | Glissement 70 px | Annulé, rien enregistré, retour en place |
| V4 | Geste court et rapide (50 px / 40 ms) | Validé par la vitesse |
| V5 | Geste court et lent (50 px / 800 ms) | Non validé |
| V6 | Geste hésitant | Non validé — la vitesse doit aller dans le sens du déplacement |
| V7 | Interruption au-delà du seuil | Traitée comme une validation |
| V8 | Double geste pendant l'envol | Une seule carte consommée |
| V9 | Endurance, 30 gestes cadencés | 30 enregistrements, aucune fuite d'état |
| V10 | Boutons « À revoir » / « Je savais » | Fonctionnels |

### 4.4 Tests du contenu

| # | Cas | Attendu |
|---|---|---|
| C1 | Effectifs | 589 questions, 479 connaissance dont 60 pièges, 110 situations |
| C2 | Structure | 3 distracteurs distincts, `exp` et `diff` présents, aucun doublon d'énoncé |
| C3 | Biais de longueur | La bonne réponse est la plus longue dans ≤ 32 % des cas |
| C4 | Stratégie « plus longue » | Rapporte moins de 33 % sur 40 examens simulés |
| C5 | Indices syntaxiques | ≤ 5 % de questions avec parenthèse, chiffre, nuance ou absolus exclusifs |
| C6 | Banc de solutions | 36 questions, aucune entrée présente dans les distracteurs |
| C7 | Questions pièges | 60, réparties 12 par thématique, énoncé contenant « FAUSSE » |
| C8 | Difficulté | Répartition équilibrée, au moins 20 questions de chaque niveau par thématique |
| C9 | Justifications | `de[]` de même longueur que `d[]`, aucune entrée vide |
| C10 | Couverture des explications | ≥ 55 % des réponses fausses reçoivent un texte |

### 4.5 Tests de l'examen

| # | Cas | Attendu |
|---|---|---|
| E1 | Composition | 40 questions, 28 connaissance + 12 situations, sur 100 sujets |
| E2 | Doublons | Aucun doublon interne |
| E3 | Niveau | Aucune question hors niveau |
| E4 | Options | 4 options, 1 seule correcte, sans doublon |
| E5 | Thématiques | Les 5 présentes dans chaque sujet |
| E6 | Mélange, recouvrement | ≤ 1,5 question commune entre examens consécutifs au même niveau |
| E7 | Mélange, couverture | 100 % du vivier tiré sur 100 examens |
| E8 | Mélange, uniformité | Écart max-min ≤ 1 **à l'intérieur de chaque sac** |
| E9 | Délai de retour | ≥ 8 examens en moyenne, moins de 5 % de retours en moins de 3 |
| E10 | Chronomètre | 15 min restantes après 30 min hors application |
| E11 | Expiration | Clôture automatique au-delà de 45 min |
| E12 | Seuil | 31/40 refusé, 32/40 accepté |
| E13 | Copie | Les 40 questions et réponses conservées |
| E14 | Pièges | Au moins un par examen dans la grande majorité des cas |

### 4.6 Tests des explications

| # | Cas | Attendu |
|---|---|---|
| X1 | Bonne réponse | `whyWrong` ne retourne rien |
| X2 | Absence de réponse | `whyWrong` ne retourne rien, la fenêtre le signale |
| X3 | Questions pièges | Les 60 traitées par la règle systématique |
| X4 | Champ `de` | Prioritaire sur toutes les règles automatiques |
| X5 | Fenêtre ouverte | 4 propositions, bonne marquée, choisie marquée |
| X6 | Fond | Défilement bloqué pendant l'ouverture, rétabli à la fermeture |
| X7 | Question juste | Aucune option marquée comme erreur |
| X8 | QCM | Justification affichée après une mauvaise réponse |

### 4.7 Tests fonctionnels et d'accessibilité

| # | Cas | Attendu |
|---|---|---|
| F1 | QCM | 4 options, 1 correcte, correction affichée |
| F2 | Filtres de difficulté | Fiches et QCM, homogènes, comptes cohérents |
| F3 | Écran Progrès | Anneau, 5 barres, ligne de seuil, 3 indicateurs |
| F4 | Export / import | Fichier et code texte fonctionnels |
| F5 | Reprise de session | Même carte, même paquet, mêmes filtres après fermeture |
| F6 | Sacs persistés | Présents dans `localStorage` après fermeture |
| F7 | Paquet obsolète | Reconstruit si le vivier a changé, progression conservée |
| A1 | Boutons visibles | Après retournement, les contrôles sont affichés |
| A2 | Mouvements réduits | `prefers-reduced-motion: reduce` neutralise les animations |
| A3 | Navigation manuelle | Précédente / Suivante / Mélanger fonctionnels |

---

## 5. Écrire un test

```js
const {makeApp,wait}=require('./lib');
(async()=>{
  const w=makeApp(); const ev=c=>w.eval(c); const errs=[]; let n=0;
  const T=(c,l)=>{n++;console.log(`  ${c?'✓':'✗'} ${n}. ${l}`); if(!c)errs.push(n+'. '+l);};
  await wait(900);                       // laisser l'application démarrer

  T(ev('QUESTIONS.length')===589, 'base intacte');

  console.log(errs.length?'❌ '+errs.join('\n'):'✅ VALIDÉ');
  process.exit(errs.length?1:0);
})();
```

### Pièges à connaître

- Les `const` du script ne sont pas exposés sur `window` : passer par `w.eval('MaConstante')`
- jsdom n'implémente pas `PointerEvent` : utiliser l'assistant `PE()` de `lib.js`
- L'envol d'une carte dure 260 ms : attendre au moins 300 ms entre deux gestes simulés
- jsdom retire les propriétés `-webkit-` inconnues du CSSOM : les vérifier dans le texte source
- Lire l'état **pendant** le geste, avant `pointerup` : le relâchement réinitialise tout
- Après un `renderProgress()` ou un `renderExam()`, les éléments du DOM sont recréés : re-interroger le document avant de cliquer
- **Mesurer le mélange à niveau constant.** En alternant CR et CSP, seules les questions « CR + CSP » peuvent être communes
- **Mesurer l'uniformité par sac**, pas globalement : une question d'un petit vivier sort légitimement plus souvent

---

## 6. Procédure avant chaque mise en ligne

1. **Syntaxe** — `node --check` sur le script extrait
2. **Tests de rendu** — section 4.1, en priorité R1
3. **Tests du geste et de validation** — sections 4.2 et 4.3
4. **Tests du contenu** — section 4.4, obligatoire si des questions ont été ajoutées
5. **Tests de l'examen** — section 4.5
6. **Tests fonctionnels** — section 4.7
7. **Validation sur iPhone** — la seule qui juge la fluidité réelle. Vérifier : glissement lent, glissement rapide, appui long immobile, appui maintenu sur le côté, défilement à une main, geste interrompu
8. **Mise en ligne** — remplacer `index.html` dans le dépôt, puis « Commit changes »

---

## 7. Historique des causes de saccades identifiées

| Version | Cause | Correction |
|---|---|---|
| v13 | Deux propriétaires du `transform` | Propriétaire unique, règle CSS excluant la carte |
| v13 | Aucun filtrage du pointeur | Mémorisation du `pointerId` |
| v13 | Une écriture de `transform` par évènement | Groupage dans `requestAnimationFrame` + `translate3d` |
| v13 | Double validation pendant l'envol | Verrou jusqu'au rendu suivant |
| v14 | Animation de `box-shadow` | Calque `::after` animé en `opacity` |
| v14 | `touch-action: pan-y` sans gestion du `pointercancel` | Interruption traitée comme une fin de geste |
| v14 | `display:none` → `flex` pendant l'interaction | `visibility` + `opacity`, place réservée |
| v15 | `touch-action: none` bloquant le défilement | Retour à `pan-y` avec verrouillage d'axe à 1,4 |
| v18 | Fausses cartes empilées, rendu figé | Calque d'ombre unique, dynamique |
| v19 | Calque d'ombre suivant partiellement la fiche | Suivi exact, même rotation, calque réduit |

---

## 8. Si les saccades reviennent

Pistes à explorer dans l'ordre :

1. **Ombre portée statique de la carte** (`box-shadow`) — même statique, elle doit être composée à chaque déplacement. Tester en la retirant temporairement
2. **`backdrop-filter`** — si un flou d'arrière-plan est ajouté un jour, c'est un coût majeur sur iPhone
3. **Taille du calque** — une carte occupant presque tout l'écran coûte plus cher à composer
4. **Nombre de nœuds dans la carte** — le banc de solutions peut afficher 10 éléments ; vérifier si les saccades n'apparaissent que sur ces questions-là
5. **La fenêtre de détail** — elle contient jusqu'à 4 propositions plus l'explication ; vérifier son ouverture sur les questions les plus longues
6. **Mémoire** — l'application charge 589 questions et un dictionnaire de 209 notions. Vérifier l'onglet Mémoire de l'inspecteur Safari sur un appareil ancien
