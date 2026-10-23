# Mission : finir « Capsuel », la capsule temporelle de mon groupe de potes

Salut. Je te confie un projet **déjà avancé** à terminer, vérifier et livrer prêt à
l'emploi. Lis ce prompt en entier avant de toucher au code.

## 0. Ce que tu reçois

Archive `capsuel-base.zip` = le dossier `capsuel/` :
- une **PWA React sans build chain** (React + Babel standalone servis depuis `vendor/`,
  chaque fichier `.jsx` expose ses composants via `Object.assign(window, …)`, pas d'ESM,
  pas d'import) avec **14 univers** (Le Fil, Le Seuil, Bibliothèque, Carte, Anecdotes,
  Mind-map, Stats, Citations, Bandes-sons, Polaroids, Reliques, Future Self,
  Attrape-Rêves, Encyclopédie) ;
- un **serveur Node auto-hébergé sans dépendance npm** (`server/`) qui réimplémente le
  contrat d'API attendu par le client (`src/prolive-client.js`) : porte d'entrée par
  passphrase, synchronisation, médias, WebSocket de présence ;
- déjà ajoutés (à vérifier, pas à refaire) : le bouton **« + »** (`src/quick-add.jsx`),
  la lecture **EXIF** date + GPS (`src/exif.js`), la **porte d'entrée** (`src/join-gate.jsx`),
  un **constructeur de site** façon Wix avec scripts façon Scratch « Quand [événement] →
  [action] » (`src/site-model.js`, `src/site-runtime.jsx`, `src/site-editor.jsx`,
  `src/pages/site.jsx`, `styles/site.css`) ;
