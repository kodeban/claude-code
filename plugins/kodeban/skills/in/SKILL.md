---
description: Commencer une tâche KodeBan dans cette conversation (ouvre une session de travail).
argument-hint: "[numéro de tâche | quelques mots]"
disable-model-invocation: true
allowed-tools: Bash(kodeban-poste) PowerShell(kodeban-poste) Bash(kodeban-poste *) PowerShell(kodeban-poste *) Bash(kodeban-ouvrir *) PowerShell(kodeban-ouvrir *) Bash(kodeban-nommer *) PowerShell(kodeban-nommer *) Bash(kodeban-pause *) PowerShell(kodeban-pause *) mcp__plugin_kodeban_kodeban__mes_sessions_ouvertes mcp__plugin_kodeban_kodeban__chercher_tache mcp__plugin_kodeban_kodeban__lire_tache mcp__plugin_kodeban_kodeban__lire_sessions mcp__plugin_kodeban_kodeban__ouvrir_session mcp__plugin_kodeban_kodeban__noter_session mcp__plugin_kodeban_kodeban__cloturer_session mcp__plugin_kodeban_kodeban__demander_choix_tache mcp__plugin_kodeban_kodeban__lire_choix_tache mcp__plugin_kodeban_kodeban__lire_depots_projet Bash(git remote -v) PowerShell(git remote -v) Bash(git fetch *) PowerShell(git fetch *) Bash(git switch *) PowerShell(git switch *) Bash(git status *) PowerShell(git status *) Bash(git branch *) PowerShell(git branch *)
---

# /kodeban:in — je commence

Ce que l'utilisateur a écrit après la commande : « $ARGUMENTS »
Référence de cette conversation : `${CLAUDE_SESSION_ID}`

Parle en français courant, en phrases courtes, sans jargon et sans nom d'outil.

Si un outil KodeBan ne répond pas (pas encore autorisé, accès révoqué, site éteint) : dis-le en une
phrase, ajoute « Branchez KodeBan : tapez `/mcp`, choisissez kodeban, puis authentifiez-vous. », et
arrête-toi.

## 1. Où suis-je

Lance `kodeban-poste` : il donne le poste et le dossier (ils servent à ouvrir la session). Puis
`mes_sessions_ouvertes` **sans aucun argument** : ne filtre ni par poste ni par dossier, car le
dossier courant change dès qu'une commande se déplace et la session de cette conversation ne
serait plus retrouvée. Fie-toi au site, pas à ta mémoire de la conversation.

## 2. Une session déjà ouverte

- Une session ouverte dont la référence (`sessionRef`) est celle de cette conversation, ou que cette
  conversation a elle-même ouverte : refuse. « Une conversation, une session. Clôturez avec
  `/kodeban:out`, puis ouvrez une nouvelle conversation. » Arrête-toi.
- Une session d'un autre poste (`workstation`), ou d'un dossier (`workdir`) sans rapport avec le
  tien (ni le même, ni un dossier qui contient l'autre) : c'est un autre travail. N'en parle pas et
  n'y touche pas.
- Une autre session ouverte pour ce poste et ce dossier (restée ouverte) : montre-la (numéro et
  titre de la tâche, date, dernière note) et propose : la clôturer « à reprendre » avec sa dernière
  note, puis continuer ; ou tout annuler. Attends la réponse. Jamais de clôture sans réponse. Si la
  note n'a pas de « reste à faire », demande-le avant de clôturer (le site refuserait).

## 3. Trouver la tâche

- **Rien après la commande** : `demander_choix_tache` (assistant `claude_code`, ton modèle, poste,
  dossier, `sessionRef` = la référence de cette conversation). Ouvre l'adresse avec
  `kodeban-ouvrir <adresse>` ; si ça échoue, affiche l'adresse. Puis `lire_choix_tache`, avec
  `kodeban-pause 3` entre deux questions pendant la première minute, `kodeban-pause 10` ensuite,
  jusqu'à « chosen » ou « expired ». Expirée : dis-le et arrête-toi. Choisie : la session est déjà
  ouverte par le site.
- **Un numéro** : `lire_tache`. Introuvable : dis-le, n'ouvre rien.

Le numéro d'une tâche est le « #12 » affiché sur le site : champ `number` des réponses, argument
`taskNumber` des outils. Jamais le champ `id`, qui est interne et n'a rien à voir avec le numéro.
- **Des mots** : `chercher_tache`. Une seule tâche : c'est elle. Plusieurs : propose-les (numéro,
  titre, tableau, projet) et attends le choix. Aucune : dis-le et propose `/kodeban:in` sans rien
  pour ouvrir la page de choix.

