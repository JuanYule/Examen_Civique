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
