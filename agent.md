Agent Guidelines (Architecture + Clean Code)

1. Objectif global
   Ce projet suit une architecture stricte inspirée de Clean Architecture / Hexagonal Architecture.
   Le but est de produire un code lisible, maintenable, testable et évolutif.
   La logique métier doit rester pure et indépendante.
   Les règles d’architecture ne doivent jamais être violées.

2. Architecture du projet

2.1 Règle principale : sens des dépendances
Les dépendances doivent toujours pointer vers l’intérieur :

presentation → application → domain

La couche infrastructure peut dépendre de application/domain, mais domain ne dépend jamais de infrastructure.

2.2 Responsabilité des dossiers

src/domain/
Contient uniquement la logique métier pure : entités, value objects, services métier, events, erreurs métier.
Interdictions : aucun code lié à une base de données, aucun appel réseau, aucune logique d’interface, aucun accès à des fichiers, aucun framework.
Cette couche doit être testable sans mocks.

src/application/
Contient les workflows (use cases).
Les use cases orchestrent la logique domain et appellent le monde extérieur uniquement via des interfaces (ports).
Interdictions : pas d’accès direct à une base de données, pas d’appel direct à des services externes, pas de logique UI, pas de logique HTTP.

src/application/ports/
Contient uniquement des interfaces définissant les dépendances externes :
repositories, services externes (notifications, paiement, stockage, logs, etc.), authentification, etc.

src/infrastructure/
Contient toutes les implémentations techniques concrètes :
accès base de données, appels externes, stockage fichiers, logs, implémentations des repositories, implémentations des services externes.
Règles : infrastructure implémente les ports définis dans application.
Infrastructure ne doit jamais exposer ses types internes vers domain/application.

src/presentation/
Contient tout ce qui gère l’entrée/sortie utilisateur :
API, controllers, routes, UI, pages, composants, validation des entrées, middlewares.
Règles : aucune logique métier ici.
La présentation appelle uniquement des use cases.
La présentation ne doit jamais appeler infrastructure directement.

src/main/
Contient uniquement l’assemblage du projet : wiring, injection des dépendances, configuration runtime, démarrage du serveur ou de l’application.
Interdiction : aucune logique métier.

src/shared/
Contient uniquement des outils génériques réellement réutilisables (types, helpers généraux).
Interdiction : ne pas mettre de logique métier ici.

3. Règles strictes Clean Code

3.1 Fonctions
Chaque fonction doit être très courte (idéalement < 20 lignes).
Chaque fonction doit faire une seule chose.
Si une fonction est difficile à nommer clairement, elle fait trop de choses.
Chaque fonction doit avoir une utilité unique et évidente.

3.2 Nommage
Le code doit être compréhensible juste en lisant les noms.
Les noms doivent être explicites, même s’ils sont longs.
Éviter les noms vagues comme data, tmp, handle, process.

Les noms doivent refléter exactement l’intention métier ou technique.

3.3 Interdiction des booléens en paramètre
Aucun booléen ne doit être passé en paramètre d’une fonction.

Interdit : doSomething(entity, true)
Correct : doSomethingInSpecificMode(entity)
Ou mieux : deux fonctions séparées avec des noms explicites.

3.4 Conditions
Éviter l’imbrication profonde.
Utiliser des early returns.
Extraire les conditions complexes dans des fonctions bien nommées.

3.5 Gestion des erreurs
Ne jamais retourner null/undefined silencieusement.
Les erreurs doivent être explicites et compréhensibles.
Créer des erreurs spécifiques plutôt que des erreurs génériques.

3.6 Types / Objets métier
Éviter de passer des primitives partout si cela rend le code flou.
Préférer des objets dédiés (Value Objects) pour représenter les concepts importants du métier.

3.7 Valeurs magiques interdites
Aucune valeur hardcodée incompréhensible.
Utiliser des constantes explicites et nommées.

3.8 Commentaires
Le code doit être auto-explicatif.
Les commentaires sont autorisés uniquement pour expliquer une règle métier ou une contrainte importante.

4. Pattern obligatoire des Use Cases
   Chaque use case doit suivre ce schéma :

* input structuré (DTO ou équivalent)
* validation basique
* exécution de logique domain
* persistance via un repository défini par un port
* output structuré

Un use case ne doit jamais contenir :
requêtes directes base de données, logique réseau, logique UI, logique framework.

5. Repositories
   Les repositories doivent exposer uniquement des objets du domain (ou des structures neutres), jamais des objets spécifiques à l’infrastructure.
   Ils doivent cacher totalement la structure de stockage.
   Ils doivent proposer des méthodes explicites : findById, findByEmail, save, delete, etc.
   Aucune requête libre ou dynamique ne doit être exposée au reste du code.

6. Tests
   Domain : tests unitaires maximaux.
   Application : tests de use cases avec dépendances mockées via ports.
   Infrastructure : tests d’intégration.
   Presentation : tests API / UI / end-to-end selon le projet.

7. Patterns interdits
   Pas de classes énormes.
   Pas de services “fourre-tout”.
   Pas de fichiers gigantesques (>300 lignes).
   Pas de fichier “utils/helpers” qui devient un dump.
   Pas de dépendances circulaires.

8. Règles de génération de code
   Toujours respecter les frontières d’architecture.
   Ne créer que le strict nécessaire.
   Ne pas sur-architecturer.
   Favoriser la clarté et la simplicité.

Résumé final
Le projet doit rester clean, modulaire, prévisible et testable.
L’architecture est non négociable.
Les règles Clean Code sont obligatoires.
