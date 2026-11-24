---
description: Clôturer la session KodeBan de cette conversation (terminée ou à reprendre).
argument-hint: "[pause | !]"
disable-model-invocation: true
allowed-tools: Bash(kodeban-poste) PowerShell(kodeban-poste) Bash(kodeban-poste *) PowerShell(kodeban-poste *) Bash(kodeban-ouvrir *) PowerShell(kodeban-ouvrir *) Bash(kodeban-pause *) PowerShell(kodeban-pause *) Bash(git log *) Bash(git branch *) Bash(git rev-parse *) mcp__plugin_kodeban_kodeban__mes_sessions_ouvertes mcp__plugin_kodeban_kodeban__lire_sessions mcp__plugin_kodeban_kodeban__noter_session mcp__plugin_kodeban_kodeban__ajouter_reference_session mcp__plugin_kodeban_kodeban__cloturer_session mcp__plugin_kodeban_kodeban__lire_tache mcp__plugin_kodeban_kodeban__demander_cloture_session mcp__plugin_kodeban_kodeban__lire_cloture_session
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

S'il n'y en a pas, fais choisir à l'utilisateur la session à clôturer, puis arrête-toi (pas
d'étape 2 : la note de la session est gardée telle quelle, le site la complète si elle est vide) :
- dis-le en une ligne : « Aucune session ouverte dans cette conversation : choisissez celle à
  clôturer dans la page qui s'ouvre. » ;
- `kodeban-poste`, puis `demander_cloture_session` (assistant `claude_code`, poste, dossier) ;
- ouvre l'adresse avec `kodeban-ouvrir <adresse>` ; si ça échoue, affiche l'adresse ;
- puis `lire_cloture_session`, avec `kodeban-pause 3` entre deux questions pendant la première
  minute, `kodeban-pause 10` ensuite, jusqu'à « chosen » ou « expired » ;
- « chosen » : « Session de la tâche #<numéro> « <titre> » clôturée, à reprendre. » (numéro =
  champ `number` de la tâche) ; « expired » : « L'adresse a expiré : rien n'a été clôturé. ».

## 2. La note finale

Rédige et montre la note finale, courte :
- ce qui a été fait ;
- les décisions, chacune avec sa raison ;
- les commits ;
- ce qu'il reste à faire ;
- la prochaine étape.

Sans mot de passe, sans clé, sans donnée d'un client. Jamais la conversation recopiée.

## 3. L'accord

- Avec `!` (seul, ou avec des mots) : ne pose aucune question et n'attends rien. La note montrée
  vaut accord : passe aussitôt à la clôture, « terminée ».
- Avec `pause` : demande seulement l'accord sur la note ; la session sera clôturée « à reprendre ».
  Attends la réponse.
- Sinon : demande « Terminée, ou à reprendre ? » et l'accord sur la note. Attends la réponse.

Corrige la note si l'utilisateur le demande, puis redemande l'accord (sauf avec `!`).

## 4. Clôturer

Ajoute à la session les commits de cette conversation qui n'y sont pas encore
(`ajouter_reference_session`, avec la branche), puis `cloturer_session` (`done` = terminée,
`resume` = à reprendre) avec la note.

Si le site refuse (note incomplète) : complète ce qui manque — demande à l'utilisateur si tu ne le
sais pas — et recommence. Jamais de contournement.

## 5. Une ligne pour finir

Où se trouve maintenant la tâche (colonne, tableau), ou « À reprendre : la tâche n'a pas bougé. »
