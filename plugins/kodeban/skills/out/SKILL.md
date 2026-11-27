---
description: Clôturer la session KodeBan de cette conversation (terminée ou à reprendre).
argument-hint: "[pause | !]"
disable-model-invocation: true
allowed-tools: Bash(kodeban-poste) PowerShell(kodeban-poste) Bash(kodeban-poste *) PowerShell(kodeban-poste *) Bash(kodeban-ouvrir *) PowerShell(kodeban-ouvrir *) Bash(kodeban-pause *) PowerShell(kodeban-pause *) Bash(git log *) PowerShell(git log *) Bash(git branch *) PowerShell(git branch *) Bash(git rev-parse *) PowerShell(git rev-parse *) Bash(git status *) PowerShell(git status *) Bash(git rev-list *) PowerShell(git rev-list *) Bash(git push *) PowerShell(git push *) mcp__plugin_kodeban_kodeban__mes_sessions_ouvertes mcp__plugin_kodeban_kodeban__lire_sessions mcp__plugin_kodeban_kodeban__noter_session mcp__plugin_kodeban_kodeban__ajouter_reference_session mcp__plugin_kodeban_kodeban__cloturer_session mcp__plugin_kodeban_kodeban__lire_tache mcp__plugin_kodeban_kodeban__demander_cloture_session mcp__plugin_kodeban_kodeban__lire_cloture_session
---

# /kodeban:out — j'ai fini, ou je m'arrête

Ce que l'utilisateur a écrit après la commande : « $ARGUMENTS »
Référence de cette conversation : `${CLAUDE_SESSION_ID}`

Parle en français courant, en phrases courtes, sans jargon et sans nom d'outil.

Si un outil KodeBan ne répond pas (pas encore autorisé, accès révoqué, site éteint) : dis-le en une
phrase, ajoute « Branchez KodeBan : tapez `/mcp`, choisissez kodeban, puis authentifiez-vous. », et
arrête-toi.

## 1. La session de cette conversation

