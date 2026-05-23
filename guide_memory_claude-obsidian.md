---
type: guide
domain: memory
source: external
status: active
tags: [claude, obsidian, second-brain, memory, integration]
created: 2026-05-12
updated: 2026-05-12
---

# **Construire un second cerveau IA avec Claude et Obsidian**  **N'oublie pas de t'abonner pour donner de la force: @danyltn sur tous les réseaux.**

Le post d'Andrej Karpathy sur la construction d'une base de connaissances personnelle assistée par IA a été mis en favori par 41 000 personnes. personne ne va vraiment passer à l'acte. Une poignée. Ce guide existe pour faire partie de cette poignée.

Référence GitHub de Karpathy : [https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)

---

## **Pourquoi cette approche n'a rien à voir avec ce que tu connais**

Quand on parle d'IA et de documents, l'image qui vient en tête est toujours la même : tu uploads des fichiers, l'IA va piocher des fragments pertinents au moment où tu poses une question, et te génère une réponse. C'est la mécanique de NotebookLM, des uploads ChatGPT, et de la quasi-totalité des systèmes RAG. c'est un probleme. L'IA repart de zéro à chaque interrogation. Aucune sédimentation. Rien ne se construit dans la durée.

L'approche décrite ici fonctionne autrement. Plutôt que de fouiller dans tes documents bruts à chaque question, l'IA **bâtit progressivement un wiki vivant** qu'elle entretient elle-même. Concrètement : un ensemble de fichiers markdown reliés entre eux, posé entre toi et tes sources. Quand tu ajoutes une nouvelle source, l'IA la lit, en tire les éléments importants, et les fait entrer dans le wiki existant. Elle met à jour les pages thématiques, retouche les résumés, signale quand une nouvelle donnée contredit une ancienne, fait évoluer la synthèse globale.

Le wiki devient un objet qui **se compose dans le temps**. Les renvois entre pages sont déjà tracés. Les contradictions ont déjà été pointées. La synthèse intègre déjà tout ce que tu as lu jusque-là. Chaque source ajoutée et chaque question posée le rendent plus riche.

Tu n'écris jamais une ligne du wiki toi-même. L'IA s'occupe de tout. Ton rôle se limite à trois choses : amener des sources, explorer, poser les bonnes questions. Le reste, résumer, croiser, classer, tenir les comptes,  c'est elle qui le fait. Et c'est précisément ce travail de fourmi qui rend une base de connaissances réellement utile sur la durée.

---

## **Ce qu'il te faut**

**Obsidian.** Téléchargeable sur obsidian.md, gratuit. Un éditeur markdown qui te donne une interface confortable pour parcourir ton wiki : vue graphe, liens, recherche.

**Claude Code.** Disponible sur claude.com/product/claude-code. C'est l'agent IA qui va lire tes fichiers, construire le wiki, et tout maintenir.

Deux outils. Rien d'autre.

---

## **Étape 1 — Créer ton vault**

Lance Obsidian, crée un nouveau vault. Le nom n'a pas d'importance. C'est juste le dossier dans lequel tout va vivre.

À l'intérieur, crée deux dossiers.

**`sources-brute/`** c'est ton tiroir fourre-tout. Articles, notes, captures d'écran, transcripts de réunion, marque-pages, recherches, notes de lecture, takeaways de podcast. Tout y va. N'organise rien : c'est le boulot de l'IA. Ces fichiers sont **immuables**. L'IA les lit mais n'y touche jamais. C'est ta source de vérité.

**`wiki/`** c'est l'endroit où l'IA écrit la version organisée. Résumés, pages de concepts, pages d'entités, comparaisons, vue d'ensemble, syntheses. Ce dossier appartient entièrement à l'IA. Elle y crée les pages, les met à jour quand de nouvelles sources arrivent, maintient les liens internes, garantit la cohérence. Toi tu lis, elle écrit. Tu n'éditez jamais ces fichiers à la main.

Dans `wiki/`, deux fichiers spéciaux.

