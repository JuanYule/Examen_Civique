# Journal des mises à jour — App de préparation à l'examen civique

Application web autonome (fiches + QCM) pour préparer l'examen civique français (niveaux CR et CSP), hébergée sur GitHub Pages.

**Dépôt GitHub :** `github.com/JuanYule/Examen_Civique`
**URL en ligne :** `https://juanyule.github.io/Examen_Civique/`
**Déploiement retenu :** GitHub Pages (validé et fonctionnel — ne pas changer de méthode d'hébergement sauf demande explicite).

---

## v1 — Version initiale
- 362 questions couvrant les 5 thématiques officielles (Principes et valeurs de la République, Système institutionnel et politique, Droits et devoirs, Histoire/géographie/culture, Vivre dans la société française).
- Questions issues des deux listes officielles du ministère de l'Intérieur (CR = carte de résident, CSP = carte de séjour pluriannuelle), avec réponses et distracteurs rédigés à partir de connaissances générales vérifiées (les réponses officielles ne sont pas publiées par le ministère).
- Deux modes : Fiches (flashcards à retourner) et QCM (4 choix, sessions de 10 questions).
- Filtres par niveau (CR/CSP) et par thématique.
- Suivi de progression par thématique.
- Stockage : `localStorage` (fonctionne uniquement en usage local, pas dans l'aperçu Claude).

## v2 — Compatibilité stockage
- Ajout d'un système de stockage hybride (`window.storage` si disponible dans un artifact Claude, sinon `localStorage`) pour fiabiliser la sauvegarde selon le contexte d'ouverture du fichier.

## v3 — Hébergement web (résolution du problème d'ouverture sur iPhone)
- **Problème rencontré :** iOS bloque systématiquement l'ouverture de fichiers HTML locaux via "Ouvrir dans Safari" (restriction système, pas un bug de l'app).
- **Solution retenue :** hébergement du fichier sur GitHub Pages → vraie URL `https://...`, ajoutable normalement à l'écran d'accueil depuis Safari, avec sauvegarde de progression stable et permanente.

## v4 — Enrichissement du contenu et nouvelles fonctionnalités (version actuelle)
- **Réponses enrichies** : les questions à réponse ouverte/multiple (ex. "Qui était un·e écrivain·e français·e célèbre ?", "Quel peintre est français ?", "Qui était une chanteuse française célèbre ?") indiquent désormais explicitement d'autres réponses valables acceptées, avec un contexte biographique bref sur la réponse principale donnée.
- **Export / Import de la progression** (écran "Progrès") :
  - Export sous forme de fichier `.json` téléchargeable.
  - Export sous forme de code texte compact (copié dans le presse-papiers), utile si le téléchargement de fichier ne fonctionne pas sur l'appareil.
  - Import via sélection de fichier ou collage du code texte.
- **Refonte des couleurs (théorie de l'apprentissage)** : chaque thématique a désormais une couleur ET une icône fixes et cohérentes dans toute l'app (fiches, QCM, accueil, progrès), basé sur la théorie du double codage (Paivio) et l'effet Von Restorff — associer une information à un repère visuel distinct et constant améliore la rétention mémorielle de 25 à 30 % selon plusieurs études citées.
  - 🏛️ Bleu — Principes et valeurs de la République
  - ⚖️ Violet — Système institutionnel et politique
  - 📜 Vert — Droits et devoirs
  - 🗺️ Terracotta — Histoire, géographie et culture
  - 🏘️ Magenta — Vivre dans la société française

## v13 — Correction du tremblement au glissement (version actuelle)
Le glissement introduit en v12 tremblait, surtout lors d'un appui maintenu sur le côté. Quatre causes distinctes ont été identifiées et corrigées.

- **Cause 1 — deux propriétaires du `transform`.** Un appui sur une carte déjà retournée déclenchait l'animation CSS `tap-pulse`, qui écrit elle aussi dans `transform` et entrait en conflit avec la position suivie par le doigt : la carte sautait. Le `transform` de la carte a désormais un propriétaire unique, le geste. La règle CSS exclut explicitement la carte (`.tap-pulse:not(.flashcard)`) et les classes de transition neutralisent toute animation concurrente.
- **Cause 2 — aucun filtrage du pointeur.** Un second doigt ou le contact d'une paume émettait des `pointermove` que l'application traitait comme le doigt principal, projetant la carte au hasard. Le pointeur est maintenant mémorisé au premier contact ; tous les autres sont ignorés, y compris leurs relâchements.
- **Cause 3 — une écriture par évènement au lieu d'une par image.** Les évènements de pointeur arrivent plus vite que le rafraîchissement de l'écran. Les mises à jour sont désormais groupées dans `requestAnimationFrame` (une seule écriture par image, vérifiée par test : 0 écriture immédiate sur 120 évènements) et utilisent `translate3d` pour forcer la composition GPU. Techniques confirmées par les implémentations de référence consultées (issue #207 de react-multi-carousel sur le jitter au drag, et les recommandations sur `translate3d`).
- **Cause 4 — double validation possible.** Pendant les 260 ms d'envol de la carte, un second geste pouvait valider une deuxième fois et faire sauter une carte. Un verrou bloque tout nouveau geste jusqu'au rendu de la carte suivante.
- **Durcissement iOS** : `-webkit-touch-callout:none`, `-webkit-user-drag:none` et suppression du surlignage tactile, pour éviter menu contextuel et sélection de texte sur appui long.

**Bug corrigé au passage** : le paquet restait figé à 362 cartes. Une session enregistrée avant l'ajout des mises en situation était restaurée telle quelle, si bien que les 80 nouvelles questions n'étaient jamais proposées. La session n'est désormais reprise que si elle couvre encore exactement le vivier courant ; sinon le paquet est reconstruit, la progression étant conservée.

**Dix campagnes de validation** : (1) propriétaire unique du `transform` ; (2) appui long immobile avec bruit de capteur simulé — 180 évènements, la carte ne bouge pas ; (3) appui maintenu sur le côté et vérification du groupage par image ; (4) multi-touch, second doigt et second appui ; (5) linéarité du suivi, rotation et opacité ; (6) seuils de validation et annulation ; (7) vitesse, hésitation, défilement vertical, `touch-action` ; (8) reconstruction du paquet obsolète ; (9) endurance sur 40 gestes cadencés et double validation ; (10) non-régression complète — banc de solutions, boutons, QCM, 30 sujets d'examen, chronomètre, seuil, écran Progrès, export/import et reprise de session.

## v12 — Glissement des fiches
- **Validation par glissement** : sur une fiche retournée, glisser vers la droite marque « je savais », vers la gauche « à revoir ». Les deux boutons sont conservés — non par redondance, mais pour l'accessibilité (VoiceOver, motricité réduite) et la découvrabilité.
- **Comportement physique** : la carte suit le doigt au pixel près, sans retard, et pivote légèrement (0,035° par pixel) comme un objet posé. Les mentions « Je savais » et « À revoir » apparaissent progressivement selon la distance parcourue, atteignant leur pleine opacité au seuil de validation.
- **Double critère de validation, comme dans les applications d'Apple** : la distance (88 px) **ou** la vitesse du geste (0,45 px/ms au-delà de 24 px). Un mouvement bref mais franc valide donc aussi, sans devoir traverser tout l'écran.
- **Retour élastique** : en deçà du seuil, la carte revient en place en 380 ms avec une courbe d'amortissement, sans à-coup.
- **Verrouillage d'axe** : au-delà de 7 px, l'application détermine si le geste est horizontal ou vertical. Un geste vertical rend immédiatement la main au défilement de la page (`touch-action: pan-y`), pour ne jamais bloquer le scroll.
- **Le geste ne peut pas être confondu avec un appui** : un glissement n'entraîne jamais le retournement de la carte, et le clic parasite émis en fin de geste est neutralisé.
- **Deux bugs trouvés pendant les tests et corrigés** :
  1. *Vitesse aberrante* — elle était mesurée entre deux points consécutifs, donc parfois sur 1 ms, ce qui produisait des valeurs absurdes et pouvait valider un geste lent par accident. Elle est désormais lissée sur une fenêtre de 100 ms, avec un intervalle minimum de 12 ms.
  2. *Geste hésitant* — un doigt parti à droite puis revenu à gauche pouvait valider dans le mauvais sens. La validation par vitesse exige maintenant que la vitesse finale aille dans le sens du déplacement.
  3. *État résiduel* — l'état du geste n'était pas réinitialisé au changement de carte ; un geste interrompu pouvait le laisser actif.
- **Quatre campagnes de test** : (1) logique du geste — seuils, rotation, opacité, verrouillage d'axe, calcul de vitesse ; (2) effets réels — validation gauche/droite, annulation, persistance, mise à jour de la barre de maîtrise ; (3) cas limites — geste rapide et court, lent et court, hésitant, interrompu, appui simple, clic parasite, boutons ; (4) non-régression complète — bancs de solutions, QCM, génération d'examens, chronomètre, seuil de réussite, écran Progrès, export/import et reprise de session après fermeture de l'application.

## v11 — Suivi des examens dans l'écran Progrès
- **Nouvelle section « Examens blancs »** dans l'onglet Progrès, conçue autour de la question utile pour l'utilisateur : « est-ce que je progresse vers le seuil de 32/40 ? », plutôt qu'une simple liste de chiffres.
- **Trois indicateurs clés** : dernier score (avec l'écart chiffré par rapport à la tentative précédente, en vert si progression, en rouge si recul), meilleur score, et nombre d'examens réussis sur le total.
- **Graphique d'évolution** : barres des 10 dernières tentatives, colorées selon le résultat (vert ≥ 32, orange 26-31, rouge < 26), avec la **ligne de seuil à 32/40 tracée en pointillés verts** — le seuil est ainsi visible en permanence comme repère, sans avoir à faire le calcul.
- **Conseil contextuel** : indique le nombre exact de bonnes réponses manquantes pour atteindre le seuil, ou invite à entretenir son niveau si le seuil est déjà atteint.
- **Historique détaillé** : 8 dernières tentatives, du plus récent au plus ancien, avec date, heure, niveau (CR/CSP), durée réelle, mention « temps écoulé » le cas échéant, et badge réussi/échoué.
- **État vide soigné** : quand aucun examen n'a été passé, un bloc explicatif propose directement de lancer un examen blanc plutôt que d'afficher un graphique vide.
- **Tests ajoutés** : état vide, calcul des indicateurs, tendance positive et négative, proportionnalité des barres, tri chronologique, plafonnement à 10 barres et 8 lignes avec mention des tentatives masquées, intégration bout-en-bout (examen réellement passé apparaissant dans Progrès), et retour à l'état vide après réinitialisation.

## v10 — Mises en situation enrichies et méthode D.V.R.E (version actuelle)
- **Base portée à 442 questions** : 362 questions de connaissance + **80 mises en situation** (contre 46 en v9).
- **34 nouvelles mises en situation** couvrant des points de droit jusque-là absents : garde à vue et droit à l'avocat, référé prud'homal, droit au compte auprès de la Banque de France, violation de domicile par le bailleur, garantie légale de conformité, signalement PHAROS, instruction en famille et autorisation du rectorat, fraude à la carte Vitale, harcèlement moral au travail, harcèlement scolaire (3018), achat de vote, discrimination au logement et au commerce, maisons France Services.
- **Sources** : questions rédigées en propre, à partir des points de droit identifiés dans un document fourni par l'utilisateur et sur parcours-civique.fr. Ces deux sources sont privées et non affiliées au gouvernement (le site l'indique lui-même) : leurs questions ne sont pas officielles et n'ont pas été recopiées.
- **Application de la méthode D.V.R.E** (Dialogue, Valeurs, Recours, Éliminer les extrêmes) :
  - 9 questions existantes reformulées : elles posaient une question de connaissance déguisée au lieu d'appeler une action. Elles demandent désormais « que faites-vous ? » et la bonne réponse est un comportement.
  - Priorité au dialogue avant la sanction lorsqu'il n'y a ni urgence ni danger (voisinage, école, employeur).
  - Chaque situation propose des distracteurs de type « ne rien faire », « contourner la procédure » ou « se faire justice soi-même ».
  - La bonne réponse identifie l'institution compétente (mairie, préfecture, Défenseur des droits, inspection du travail, CPAM, France Travail, PHAROS, secours 15/17/18/112).
- **Contrôles automatisés ajoutés** : aucune bonne réponse incivique, 3 distracteurs distincts par question, explication obligatoire, scénario présent dans l'énoncé. 0 erreur structurelle sur 80 situations.
- **Revalidation** : 100 examens simulés (composition 28/12 respectée 100 fois sur 100, 0 doublon, 0 question hors niveau, 5 thématiques par sujet, 442 questions mobilisées), chronomètre et seuil de réussite revérifiés, non-régression sur les bancs de solutions, graphiques et reprise de session.

## v9 — Mode Examen blanc (version actuelle)
- **Nouvel onglet « Examen »** reproduisant les conditions officielles : 40 questions QCM (4 réponses, 1 correcte), 45 minutes, seuil de réussite à 32/40 (80 %), au niveau CR ou CSP au choix.
- **Composition conforme** : 28 questions de connaissance + 12 mises en situation, réparties sur les cinq thématiques officielles, tirées au sort à chaque tentative.
- **46 mises en situation créées** (base portée à 408 questions). Le ministère ne publie pas les mises en situation officielles : elles ont été rédigées d'après le format décrit et les principes de la formation civique.
- **Chronomètre à horloge absolue** : le temps est calculé depuis l'horodatage de départ, donc il continue de courir si l'utilisateur quitte l'application, verrouille son téléphone ou change d'onglet. À la reprise, le temps réellement écoulé est recalculé ; si les 45 minutes sont dépassées, l'examen est clôturé automatiquement. Alerte visuelle à 10 min puis 5 min.
- **Déroulé fidèle** : aucune correction pendant l'épreuve, navigation libre entre les 40 questions via une grille, reprise possible d'un examen en cours après fermeture de l'app.
- **Écran de résultats** : score et mention réussi/non atteint, durée réelle, questions sans réponse, performance par type (connaissance / situation), analyse par thématique triée de la plus faible à la plus solide avec conseil de révision, et corrigé complet des erreurs avec explication.
- **Validation : 100 examens simulés avant livraison** — taille (40), composition (28/12 sur 100/100), aucun doublon, aucune question hors niveau, options valides (4 choix, 1 seule bonne réponse, sans doublon), 5 thématiques couvertes dans chaque sujet, 406 questions différentes mobilisées sur 408. Chronomètre vérifié (45 min au départ, 15 min restantes après 30 min hors application, clôture automatique à expiration) et seuil de réussite testé aux bornes exactes (31/40 refusé, 32/40 accepté).

## v8 — Séparation progression / position et reprise de session (version actuelle)
- **Problème identifié** : la barre en haut de l'écran Fiches affichait la *position dans le paquet* (« 4/362 »), mais ressemblait exactement à une barre de progression. Elle repartait à zéro à chaque ouverture — ce qui donnait l'impression, à tort, que la progression était perdue.
- **Correction, alignée sur les principes d'interface d'Apple** (un indicateur de progression doit refléter un état durable ; un état temporaire ne doit pas lui ressembler) :
  - **Barre de maîtrise** (verte, persistée) : « X / Y maîtrisées dans cette sélection ». C'est la seule barre de l'écran.
  - **Position dans le paquet** : ligne discrète en texte, sans barre (« Carte 13 sur 362 »), avec un bouton « Recommencer ».
- **Reprise de session (restauration d'état, logique d'Apple Books)** : l'app rouvre exactement sur la carte où l'utilisateur s'était arrêté, avec le même ordre de paquet et les mêmes filtres (niveau CR/CSP et thème) restaurés automatiquement.
- **Tests ajoutés** : simulation de trois lancements successifs de l'app avec stockage persistant partagé, vérifiant la reprise de la carte, de l'index, du compte de maîtrise, du filtre thématique, et le comportement du bouton « Recommencer ».

## v7 — Banc de solutions élargi (version actuelle)
- Chaque question à réponses variables propose désormais **jusqu'à 10 réponses alternatives** (323 au total, contre 210 en v6), tirées de l'ensemble réel des réponses acceptables.
- Les bancs plus courts correspondent à des **ensembles finis**, où il n'existe pas 10 réponses valables : DOM insulaires (3), pays fondateurs de l'UE (5), façades maritimes (5), territoires de l'océan Indien (5), conditions du permis (6), symboles de la République (7-9), pays frontaliers (7-9).
- **Trois passes de vérification** :
  1. Cohérence interne : pas de doublon dans un banc, pas de répétition de la réponse principale, aucun banc vide.
  2. Contrôle anti-contradiction : aucune réponse du banc ne figure parmi les distracteurs (mauvaises réponses) de la même question — sinon l'app se contredirait entre le QCM et la fiche.
  3. Rendu et graphiques : les 36 bancs testés un par un en rendu réel (jsdom), affichage/masquage correct, QCM, anneau et barres.
- **Correction de robustesse détectée en passe 3** : les animations de l'anneau et des barres dépendaient de `requestAnimationFrame`, qui ne se déclenche pas quand l'onglet est en arrière-plan (les graphiques pouvaient rester vides). Remplacées par une transition CSS directe et une animation `scaleX`, sans dépendance à rAF.

## v6 — Banc de solutions et graphiques interactifs (version actuelle)
- **Analyse des 362 questions en deux passes** : détection par motif syntaxique (49 candidats), puis tri manuel pour distinguer les questions à ensemble réellement ouvert (36) de celles à réponse unique malgré la formulation "lequel de ces" (13, ex. numéros d'urgence, "quel fleuve traverse Paris").
- **Banc de solutions** : les 36 questions à réponses multiples affichent désormais un bloc "Autres réponses acceptées" (210 réponses alternatives au total), visible sur la fiche et après réponse en QCM. Un badge "Réponses multiples" signale ces questions dès le recto.
- **Graphiques interactifs (écran Progrès)** :
  - Anneau de progression animé (SVG, `stroke-dashoffset`) avec pourcentage global et légende CR/CSP.
  - Barres horizontales par thématique, animées au chargement, dépliables au toucher pour afficher le détail (maîtrisées / restantes).
- **Vérification** : validation syntaxique via `node --check` et tests fonctionnels automatisés via jsdom (rendu, navigation, banc affiché/masqué selon la question, QCM, graphiques, export).

## v5 — Enrichissement complet des 362 questions (version actuelle)
- Chaque question a désormais une courte explication contextuelle (date, chiffre clé, nuance juridique, mnémotechnique) — en moyenne 75 caractères, pour ne pas surcharger la flashcard.
- Les explications ajoutent une information non redondante avec la réponse (ex. "Loi de 1905" plutôt que répéter "séparation Églises-État").
- Les ~10 questions à réponses multiples restent volontairement plus longues (elles listent les autres réponses acceptées).
- Fichier unique autonome (`index.html`), aucune dépendance externe.
- Pour toute mise à jour future : je regénère le fichier complet, et il faut le re-uploader manuellement sur GitHub (remplacer `index.html` dans le repo) — je n'ai pas d'accès direct en écriture au dépôt GitHub de l'utilisateur.
- Prochaines pistes possibles (non demandées à ce jour) : enrichir les explications des ~340 questions restantes qui n'ont pas encore de note contextuelle ; ajouter un mode "questions à revoir en priorité" basé sur les erreurs en QCM.
