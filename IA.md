# Utilisation de l'IA - TP1

Ce document décrit les cas réels où j'ai utilisé l'IA pour m'aider durant le TP1.

---

## Utilisation 1 : Structure du JSON pour l'API Gemini et rôle système

- **Problème rencontré :**
  Quand j'ai voulu ajouter le rôle système dans la requête avec `"role": "system"` dans le tableau `contents`, l'API Gemini a renvoyé une erreur 400 Bad Request. Je ne comprenais pas où placer les consignes du rôle système.
- **Raison de l’utilisation de l’IA :**
  Savoir exactement quelle structure JSON attend l'API REST de Gemini pour le rôle système et les messages.
- **Prompt utilisé :**
  ```text
  Pourquoi l'API Gemini me renvoie une erreur 400 quand je mets "role": "system" dans contents, et comment envoyer correctement le rôle système et la question en JSON ?
  ```
- **Réponse de l'IA :**
  > « Dans l'API Gemini, le tableau `contents` n'accepte que les rôles `user` et `model`. Le rôle `system` n'est pas autorisé dans `contents`.
  > Pour donner des consignes système au modèle, il faut utiliser l'objet `system_instruction` à la racine du JSON :
  > ```json
  > {
  >   "system_instruction": {
  >     "parts": [{ "text": "Tu es un coach de voyage écologique..." }]
  >   },
  >   "contents": [
  >     { "role": "user", "parts": [{ "text": "Ma question..." }] }
  >   ]
  > }
  > ``` »
- **Ce que vous avez appris :**
  J'ai compris que Gemini ne fonctionne pas comme OpenAI : les instructions système sont séparées dans `system_instruction`, et la conversation interactive est dans `contents`. J'ai aussi retenu que l'API n'a pas de mémoire ; il faut donc renvoyer tout l'historique dans `contents` à chaque fois pour que le bot suive la conversation.

---

## Utilisation 2 : Ajouter des messages au JSON avec JsonPointer

- **Problème rencontré :**
  Je devais ajouter la nouvelle question et la réponse reçue dans le JSON pour garder l'historique, mais avec la bibliothèque `jakarta.json`, les objets sont immuables (on ne peut pas simplement faire un `add` direct sur l'objet).
- **Raison de l’utilisation de l’IA :**
  Comprendre comment marche `JsonPointer` et comment ajouter un élément à la fin du tableau `contents`.
- **Prompt utilisé :**
  ```text
  En Java avec jakarta.json, comment utiliser JsonPointer pour ajouter un nouveau message à la fin du tableau contents d'un JsonObject existant ?
  ```
- **Réponse de l'IA :**
  > « Les objets JSON de `jakarta.json` ne sont pas modifiables directement. On utilise `JsonPointer` avec le chemin `/contents/-` où le tiret `-` signifie 'ajouter à la fin du tableau' :
  > ```java
  > JsonPointer pointer = Json.createPointer("/contents/-");
  > this.requeteJson = this.pointer.add(this.requeteJson, nouveauMessageJson);
  > ```
  > La méthode `add` renvoie un nouvel objet JSON avec le message inséré. »
- **Ce que vous avez appris :**
  J'ai appris que modifier un `JsonObject` crée en réalité une nouvelle copie qu'il faut réaffecter à la variable. Le symbole `/-` dans le chemin est un raccourci très pratique pour dire « ajoute cet élément tout à la fin de la liste », sans avoir besoin de compter combien il y en a déjà.

---

## Utilisation 3 : Problème avec la clé GEMINI_KEY (System.getenv retourne null)

- **Problème rencontré :**
  J'avais créé la variable d'environnement `GEMINI_KEY` sous Windows avec PowerShell, mais dans mon code Java `System.getenv("GEMINI_KEY")` renvoyait toujours `null`, ce qui bloquait le démarrage de l'application.
- **Raison de l’utilisation de l’IA :**
  Comprendre pourquoi IntelliJ ne voyait pas ma variable et trouver la méthode la plus simple pour la faire fonctionner avec le serveur Payara.
- **Prompt utilisé :**
  ```text
  J'ai créé la variable GEMINI_KEY dans Windows, mais System.getenv("GEMINI_KEY") retourne null dans IntelliJ quand je lance le serveur. Comment faire ?
  ```
- **Réponse de l'IA :**
  > « Si IntelliJ était déjà ouvert quand vous avez créé la variable dans Windows, il ne peut pas la voir sans être redémarré.
  > Le plus simple est de l'ajouter directement dans votre configuration d'exécution :
  > 1. Allez dans `Run` > `Edit Configurations...`.
  > 2. Sélectionnez votre serveur Payara/Tomcat.
  > 3. Dans `Environment variables`, écrivez : `GEMINI_KEY=votre_cle_ici`.
  > 4. Cliquez sur OK et relancez le serveur. »
- **Ce que vous avez appris :**
  J'ai compris qu'un programme déjà ouvert ne met pas à jour ses variables tout seul. Mettre la clé dans la configuration d'IntelliJ est plus rapide et évite surtout d'écrire la clé en dur dans les fichiers Java, ce qui protège la clé secrète pour ne pas la publier sur GitHub par accident.

---

## Utilisation 4 : Tester les erreurs demandées et annuler les modifications avec Git

- **Problème rencontré :**
  Le TP demandait de tester volontairement deux erreurs (une fausse clé et le rôle `"system"` dans `contents`) pour copier les messages d'erreur, puis de revenir au code propre. J'avais peur de casser mon projet ou de mélanger mes commits.
- **Raison de l’utilisation de l’IA :**
  Être sûr de la manipulation avec Git pour annuler ces modifications de test sans perdre mon travail.
- **Prompt utilisé :**
  ```text
  J'ai modifié mon code pour provoquer les erreurs du TP1. Comment annuler proprement ces modifications de test avec Git ou IntelliJ pour retrouver mon code d'avant sans faire de bêtise ?
  ```
- **Réponse de l'IA :**
  > « Tant que vous n'avez pas fait de commit :
  > 1. Vérifiez vos modifications avec `git diff`.
  > 2. Pour annuler et remettre les fichiers à l'état du dernier commit, faites :
  > `git restore .` (dans le terminal)
  > ou dans IntelliJ : clic droit sur le fichier > `Git` > `Rollback`.
  > Vos fichiers reviendront exactement comme ils étaient. »
- **Ce que vous avez appris :**
  J'ai appris qu'en faisant un commit propre avant de commencer des tests, on ne risque rien. On peut tester les pannes en toute sécurité, puis annuler les changements d'un seul clic avec le rollback ou la commande `git restore`, sans laisser de mauvaises traces dans le projet.