Appelle `mes_sessions_ouvertes` **sans aucun argument** (ni poste ni dossier : le dossier courant
a pu changer depuis l'ouverture). La session de cette conversation est celle dont la référence
(`sessionRef`) est celle de cette conversation, ou celle que cette conversation a ouverte, quels
que soient son poste et son dossier. S'il y en a une : passe à l'étape 2.

Si la liste est vide (aucune session ouverte nulle part) : « Aucune session ouverte. » et rien
d'autre.

Si la liste n'est pas vide mais qu'aucune n'est celle de cette conversation, fais choisir à l'utilisateur la session à clôturer, puis arrête-toi (pas
d'étape 2 : la note de la session est gardée telle quelle, le site la complète si elle est vide) :
- dis-le en une ligne : « Aucune session ouverte dans cette conversation : choisissez celle à
  clôturer dans la page qui s'ouvre. » ;
- `kodeban-poste`, puis `demander_cloture_session` (assistant `claude_code`, poste, dossier) ;
- ouvre l'adresse avec `kodeban-ouvrir <adresse>` ; si ça échoue, affiche l'adresse ;
- puis `lire_cloture_session`, avec `kodeban-pause 3` entre deux questions pendant la première
  minute, `kodeban-pause 10` ensuite, jusqu'à « chosen » ou « expired » ;
- « chosen » : « Session de la tâche #<numéro> « <titre> » clôturée, à reprendre. » (numéro =
  champ `number` de la tâche) ; sans session dans la réponse (tâche supprimée depuis) : « Session
  clôturée, à reprendre. » ; « expired » : « L'adresse a expiré : rien n'a été clôturé. ».

## 2. La note finale

Rédige et montre la note finale, courte :
- ce qui a été fait ;
- les décisions, chacune avec sa raison ;
- les commits ;
- ce qu'il reste à faire ;
- la prochaine étape.

Sans mot de passe, sans clé, sans donnée d'un client. Jamais la conversation recopiée.

Lis la tâche (`lire_tache`). Si elle a une adresse de suivi (`followEmail`), ajoute à la note un
**message au demandeur** : la personne qui suit la tâche, souvent hors de l'équipe. Trois à cinq
phrases, en mots simples : ce qui a changé pour elle, ce qu'elle peut faire maintenant. Ni jargon, ni
nom de fichier, ni commit, ni nom d'outil. Montre-le sous la note, avec l'adresse. Il ne part jamais
d'ici : quand l'utilisateur passera la tâche dans une colonne de fin, le site lui proposera ce message
à relire avant de l'envoyer.

### Travail en cours

Rempli d'office, sans rien demander. Lance, tels quels : `git status --porcelain`,
`git branch --show-current`, puis `git rev-list --count '@{u}..HEAD'` (commits pas encore poussés ;
garde les guillemets). Si ce dernier dit qu'il n'y a pas de branche suivie : la branche n'a jamais
été poussée, tous ses commits sont à pousser (leur nombre : `git rev-list --count HEAD --not --remotes`).

En une ou deux lignes : la branche, poussée ou non (combien de commits à pousser), les fichiers
modifiés laissés sur place, et le poste (donné par `kodeban-poste`). Tout est poussé et rien n'est
modifié : « rien en cours », exactement ces mots (le site les lit comme « rien en attente » et
n'affiche alors aucun avertissement sur la session). Pas un dépôt Git (« not a git repository ») ou Git absent : n'écris
rien.

**En attente de** : seulement si une question à une personne bloque la suite (qui, et quoi).
Sinon, rien.

### Où on en est

C'est le texte de la tâche, pas de la session : celui qu'on lit en premier pour la reprendre. Lis
le texte actuel avec `lire_sessions` (champ `handover` : son texte et sa `version` ; `null` s'il n'y
en a pas encore). Réécris-le en entier avec ce que cette session a appris : jamais de notes
empilées les unes sur les autres. Écris-le pour un développeur qui reprend sans rien connaître et
n'a pas suivi la conversation. Court, en Markdown simple, avec ces rubriques, dans cet ordre :
- **But** ;
- **Ce qui marche** ;
- **Décisions qui tiennent**, chacune avec sa raison ;
- **Pistes abandonnées**, chacune avec sa raison ;
- **Reste à faire** ;
- **Où est le code** ;
- **Comment vérifier**.

Écris-le aussi bien pour une clôture « terminée » que « à reprendre ». Même règle que la note : ni
mot de passe, ni clé, ni donnée d'un client.

Montre, ensemble : la note, le travail en cours, l'attente s'il y en a une, puis « Où on en est ».
Le même accord vaut pour tout.

## 3. L'accord

- Avec `!` (seul, ou avec des mots) : ne pose aucune question et n'attends rien. Ce qui est montré
  vaut accord : passe aussitôt à la clôture, « terminée ».
- Avec `pause` : demande seulement l'accord sur ce qui est montré ; la session sera clôturée « à
  reprendre ». Attends la réponse.
- Sinon : demande « Terminée, ou à reprendre ? » et l'accord sur ce qui est montré. Attends la
  réponse.

Corrige la note ou « Où on en est » si l'utilisateur le demande, puis redemande l'accord (sauf avec
`!`).

### Commits pas encore poussés

Seulement pour une clôture « à reprendre » avec des commits pas encore poussés. Demande : « Je
pousse les <n> commits de la branche <branche> ? » — avec `pause`, dans la même question que
l'accord ; sinon, dès que l'utilisateur a répondu « à reprendre ». Attends la réponse.
- Oui : `git push` (la branche courante, telle quelle) ; sans branche suivie :
  `git push -u origin <branche>`. Puis mets à jour le travail en cours (« poussée »).
- Non : ne pousse pas ; on clôture quand même.
- Le push est refusé (droits, conflit) : dis-le en une ligne, écris-le dans le travail en cours
  (« push refusé : <raison courte> »), et clôture quand même.

Jamais de commit, de nouvelle branche ni de remisage faits par cette commande : les fichiers non
commités sont seulement signalés dans le travail en cours.

## 4. Clôturer

Ajoute à la session les commits de cette conversation qui n'y sont pas encore
(`ajouter_reference_session`, avec la branche), puis `cloturer_session` (`done` = terminée,
`resume` = à reprendre) avec la note, `workInProgress` (le travail en cours, s'il y en a un),
`waitingFor` (l'attente, s'il y en a une), `handover` (« Où on en est ») et `handoverVersion` (la
version lue avec `lire_sessions` ; ne l'envoie pas si `handover` était `null`). Le message au
demandeur va dans `followUpMessage`, seulement pour une clôture « terminée » : à reprendre, le
travail n'est pas fini, ne l'envoie pas.

Si le site refuse :
- « Où on en est » a changé entre-temps (`handover_conflict`) : rien n'est clôturé, la session reste
  ouverte. Relis-le avec `lire_sessions`, refonds le nouveau texte avec ce que cette session a
  appris, remontre-le et redemande l'accord. Même avec `!` : dis-le en une ligne (« Quelqu'un a
  modifié « Où on en est » entre-temps : voici le texte refondu. ») et redemande.
- Un texte trop long : raccourcis-le (jamais coupé au milieu) et recommence.
- Note incomplète : complète ce qui manque — demande à l'utilisateur si tu ne le sais pas — et
  recommence.

Jamais de contournement.

## 5. Une ligne pour finir

Où se trouve maintenant la tâche (colonne, tableau), ou « À reprendre : la tâche n'a pas bougé. »
Avec un message au demandeur : ajoute « Le message pour <adresse> vous sera proposé quand la tâche
passera dans une colonne de fin. »
