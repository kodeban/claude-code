# Le paquet KodeBan pour Claude Code

Quatre commandes pour piloter votre travail dans [KodeBan](https://kodeban.fr) depuis Claude Code :

| Commande | Ce qu'elle fait |
|---|---|
| `/kodeban:in` | je commence : choisir la tâche et ouvrir une session de travail |
| `/kodeban:out` | j'ai fini, ou je m'arrête : clôturer la session avec sa note |
| `/kodeban:todo` | je note une chose pour plus tard, sans quitter ce que je fais |
| `/kodeban:prompt` | j'utilise un prompt de la banque de prompts |

Rien ne se lance tout seul : seules ces quatre commandes agissent, et seulement quand vous les tapez.

Présentation illustrée : https://kodeban.fr/mcp

## Installer

Il vous faut un compte sur https://kodeban.fr. Aucune clé à créer ni à copier.

**1. Ajouter le paquet**

- Dans le panneau Claude Code de VS Code : tapez `/plugins`, onglet **Marketplaces**, ajoutez
  `kodeban/claude-code`, puis installez **kodeban** depuis l'onglet **Plugins**.
- Ou dans Claude Code en terminal :
  ```
  /plugin marketplace add kodeban/claude-code
  /plugin install kodeban@kodeban
  ```

Ouvrez ensuite une nouvelle conversation.

**2. Se connecter à KodeBan**

Tapez `/mcp`, choisissez **kodeban**, puis **s'authentifier**. Votre navigateur ouvre kodeban.fr
(connectez-vous si besoin) : cliquez sur **Autoriser**. C'est tout : revenez dans Claude Code.

L'accès ainsi donné apparaît dans KodeBan, **Mon profil › Clés d'accès** (« Claude Code (autorisé le
…) »). Il permet de lire et de modifier vos projets et tâches avec vos droits, jamais de supprimer.

Si vous aviez branché KodeBan à la main (`claude mcp add … kodeban2 …`), retirez ce branchement :
`claude mcp remove kodeban2`. Sinon, les outils seraient en double.

## Les commandes

**`/kodeban:in`** — trois façons :
- `/kodeban:in` : la page de choix s'ouvre, vous cliquez sur la tâche (avec Chrome ou Edge comme
  navigateur par défaut : dans une fenêtre à part, sans onglets ; sinon dans un onglet) ;
- `/kodeban:in 57` : la tâche n° 57 ;
- `/kodeban:in charte graphique` : la tâche qui correspond à ces mots (s'il y en a plusieurs, elles vous sont proposées).

Si le projet de la tâche a des dépôts Git notés sur le site (fiche du projet › « Environnement
technique »), Claude vérifie que le dossier de la conversation est l'un d'eux. Sinon, il vous le
dit et vous demande s'il continue ici.

Une conversation = une session. Votre première demande après `/kodeban:in` devient la demande de
départ de la session. Ensuite, Claude tient la note de la session à jour au fil du travail.

**`/kodeban:out`** : Claude montre la note finale (ce qui a été fait, décisions, commits, reste à
faire, prochaine étape) et demande « terminée ou à reprendre ? ». `/kodeban:out pause` clôture
« à reprendre » sans poser la question. `/kodeban:out !` montre la note et clôture aussitôt
« terminée », sans rien demander.

**`/kodeban:todo vérifier les marges à l'impression`** : crée une tâche dans la première colonne du
tableau de la session, sans toucher à la session.

**`/kodeban:prompt revue de code`** : cherche un prompt dans votre banque de prompts (les vôtres et
ceux partagés par votre organisation) et l'applique à la conversation ; sans mots, vos favoris. Le
prompt d'un collègue vous est d'abord montré en entier : rien n'est appliqué sans votre accord.

## Autorisations

Pendant le travail (par exemple après un commit), Claude Code demande une fois l'accord pour chaque
outil KodeBan. Répondez « toujours autoriser », ou ajoutez dans vos réglages
(`~/.claude/settings.json`) :

```json
{ "permissions": { "allow": ["mcp__plugin_kodeban_kodeban"] } }
```

## Mettre à jour

Dans le panneau de VS Code : `/plugins` › **kodeban** › mettre à jour. En terminal :

```
/plugin marketplace update kodeban
/plugin update kodeban@kodeban
```

puis ouvrez une nouvelle conversation.

## Désinstaller

`/plugins` (ou `/plugin uninstall kodeban@kodeban` en terminal). Puis, dans KodeBan, **Mon profil ›
Clés d'accès** : révoquez l'accès « Claude Code (autorisé le …) ».

## En cas de souci

- **« Branchez KodeBan »** : `/mcp` › kodeban › s'authentifier, puis **Autoriser** dans le navigateur.
- **Accès révoqué dans le profil** : même geste, une nouvelle autorisation est créée.
- **La page de choix ne s'ouvre pas** : l'adresse est affichée, ouvrez-la vous-même.
- **Accès de votre organisation expiré** : voyez avec l'administrateur de votre organisation.
- **Essai sur un site KodeBan local** : le paquet vise kodeban.fr ; branchez le site local à la main
  (`claude mcp add --transport http kodeban-local http://127.0.0.1:5180/api/mcp`), puis `/mcp`.