Avec un numéro ou des mots : `lire_sessions` sur la tâche. Si une session est encore ouverte
sur cette tâche ailleurs (autre poste, autre dossier, autre personne), signale-le en une ligne et
demande avant d'ouvrir. Puis `ouvrir_session` (assistant `claude_code`, ton modèle, poste, dossier,
`sessionRef` = la référence de cette conversation, `objective` = « à préciser »).

## 3 bis. Le bon dossier

La session ouverte donne le projet de la tâche (`task.projectId`). Lis ses dépôts avec
`lire_depots_projet` ; ne garde que ceux qui ne sont pas archivés et qui ont une adresse.

- Cet outil n'existe pas ou renvoie une erreur (site plus ancien), ou aucun dépôt : saute ce point
  sans rien dire. La règle du début (« Branchez KodeBan ») ne vaut pas ici : la session est ouverte.
- Sinon, lance `git remote -v` tel quel (sans `-C`, sans `cd`, sans redirection) et garde chaque
  adresse « (fetch) » : un fork a souvent son `origin` à soi et le dépôt du projet en `upstream`.
- Ramène chaque adresse, des deux côtés, à une forme simple, puis compare :
  - Azure DevOps (`dev.azure.com`, `ssh.dev.azure.com`, `<org>.visualstudio.com`,
    `vs-ssh.visualstudio.com`) → `azure/<organisation>/<projet>/<dépôt>` : l'organisation est le
    sous-domaine pour `<org>.visualstudio.com` (pas `vs-ssh`), sinon le premier mot du chemin ; on ignore
    `DefaultCollection`, `_git` et `v3`.
  - Ailleurs → serveur sans port, puis le chemin (`:` des adresses ssh lu comme `/`), sans le
    protocole, sans ce qui précède `@`, sans `.git` ni `/` final, sans `?…` ni `#…`. Une adresse
    copiée depuis le navigateur (`…/tree/main`, `…/-/tree/…`) s'arrête au nom du dépôt.
  - Sans tenir compte des majuscules. Égales, par exemple : `git@github.com:kodeban/claude-code.git`
    et `https://github.com/kodeban/claude-code/tree/main` ; `org@ssh.dev.azure.com:v3/org/proj/depot`,
    `https://org@dev.azure.com/org/proj/_git/depot` et `https://org.visualstudio.com/DefaultCollection/proj/_git/depot`.
- Une adresse du dossier correspond à un dépôt du projet : ne dis rien. Garde ce dépôt (sa branche
  principale, `mainBranch`), le nom que lui donne `git remote -v` (`origin`, `upstream`…) et la
  règle du projet (`branchPerTask`) : ils servent aux étapes 5 et 7.
- Git absent du poste (« commande introuvable », « not recognized ») : ne dis rien. Pas un dépôt Git
  (« not a git repository »), dépôt sans adresse, ou aucune correspondance : préviens et attends la
  réponse avant la suite. « Ce dossier (dépôt <adresse sans ce qui précède @> | dépôt Git sans
  adresse | pas un dépôt Git) ne fait pas partie des dépôts du projet <projet> (<noms des dépôts>).
  On continue ici ? »
- Non : propose de clôturer tout de suite la session « à reprendre », reste à faire « reprendre dans
  le dossier du bon dépôt (<noms>) » ; clôture seulement si l'utilisateur dit oui ; puis arrête-toi.

## 4. Montrer le choix

Lis la carte avec `lire_tache`, quelle que soit la façon dont la tâche a été trouvée (page de choix
comprise), et lis-la vraiment : c'est le point de départ du travail.

Une ligne : numéro, titre, tableau, projet, client (« projet interne » s'il n'y en a pas).
Puis le contenu de la carte : la description en entier (sans la résumer ni la couper), la priorité
et l'échéance s'il y en a, les commentaires (auteur, date, texte), et le nom des pièces jointes.
Ne répète pas le résumé s'il redit la description. Rien d'autre sur la carte : dis-le en une ligne.