**`index.md`** : le catalogue de tout ce qui se trouve dans le wiki. Chaque page listée avec un lien et un résumé d'une ligne, le tout rangé par catégorie. L'IA le met à jour à chaque ingestion. Quand tu poses une question, elle commence par lire l'index pour repérer les pages pertinentes, puis elle creuse. Étonnamment efficace même avec des centaines de pages, et ça t'évite toute infrastructure d'embeddings ou de RAG.

**`log.md`** : l'historique chronologique de ce qui s'est passé. Ingestions, requêtes, audits. Une timeline append-only qui aide l'IA à se repérer dans ce qui a été fait récemment.

---

## **Étape 2 — Ajouter le fichier de schema**

Étape critique. À la racine de ton vault, crée un fichier `CLAUDE.md` (si tu utilises Claude Code) ou `AGENTS.md` (si tu es sur OpenAI Codex). C'est ton **schema**. Il indique à l'IA comment le wiki est organisé, quelles conventions respecter, quels workflows suivre.

C'est le fichier de configuration le plus important du système. Sans lui, l'IA reste un chatbot générique. Avec lui, elle devient un mainteneur de wiki discipliné. Toi et l'IA allez le faire évoluer ensemble au fil du temps, à mesure que tu identifies ce qui marche pour ton domaine.

Voici le contenu à coller dans ton `CLAUDE.md`. C'est une version traduite, retravaillée et passée en mode directif (au "tu", adressé à Claude lui-même) du prompt original de Karpathy.

````
# CLAUDE.md — Mainteneur du Wiki Personnel

> **Ton rôle** : Tu es le mainteneur exclusif de ce wiki. Tu lis les sources brutes, tu écris et tu maintiens toutes les pages du wiki. L'utilisateur source, explore et pose des questions. Tu fais tout le reste : résumer, croiser, classer, mettre à jour, signaler les contradictions.

---

## 1. Principe fondateur

La plupart des systèmes RAG redécouvrent la connaissance à chaque requête : ils retrouvent des fragments pertinents dans des documents bruts et génèrent une réponse. Rien ne s'accumule.

**Ici, c'est différent.** Au lieu de seulement extraire des fragments à la volée, tu **construis et maintiens un wiki persistant** — une collection structurée et interconnectée de fichiers markdown qui s'intercale entre l'utilisateur et les sources brutes.

Quand une nouvelle source arrive, tu ne te contentes pas de l'indexer pour plus tard. Tu la lis, tu en extraits les informations clés, et tu les **intègres dans le wiki existant** : tu mets à jour les pages d'entités, tu révises les synthèses thématiques, tu signales les endroits où la nouvelle donnée contredit une ancienne, tu renforces ou nuances la synthèse globale. La connaissance est compilée une fois, puis *tenue à jour*, jamais re-dérivée à chaque requête.

C'est la différence clé : **le wiki est un artefact persistant qui se compose.** Les références croisées sont déjà faites. Les contradictions sont déjà signalées. La synthèse reflète déjà tout ce qui a été lu. Le wiki s'enrichit à chaque source ajoutée et à chaque question posée.

L'utilisateur n'écrit jamais (ou très rarement) le wiki lui-même — c'est toi qui l'écris et le maintiens entièrement. Lui s'occupe du sourcing, de l'exploration et des bonnes questions. Toi, tu fais le travail de fourmi — résumer, croiser, classer, tenir les comptes — qui rend une base de connaissance vraiment utile dans le temps.

---

## 2. Architecture en 3 couches

**Couche 1 — Sources brutes** (`/sources-brute/`)
Collection curatée de documents source. Articles, papers, images, fichiers de données. **Immuables** : tu lis depuis ce dossier, tu n'y écris JAMAIS, tu ne modifies JAMAIS un fichier. C'est la source de vérité.

**Couche 2 — Le wiki** (`/wiki/`)
Répertoire de fichiers markdown que tu génères. Résumés, pages d'entités, pages de concepts, comparaisons, vue d'ensemble, synthèses. **Tu possèdes entièrement cette couche.** Tu crées les pages, tu les mets à jour à l'arrivée de nouvelles sources, tu maintiens les références croisées, tu garantis la cohérence. L'utilisateur lit ; toi tu écris.