- des tests : `server/test/smoke.mjs` (85 tests d'API), `tools/check-jsx.mjs`,
  `tools/parcours-e2e.mjs` (parcours Chrome), `tools/make-test-photo.mjs` (fabrique une
  photo avec EXIF date + GPS) ;
- la doc : `README.md` et `README-DEPLOIEMENT.md` — **lis-les en premier**.

Deux dossiers de **référence** (à lire, pas à servir tels quels) :
- `reference-ame/` : `BMAD_PROJECT_VALUES.md` et `DESIGN_BRIEF.md` = l'âme du projet
  (direction artistique, ton, ce qu'il ne faut jamais faire). Ils citent ma capsule
  privée : inspire-toi de l'esprit, ne recopie **aucun** contenu personnel.
- `reference-deezer/` : le téléchargement Deezer déjà écrit pour ma capsule privée
  (`tools/deemix-helper.py`, `tools/deemix-batch.py`, `tools/vendor/` = librairies Python
  embarquées, `tools/deemix-settings/`, `src/pages/playlists.jsx` avec la fenêtre
  « Télécharger depuis Deezer », `AIDE_DEEZER.html`). À intégrer (cf. §3.10).

`PROMPT-GLM.md` (ce texte) est aussi dans l'archive : relis-le si ta session redémarre.

## 1. ⛔ Règle n°1 : NE PAS RÉÉCRIRE

Une tentative précédente a **jeté ce projet et tout refait en Next.js + Prisma**. Résultat
refusé : les 14 univers ont disparu et le site était ouvert à n'importe qui. Donc :
- tu **gardes** cette base, cette architecture (pas de Next.js, pas de bundler, pas de
  TypeScript, pas de framework serveur) et **les 14 univers** ;
- tu **complètes, corriges, testes et peaufines** ;
- toute nouvelle librairie front est **copiée dans `vendor/`** (aucun CDN : le site doit
  marcher sans Internet sauf les tuiles de carte) ;
- le serveur reste **sans dépendance npm** (ou presque : justifie chaque exception).

## 2. Le contexte

- **10 potes de lycée**, non-développeurs, surtout **sur téléphone** (iPhone et Android).
- Hébergement : mon PC gamer « banane » — **Ubuntu Server 26.04, Node 22 déjà installé**,
  Tailscale déjà installé avec certificats HTTPS. Le PC n'est **pas allumé 24 h/24** : il
  est réveillé à distance (Wake-on-LAN) quand quelqu'un veut consulter la capsule.
- Mise en ligne prévue : serveur Node sur `127.0.0.1:8787`, exposé en HTTPS public par
  **Tailscale Funnel** (`tailscale funnel --bg 8787` → `https://banane.<tailnet>.ts.net/`).
  Les potes ouvrent le lien et tapent la passphrase ; ensuite on leur **propose
  d'installer** la capsule comme une app (téléphone et PC), cf. §3.8.
- Nom du groupe = titre par défaut de la capsule : **« Nik la société capitaliste entre
  cop1 »** (modifiable dans l'app). Ne mets **aucune** passphrase en dur : je la
  choisirai au déploiement avec `cli.js passphrase`.
- Tout le texte visible est **en français**. Ton chaleureux, un peu chaotique, « entre
  potes ». **Aucun faux contenu humain** : n'invente jamais de nom, d'anecdote, de photo ou
  de souvenir ; les zones vides affichent un texte d'invitation (« Rien ici encore… »).
- **Aucun contenu personnel** de ma capsule privée : pas de « Lamine », « Cypis »,
  « Gringo », « Finley », « F + G », fluoxétine, Amsterdam en dur. Pas de paiement ni de
  licence visibles par les potes. (Le téléchargement Deezer, lui, est **voulu** : §3.10.)

## 3. Ce qui doit marcher à la livraison (critères d'acceptation)

### 3.1 Accès réservé au groupe
- Sans la passphrase (ou une invitation), **on ne voit rien** : ni souvenirs, ni photos,
  ni noms. Vérifie que ni l'API ni les fichiers statiques ne laissent fuir du contenu
  (teste avec `curl` sans cookie sur toutes les routes, y compris `/api/.../media/...`).
- Le limiteur anti-brute-force ne doit **pas bloquer tout le groupe** : derrière Tailscale
  Funnel, toutes les connexions arrivent de `127.0.0.1`. Ajoute `TRUST_PROXY=1` dans
  `.env` pour lire `X-Forwarded-For`, et sinon compte par session plutôt que par IP.

### 3.2 Le bouton « + » (porte d'entrée principale)
- Visible partout, gros, utilisable au pouce sur téléphone.
- Types : **photo(s)** (plusieurs d'un coup), **vidéo**, **audio/musique**, **anecdote**,
  **citation**, **lien** (YouTube, Deezer, Spotify…), **autre** (texte libre).
- Chaque dépôt est **rangé automatiquement** : dans Le Fil à sa date ET dans l'univers
  logique (photo → Polaroids + Carte si GPS ; anecdote → Anecdotes ; citation →
  Citations ; musique/lien musical → Bandes-sons ; etc.). Documente la table de rangement.
- L'auteur est le pseudo choisi à l'entrée (pas de saisie à chaque fois).

### 3.3 Métadonnées photo (EXIF) — automatique
- Date de **prise de vue** → la photo se range à cette date (pas la date d'envoi).
- GPS → **épingle automatique sur la carte** du groupe + nom du lieu en français
  (géocodage inverse via le serveur, avec cache, ≤ 1 requête/s vers Nominatim).
- ⚠ **iPhone** : les photos sont souvent en **HEIC**, et Safari peut retirer le GPS à
  l'envoi. Il faut : (a) accepter le HEIC (lire l'EXIF du HEIC ; pour l'affichage,
  convertir en JPEG dans le navigateur avec une lib copiée dans `vendor/`, ou côté
  serveur si impossible) ; (b) quand il n'y a **pas** de date ou de GPS, proposer dans le
  formulaire « + » de **choisir la date** et de **chercher le lieu** (recherche de lieu)
  ou de **poser l'épingle sur la carte** — en 2 gestes maximum.
- Fixtures de test : JPEG avec date+GPS, JPEG sans EXIF, PNG, HEIC (si tu peux en
  produire un), photo pivotée (orientation EXIF respectée à l'affichage).

### 3.4 Tout éditer, façon Wix / Canva, sans savoir coder
- Les potes doivent pouvoir **personnaliser 100 % de la capsule** : le Seuil (accueil),
  les **univers existants** (nom, couleurs, fond d'écran, ordre, affichage oui/non), les
  pages du constructeur de site, les blocs (texte, titre, photo, galerie, vidéo, audio,
  carte, compte à rebours, bouton, sticker, dessin à main levée…), avec glisser-déposer,
  redimensionnement, alignement magnétique, annuler/rétablir, dupliquer, supprimer.
- La logique « façon Scratch » (`Quand [clic / survol / apparition / date] → [actions
  animées : apparaître, tourner, rebondir, aller à une page, jouer un son, confettis…]`)
  doit être **simple à assembler** et avoir un bouton « essayer ».
- Pense « ado de 17 ans sur téléphone » : chaque outil s'explique de lui-même, des
  modèles prêts à l'emploi (« page anniversaire », « page voyage », « mur de photos »),
  et **rien ne peut casser la capsule** (annuler partout, suppression confirmée).
- Les modifications sont **sauvegardées sur le serveur** et vues par tout le groupe.

### 3.5 Dix potes en même temps, sans perte
- Le modèle actuel pousse l'**état complet** avec horodatage et renvoie `409` + une
  fenêtre de conflit. À 10, ça donnera des conflits sans arrêt. **C'est le risque n°1.**
- Exige : deux potes qui **ajoutent** des choses différentes en même temps ne doivent
  **jamais** voir de conflit ni perdre quoi que ce soit (fusion côté serveur par entité,
  via les `id`, ou opérations unitaires ajouter / modifier / supprimer). La fenêtre de
  conflit ne doit apparaître que si deux personnes modifient **le même** élément.
- Les ajouts des autres apparaissent **sans recharger** (WebSocket déjà présent).
- **Test de charge obligatoire** : script `tools/test-10-potes.mjs` qui simule 10 clients
  qui rejoignent, ajoutent chacun 20 souvenirs (dont des photos) en parallèle et éditent
  le site ; résultat attendu : 200 souvenirs présents, 0 perte, 0 conflit injustifié.

### 3.6 Serveur à la demande (le PC n'est pas allumé en permanence)
- Le service **démarre tout seul au boot** (systemd, `deploy/capsuel.service`).
- Ajoute une **extinction automatique quand personne ne s'en sert** : le serveur écrit
  l'heure de sa dernière activité (requête ou WebSocket ouvert) ; un timer systemd
  (`deploy/capsuel-idle.timer` + `.service`, toutes les 5 min) éteint le PC si aucune
  activité depuis `IDLE_MINUTES` (défaut 60) **et** démarrage depuis plus de 20 min, en
  appelant `/usr/local/bin/safepoweroff.sh` (script existant sur banane qui refuse
  d'éteindre si un autre travail tourne). **Désactivé par défaut** (`IDLE_SHUTDOWN=0`),
  documenté. Ne touche à rien d'autre sur la machine.
- **Quand le serveur dort**, l'app installée (ou déjà ouverte une fois) doit quand même
  s'ouvrir et afficher **la dernière version synchronisée** de la capsule (l'app est déjà
  « hors-ligne d'abord »), avec un bandeau clair « La capsule dort 💤 » ; les dépôts faits
  pendant ce temps sont gardés et envoyés au réveil. Si `WAKE_URL` est renseigné dans la
  config, le bandeau affiche un bouton « Réveiller la capsule » qui ouvre cette adresse
  (je fournirai le relais de réveil moi-même ; toi tu prévois juste le bouton et la
  config, puis la reconnexion automatique dès que le serveur répond).

### 3.7 Mobile, beauté, robustesse
- Vérifie chaque écran à **375 px de large** (Le Fil, « + », Carte, porte d'entrée,
  éditeur, univers) : rien de coupé, rien d'inatteignable.
- Photos redimensionnées dans le navigateur avant envoi ; miniatures ; vidéos jusqu'à
  ~200 Mo ; message clair si trop gros.
- Export `.capsule` et sauvegarde serveur (`cli.js backup`, `deploy/backup.sh`) intacts.
- Si tu modifies des fichiers mis en cache, **incrémente `VERSION` dans `sw.js`**.

### 3.8 Installer la capsule comme une app (téléphone + PC)
- L'installation est **encouragée** : après l'entrée dans la capsule, une invitation
  douce « Installer la capsule » (refusable, ne revient pas sans arrêt).
- Android / Chrome / Edge PC : vrai bouton d'installation (`beforeinstallprompt`).
- iPhone (Safari) : mini-guide illustré « Partager → Sur l'écran d'accueil ».
- Nom, icône et couleur de l'app = ceux du groupe ; l'app installée ouvre directement la
  capsule du groupe (pas la page d'accueil commerciale), et marche hors-ligne (§3.6).

### 3.9 La boîte à idées du groupe (publique)
- Un endroit visible (bouton dans le chrome + entrée dans le « + ») où chaque pote peut
  écrire : une **idée d'amélioration**, une **fonctionnalité qu'il voudrait mais n'a pas
  réussi à faire** avec l'éditeur, ou un **bug**. Champs : texte libre, catégorie
  (idée / « j'ai pas réussi à… » / bug), capture d'écran facultative ; l'auteur et la
  date sont posés automatiquement ; l'univers ou la page où il se trouvait aussi.
- **Publique dans le groupe** : tout le monde voit les idées, peut mettre un **+1** et
  commenter. Le mécène (moi) peut changer le statut : nouvelle / vue / prévue / faite.
- Dans l'éditeur, un lien « Je n'y arrive pas → le suggérer » pré-remplit une idée avec
  le contexte (bloc, page, ce qu'il essayait de faire).
- **Export pour moi** : un bouton (mécène) et une commande `cli.js idees` qui produisent
  un fichier Markdown propre (idées triées par +1, avec statut, auteur, date, contexte),
  que je pourrai transmettre tel quel à une IA pour faire évoluer la capsule.

### 3.10 Télécharger de la musique depuis Deezer
- Dans Bandes-sons (et dans le « + » → musique) : coller un lien Deezer (titre, album ou
  playlist) → le **serveur** télécharge les morceaux → ils arrivent dans la bibliothèque
  du groupe comme pistes locales (avec `sourceUrl` d'origine), lisibles par tous.
- Pars de `reference-deezer/` (même librairie deemix, mêmes réglages), mais le
  téléchargement tourne **sur le serveur** (Python 3 de banane ; dépendances installées
  par `deploy/install.sh` ou prises dans `tools/vendor/`), plus sur le PC de chacun.
- Le jeton Deezer (ARL) est **mon secret** : stocké côté serveur seulement (`data/` ou
  `.env`, commande `cli.js deezer-arl`), **jamais** envoyé au navigateur, jamais dans les
  logs, jamais dans l'archive livrée. Sans jeton configuré, la fonction affiche un message
  clair et le reste de l'app marche.
- File d'attente côté serveur (un téléchargement à la fois), progression visible dans
  l'app, réservé aux membres connectés, limite raisonnable par pote et par jour.

## 4. Déploiement : ce que je veux dans le dossier livré
- `deploy/install.sh` qui marche sur **Ubuntu 26.04 avec Node 22 déjà présent** (ne
  réinstalle pas Node s'il est ≥ 20), crée l'utilisateur système, copie dans
  `/opt/capsuel`, crée `.env`, initialise la capsule, installe et démarre le service.
  Idempotent : relancé pour une mise à jour, il ne touche **jamais** à `data/` ni `.env`.
- `README-DEPLOIEMENT.md` mis à jour avec la voie **Tailscale Funnel** en premier
  (commandes exactes, `APP_ORIGIN=https://banane.<tailnet>.ts.net`, WebSocket qui passe).
- Le dossier livré ne contient **pas** de `data/` réel (ni passphrase, ni clés, ni photos
  de test) : `cli.js init` les crée sur le serveur.

## 5. Méthode
1. Lis `README.md`, `README-DEPLOIEMENT.md`, `src/prolive-client.js`, `server/lib/api.js`.
2. Lance `node server/cli.js init` puis `node server/index.js`, puis tous les tests
   existants. Note ce qui casse **avant** de modifier quoi que ce soit.
3. Fais un plan court (dans `CHANGELOG-GLM.md`), puis avance point par point.
4. Teste **en vrai dans un navigateur** (porte d'entrée → pseudo → « + » photo avec GPS →
   la photo est datée et sur la carte → un 2ᵉ navigateur la voit sans recharger →
   édition du site → le 2ᵉ navigateur voit la modif), en largeur bureau ET 375 px.
5. ⚠ Ta sandbox peut **se réinitialiser** : fais un `git commit` après chaque étape et
   garde une archive à jour du projet (`tar` du dossier `capsuel/`) dans un dossier de
   téléchargement, pour ne jamais tout perdre.
6. Utilise des sous-agents si tu en as, mais vérifie toi-même leur travail.

## 6. Livrable final
- Le dossier `capsuel/` complet en **une archive** (`capsuel-final.tar` ou `.zip`),
  sans `data/`, sans `node_modules`.
- `CHANGELOG-GLM.md` à la racine : ce que tu as changé (fichier par fichier, en bref),
  ce que tu as testé et comment (résultats chiffrés des tests), ce qui reste imparfait.
- Tous les tests au vert : `smoke.mjs`, `check-jsx.mjs`, `parcours-e2e.mjs`,
  `test-10-potes.mjs` + tes nouveaux tests (boîte à idées, installation, mode « capsule
  qui dort », Deezer sans jeton = message propre).

Compris ? Commence par l'étape 1 et 2, dis-moi ce que tu trouves, puis enchaîne sans
t'arrêter jusqu'au livrable final.