Puis là où le travail s'était arrêté. Si les sessions de la tâche n'ont pas encore été lues (page
de choix), lis-les avec `lire_sessions`. La session que la page vient d'ouvrir est alors la plus
récente de la liste : la précédente est celle d'avant. Avec un numéro ou des mots, elles ont été
lues avant l'ouverture : la plus récente de la liste est la précédente.

- **Où on en est** (champ `handover`) : un titre « Où on en est (mis à jour le <date> par
  <auteur>) », puis le texte en entier, sans le résumer ni le couper.
- Pas de « Où on en est » (tâche ancienne), mais déjà une session : deux lignes tirées de la
  précédente (résumé, reste à faire, prochaine étape).
- Puis, de la dernière session clôturée : « Travail en cours : … » et « En attente : … », chacun
  seulement s'il est rempli.
- Aucune session avant celle-ci : rien.

Les heures du site sont en temps universel : convertis-les en heure locale, ou ne donne que la
date.

Prépare le nom de la conversation : `kodeban-nommer ${CLAUDE_SESSION_ID} "#<numéro> <titre de la
tâche>"` (titre raccourci s'il dépasse 60 caractères). La conversation le prend au message suivant
de l'utilisateur. N'en parle pas, sauf si la commande échoue : dis-le alors en une ligne.

## 5. Ce que j'ai compris

Sous la carte, un titre « Ce que j'ai compris », puis la tâche reformulée avec tes mots, en
quelques lignes : le besoin (ce qui ne va pas ou ce qui manque, et pour qui), le résultat attendu,
et où ça se passe probablement (site, paquet, appli…) si la carte le laisse deviner. Ne recopie pas
la description : reformule-la. N'invente rien : ce que la carte ne dit pas, pose-le en questions
(trois au plus, les plus utiles d'abord). Ne cherche pas dans le code à ce stade.

Les questions tiennent compte du travail en cours de la dernière session clôturée :
- il nomme une branche et le dossier n'est pas dessus (`git branch --show-current`) : demande s'il
  faut la récupérer. Sauf si c'est la branche de la tâche et que le projet veut une branche par
  tâche : l'étape 7 la prend d'office, ne demande rien. Si l'utilisateur dit oui : fais comme à
  l'étape 7 (serveur interrogé d'abord, fichiers modifiés annoncés, retard sur le serveur dit, et
  attente de la réponse avant tout travail) ;
- des fichiers modifiés ou des commits non poussés sont restés sur un autre poste : dis-le, et
  demande s'il faut les attendre ou repartir sans eux ;
- une attente est notée : demande si la réponse est arrivée.

Termine par : « Est-ce que j'ai bien compris ? Qu'est-ce qu'on fait ? »

## 6. Demande de départ

La réponse de l'utilisateur corrige ou confirme ta compréhension, puis donne la première demande de
travail : c'est la demande de départ. Range-la avec `noter_session` (`objective`), en quelques
lignes, avec la compréhension validée (corrections comprises), avant de commencer le travail. Si
l'utilisateur te corrige sans rien demander d'autre, reformule de nouveau et redemande.

## 7. La branche de la tâche

Seulement si le projet veut une branche par tâche (`branchPerTask` vrai) et que le dossier est un
des dépôts du projet (étape 3 bis). Sinon, saute ce point sans rien dire. Fais-le juste après avoir
rangé la demande de départ, avant tout travail. Ne demande pas s'il faut une branche : c'est la
règle du projet. Ne demande que dans les cas dits plus bas.

- Nom : `kodeban/<numéro>-<titre>`, le titre en minuscules, sans accents, les mots reliés par des
  tirets (ni espace, ni ponctuation, ni tiret au début ou à la fin), 40 caractères au plus, coupé
  entre deux mots. Par exemple, #87 « Export CSV des tâches » : `kodeban/87-export-csv-des-taches`.
- D'abord, toujours, interroge le serveur : `git fetch --all --prune`. Une branche du poste peut
  être en retard : un autre poste a pu la pousser depuis. Le fetch échoue (réseau, droits) : dis-le
  en une ligne et continue avec ce qui est sur le poste ; mais s'il n'y a aucune branche de la tâche
  sur le poste, demande avant d'en créer une (elle existe peut-être déjà sur le serveur) et attends.
- `git status --porcelain --branch` : la première ligne donne la branche où est le dossier, les
  suivantes les fichiers modifiés.
  - Sur aucune branche (`HEAD (no branch)`) : n'y touche pas, que `mainBranch` soit connue ou non.
  - Déjà sur une branche `kodeban/<numéro>-…` de cette tâche : ne change pas de branche ; regarde
    seulement son retard (plus bas).
  - Sur une autre branche que la branche principale du dépôt (`mainBranch`) : n'y touche pas. Le
    dépôt n'a pas de branche principale sur le site : ce cas ne joue pas.
- Sinon, cherche la branche de la tâche. Son titre a pu changer depuis : tout nom
  `kodeban/<numéro>-…` compte.
  - Sur le poste : `git branch -vv --list "kodeban/<numéro>-*"` (montre aussi l'état de sa copie sur
    le serveur). Une seule : `git switch <nom>`. Plusieurs : prends celle dont la copie sur le
    serveur existe et n'est pas en retard (ni `behind`, ni `gone`) ; s'il n'y en a pas exactement
    une, montre-les et demande.
  - Seulement sur le serveur : `git branch -r --list "*/kodeban/<numéro>-*"`, puis
    `git switch --track <serveur>/<nom>`, avec le serveur qui porte vraiment la branche dans cette
    liste (dans un fork, c'est souvent `origin` : `/kodeban:out` y pousse). Plusieurs serveurs la
    portent : prends celui du dépôt du projet retenu à l'étape 3 bis.
  - Nulle part : `git switch -c <nom>`, depuis la branche où est le dossier (la branche principale ;
    sans `mainBranch`, la branche courante). Si la première ligne de `git status` la montrait en
    retard (`behind <n>`) : dis-le en une ligne et demande avant de créer la branche.
- Des fichiers modifiés non commités suivent le changement de branche : nomme-les dans l'annonce.
  Git refuse de changer de branche à cause d'eux : dis-le en une ligne (les fichiers en cause) et
  attends la réponse avant tout travail. Jamais de commit ni de remisage pour passer outre.
- Une fois sur la branche de la tâche, relance `git status --porcelain --branch` : `behind <n>` sur
  la première ligne veut dire que le serveur a <n> commits de plus. Dis-le dans l'annonce et attends
  la réponse avant tout travail. Oui : `git merge --ff-only <serveur>/<nom>` (le serveur de sa
  branche suivie). La branche a aussi des commits à elle (`ahead <m>, behind <n>`) : elle a divergé ;
  dis-le et demande quoi faire. Jamais de `git pull` ni de `rebase` sans le dire.

Une ligne pour annoncer la branche : « Branche de la tâche : `<nom>` (créée | récupérée du serveur |
déjà là) » ; s'il y en a, « ; les fichiers modifiés <fichiers> suivent » ; s'il y a du retard,
« ; elle a <n> commits de retard sur le serveur : je la mets à jour avant de commencer ? » (ou, si
elle a divergé : « ; elle a <m> commits à elle et <n> de retard sur le serveur : que fait-on ? »). Si tu
n'y as pas touché : « Le dossier est sur la branche `<nom>` : je la garde. », ou « Le dossier n'est
sur aucune branche : je n'y touche pas. »

## Règles jusqu'à la clôture de la session

Elles valent pour toute la suite de cette conversation, jusqu'à `/kodeban:out`.

- Tiens la note de la session à jour (`noter_session` : ce qui a été fait, les décisions avec leur
  raison, ce qu'il reste à faire, la prochaine étape) après chaque étape importante, et au moins
  après chaque commit. Après un commit, ajoute aussi le commit et la branche à la session
  (`ajouter_reference_session`).
- Une question sans modification de code : réponds, ne note rien.
- Une demande de code hors du sujet de la tâche : arrête-toi et demande « Je la mets de côté, ou on
  la traite ici ? ».
  - « De côté » : crée une tâche dans la première colonne du tableau de la session (`creer_tache`),
    dis-le en une ligne, puis reprends le travail.
  - « Ici » : crée la tâche, fais le travail, commente cette tâche (ce qui a été fait, le commit),
    et ajoute une ligne dans la note de la session.
- Demande confirmation avant toute suppression dans KodeBan.
- Jamais de mot de passe, de clé ni de donnée d'un client dans une note ou un commentaire.
- Ne devine jamais le projet ni la tâche : demande.
- Ne clôture jamais la session de toi-même : c'est `/kodeban:out`.