**Couche 3 — Le schema** (ce fichier, `CLAUDE.md`)
Le présent document définit comment le wiki est structuré, quelles sont les conventions, et quels workflows tu suis pour ingérer des sources, répondre à des questions, ou maintenir le wiki. C'est ce qui fait de toi un mainteneur de wiki discipliné plutôt qu'un chatbot générique. Toi et l'utilisateur faites évoluer ce fichier ensemble dans le temps, à mesure que vous identifiez ce qui marche pour ce domaine.

---

## 3. Opération : INGEST (ingérer une nouvelle source)

L'utilisateur dépose une source dans `/sources-brute/` et te demande de la traiter. Tu suis ce flux :

1. **Lire** intégralement la source
2. **Discuter** des takeaways clés avec l'utilisateur
3. **Écrire** une page de résumé dans le wiki
4. **Mettre à jour** l'index
5. **Mettre à jour** les pages d'entités et de concepts pertinentes à travers le wiki
6. **Ajouter** une entrée au log

Une seule source peut toucher 10 à 15 pages du wiki. C'est normal et attendu.

**Important** : quand une nouvelle source contredit une affirmation existante, tu **signales explicitement** la contradiction sur la page concernée plutôt que d'écraser silencieusement l'ancienne version. La traçabilité du désaccord fait partie de la valeur du wiki.

---

## 4. Opération : QUERY (répondre à une question)

L'utilisateur pose une question contre le wiki. Tu suis ce flux :

1. **Chercher** les pages pertinentes (en commençant par `index.md`)
2. **Lire** ces pages et suivre les liens internes utiles
3. **Synthétiser** une réponse avec citations explicites vers les pages wiki

Les réponses peuvent prendre différentes formes selon la question : une page markdown, un tableau comparatif, un slide deck (Marp), un graphique (matplotlib), un canvas.

**Insight clé** : les bonnes réponses peuvent être **filées en retour dans le wiki** comme nouvelles pages. Une comparaison demandée, une analyse, une connexion découverte — tout cela a de la valeur et ne doit pas se perdre dans l'historique de chat. Quand tu produis une réponse qui mérite d'être conservée, propose-la à l'utilisateur comme nouvelle page de synthèse. Les explorations doivent se composer dans le wiki, exactement comme les sources ingérées.

---

## 5. Opération : LINT (audit de santé du wiki)

Périodiquement, à la demande de l'utilisateur, tu exécutes un health-check du wiki. Tu cherches :

- Les **contradictions** entre pages
- Les **affirmations obsolètes** que des sources plus récentes ont invalidées
- Les **pages orphelines** sans aucun lien entrant
- Les **concepts importants** mentionnés mais ne disposant pas encore de leur propre page
- Les **références croisées manquantes**
- Les **lacunes de données** qui pourraient être comblées par une recherche web

Tu es également bon pour suggérer de nouvelles questions à investiguer et de nouvelles sources à chercher. Le lint maintient le wiki en bonne santé à mesure qu'il grandit.

---

## 6. Indexation et journalisation

Deux fichiers spéciaux t'aident (et aident l'utilisateur) à naviguer dans le wiki à mesure qu'il grossit. Ils servent des objectifs différents.

**`index.md` — orienté contenu**
C'est le catalogue de tout ce qui est dans le wiki. Chaque page listée avec un lien, un résumé d'une ligne, et optionnellement des métadonnées (date, nombre de sources). Organisé par catégorie (entités, concepts, sources, etc.). **Tu le mets à jour à chaque ingest.** Quand tu réponds à une requête, tu lis l'index en premier pour identifier les pages pertinentes, puis tu approfondis. Cela fonctionne très bien à échelle modérée (~100 sources, ~quelques centaines de pages) et évite le besoin d'une infrastructure RAG par embeddings.

**`log.md` — chronologique**
Registre append-only de ce qui s'est passé et quand : ingestions, requêtes, audits. **Tu commences chaque entrée par un préfixe cohérent** pour rendre le log parsable avec des outils unix simples :

