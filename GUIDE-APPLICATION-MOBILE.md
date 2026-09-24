# De l'application web à une vraie application mobile

Guide complet, pas à pas, pour transformer `index.html` en application installable sur Android et iPhone.

**Public visé** : quelqu'un qui n'a jamais fait de développement mobile.
**Ordinateur** : Windows 11.
**Point de départ** : l'application web existante, `github.com/JuanYule/Examen_Civique`.

---

## Sommaire

- [0. À lire avant tout](#0-à-lire-avant-tout)
- [1. Quelle technologie choisir](#1-quelle-technologie-choisir)
- [2. Logiciels à installer](#2-logiciels-à-installer)
- [3. Créer le projet](#3-créer-le-projet)
- [4. Lancer l'application sur un émulateur](#4-lancer-lapplication-sur-un-émulateur)
- [5. Lancer sur ton vrai téléphone Android](#5-lancer-sur-ton-vrai-téléphone-android)
- [6. Icône, nom et écran de démarrage](#6-icône-nom-et-écran-de-démarrage)
- [7. Améliorations natives](#7-améliorations-natives)
- [8. Publier sur le Play Store](#8-publier-sur-le-play-store)
- [9. Le cas de l'iPhone](#9-le-cas-de-liphone)
- [10. Mettre à jour l'application](#10-mettre-à-jour-lapplication)
- [11. Récapitulatif des points de contrôle](#11-récapitulatif-des-points-de-contrôle)
- [12. Problèmes fréquents](#12-problèmes-fréquents)
- [13. Budget et calendrier](#13-budget-et-calendrier)

---

## 0. À lire avant tout

### La contrainte à connaître dès maintenant

| Plateforme | Depuis Windows 11 | Explication |
|---|---|---|
| **Android** | ✅ Possible de bout en bout | Android Studio existe sur Windows |
| **iPhone** | ❌ Impossible directement | Apple impose Xcode, qui n'existe que sur macOS |

Ce n'est pas contournable par une astuce de configuration : Apple ne fournit pas ses outils de compilation ailleurs que sur Mac. Les trois solutions réelles sont détaillées en [section 9](#9-le-cas-de-liphone).

### Ce que ce projet ne demande pas

Tu n'as **pas** à apprendre Java, Kotlin, Swift ou React Native. Ton `index.html` fonctionnera tel quel à l'intérieur de l'application. Ce que tu vas apprendre relève de l'outillage — installer, lancer, compiler — pas d'un nouveau langage.

### Faut-il vraiment une application native ?

Question honnête à se poser avant d'investir du temps.

| | Application web actuelle (PWA) | Application native |
|---|---|---|
| Coût | 0 € | 25 € Android, 99 €/an iPhone |
| Mise à jour | Copier-coller, effet immédiat | Nouvelle version soumise au store |
| Installation | Ajouter à l'écran d'accueil depuis Safari | Depuis le store |
| Fonctionne hors ligne | Oui | Oui |
| Vibration sur iPhone | Non | Oui |
| Notifications | Limitées | Complètes |
| Visible par d'autres | Non | Oui, via le store |
| Délai de publication | Aucun | 1 à 3 jours, plus 14 jours de test obligatoire sur Android |

**Le vrai gain d'une application native** : la visibilité sur les stores, les notifications, et la vibration sur iPhone. Si l'application ne sert qu'à toi, la version web actuelle reste plus pratique. Si tu veux la partager largement, l'application native se justifie.

---

## 1. Quelle technologie choisir

J'ai comparé les trois voies possibles.

| | **Capacitor** | React Native | Flutter |
|---|---|---|---|
| Réutilise ton code | ✅ Presque tel quel | ❌ Tout à réécrire | ❌ Tout à réécrire |
| Langage à apprendre | Aucun | JavaScript + React | Dart |
| Temps estimé | 1 à 2 jours | 3 à 6 semaines | 4 à 8 semaines |
| Performance | Bonne pour ce type d'application | Excellente | Excellente |
| Risque de casser l'existant | Faible | Total | Total |

**Recommandation : Capacitor.** Ton application est du HTML, du CSS et du JavaScript ; Capacitor l'enveloppe dans une coque native sans la réécrire. Les deux autres imposeraient de tout refaire pour un bénéfice invisible sur une application de fiches et de QCM.

Ce que Capacitor apporte concrètement : un vrai fichier installable, une présence sur les stores, un accès aux fonctions du téléphone (vibration, notifications, partage), et le fonctionnement hors ligne garanti.

### Éditeur de code

**Visual Studio Code**, pour trois raisons : gratuit, léger, et de loin le plus documenté — toute recherche d'aide en ligne suppose que tu l'utilises. Android Studio sera installé aussi, mais uniquement comme outil de compilation et d'émulation, pas pour écrire du code.

---

## 2. Logiciels à installer

Ordre à respecter. Compte 1 h 30 à 2 h, dont beaucoup d'attente.

### 2.1 Node.js — le moteur des outils

Capacitor 8 exige **Node.js 22 ou supérieur**.

1. Aller sur **nodejs.org**
2. Télécharger la version **LTS** (Long Term Support), pas la « Current »
3. Lancer l'installateur, tout accepter par défaut
4. **Important** : laisser cochée la case « Add to PATH »

> ### ✅ Point de contrôle 1
> Ouvrir le menu Démarrer, taper `cmd`, ouvrir l'**Invite de commandes**, puis :
> ```
> node --version
> npm --version
> ```
> Attendu : un numéro commençant par `v22` ou plus, et un numéro pour npm.
> **Si « n'est pas reconnu »** : redémarrer l'ordinateur, puis réessayer.

### 2.2 Visual Studio Code — l'éditeur

1. Aller sur **code.visualstudio.com**
2. Télécharger pour Windows, installer
3. **Cocher** « Ajouter au menu contextuel de l'Explorateur » — tu pourras ouvrir un dossier par clic droit

Extensions à installer (icône carrés à gauche, rechercher, cliquer sur Install) :

| Extension | Utilité |
|---|---|
| French Language Pack | Interface en français |
| Live Server | Tester la page web instantanément |
| Prettier | Met en forme le code proprement |
| Auto Rename Tag | Renomme automatiquement les balises HTML par paire |

> ### ✅ Point de contrôle 2
> VS Code s'ouvre et les quatre extensions apparaissent dans l'onglet Extensions, section « Installed ».

### 2.3 Git — pour publier sur GitHub

1. Aller sur **git-scm.com**, télécharger pour Windows
2. Pendant l'installation, une seule option compte : à « Choosing the default editor », sélectionner **Visual Studio Code**
3. Tout le reste par défaut

> ### ✅ Point de contrôle 3
> ```
> git --version
> ```
> Attendu : un numéro de version.

### 2.4 Android Studio — le plus long

Environ 1 Go à télécharger, plus 3 à 6 Go de composants. Prévoir du temps et de l'espace disque.

1. Aller sur **developer.android.com/studio**
2. Télécharger, lancer l'installateur
3. Au premier démarrage, choisir **Standard** dans l'assistant
4. Accepter les licences, laisser télécharger

Capacitor 8 exige **Android Studio Otter (2025.2.1) ou plus récent**. Java 21 est inclus, aucune installation séparée n'est nécessaire.

Ensuite, vérifier le SDK : **More Actions → SDK Manager**
- Onglet **SDK Platforms** : cocher la version d'Android la plus récente
- Onglet **SDK Tools** : cocher **Android SDK Build-Tools**, **Android SDK Platform-Tools**, **Android Emulator**
- **Apply**, attendre

> ### ✅ Point de contrôle 4
> Android Studio s'ouvre sur l'écran d'accueil, et le SDK Manager affiche au moins une plateforme Android installée.
> Noter le chemin du SDK affiché en haut du SDK Manager, du type `C:\Users\TonNom\AppData\Local\Android\Sdk` — il servira si un problème survient.

### 2.5 Récapitulatif

| Logiciel | Rôle | Taille | Obligatoire |
|---|---|---|---|
| Node.js LTS | Fait tourner les outils | ~50 Mo | Oui |
| Visual Studio Code | Écrire le code | ~100 Mo | Oui |
| Git | Publier sur GitHub | ~50 Mo | Oui |
| Android Studio | Compiler et émuler Android | ~4 à 7 Go | Oui |
| Google Chrome | Déboguer l'application | ~100 Mo | Recommandé |

---

## 3. Créer le projet

### 3.1 Structure

Créer un dossier `C:\Projets\examen-civique-app`. Il contiendra :

```
examen-civique-app/
├── www/                  ← ton application web
│   └── index.html            (le fichier existant, inchangé)
├── android/              ← projet Android, généré automatiquement
├── ios/                  ← projet iPhone, généré (inutilisable sur Windows)
├── resources/            ← icône et écran de démarrage
├── capacitor.config.json ← configuration de l'application
├── package.json          ← liste des outils utilisés
└── node_modules/         ← outils installés (ne jamais toucher)
```

**Règle d'or** : tu ne modifies que `www/index.html`. Les dossiers `android/` et `ios/` sont régénérés automatiquement — toute modification manuelle y sera perdue.

### 3.2 Initialiser

Ouvrir le dossier dans VS Code (clic droit → « Ouvrir avec Code »), puis ouvrir le terminal intégré : menu **Terminal → Nouveau terminal**. Taper les commandes une par une, en attendant la fin de chacune.

```bash
npm init -y
npm install @capacitor/core @capacitor/cli
npx cap init
```

Trois questions sont posées :

| Question | Réponse à donner | Remarque |
|---|---|---|
| App name | `Examen Civique` | Le nom affiché sous l'icône |
| App Package ID | `com.juanyule.examencivique` | **Définitif** : impossible à changer après publication |
| Web asset directory | `www` | Où se trouve ton HTML |

> ⚠️ **L'identifiant de paquet est définitif.** Il identifie l'application sur les stores. Format : `com.tonnom.nomapp`, en minuscules, sans accent ni espace ni tiret.

### 3.3 Placer l'application

```bash
mkdir www
```

Copier ton `index.html` actuel dans le dossier `www`.

> ### ✅ Point de contrôle 5
> Dans VS Code, l'explorateur affiche `www/index.html`, `capacitor.config.json` et `package.json`.
> Clic droit sur `www/index.html` → **Open with Live Server** : l'application s'ouvre dans le navigateur et fonctionne.

### 3.4 Ajouter la plateforme Android

```bash
npm install @capacitor/android
npx cap add android
npx cap sync
```

`npx cap sync` copie `www/` vers le projet Android. **À relancer après chaque modification du HTML** — c'est l'oubli le plus fréquent.

> ### ✅ Point de contrôle 6
> Un dossier `android/` est apparu, contenant lui-même `app/`, `gradle/`, etc.
> Le terminal affiche `✔ Copying web assets` et `✔ Updating Android plugins` sans erreur en rouge.

---

## 4. Lancer l'application sur un émulateur

L'émulateur est un téléphone Android simulé sur ton PC.

### 4.1 Créer l'émulateur

1. Android Studio → **More Actions → Virtual Device Manager**
2. **Create Device**
3. Choisir **Pixel 7** (bon compromis de taille)
4. Choisir la dernière image système proposée ; si un bouton **Download** apparaît, cliquer et attendre
5. **Finish**

### 4.2 Lancer

```bash
npx cap open android
```

Android Studio s'ouvre sur le projet. Attendre que la barre de progression en bas soit terminée — **le premier chargement (« Gradle sync ») peut prendre 10 à 20 minutes**. C'est normal, ne rien toucher.

Ensuite : sélectionner l'émulateur dans le menu déroulant en haut, puis cliquer sur le **triangle vert ▶**.

> ### ✅ Point de contrôle 7
> L'émulateur démarre et ton application s'affiche.
> Vérifier : les fiches défilent, le glissement gauche/droite fonctionne, le QCM répond, un examen blanc démarre.
> **Astuce** : le glissement se simule en cliquant et faisant glisser à la souris.

---

## 5. Lancer sur ton vrai téléphone Android

L'émulateur ne reproduit pas fidèlement le tactile. Cette étape est indispensable.

### 5.1 Activer le mode développeur sur le téléphone

1. **Paramètres → À propos du téléphone**
2. Appuyer **7 fois** sur **Numéro de build** — un message confirme l'activation
3. Revenir dans **Paramètres → Système → Options pour les développeurs**
4. Activer **Débogage USB**

### 5.2 Connecter

1. Brancher le téléphone en USB
2. Sur le téléphone, accepter **« Autoriser le débogage USB ? »**
3. Dans Android Studio, ton téléphone apparaît dans le menu déroulant des appareils
4. Le sélectionner, cliquer sur ▶

> ### ✅ Point de contrôle 8
> L'application s'installe sur ton téléphone et s'ouvre.
> **À tester sérieusement** : glissement lent, glissement rapide, appui long immobile, défilement à une main, rotation de l'écran, sortie et retour dans l'application.

### 5.3 Déboguer

Sur ton PC, ouvrir Chrome et saisir `chrome://inspect`. Ton application apparaît ; cliquer sur **inspect** ouvre les outils de développement **connectés à ton téléphone**. Tu vois la console, tu peux exécuter du JavaScript à distance. C'est l'outil le plus utile de tout ce guide.

---

## 6. Icône, nom et écran de démarrage

### 6.1 Préparer les images

| Fichier | Taille | Contenu |
|---|---|---|
| `resources/icon.png` | 1024 × 1024 px | L'icône, sans coins arrondis (le système s'en charge) |
| `resources/splash.png` | 2732 × 2732 px | Écran de démarrage, motif centré dans le tiers du milieu |

Outils gratuits pour les créer : **Canva**, **Figma**, ou **GIMP**.

Suggestion cohérente avec l'application : fond bleu marine `#000091`, monogramme « RF » ou une silhouette de colonne blanche au centre.

### 6.2 Générer toutes les tailles

```bash
npm install @capacitor/assets --save-dev
npx capacitor-assets generate --android
npx cap sync
```

Toutes les déclinaisons nécessaires sont produites automatiquement.

> ### ✅ Point de contrôle 9
> Relancer l'application. L'icône personnalisée apparaît sur l'écran d'accueil du téléphone, et l'écran de démarrage s'affiche au lancement.

---

## 7. Améliorations natives

Facultatif, mais c'est le principal intérêt d'une application native.

### 7.1 Vibration

Ton code utilise `navigator.vibrate()`, qui ne fonctionne pas sur iPhone. Le module natif fonctionne partout.

```bash
npm install @capacitor/haptics
npx cap sync
```

Dans `www/index.html`, section `[J5]`, remplacer la fonction `haptic` par :

```js
function haptic(ms){
  // Version native : fonctionne aussi sur iPhone, contrairement à navigator.vibrate
  if(window.Capacitor?.Plugins?.Haptics){
    window.Capacitor.Plugins.Haptics.impact({style:'LIGHT'});
    return;
  }
  if(navigator.vibrate){ try{ navigator.vibrate(ms); }catch(e){} }
}
```

### 7.2 Barre d'état

```bash
npm install @capacitor/status-bar
npx cap sync
```

Dans `capacitor.config.json` :

```json
{
  "plugins": {
    "StatusBar": { "style": "DARK", "backgroundColor": "#000091" }
  }
}
```

### 7.3 Bouton retour d'Android

Sans cela, le bouton retour ferme l'application au lieu de revenir à l'écran précédent.

```bash
npm install @capacitor/app
npx cap sync
```

À ajouter dans la section `[J17]` de ton HTML :

```js
// Bouton retour d'Android : revient à l'accueil, ne ferme l'application
// que si l'on y est déjà.
if(window.Capacitor?.Plugins?.App){
  window.Capacitor.Plugins.App.addListener('backButton', ({canGoBack})=>{
    const accueil = document.getElementById('screen-home').classList.contains('active');
    if(accueil) window.Capacitor.Plugins.App.exitApp();
    else goTab('home');
  });
}
```

> ### ✅ Point de contrôle 10
> Sur le téléphone : la vibration se déclenche à la validation d'une fiche, la barre d'état est bleue, et le bouton retour ramène à l'accueil sans fermer l'application.

---

## 8. Publier sur le Play Store

### 8.1 Créer le compte

1. **play.google.com/console**
2. Compte développeur : **25 $ US, paiement unique et définitif**
3. Vérification d'identité : pièce d'identité demandée, **délai de 1 à 3 jours**

> ⚠️ **Contrainte majeure à connaître dès maintenant.** Pour un compte personnel créé après le 13 novembre 2023, Google impose un **test fermé avec au moins 12 testeurs inscrits en continu pendant 14 jours** avant d'autoriser la publication publique. Les testeurs doivent être 12 comptes Google distincts, sur de vrais appareils. Ce n'est pas contournable. **Commence à recruter tes 12 personnes dès aujourd'hui** — c'est le principal facteur de délai, bien avant la technique.

### 8.2 Signer l'application

La clé de signature prouve que les mises à jour viennent bien de toi.

Dans Android Studio : **Build → Generate Signed Bundle / APK → Android App Bundle → Create new...**

| Champ | Valeur |
|---|---|
| Key store path | `C:\Projets\cles\examen-civique.jks` |
| Password | Un mot de passe fort |
| Alias | `examencivique` |
| Validity | 25 ans minimum |

> 🔴 **Sauvegarde ce fichier `.jks` et son mot de passe dans au moins deux endroits sûrs.**
> Perdre cette clé signifie **ne plus jamais pouvoir mettre à jour l'application**. Il faudrait la republier sous une nouvelle identité, en perdant les utilisateurs et les avis. C'est l'erreur irréparable la plus courante.

### 8.3 Produire le fichier à envoyer

**Build → Generate Signed Bundle / APK → Android App Bundle**, choisir la clé créée, variante **release**.

Le fichier apparaît dans `android/app/release/app-release.aab`.

### 8.4 Ce que Google exige

| Élément | Détail |
|---|---|
| Fichier `.aab` | Produit à l'étape précédente |
| Icône | 512 × 512 px |
| Image de présentation | 1024 × 500 px |
| Captures d'écran | 2 à 8, prises depuis ton téléphone |
| Description courte | 80 caractères maximum |
| Description longue | 4000 caractères maximum |
| Politique de confidentialité | **Obligatoire**, URL publique |
| Classification du contenu | Questionnaire à remplir |
| Public cible | Adultes |

**Politique de confidentialité** : ton application ne collecte aucune donnée, tout reste sur l'appareil. Une page simple suffit. Elle peut être hébergée gratuitement sur ton dépôt GitHub, en ajoutant un fichier `privacy.md`.

Modèle utilisable tel quel :

> **Politique de confidentialité — Examen Civique**
> Cette application ne collecte, ne transmet et ne partage aucune donnée personnelle. La progression est enregistrée uniquement sur l'appareil de l'utilisateur et n'est envoyée à aucun serveur. L'application ne contient ni publicité, ni outil de mesure d'audience. Elle ne demande aucune autorisation d'accès au téléphone.
> Contact : *ton adresse électronique*

> ### ✅ Point de contrôle 11
> Le tableau de bord de la Play Console affiche toutes les sections en vert, et le fichier `.aab` est accepté sans avertissement bloquant.

---

## 9. Le cas de l'iPhone

Rappel : Xcode n'existe que sur macOS. Voici les trois voies réelles.

### Option A — Rester sur la version web (recommandée pour commencer)

Ton application fonctionne **déjà** sur iPhone via Safari et l'ajout à l'écran d'accueil. Coût nul, aucune contrainte, aucun délai.

Limites : pas de vibration, pas de notifications, absente de l'App Store.

**C'est le meilleur rapport effort/bénéfice tant que l'application te sert à toi et à quelques proches.**

### Option B — Compilation dans le nuage

Des services fournissent des Mac à distance qui compilent ton projet.

| Service | Offre gratuite | Difficulté |
|---|---|---|
| **Codemagic** | 500 min/mois | Moyenne |
| **GitHub Actions** | 2000 min/mois sur dépôt public | Élevée |
| **Ionic Appflow** | Essai limité | Faible |

Il faut malgré tout un **compte Apple Developer à 99 $ US par an** pour signer et publier.

Codemagic est le plus abordable : tu connectes ton dépôt GitHub, il détecte Capacitor, et te guide pour les certificats.

### Option C — Emprunter ou louer un Mac

- Un Mac d'occasion (Mac mini M1) suffit largement
- Un Mac loué à l'heure (MacinCloud, environ 1 $/h) permet une publication ponctuelle
- Emprunter le Mac d'un proche une demi-journée suffit pour la première publication

### Recommandation

**Commence par Android seul.** Tu apprends toute la chaîne sur la plateforme qui fonctionne depuis ton PC. L'iPhone reste couvert par la version web. Quand l'application Android sera publiée et stable, tu décideras si l'App Store vaut les 99 $ annuels et la complexité supplémentaire.

---

## 10. Mettre à jour l'application

### Après chaque modification du HTML

```bash
npx cap sync
npx cap open android
```

Puis relancer avec ▶.

> ⚠️ Oublier `npx cap sync` est l'erreur la plus fréquente : tu modifies le HTML, rien ne change, et tu cherches un bug qui n'existe pas.

### Pour publier une nouvelle version

Dans `android/app/build.gradle`, incrémenter :

```gradle
versionCode 2          // un entier, +1 à chaque publication, obligatoire
versionName "1.1"      // le numéro visible par les utilisateurs
```

Puis regénérer le `.aab` signé **avec la même clé**, et l'envoyer sur la Play Console.

---

## 11. Récapitulatif des points de contrôle

| # | Étape | Vérification |
|---|---|---|
| 1 | Node.js | `node --version` renvoie v22 ou plus |
| 2 | VS Code | Les quatre extensions sont installées |
| 3 | Git | `git --version` renvoie un numéro |
| 4 | Android Studio | SDK Manager affiche une plateforme installée |
| 5 | Projet créé | `www/index.html` s'ouvre avec Live Server |
| 6 | Android ajouté | Le dossier `android/` existe, `sync` sans erreur |
| 7 | Émulateur | L'application tourne, fiches et QCM fonctionnent |
| 8 | Téléphone réel | Installée, gestes tactiles corrects |
| 9 | Identité visuelle | Icône et écran de démarrage personnalisés |
| 10 | Fonctions natives | Vibration, barre d'état, bouton retour |
| 11 | Play Console | Toutes les sections en vert, `.aab` accepté |
| 12 | Test fermé | 12 testeurs inscrits pendant 14 jours |
| 13 | Publication | Application visible sur le Play Store |

---

## 12. Problèmes fréquents

| Message ou symptôme | Cause | Solution |
|---|---|---|
| `node n'est pas reconnu` | PATH non pris en compte | Redémarrer l'ordinateur |
| `npx cap add android` échoue | `www` absent ou vide | Créer `www` et y placer `index.html` |
| Gradle sync interminable | Premier téléchargement | Attendre, jusqu'à 20 min ; connexion stable requise |
| `SDK location not found` | Chemin du SDK inconnu | Créer `android/local.properties` avec `sdk.dir=C\:\\Users\\TonNom\\AppData\\Local\\Android\\Sdk` |
| Écran blanc au lancement | `sync` oublié | `npx cap sync` |
| Modifications invisibles | `sync` oublié | `npx cap sync` |
| Téléphone non détecté | Débogage USB inactif | Réactiver, changer de câble (certains ne transmettent que le courant) |
| L'émulateur ne démarre pas | Virtualisation désactivée | Activer VT-x ou AMD-V dans le BIOS |
| `.aab` refusé par Google | Version déjà utilisée | Incrémenter `versionCode` |
| Gestes moins fluides que dans Safari | WebView Android ancienne | Mettre à jour Android System WebView depuis le Play Store |

---

## 13. Budget et calendrier

### Coûts

| Poste | Android | iPhone |
|---|---|---|
| Compte développeur | 25 $ une fois | 99 $ par an |
| Logiciels | 0 € | 0 € |
| Compilation dans le nuage | — | 0 à 30 $/mois |
| **Première année** | **25 $** | **99 $ minimum** |

### Calendrier réaliste

| Phase | Durée | Remarque |
|---|---|---|
| Installation des logiciels | 2 h | Beaucoup d'attente |
| Création du projet | 1 h | |
| Premier lancement en émulateur | 1 h | Gradle est lent la première fois |
| Test sur téléphone réel | 30 min | |
| Icône et écran de démarrage | 1 à 2 h | Selon le soin apporté au visuel |
| Fonctions natives | 1 h | Facultatif |
| Préparation Play Console | 2 à 3 h | Captures, descriptions, confidentialité |
| Vérification du compte Google | 1 à 3 jours | Attente |
| **Test fermé obligatoire** | **14 jours minimum** | **Le vrai goulot d'étranglement** |
| Examen par Google | 1 à 7 jours | |
| **Total jusqu'à la publication** | **3 à 4 semaines** | Dont 3 semaines d'attente |

**La technique représente environ deux jours de travail. Le reste est de l'attente administrative.** D'où le conseil de commencer à recruter tes 12 testeurs dès maintenant, en parallèle du développement.

---

## Par où commencer, concrètement

Aujourd'hui, dans cet ordre :

1. Installer Node.js, puis vérifier le **point de contrôle 1**
2. Installer VS Code et ses extensions, **point de contrôle 2**
3. Installer Git, **point de contrôle 3**
4. Lancer le téléchargement d'Android Studio et faire autre chose pendant ce temps
5. Pendant l'attente : dresser la liste de 12 personnes disposant d'un téléphone Android et d'un compte Google, prêtes à tester pendant deux semaines

Puis, dans les jours suivants, dérouler les sections 3 à 5 jusqu'à voir ton application tourner sur ton propre téléphone. C'est l'étape la plus satisfaisante, et elle est atteignable en une soirée une fois les logiciels installés.