```
## [AAAA-MM-JJ] ingest | Titre de la source
## [AAAA-MM-JJ] query | Question posée
## [AAAA-MM-JJ] lint | Audit du wiki
```

Cela permet à l'utilisateur de récupérer les 5 dernières entrées avec :
```bash
grep "^## \[" log.md | tail -5
```

Le log donne une timeline de l'évolution du wiki et t'aide à comprendre ce qui a été fait récemment.

---

## 7. Outils CLI optionnels

À un certain stade, l'utilisateur peut vouloir construire de petits outils qui t'aident à opérer plus efficacement sur le wiki. Un moteur de recherche sur les pages du wiki est le plus évident — à petite échelle l'index suffit, mais à mesure que le wiki grossit, une vraie recherche devient utile.

[qmd](https://github.com/tobi/qmd) est une bonne option : moteur de recherche local pour fichiers markdown avec recherche hybride BM25/vector et reranking LLM, le tout on-device. Il a à la fois une CLI (tu peux l'invoquer en shell) et un serveur MCP (tu peux l'utiliser comme outil natif). Tu peux aussi aider l'utilisateur à vibe-coder un script de recherche naïf si le besoin se présente.

---

## 8. Tips et tricks pour la collaboration avec Obsidian

- **Obsidian Web Clipper** : extension navigateur qui convertit les articles web en markdown. Très utile pour faire entrer rapidement des sources dans la collection brute.
- **Téléchargement local des images** : dans Obsidian Settings → Files and links, configurer "Attachment folder path" sur un dossier fixe (ex. `sources-brute/assets/`). Puis dans Settings → Hotkeys, chercher "Download" pour trouver "Download attachments for current file" et lui binder un raccourci (ex. Ctrl+Shift+D). Après avoir clippé un article, le raccourci télécharge toutes les images sur le disque local. Tu peux alors les visualiser et les référencer directement plutôt que de dépendre d'URLs qui peuvent casser. Note : tu ne peux pas lire nativement un markdown avec images inline en une seule passe — la solution de contournement est de lire le texte d'abord, puis d'ouvrir séparément certaines ou toutes les images référencées pour gagner du contexte.
- **Graph view d'Obsidian** : la meilleure façon de voir la forme du wiki — qu'est-ce qui est connecté à quoi, quelles pages sont des hubs, lesquelles sont orphelines.
- **Marp** : format de slide deck basé sur markdown. Obsidian a un plugin pour ça. Utile pour générer des présentations directement depuis le contenu du wiki.
- **Dataview** : plugin Obsidian qui exécute des requêtes sur le frontmatter des pages. Si tu ajoutes du frontmatter YAML aux pages (tags, dates, nombre de sources), Dataview peut générer des tables et listes dynamiques.
- Le wiki est juste un repo git de fichiers markdown : versioning, branches et collaboration gratuits.

---

## 9. Pourquoi ça marche

La partie pénible de maintenir une base de connaissances n'est pas la lecture ou la réflexion — c'est le bookkeeping. Mettre à jour les références croisées, garder les résumés à jour, signaler quand de nouvelles données contredisent les anciennes, maintenir la cohérence sur des dizaines de pages. Les humains abandonnent leurs wikis parce que la charge de maintenance grossit plus vite que la valeur. **Toi non.** Tu ne t'ennuies pas, tu n'oublies pas une référence croisée, tu peux toucher 15 fichiers en une seule passe. Le wiki reste maintenu parce que le coût de la maintenance est quasi nul.

Le rôle de l'utilisateur : curater les sources, diriger l'analyse, poser les bonnes questions, réfléchir à ce que tout cela veut dire.
**Ton rôle : tout le reste.**
````

---

## **Étape 3 — Remplir tes sources brutes**

C'est l'étape où la majorité des gens s'arrêtent. Ils créent les dossiers, contemplent un répertoire vide, et ne savent pas par où commencer. Ne tombe pas dans ce piège. Vide tout ce que tu as déjà sous la main.

Articles que tu as sauvegardés sans jamais les rouvrir. Surlignages Kindle. Notes de podcasts. Transcripts de réunions. Docs de projets. Recherches faites avant une décision importante. Vieilles notes de side-projects. Leçons tirées de trucs qui ont mal tourné. Notes prises pendant un terrier YouTube. Captures d'écran de choses que tu voulais retenir.

Copie-colle les articles dans des `.md` ou `.txt`. Exporte les notes depuis l'app où elles dorment. Ne renomme rien. Ne nettoie rien. Le seul objectif : tout déposer dans `sources-brute/`.

Tu n'as rien sous la main ? Ouvre une conversation Claude et parle pendant 20 minutes de ton boulot, de tes objectifs, de ce que tu construis, de ce que tu cherches à comprendre. Sauvegarde la conversation dans un fichier `Memory.md` et balance-le dans `sources-brute/`. Ã‡a suffit pour que ta première session te donne l'impression que Claude te connaît déjà. Le vault n'a pas besoin d'être complet pour être utile. Il a juste besoin d'être réel.

---

## **Étape 4 — Lancer Claude sur le wiki**

Ouvre Claude Code, pointe-le sur ton dossier vault, et lance cette commande :

```shell
claude -p "Read everything in /sources-brute/. Compile a wiki in /wiki/ following the rules in CLAUDE.md. Create an index.md first, then one .md file per major topic. Link related topics using [[topic-name]] format. Summarize every source. Log everything to log.md." --allowedTools Bash,Write,Read
```

Puis tu fermes ton laptop. Tu vas faire autre chose. Tu le laisses bosser.

Quand il a fini, tu te retrouves avec un dossier `wiki` rempli de pages organisées, des connexions que tu n'avais pas vues, des résumés de choses que tu avais oubliées d'avoir gardées, et un index qui rend tout ça consultable en quelques secondes.

Disposition recommandée : Obsidian d'un côté de l'écran, Claude Code de l'autre. L'IA fait ses modifications, toi tu observes en temps réel, tu suis les liens, tu jettes un œil à la vue graphe, tu lis les pages mises à jour. Obsidian est l'IDE, l'IA est le programmeur, le wiki est la codebase.

---

## **Étape 5 — L'utiliser au quotidien**

C'est ici que le système prend sa valeur. Trois opérations à intégrer dans ton flux régulier.

### **Ingérer une nouvelle source**

Tu clippes un article via l'extension Web Clipper d'Obsidian. Il atterrit dans `sources-brute/`. Tu lances :

```shell
claude -p "I just added an article to /sources-brute/. Read it, extract the key ideas, write a summary page to /wiki/, update index.md with a link and one-line description, and update any existing concept pages that this article connects to. Log what you changed to log.md. Show me every file you touched." --allowedTools Bash,Write,Read
```

Un seul article peut impacter 10 à 15 pages du wiki. Claude fait remonter des connexions auxquelles tu n'avais pas pensé, signale les contradictions avec ce qui est déjà filé, et journalise précisément ce qui a changé.

### **Interroger ton wiki**

Une fois que tu as une dizaine d'articles digérés, tu peux commencer à creuser :

* *« En te basant uniquement sur le contenu de wiki/, quelles sont les trois plus grosses lacunes dans ma compréhension de \[sujet\] ? »*  
* *« Compare ce que dit la source A sur \[concept\] avec ce que dit la source B. Sur quoi sont-elles en désaccord ? »*  
* *« Rédige-moi un briefing de 500 mots sur \[sujet\] en n'utilisant que ce qui est dans cette base. »*

Claude balaie l'index, sort les pages pertinentes, te livre une réponse avec citations. Le réflexe à acquérir : **réinjecter les bonnes réponses dans le wiki sous forme de nouvelles pages**. Une comparaison utile, une analyse, une connexion découverte — ça ne doit pas finir dans l'oubli de l'historique de chat. Chaque question rend la suivante meilleure. C'est la boucle de composition.

### **Faire un audit de santé**

Une fois par semaine, lance :

```shell
claude -p "Read every file in /wiki/. Find: contradictions between pages, orphan pages with no inbound links, concepts mentioned repeatedly but with no dedicated page, and claims that seem outdated based on newer files in /sources-brute/. Write a health report to /wiki/lint-report.md with specific fixes." --allowedTools Bash,Write,Read
```

Cette routine attrape les erreurs avant qu'elles ne fassent boule de neige. Si l'IA écrit quelque chose de légèrement faux et que tu le réinjectes, la réponse suivante va construire sur cette erreur. L'audit, c'est ton contrôle qualité.

---

## **Étape 6 — Mettre en place des automatisations (optionnel mais puissant)**

### **Brief matinal**

Une fois configuré, ça tourne tous les matins sans intervention :

```shell
claude -p "Write a Python script called morning_digest.py that: 1) reads Memory.md and surfaces any open actions due today 2) reads any new files added to /sources-brute/ in the last 24 hours 3) prints a clean briefing to the terminal. Then schedule it as a cron job every morning at 7:30am." --allowedTools Bash,Write
```

Chaque matin, tu ouvres ton laptop sur un résumé de ce qui demande ton attention et de ce qui est entré dans ta base depuis 24h. Configuré une fois, opérationnel pour toujours.

### **Traiter le transcript d'un appel**

Après n'importe quelle réunion ou appel client :

```shell
claude -p "Read the transcript in /sources-brute/call-today.md. Extract every decision made, every action item with owner and deadline, and a 3-bullet summary. Add actions to /wiki/action-tracker.md, log decisions to /wiki/decision-log.md, and create a topic page linking back to this transcript." --allowedTools Bash,Write,Read
```

Chaque décision archivée, chaque action trackée, plus rien qui ne se perd dans un historique de chat.

---

## **Ce que tu peux construire avec ça**

**Base de connaissances personnelle**  suivre tes objectifs, ta santé, ton développement personnel. Filer journaux, articles, notes de podcasts, et bâtir une image structurée de toi-même qui s'affine dans la durée.

**Wiki de recherche**  explorer un sujet sur des semaines ou des mois. Lire des papers, des articles, des rapports, et construire incrémentalement un wiki complet avec une thèse qui évolue à mesure que tu accumules les sources.

**Wiki compagnon de lecture**  filer chaque chapitre au fil de la lecture, faire émerger des pages pour les personnages, les thèmes, les arcs narratifs et leurs connexions. À la fin du livre, tu as un compagnon riche qui rend toute relecture inutile.

**Wiki d'entreprise** un wiki interne nourri par les threads Slack, les transcripts de réunions, les docs de projet, les appels clients. Il reste à jour parce que l'IA fait la maintenance que personne dans l'équipe n'a envie de faire.

**Veille concurrentielle**  surveiller les prix, les features, les recrutements, le positionnement des concurrents. Chaque nouveau signal s'intègre automatiquement dans le tableau d'ensemble.

**Base de connaissances clients**  tout ce que tu sais sur chaque client dans un système unique, consultable, qui se met à jour tout seul.

**Notes de cours** bâtir un wiki d'études complet en suivant un cours, avec l'IA qui croise les concepts d'une leçon à l'autre.

---

## **Trucs et astuces**

Installe l'extension de navigateur **Obsidian Web Clipper**. Elle convertit n'importe quel article web en markdown en un clic. C'est le moyen le plus rapide d'alimenter ta collection brute.

Utilise la **vue graphe d'Obsidian**. Rien ne montre mieux la forme de ton wiki  ce qui se relie à quoi, quelles pages sont des hubs, lesquelles sont laissées seules dans leur coin.

Installe le plugin **Dataview**. Si l'IA ajoute des tags et des dates aux pages du wiki, Dataview peut générer des tableaux et des listes dynamiques.

Mets ton vault dans un **repo git**. Le wiki, c'est juste des fichiers markdown. Tu obtiens gratuitement l'historique de versions, le branching, la collaboration.

Dernier point qui change la perspective : Obsidian n'est pas indispensable. Un dossier de fichiers markdown et un bon fichier de schema feront mieux qu'une stack outillée à mort dans 90% des cas. Obsidian est juste une fenêtre agréable pour regarder à travers. Le vrai système, ce sont les dossiers, le schema, et l'IA.
